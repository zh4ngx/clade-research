# Surprise-Driven Memory & Practical Architecture

> Gap analysis: bootstrapping, credit assignment, and pragmatic first steps.

Companion to [heterogeneous-cognitive-architecture.md](heterogeneous-cognitive-architecture.md), [snn-gate-and-memory-design.md](snn-gate-and-memory-design.md), and [coupled-dynamical-architecture.md](coupled-dynamical-architecture.md). Source: CC gap analysis.

## 1. Start with MLP, Not SNN

The 5-neuron SNN gate uses priority ordering — not a real SNN mechanism (basal ganglia use lateral inhibition, winner-take-all). With 5 outputs, a softmax MLP captures 90% of functionality with full differentiability. SNN doesn't earn its complexity until temporal dynamics matter.

Start with differentiable gating MLP. Prove architecture works with gradients. Replace with SNN in phase 2.

## 2. Dual Memory: Episodic vs Factual

| Role | Exact Recall? | Example |
|------|--------------|---------|
| Episodic | No — reconstruction fine | "We discussed schema design" |
| Semantic/Factual | Yes — lossy is catastrophic | "grep returned 3 modified files" |

Tool outputs, file contents, instructions must be verbatim. "Something about tables" for a grep result is a broken tool. Fix: dual memory (associative + exact ring buffer). Gate routes by pathway. Biologically grounded: hippocampus has separate episodic (CA3→CA1) and semantic (entorhinal grid cells) pathways.

## 3. OODA Loop

| Phase | Component | Latency |
|-------|-----------|---------|
| Observe | SSM streaming | Per-token (ms) |
| Orient | Memory retrieval | Per-event (10-100ms) |
| Decide | Gate spike | Per-decision (1-10ms) |
| Act | LM head / tool | Per-action (100ms-sec) |

Orient must complete before Decide. Memory latency > token rate → gate must wait or fire without context. This is the pulse-feedback timing constraint.

## 4. Credit Assignment Across Timescales

| Layer | Timescale | Mechanism |
|-------|-----------|-----------|
| SSM | Per-token (ms) | Gradient descent |
| Memory | Per-event (sec) | Write/decay per cycle |
| Gate | Per-episode (min) | RL reward |

Failure attribution: wrong SSM encoding? Gate timed wrong? Memory returned garbage? Options: counterfactual replay, advantage decomposition, or pulse-feedback sequences that encode blame.

## 5. Surprise-Triggered Retrieval

```
surprise_t = CE(predicted_token, actual_token)
if surprise_t > threshold:
    trigger_memory_retrieval(project(h_t))
```

Hippocampus activates on novel stimuli. System retrieves only when needed. SSM surprise → additional input to gate's "retrieve" neuron.

## 6. Bootstrapping Curriculum

Surprise is both retrieval trigger AND write signal:

1. Phase 0: SSM alone, gate always generates
2. Phase 1: Add memory with supervised retrieval labels
3. Phase 2: Surprise-triggered retrieval — system discovers retrieval reduces surprise
4. Phase 3: Write decisions — store high-surprise moments preferentially

Simplest possible bootstrapping curriculum.

## 7. K=4 as Resonant Encoding

At edge of chaos (λ ≈ 0), information transmission is bounded. SSM → CNS projection must compress into patterns that survive lattice dynamics — resonant encoding, not linear projection. SSM output projection must be co-designed with CA rule set.

## 8. Vulkan Compute for BSCA

XNOR + popcount maps to `VK_SUBGROUP_FEATURE_ARITHMETIC_BIT` on RDNA3 — 1/4 SP throughput, zero divergence for binary ops. BSCA fabric runs as Vulkan compute shader alongside SSM on same GPU. No PCIe transfer.

RX 7900 XTX (24 GB): memory matrix (3 MB) + BSCA lattice (~64 KB) fits in SRAM alongside 3B SSM.

---

**Core claim:** Surprise-driven memory bootstrapping is the testable, falsifiable contribution. A system that discovers memory value through prediction error, stores surprise, retrieves on future surprise, develops genuine uncertainty from genuinely lossy recall. No transformer makes this claim.
