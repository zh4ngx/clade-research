# Coupled Dynamical Systems Architecture

Related: [heterogeneous-cognitive-architecture](heterogeneous-cognitive-architecture.md), [snn-gate-and-memory-design](snn-gate-and-memory-design.md), [bsca-fabric](bsca-fabric.md), [pulse-feedback](pulse-feedback.md)

**A competing view to the pipeline synthesis.** The five-agent consensus built a pipeline: SSM → Gate → (Memory or Action). This note proposes a coupled oscillator: SSM ↔ BSCA CA Fabric (which intrinsically holds memory). A closed loop, not a feed-forward flowchart.

Source: GC gap analysis, challenging the 5-agent consensus.

## 1. The PCIe Bottleneck Kills O(1)

The consensus: run SSM on GPU, BSCA/SNN on FPGA, each on optimal hardware.

The problem: if the SNN gates every token (deciding generate/retrieve/tool-use), you pass a dense hidden state (4096-dim floats) from GPU, across PCIe, to FPGA, wait for the spike cascade, and send the binary decision back. PCIe latency and bandwidth turn the fast SSM into a stuttering mess.

You cannot split a tight cognitive control loop across a high-latency hardware bus.

**Two paths:**

| Path | How | Tradeoff |
|------|-----|----------|
| **Unified Silicon (SoC)** | Apple M-series or AMD APU — GPU cores and CPU/NPU share the same physical RAM. No PCIe copy. | Hardware-constrained, but latency is solved |
| **Synthesized Mamba** | Deeply quantize a tiny SSM (130M–370M params, INT4) and map it onto FPGA DSP slices alongside [bsca-fabric](bsca-fabric.md). The FPGA becomes the entire brain. | Less SSM capacity, but zero inter-chip latency |

The pipeline architecture assumed hardware heterogeneity was free. It isn't — the bus is the bottleneck.

## 2. VQ Bottleneck as Thalamus

The consensus: linear projection from continuous SSM vectors to discrete SNN spikes. Flagged as "hard open problem."

The problem: linear projection of dense floats to spike thresholds is biologically inaccurate and mathematically unstable. Massive impedance mismatch between dense/continuous and sparse/discrete.

**The fix: Vector Quantized (VQ) bottleneck between SSM and CNS.**

Borrowing from VQ-VAEs (audio/image generation):
1. Force the continuous SSM hidden state to map to a discrete codebook of concepts
2. Codebook of 10,000 entries → 10,000 input neurons on the BSCA fabric
3. The SSM activates a sparse set of indices (e.g., `[URGENT]`, `[DATABASE]`, `[USER_QUERY]`)
4. These indices become instantaneous binary spikes (1s) fed into the SNN

The VQ layer naturally acts as the "Thalamus" — perfectly bridging continuous and discrete worlds. It's differentiable, mathematically sound, and produces the sparse activation pattern the BSCA fabric expects.

## 3. Attractor Memory, Not Matrix Memory

The consensus: associative memory as a passive PyTorch matrix `M` queried via dot-product similarity.

The problem: if you're already building a [bsca-fabric](bsca-fabric.md) CA fabric for the CNS, a separate passive matrix is redundant and philosophically misaligned with the active biological architecture.

**Merge memory and CNS.** In dynamical systems, memories are attractor basins.

- The BSCA fabric IS the memory. Memories are stable topological patterns (gliders, oscillators) within the CA grid, encoded via STDP weight changes
- When the SSM sends a cue, it perturbs the CA grid
- The grid collapses into the nearest stable attractor state (the memory)
- The output isn't a retrieved vector — it's a temporal sequence of spikes ([pulse-feedback](pulse-feedback.md)) emitted by that attractor

This unifies System 1 routing and memory into a single, active, power-efficient spatial substrate. No separate memory layer needed.

## 4. Top-Down Modulation of SSM Parameters

The consensus: SNN sits at the output of the SSM, acting as a switchboard (generate vs retrieve vs tool).

The problem: biological prefrontal cortex doesn't just gate outputs — it exerts massive top-down control over the sensory cortex itself (attention).

**The fix: SNN spike trains feed back into the SSM to dynamically alter its internal parameters.**

In Mamba, the core parameters (step size Δ, matrices B and C) are data-dependent. The SNN can modulate these:

- High-uncertainty state → SNN fires pulse sequence that lowers Δ, forcing the SSM to "slow down" and integrate context more carefully
- Low-uncertainty state → Δ increases, SSM processes faster

The SNN isn't just routing the SSM's output — it's actively tuning the working memory dynamics of the SSM in real-time. This is biological attention made concrete.

## The Architectural Shift

```
Pipeline (5-agent consensus):
  SSM ──→ Gate ──→ (Memory or Action)
  Feed-forward, modular, clean separation

Coupled Oscillator (this proposal):
  SSM ←──[VQ Bottleneck]──→ BSCA CA Fabric (= Memory + CNS)
  Closed loop, coupled dynamics, no separate memory subsystem
```

The pipeline is easier to reason about and prototype. The coupled oscillator is harder to build but more faithful to biological dynamics and avoids the PCIe bottleneck. The VQ bottleneck is the key enabling mechanism — it makes the continuous ↔ discrete bridge differentiable and tractable.

## Open Questions

- Can a 130M INT4 Mamba on FPGA DSP slices actually perform useful language processing?
- What codebook size balances expressivity vs BSCA fabric input width?
- Does attractor memory (CA basins) actually store and retrieve as well as a matrix for the kinds of facts an agent needs?
- How do you train a coupled oscillator end-to-end? The VQ bottleneck helps (differentiable), but the CA dynamics are still non-differentiable
- Is this a research question or an engineering question? (Probably both — the VQ→CA interface is research, the SoC deployment is engineering)

---
#clade #architecture #coupled-oscillator #vq-vae #attractor-memory #bio-inspired
