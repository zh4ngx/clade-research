---
date: 2026-04-04
tags: [neuromorphic, FPGA, SNN, BNN, cellular-automata, hardware, architecture, clade]
aliases: [Binarized Spiking Cellular Automata, BSCA, Agentic OS Fabric]
---

# The BSCA Fabric: Binarized Spiking Cellular Automata

Related: [bms-edge-ml](bms-edge-ml.md), [dual-process-routing](dual-process-routing.md), [hardware-polymorphism](hardware-polymorphism.md)

## The Sensory-Cognitive Split

A local-first, always-on agentic OS abandons the von Neumann bottleneck for a biological split:

- **Interface / Cerebral Cortex:** Small quantized SLM (e.g., 8B Transformer). Specialized for human semantics, power-hungry, stateless. Sleeps 99% of the time, wakes only to parse language or format output.
- **Core Fabric / Autonomic Nervous System:** The BSCA. Always-on, event-driven. Routes tasks, tracks local state, stores memories — drawing fractions of a watt.

This maps directly to [dual-process-routing](dual-process-routing.md): BSCA is System 1 (amygdala — fast, reflexive, continuous), the SLM is System 2 (prefrontal cortex — deliberative, expensive, episodic).

## The BSCA Paradigm

Three paradigms unified into a single Turing-complete substrate:

1. **Cellular Automata (Topology)** — Massively parallel grid, cells communicate only with immediate neighbors. Solves global routing and wire-delay issues.
2. **Binarized Neural Networks (Spatial Efficiency)** — Synaptic weights restricted to +1 or -1. Floating-point matrix multiplication replaced by hardware logic gates.
3. **Spiking Neural Networks (Temporal Efficiency)** — Event-driven. Cells update only on discrete spikes, no synchronous global clock.

State update for cell $i$ at time $t$ from local neighborhood $N$:

$$V_{i}(t+1) = \alpha V_{i}(t) + \sum_{j \in N} \text{XNOR}(w_{ij}, S_j(t))$$

## FPGA Hardware Mapping

Modern FPGAs natively support this without expensive tensor cores:

| Resource | Role |
|----------|------|
| **6-Bit LUTs** | Cellular update rules and boolean logic (XNOR, AND, Majority voting) |
| **BRAM** | Distributed memory for localized membrane potential ($V_i$) and latent weight counters — no central RAM fetches |
| **Bit-Shifts (`>>`)** | Temporal voltage decay ($\alpha$) at near-zero power cost |

This is [hardware-polymorphism](hardware-polymorphism.md) made concrete: the agent IS the FPGA fabric, not a process running ON it.

## Training & Plasticity

Standard backpropagation fails (non-differentiable spikes, 1-bit weights). Two learning modes:

### Offline Compilation (GPU Foundry)
- **Surrogate Gradients** — replace step-function spike with continuous approximation during backward pass
- **Straight-Through Estimators (STE)** — maintain high-precision shadow weights during training, snap to binary for forward pass, pipe gradient through

### On-Device Adaptation (STDP)
Spike-Timing-Dependent Plasticity — unsupervised local learning via causality ($\Delta t = t_{post} - t_{pre}$):

- **LTP (Long-Term Potentiation):** Input precedes output → causality → strengthen
- **LTD (Long-Term Depression):** Input follows output → uncorrelated → weaken

**Shadow Weight Bridge:** Continuous STDP applied to binary hardware via a *Latent Weight* (integer counter in BRAM). STDP adds/subtracts from the counter. When it overflows/underflows a threshold, the *Expressed Weight* (1-bit logic gate) flips.

## Intelligence Encoding & Routing

STDP's temporal asymmetry breaks the CA grid's structural symmetry, carving unidirectional functional pathways by rewarding forward causality (LTP) and penalizing backward echoes (LTD).

Tool execution and contextual behavior encoded through **Attractor Basins** — spike waves interact with recurrent pathways, the graph settles into stable spatiotemporal patterns. Activating a tool collapses the network into its associated attractor; polymorphic cell clusters handle multiple contextual meanings dynamically.

## Deep Research: Next Steps

- **BCM Theory** — sliding modification threshold ($\theta_M$) for synaptic homeostasis, preventing runaway excitation
- **Metaplasticity** — neuron's activity history dictates its future LTP/LTD threshold
- **FPGA Implementation** — formulating BCM sliding thresholds using only integer accumulators and bit-shifts in BRAM

---
#neuromorphic #fpga #snn #bnn #cellular-automata
