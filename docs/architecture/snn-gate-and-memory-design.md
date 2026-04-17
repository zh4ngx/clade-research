# SNN Gate & Memory Design

> Concrete design for the thalamic gate and hippocampal memory subsystem.

Companion to [heterogeneous-cognitive-architecture.md](heterogeneous-cognitive-architecture.md). Source: Qwen deep-dive on the three-layer architecture.

## SNN Gating Layer

### Inputs

The SNN receives two input streams:

1. **SSM hidden state** `h_t` (e.g., 768-dim from a 3B SSM) — "what's happening right now"
2. **Memory context vector** `m_t` — compressed summary of the most recent memory retrieval (default/empty if no recent query)

### Output Neurons

Each SNN output neuron corresponds to a pathway. Membrane potential `v` fires when `v > threshold`:

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

Decision mechanism: spike competition. At each timestep (or every N timesteps), the gate evaluates which pathway activates. Multiple simultaneous spikes resolved by priority ordering.

### Training Protocol

Spikes are non-differentiable step functions. Three approaches:

| Method | How | Tradeoff |
|--------|-----|----------|
| Surrogate gradients | Replace step with smooth approximation during backprop | Standard, but gradients are approximate |
| Straight-through estimator | Pretend derivative of step is 1 | Simple, noisy gradients |
| RL with discrete actions | Treat each spike as a discrete action, reward for correct output | No gradients through gate, handles discreteness (PPO/DQN) |

**Phased training:**
1. Pretrain SSM + projection so SNN receives good representations
2. Freeze SSM, train SNN gate with surrogate gradients on supervised objective (correct pathway per timestep)
3. Fine-tune end-to-end with RL — reward is task completion, SNN learns timing of decisions

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
| Interference | Similar memories blend | Writing A near B corrupts both — system learns robustness |
| Decay | Unused memories fade | Unretrieved items lose amplitude, encourages recency |
| Cue-based retrieval | Smell triggers childhood memory | Retrieval cue is projection of SSM state |
| Reconstruction | Retrieve fragments, not the memory | Weighted sum of all stored items, inherently lossy |
| Plasticity | Memories change when recalled | Retrieved item modified before re-writing (reconsolidation) |

### How This Differs from Attention

Structurally similar (weighted sum over stored values), but:

1. **Plasticity:** Memories change through interference and reconsolidation. Attention operates on static KV cache.
2. **Capacity limits:** Fixed matrix size. Writing beyond capacity causes destructive interference.
3. **Noisy retrieval:** Corrupted reconstructions force the SSM to fill gaps.
4. **Forgetting is a feature:** No GC needed. Unused memories fade.

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

1. **SSM** encodes input → `h_t` representing the query
2. **SNN Gate** receives `h_t`, cue matches "database schema decision", Neuron B (retrieve) spikes
3. **Memory Layer** queries associative matrix → noisy, partially decayed reconstruction: "we decided... something about... tables..."
4. **SNN Gate** receives retrieved memory, Neuron A (generate) spikes
5. **Output:** "Last week we discussed the schema. We decided on a denormalized structure, though I'm fuzzy on the exact columns."

Partially correct with expressed uncertainty — because memory retrieval was genuinely lossy. More honest than a Transformer that confidently retrieves but may hallucinate.

## Research Contribution

1. **Genuine uncertainty** — agent admits it because memory is lossy, not because prompted
2. **No KV cache explosion** — O(N log N) perception, O(1) gate, O(num_memories) retrieval
3. **Persistent, evolving memory** across sessions that naturally degrades
4. **Testable hypothesis:** SSM + plastic associative memory + SNN gate vs. Transformer on long-horizon agentic tasks
