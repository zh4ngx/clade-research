# Coupled Dynamical Systems Architecture

> A competing view: closed loop, not pipeline.

Companion to [heterogeneous-cognitive-architecture.md](heterogeneous-cognitive-architecture.md) and [snn-gate-and-memory-design.md](snn-gate-and-memory-design.md). Source: GC gap analysis challenging the 5-agent consensus.

## The Shift

The five-agent consensus built a pipeline: SSM → Gate → (Memory or Action). This note proposes a coupled oscillator: SSM ↔ BSCA CA Fabric (which intrinsically holds memory). A closed loop, not a feed-forward flowchart.

## 1. The PCIe Bottleneck Kills O(1)

The consensus: run SSM on GPU, BSCA/SNN on FPGA, each on optimal hardware.

The problem: if the SNN gates every token, you pass dense hidden state (4096-dim floats) across PCIe to FPGA, wait for spike cascade, send decision back. PCIe latency destroys the SSM's O(1) advantage.

You cannot split a tight cognitive control loop across a high-latency hardware bus.

Two paths:
- **Unified Silicon (SoC):** Apple M-series, AMD APU — shared physical RAM, no PCIe copy. Hardware-constrained but latency-solved.
- **Synthesized Mamba:** Quantize a tiny SSM (130M–370M params, INT4) onto FPGA DSP slices alongside BSCA fabric. FPGA becomes the entire brain. Less SSM capacity, zero inter-chip latency.

The pipeline assumed hardware heterogeneity was free. It isn't — the bus is the bottleneck.

## 2. VQ Bottleneck as Thalamus

The consensus: linear projection from continuous SSM vectors to discrete SNN spikes. "Hard open problem."

Linear projection of dense floats to spike thresholds is biologically inaccurate and mathematically unstable. Massive impedance mismatch.

Fix: **Vector Quantized (VQ) bottleneck** between SSM and CNS (borrowed from VQ-VAEs):
1. Force continuous SSM hidden state to map to a discrete codebook of concepts
2. Codebook of 10K entries → 10K input neurons on BSCA fabric
3. SSM activates sparse set of indices (e.g., `[URGENT]`, `[DATABASE]`, `[USER_QUERY]`)
4. Indices become instantaneous binary spikes fed into SNN

Differentiable, mathematically sound, produces sparse activation the BSCA fabric expects. The VQ layer is the Thalamus.

## 3. Attractor Memory, Not Matrix Memory

Consensus: associative memory as passive PyTorch matrix queried via dot-product.

If you're already building a BSCA CA fabric for the CNS, a separate passive matrix is redundant and philosophically misaligned.

**Merge memory and CNS.** Memories are attractor basins in the CA fabric:
- BSCA fabric IS the memory — stable topological patterns (gliders, oscillators) encoded via STDP
- Cue from SSM perturbs the CA grid
- Grid collapses into nearest stable attractor (the memory)
- Output is temporal spike sequence from that attractor (pulse-feedback.md)

Unifies System 1 routing and memory into a single active substrate. No separate memory layer.

## 4. Top-Down Modulation of SSM Parameters

Consensus: SNN at SSM output, acting as switchboard.

Biological prefrontal cortex doesn't just gate outputs — it exerts top-down control over sensory cortex (attention).

Fix: **SNN spike trains feed back into SSM to alter its internal parameters.** In Mamba, step size Δ and matrices B/C are data-dependent:
- High uncertainty → SNN lowers Δ, forcing SSM to integrate more carefully
- Low uncertainty → Δ increases, SSM processes faster

The SNN actively tunes working memory dynamics in real-time. Biological attention made concrete.

## Open Questions

- Can a 130M INT4 Mamba on FPGA DSP slices perform useful language processing?
- What codebook size balances expressivity vs BSCA fabric input width?
- Does attractor memory (CA basins) store/retrieve as well as a matrix for agent-relevant facts?
- How do you train a coupled oscillator end-to-end? VQ helps (differentiable), but CA dynamics are still non-differentiable
- Is this research or engineering? (VQ→CA interface is research, SoC deployment is engineering)
