# Surprise-Driven Memory & Practical Architecture

Related: [heterogeneous-cognitive-architecture](heterogeneous-cognitive-architecture.md), [snn-gate-and-memory-design](snn-gate-and-memory-design.md), [coupled-dynamical-architecture](coupled-dynamical-architecture.md), [pulse-feedback](pulse-feedback.md), [bsca-fabric](bsca-fabric.md), [tpo-target-policy-optimization](tpo-target-policy-optimization.md)

Gap analysis challenging the 6-agent synthesis. Eight contributions ranging from architecture corrections to a novel bootstrapping mechanism. Source: CC.

## 1. Start with a Differentiable MLP, Not an SNN

The 5-neuron SNN gate (from [snn-gate-and-memory-design](snn-gate-and-memory-design.md)) uses priority ordering — which isn't a real SNN mechanism. Real basal ganglia use lateral inhibition (winner-take-all), not priority lists.

But with only 5 outputs, a softmax MLP captures 90% of the functionality with full differentiability. The SNN doesn't earn its complexity until you need temporal dynamics: burst patterns, refractory periods, spike-timing-sensitive decisions. A 5-way classifier doesn't need any of that.

**Recommendation:** Start with a differentiable gating MLP. Prove the SSM + memory + gate architecture works end-to-end with gradients. Replace with SNN in phase 2 and measure what temporal dynamics buy you.

## 2. Dual Memory: Episodic vs Factual

The notes celebrate lossy reconstructive memory as "more honest." But an agentic OS needs both:

| Memory Role | Needs Exact Recall? | Example |
|-------------|-------------------|---------|
| Episodic (what happened) | No — reconstruction fine | "We discussed schema design last week" |
| Semantic/Factual (what's true) | Yes — lossy is catastrophic | "The git status showed 3 modified files" |

The system needs to remember tool outputs, file contents, user instructions verbatim. "Something about... tables..." when the user asked "what did grep return?" is a broken tool, not honest uncertainty.

**Fix:** Dual memory — associative store (lossy, reconstructive) for episodic context, small exact buffer (ring buffer or KV) for recent tool outputs and facts. The gate routes to the correct memory by pathway. More biologically plausible: hippocampus has separate pathways for episodic (CA3→CA1 pattern completion) and semantic (entorhinal grid cells, precise encoding).

## 3. OODA Loop Framing

The architecture is Boyd's OODA loop made concrete:

| OODA Phase | Component | Latency Requirement |
|------------|-----------|-------------------|
| **Observe** | SSM streaming | Per-token (ms) |
| **Orient** | Memory retrieval | Per-event (10-100ms) |
| **Decide** | SNN gate spike | Per-decision (1-10ms) |
| **Act** | LM head / tool call | Per-action (100ms-seconds) |

Critical constraint: Orient must complete before Decide fires. If memory retrieval takes longer than token rate, the gate must wait or decide without context. This is the [pulse-feedback](pulse-feedback.md) timing problem — the pulse sequence between memory and gate must have known maximum latency, or the gate needs a confidence threshold to fire without memory.

## 4. Credit Assignment Across Timescales

The three layers learn at fundamentally different timescales:

| Layer | Learning Timescale | Update Mechanism |
|-------|-------------------|-----------------|
| SSM | Per-token (milliseconds) | Gradient descent on next-token prediction |
| Memory | Per-event (seconds) | Write/interference/decay per retrieval cycle |
| SNN gate | Per-episode (minutes) | RL reward per task completion |

When the system fails, which component gets the blame? Wrong SSM representation? Gate retrieved at wrong time? Memory returned garbage? This is temporal credit assignment across three timescales.

Options:
- **Counterfactual replay:** Re-run each component in isolation. Would the gate have decided correctly with correct memory? If yes, blame memory.
- **Advantage decomposition:** Compute advantage for each component by marginalizing out others.
- **Pulse feedback as credit signal:** Structured token sequences from [pulse-feedback](pulse-feedback.md) carry information about which upstream component failed. The strongest argument for pulse sequences over scalar rewards — they encode blame.

## 5. Surprise-Triggered Retrieval

The gate's "retrieve" neuron fires — but what causes it? The notes don't specify.

**Mechanism:** Retrieve when SSM state shows high prediction error.

```
surprise_t = CE(predicted_token, actual_token)
if surprise_t > threshold:
    trigger_memory_retrieval(project(h_t))
```

Biologically grounded — hippocampus activates on novel/surprising stimuli. Practically useful — system only retrieves when it needs to, saving bandwidth and reducing interference from unnecessary reads.

The SNN gate learns this threshold: SSM surprise signal becomes an additional input to the "retrieve" neuron, learning when surprise warrants retrieval vs. when the SSM can handle it alone.

## 6. The Bootstrapping Curriculum

How does the system learn to use memory before it has good memories? Chicken-and-egg.

**Key insight:** Surprise is both the retrieval trigger AND the write signal. The same mechanism stores and retrieves:

1. **Phase 0:** SSM alone, no memory. Gate always fires "generate." Standard SSM language model.
2. **Phase 1:** Add memory with supervised labels — "given this context, correct action is RETRIEVE." Gate learns when retrieval helps. Memory populated during training.
3. **Phase 2:** Add surprise-triggered retrieval. Gate learns to fire "retrieve" on high surprise. System discovers retrieval reduces surprise — the bootstrap point.
4. **Phase 3:** Add write decisions. Gate learns to store high-surprise moments — the brain consolidates surprising events preferentially.

Surprise is the simplest possible bootstrapping curriculum.

## 7. K=4 as Resonant Encoding Constraint

The BSCA lattice at K=4 (edge of chaos) has maximum information transmission bounded by the Lyapunov exponent λ ≈ 0. Information propagates slowly with maximal sensitivity to initial conditions.

**Implication:** The SSM → CNS projection must compress state into patterns that *survive* the lattice's edge-of-chaos dynamics. Not all projections work — only those that map to stable attractor basins. The interface isn't a linear projection; it's a **resonant encoding** — the projected pattern must be a natural mode of the lattice.

This means the SSM's output projection must be co-designed with the CA's rule set. You can't independently design the SSM and the CA and hope the interface works.

## 8. Vulkan Compute for BSCA Operations

Missing middle between "FPGA needed" and "GPU is fine." Vulkan compute subgroup operations on consumer GPUs.

Binary XNOR + population count (core BSCA operations) map to `VK_SUBGROUP_FEATURE_ARITHMETIC_BIT` — hardware-native ballot and inclusive scan. On RDNA3, subgroup operations run at 1/4 SP throughput with zero divergence penalty for binary ops.

The SNN/CA simulation runs as a Vulkan compute shader alongside SSM inference on the same GPU. No PCIe transfer. The [bsca-fabric](bsca-fabric.md) doesn't need dedicated hardware — it needs a well-written compute shader.

For RX 7900 XTX (24 GB VRAM): 1024×768 memory matrix (3 MB) + BSCA lattice (4096 cells × 4 LUTs ≈ 64 KB) fits in SRAM alongside a 3B SSM with room to spare.

---

**Bottom line:** The strongest contribution isn't the SNN gate (start as MLP) or the FPGA mapping (start as Vulkan shader). It's surprise-driven memory bootstrapping — a system that discovers the value of memory through prediction error, stores surprising moments, retrieves on future surprise, and develops genuine uncertainty from genuinely lossy recall. That's a testable, falsifiable claim that no transformer makes.

---
#clade #architecture #surprise #credit-assignment #ooda #vulkan #practical
