---
date: 2026-04-16
tags: [clade, architecture, hierarchical-control, inhibitory-gating, procedural-memory, modulation-budget, evaluation, bio-inspired]
aliases: [hierarchical control, inhibitory gating, amnesia task, procedural memory, modulation budget]
---

# Hierarchical Control Architecture (Kimi Synthesis)

Related: [heterogeneous-cognitive-architecture](heterogeneous-cognitive-architecture.md), [coupled-dynamical-architecture](coupled-dynamical-architecture.md), [surprise-driven-bootstrapping](surprise-driven-bootstrapping.md), [wasm-credit-assignment-replay](wasm-credit-assignment-replay.md), [bsca-fabric](bsca-fabric.md), [pulse-feedback](pulse-feedback.md)

A third view: not pipeline (5-agent consensus) or coupled oscillator (GC), but a **hierarchical control system** with multiple timescales and inhibitory gating. The BSCA fabric isn't just memory or routing — it's the procedural substrate.

Source: Kimi gap analysis, the seventh agent.

## 1. Inhibitory Gating, Not Excitatory

The 5-neuron gate uses excitatory selection — which neuron fires selects the pathway. Biologically wrong.

**Basal ganglia use inhibitory gating:** the default is "do nothing" (globus pallidus tonic inhibition). Specific pathways *release* inhibition to allow action.

```
Current (wrong):  Neuron A fires  → generate
Proposed (right): All neurons fire tonic inhibition →
                   Neuron A stops firing → release generate pathway
```

Why this matters:
- More robust to noise — failure mode = do nothing, not random action
- Maps to [bsca-fabric](bsca-fabric.md) attractor dynamics — cessation of a pattern is as detectable as onset
- Eliminates the "priority ordering" hack from [snn-gate-and-memory-design](snn-gate-and-memory-design.md) — competitive inhibition instead

## 2. Three Memory Systems, Not Two

CC added episodic vs semantic. Procedural is missing.

| Memory Type | Content | Example | Encoding |
|-------------|---------|---------|----------|
| Episodic | Events | "We discussed schema design" | Transient attractor basins (decay) |
| Semantic | Facts | "Database has 3 tables" | Stable fixed points (STDP-consolidated) |
| Procedural | Skills | "How to grep" / "How to format JSON" | Spatiotemporal patterns in BSCA |

Procedural memory is not declarative — can't be retrieved as a vector. It's encoded as dynamical attractors in the BSCA fabric. Tool execution isn't "retrieve then act" — it's "collapse into the tool-use attractor basin."

**Novel synthesis:** The [bsca-fabric](bsca-fabric.md) isn't just System 1 routing — it's the procedural substrate. All three memory types live in one lattice.

## 3. Modulation Budget (Safety Mechanism)

GC proposes SNN spikes modulate SSM parameters (Δ, B, C). Danger: a rogue attractor can trap the SSM in a loop.

**Fix: bounded modulation with homeostatic regeneration.**

```
Δ_t+1 = Δ_t + α·SNN_signal
Constraint: |Δ_t - Δ_0| < ε (can't drift far from pretrained)
Budget regenerates slowly (homeostasis)
```

The system must be resilient to CNS dysfunction. The SSM degrades gracefully if the BSCA fabric enters epileptic oscillation.

## 4. Self-Supervised Surprise Mining (Training Data)

The biggest gap across all agents: where does training data come from?

**Proposal: mine surprise from existing LLM training corpora.**

1. Train base SSM on next-token prediction (standard)
2. Mask random spans → SSM must predict masked content
3. High prediction error positions → synthetic "surprise"
4. Train gate: given pre-mask context, predict "would retrieval help?"
5. Memory: store masked spans, retrieve on similar context

No human labels needed. Surprise is synthetic. Gate learns from prediction error.

## 5. The Amnesia Task (Evaluation Benchmark)

First concrete evaluation metric proposed across all agents.

**Setup:** Agent completes 100-step task. Information scattered at steps 10, 30, 50, 70. At step 80, query asks about step 20.

| Baseline | Context Window | Expected |
|----------|---------------|----------|
| Transformer (4K) | Can attend back | "Cheating" — perfect recall |
| Transformer (128) | Cannot attend back | Fair comparison |
| Proposed system | SSM compression + memory retrieval | Target |

**Success criterion:** Match 4K-context transformer accuracy with <5% of the KV cache, measured on real wall-clock time.

## 6. Soft VQ with Residual (Alternative to Hard VQ)

GC's hard VQ codebook (10K entries) wastes bits: log2(10,000) ≈ 13.3 bits per token from a 12,288-bit SSM state.

**Alternative:** Soft VQ-VAE with learned prior — BSCA receives both a categorical concept ("DATABASE") and a continuous "flavor" (how database-related, uncertainty, valence).

```
SSM → Encoder → Latent z (continuous, 64 dims)
           ↓
       VQ Codebook (soft, 1024 entries)
           ↓
       + Noise (diffusion-style)
           ↓
    BSCA receives: discrete code + residual
```

More information-theoretically efficient than hard quantization.

## 7. WebGPU Sparse Ops (Alternative to Vulkan Compute)

CA updates are sparse — only cells near active spikes update. GPUs excel at dense parallel ops. Simulating CA on GPU is using a SIMD cannon to shoot mosquitos.

**Alternative:** Sparse tensor operations via WebGPU.
- Sparse matrix formats (CSR, COO)
- Indirect dispatch (only update changed cells)
- Workgroup-local barriers for neighborhood communication

4096-cell BSCA lattice at 1% activity → 40 active cells → single workgroup. Runs everywhere (browser, desktop, mobile, server). Same code for research and deployment.

**Why higher abstraction wins here:** Dense compute → close-to-metal wins (Vulkan). Sparse compute → you want the abstraction that provides indirect dispatch and sparse formats, which WebGPU has and Vulkan doesn't natively. The higher level isn't better in general — it matches the workload shape. Same reason you don't write a web server in assembly.

## 8. SR+HER: Successor Representations with Hindsight Replay for Branch Pre-Filtering

### What Successor Representations Give You

A Successor Representation (SR) decomposes the value function into transition structure and reward structure:

$$V^\pi(s) = \sum_{s'} M^\pi(s, s') \cdot r(s')$$

where $M^\pi = (I - \gamma T^\pi)^{-1}$ is the expected discounted future state occupancy under policy $\pi$, and $r$ is the per-state reward. The critical property: when the reward function changes but the transition dynamics don't, only $r$ needs updating. $M^\pi$ — the expensive part — is reusable.

For CLADE, the "states" are agent configurations (component activation patterns, capability states, lattice attractor positions). The "transitions" are what happens when a component executes — how the system state evolves. The SR captures *which future states are reachable from here*, independent of what we think about them.

### What Hindsight Experience Replay Adds

HER converts failed trajectories into training data by relabeling: "you didn't reach goal G, but you did reach state $s'$ — so here's what you'd need to do to reach $s'$ from your starting point." Every execution, successful or not, produces a valid training sample.

This matters for CLADE because most WASM branch explorations won't reach the goal. [wasm-credit-assignment-replay](wasm-credit-assignment-replay.md) forks the agent at decision points and explores branches. Without HER, failed branches are pure cost. With HER, they're training data for the SR.

### The Synthesis: Pre-Filtering WASM Branches

The expensive operation in CLADE's credit assignment is the WASM branch replay — forking the agent state, executing each branch to completion, comparing outcomes. This is essentially tree search over component decisions. With $N$ decision points and $K$ options per point, the branching factor explodes.

SR+HER proposes a cheaper alternative: **don't run the branch — predict its outcome.**

The SR matrix $M$ encodes expected future state occupancy. Given current state $s$ and a candidate branch action $a$, the predicted future state is approximately:

$$\hat{s}' \approx M^\pi(s, \cdot) \cdot \mathbf{1}_a$$

This is a **matrix-vector multiply**, not a GPU forward pass. If $M$ is $|\mathcal{S}| \times |\mathcal{S}|$ and we can keep it sparse (most state pairs never transition), the cost is roughly $\mathcal{O}(k)$ where $k$ is the number of non-zero entries in the relevant row — potentially orders of magnitude cheaper than executing the full WASM branch, which requires loading agent state, instantiating the Wasm module, running the component, and observing the result.

**The pipeline:**

1. **Pre-filter:** Use SR to predict which branches are likely to reach high-value states. Rank branches by predicted value.
2. **Selective execution:** Only execute the top-$k$ branches via full WASM replay. Discard the rest.
3. **HER update:** Relabel all executed branches (including failures) as training data for the SR.
4. **SR update:** Update $M$ with the actual transitions observed.

The SR doesn't replace the WASM replay — it *screens* for it. Only branches that the SR predicts might be valuable get the expensive full execution.

### Why This Might Work

- **The SR is a compressed model of transition dynamics.** It captures the structural regularities of the system — which components tend to lead to which outcomes — without needing to simulate the full execution.
- **HER provides dense training signal from sparse rewards.** Most branches fail, but HER turns those failures into valid SR training data. The SR improves even when the task goal isn't reached.
- **The cost asymmetry is real.** A matrix-vector multiply on a sparse SR is microseconds. A full WASM branch replay is milliseconds to seconds. If the SR's predictions are even roughly correct, the filtering saves massive compute.

### Where It Breaks Down

**High-dimensional continuous states.** SR requires discrete state enumeration for the canonical $M^\pi$ matrix. CLADE's state space — lattice attractor positions, agent memory contents, capability sets — is not naturally discrete. Discretization introduces quantization error, and the state space may be too large for exact SR. Approximate SR methods (successor features, deep SR) exist but add complexity and may lose the "cheap matrix multiply" advantage.

**Partial observability.** SR assumes the agent observes the full state. In CLADE, agents operate on local state only ([../insights.md](../insights.md) Theme 4: CRDT state is a wavefront, not a snapshot). The SR would need to operate on *local* successor representations — predicting future local states from current local observations. This is a weaker but potentially sufficient approximation. Honest assessment: unvalidated.

**Fuzzy goal structure.** HER relabels trajectories with achieved goals. This works when goals are well-defined state targets ("reach state $s'$"). CLADE's goals are often fuzzy — "sync my tablet" isn't a single state but a distribution over acceptable outcomes. Defining the goal space for HER requires agreeing on what "success" means for each task class. This is a design problem, not a mathematical one, but it's not trivial.

**SR staleness.** $M^\pi$ is computed for a specific policy $\pi$. When the BSCA fabric adapts via STDP (plastic readout weights changing), the transition dynamics shift. The SR must be re-estimated periodically, or it drifts from reality. The re-estimation cost must be weighed against the filtering savings.

### Connections to Existing Notes

**[wasm-credit-assignment-replay](wasm-credit-assignment-replay.md):**
SR+HER is a direct optimization of the WASM replay system. The replay is the expensive oracle; SR+HER is the cheap filter that decides which branches are worth replaying. The two systems are complementary, not competing.

**[bsca-fabric](bsca-fabric.md):**
The BSCA's attractor basins ARE the state space for the SR. Each attractor basin is a discrete state in the SR matrix. Transitions between basins (driven by incoming spikes and STDP weight changes) define $T^\pi$. The SR captures the dynamics of basin transitions — which attractor the system tends to flow toward from any given attractor.

**[surprise-driven-bootstrapping](surprise-driven-bootstrapping.md):**
SR and surprise-driven retrieval are alternative credit assignment mechanisms. The open question in §What's Still Open below — "Successor Representations vs surprise-driven credit assignment — which to test first?" — this section argues: test SR+HER first as a pre-filter, because it's cheaper and doesn't require the full surprise detection pipeline to be operational. SR+HER can run before the BSCA fabric is fully trained; it only needs transition data from WASM replays.

**[self-distillation-context-compression](self-distillation-context-compression.md):**
Both compress information into reusable structure. SDPO compresses interaction history into model weights. SR compresses transition dynamics into a matrix. The SR matrix is a form of distilled experience — it captures "what tends to happen" without storing individual trajectories.

**[tpo-target-policy-optimization](tpo-target-policy-optimization.md):**
TPO constructs target distributions over candidates. SR+HER constructs predictions over future states. They operate at different levels: TPO is "which action was better, given scores"; SR+HER is "which future state is reachable, given dynamics." Both replace expensive search with cheap computation. Potentially composable: use SR to pre-filter branches, TPO to construct the update target for branches that pass the filter.

**[pulse-feedback](pulse-feedback.md):**
The SR could determine which feedback pulses are worth propagating. If the SR predicts that a given component's output won't significantly change the reachable state space, the feedback pulse can be dropped — saving communication bandwidth in the swarm.

## The Proposed Hybrid

```
┌─────────────────────────────────────────────────────────────────┐
│                     SENSORY LAYER (GPU)                          │
│  SSM or LTCN — handles streaming input                           │
└──────────────────────┬──────────────────────────────────────────┘
                       │ Inhibitory gating (default: suppress)
                       ▼
┌─────────────────────────────────────────────────────────────────┐
│                SOFT VQ BOTTLENECK (GPU)                          │
│  Codebook index + residual vector                                │
└──────────────────────┬──────────────────────────────────────────┘
                       │ Sparse pulse sequence
                       ▼
┌─────────────────────────────────────────────────────────────────┐
│                     BSCA FABRIC                                  │
│  Three memory systems in one lattice:                           │
│    Episodic: Transient attractor basins                         │
│    Semantic: Stable fixed points (STDP)                         │
│    Procedural: Spatiotemporal patterns (tool sequences)        │
│  Inhibitory gating: tonic inhibition, pathway release          │
│  Surprise detection: local prediction error → broadcast         │
│  Modulation budget: bounded SSM parameter drift                 │
└──────────────────────┬──────────────────────────────────────────┘
                       │
┌─────────────────────────────────────────────────────────────────┐
│                     LEARNING SYSTEM                              │
│  SR+HER: pre-filter WASM branches via successor representation  │
│  Surprise-driven bootstrapping                                   │
│  STDP for BSCA spike-timing                                      │
│  WASM branching replay for credit assignment (post-filter)      │
│  Self-supervised surprise mining for training data               │
│  TPO for gate policy updates (post-filter)                      │
└─────────────────────────────────────────────────────────────────┘
```

## What's Still Open

- LTCN vs SSM — is continuous-time worth the added complexity before the basic architecture is validated?
- Hard VQ (GC) vs soft VQ (Kimi) — both theoretically defensible, need empirical comparison
- WebGPU sparse ops vs Vulkan compute — deployment question, not architecture question
- SR+HER state discretization — how coarse can the attractor basin quantization be before prediction quality degrades past usefulness?
- SR update frequency — how often must $M^\pi$ be re-estimated under STDP plasticity before staleness dominates?

---
#clade #architecture #hierarchical-control #inhibitory-gating #procedural-memory #evaluation
