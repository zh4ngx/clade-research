# SNN Gate & Memory Design

Related: [heterogeneous-cognitive-architecture](heterogeneous-cognitive-architecture.md), [bsca-fabric](bsca-fabric.md), [pulse-feedback](pulse-feedback.md)

Concrete design for the SNN gating layer and associative memory subsystem from [heterogeneous-cognitive-architecture](heterogeneous-cognitive-architecture.md). Source: Qwen deep-dive on the three-layer architecture.

## SNN Gating Layer

### Inputs

The SNN receives two input streams:

1. **SSM hidden state** `h_t` (e.g., 768-dim from a 3B SSM) — "what's happening right now"
2. **Memory context vector** `m_t` — compressed summary of the most recent memory retrieval (default/empty if no recent query)

### Output Neurons

Each SNN output neuron corresponds to a pathway. The neuron maintains a membrane potential `v` and fires when `v > threshold`:

```
                 ┌─────────────────────────────────┐
                 │           SNN GATE              │
                 │                                 │
    h_t ────────→│  Neuron A: "Generate token"     │─── spike → trigger LM head
                 │  Neuron B: "Retrieve memory"     │─── spike → query memory layer
    m_t ────────→│  Neuron C: "Write to memory"     │─── spike → commit h_t to memory
                 │  Neuron D: "Call tool/API"       │─── spike → trigger external tool
                 │  Neuron E: "Idle / maintain"     │─── spike → do nothing
                 └─────────────────────────────────┘
```

**Decision mechanism:** spike competition. At each timestep (or every N timesteps), the gate evaluates which pathway activates. Multiple simultaneous spikes resolved by priority ordering.

### Training Protocol

Spikes are non-differentiable step functions. Three approaches:

| Method | How | Tradeoff |
|--------|-----|----------|
| Surrogate gradients | Replace step with smooth approximation during backprop | Standard, but gradients are approximate |
| Straight-through estimator | Pretend derivative of step is 1 | Simple, noisy gradients |
| RL with discrete actions | Treat each spike as a discrete action, reward for correct output | No gradients through gate, handles discreteness properly (PPO/DQN) |

**Phased training:**
1. **Pretrain SSM + projection** so the SNN receives good representations
2. **Freeze SSM, train SNN gate** with surrogate gradients on supervised objective (correct pathway per timestep)
3. **Fine-tune end-to-end with RL** — reward is task completion, SNN learns timing of retrieval/tool/generation decisions

## Associative Memory Layer

Not a clean vector DB. Biological memory is noisy, plastic, and reconstructive.

### Architecture

```
┌─────────────────────────────────────────────────────────┐
│                 ASSOCIATIVE MEMORY                      │
│                                                         │
│  Storage: Matrix M (size: max_items × item_dim)        │
│  Trace:    τ (per-item decay/interference counter)      │
│  Pattern:  Each stored item is a noisy vector m_i       │
│                                                         │
│  WRITE: When gate spikes "write":                      │
│    m_new = compress(h_t)                                │
│    M ← M + α·(m_new · retrieval_cue^T)                  │
│    τ ← τ + decay                                        │
│                                                         │
│  READ: When gate spikes "retrieve":                    │
│    cue = project(h_t)  (query vector)                   │
│    similarity = M · cue   (dot product matching)       │
│    retrieved = softmax(similarity) × M   (weighted sum)│
│    retrieved ← retrieved + noise(σ)                     │
│    retrieved ← decay(retrieved, τ)                      │
│                                                         │
│  FORGET: Items with τ > threshold decay to zero.        │
│         Interference between similar items causes       │
│         gradual corruption.                             │
└─────────────────────────────────────────────────────────┘
```

### Properties

| Property | Biological Motivation | Computational Effect |
|----------|----------------------|---------------------|
| **Interference** | Similar memories blend together | Writing A near B corrupts both — system learns robustness |
| **Decay** | Unused memories fade | Unretrieved items lose amplitude, encourages recency |
| **Cue-based retrieval** | A smell triggers a childhood memory | Retrieval cue is projection of SSM state; similarity triggers partial recall |
| **Reconstruction** | You retrieve fragments, not the memory itself | Output is weighted sum of all stored items (like attention), inherently lossy |
| **Plasticity** | Memories change when recalled (reconsolidation) | Retrieved item is modified before re-writing |

### How This Differs from Attention

Structurally similar (weighted sum over stored values), but:

1. **Plasticity:** Memories change through interference and reconsolidation. Attention operates on static KV cache.
2. **Capacity limits:** Fixed matrix size. Writing beyond capacity causes destructive interference — old memories overwritten. This is how human memory actually works.
3. **Noisy retrieval:** Never gets "perfect" memories — corrupted reconstructions force the SSM to fill gaps.
4. **Forgetting is a feature:** No garbage collection needed. Unused memories fade. The system relies on SSM state for what matters.

## Data Flow

```
                    ┌──────────────────┐
                    │   TOKEN INPUT     │
                    └────────┬─────────┘
                             │
                    ┌────────▼─────────┐
                    │  SSM (Sensory)   │
                    │  h_t = f(h_t-1)  │
                    └────────┬─────────┘
                             │
               ┌─────────────┼─────────────┐
               │             │             │
      ┌────────▼────┐       │      ┌───────▼────────┐
      │  SNN GATE   │       │      │  MEMORY LAYER  │
      │  (Thalamus) │◄──────┤      │  (Hippocampus) │
      │             │ cues  │  ┌──►│                │
      │  Decides:   │ query │  │   │  Write/read    │
      │  - Generate │       │  │   │  Decay/forget  │
      │  - Retrieve │───────┘  │   │  Interference  │
      │  - Write    │──────────┘   └────────────────┘
      │  - Tool     │
      │  - Idle     │
      └────────┬────┘
               │
      ┌────────▼────────┐
      │  ACTION OUTPUT  │
      │  - Emit token   │
      │  - API call     │
      │  - No action    │
      └─────────────────┘
```

## Concrete Example

User asks: "What did we decide about the database schema last week?"

1. **SSM** encodes the input. Produces `h_t` representing the query.
2. **SNN Gate** receives `h_t`. Cue pattern matches "database schema decision". Neuron B (retrieve) spikes.
3. **Memory Layer** queries associative matrix. Returns a noisy, partially decayed reconstruction — "we decided... something about... tables..."
4. **SNN Gate** receives retrieved memory as context. Neuron A (generate) spikes.
5. **Output:** "Last week we discussed the schema. We decided on a denormalized structure, though I'm fuzzy on the exact columns."

The system gives a partially correct answer with expressed uncertainty — because the memory retrieval was genuinely lossy. Not a bug. More honest than a Transformer that confidently retrieves the exact vector but may hallucinate.

## Implementation Strategy

Source: QC practical hardware/software analysis.

### Hardware Assessment

| Hardware | Verdict | Why |
|----------|---------|-----|
| **FPGA + SRAM** | Overengineered for SLM scale | Memory matrix (1024 items × 768 dims) is a ~3 MB matmul — takes ~1ms on GPU. FPGA only wins at hundreds of thousands of items or sub-ms robotics latency |
| **Neuromorphic (Loihi 2)** | Interesting but impractical | Pure spike processor — can't run SSM. Intel lab access only, not commercially available. Interconnect overhead kills throughput for small-batch inference |
| **GPU (Vulkan/CPU BLAS)** | Right tool for research | Associative memory matmul fits easily in VRAM. Sub-ms retrieval at SLM scale |

**The principle:** Don't build hardware until the architecture is validated in software. A 12 MB matrix-vector multiply on GPU is perfectly adequate for research.

### Software Tiers

**Tier 1: GPU-resident associative matrix (research)**
```python
class AssociativeMemory:
    def __init__(self, max_items=1024, dim=768, device="vulkan"):
        self.M = torch.zeros(max_items, dim, device=device)
        self.τ = torch.zeros(max_items, device=device)
        self.n = 0

    def write(self, item, cue, strength=0.1):
        binding = strength * torch.outer(item, cue)
        self.M[self.n % max_items] += binding.flatten()
        self.τ[self.n % max_items] = 0
        self.n += 1

    def read(self, cue, noise=0.01):
        similarity = self.M @ cue
        weights = torch.softmax(similarity / temperature, dim=0)
        retrieved = weights @ self.M + noise * torch.randn_like(cue)
        self.τ *= 0.99  # decay all traces
        return retrieved
```
Single matmul retrieval on GPU — fast enough for research. Start here.

**Tier 2: CPU RAM with disk persistence (scale-up)**
- GPU: SSM + SNN gate
- RAM: Large memory matrix (32 GB)
- SSD: Compressed memory snapshots (cold storage for decayed traces)

At 1024 items × 768 dims × 4 bytes ≈ 3 MB — fits in RAM easily. Offloading only needed above ~10K items / ~1 GB.

**Tier 3: Production tiered memory (deployment)**
```
GPU hot cache → RAM warm matrix → SQLite metadata → Disk cold snapshots
```
Essentially L1/L2/RAM/disk tiering applied to associative memory. Only relevant after the architecture is validated and you're optimizing for deployment.

### Recommended Starting Point

1. Associative memory as PyTorch tensor on GPU alongside SSM — same device, no copying
2. `max_items = 1024` — enough for a week of agentic operation
3. Three operations only: **write**, **read**, **decay**. No plasticity, no interference, no reconsolidation yet
4. Add biological properties (plasticity, interference, reconsolidation) once the core loop works
5. Hardware optimization only when hitting scale limits (>10K items)

The hardware question becomes interesting *after* the architecture is validated. If SSM + associative memory + SNN gate beats a baseline Transformer on agentic tasks, then optimize the memory path for deployment. That's a deployment problem, not a research problem.

## Research Contribution

This system would demonstrate:

1. **Genuine uncertainty** — the agent admits uncertainty because its memory is lossy, not because you prompted it to
2. **No KV cache explosion** — SSM handles perception in O(N log N), SNN gate fires in O(1), memory retrieval is O(num_memories), not O(context²)
3. **Persistent, evolving memory** across sessions that naturally degrades — like a real agent
4. **Testable hypothesis:** Does SSM + plastic associative memory + SNN gate outperform a Transformer of similar size on long-horizon agentic tasks?

---
#clade #architecture #snn #memory #design #bio-inspired
