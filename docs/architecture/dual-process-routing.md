---
tags: [clade, agents, system-design, cognition]
date: 2026-03-26
---

# Dual-Process Agent Routing (System 1 / System 2)

**Concept:** Mapping Daniel Kahneman's dual-process theory of human cognition directly onto heterogenous computing architecture.

**System 1 (The Amygdala):**
* **Hardware:** CPU (Ryzen 9950X / Edge Microcontroller).
* **Runtime:** Wasmtime / WAMR.
* **Function:** Hard-real-time, low-latency, localized logic. Runs the immediate continuous agent loop. Handles reflexive actions, tiered memory read/writes, and basic swarm signaling. Operates in microseconds.

**System 2 (The Prefrontal Cortex):**
* **Hardware:** Accelerators (AMD 6900XT / FPGA).
* **Runtime:** Massive XNOR / BNN Lattices.
* **Function:** Heavy, deliberative computation. When System 1 encounters an unsolvable localized state, it initiates an asynchronous WASI call to System 2. The CPU agent yields its thread, allowing System 2 the necessary clock cycles to process the high-dimensional binary lattice before returning the strategic state update.

---

Related: [clade-architecture](../architecture.md), [hardware-polymorphism](hardware-polymorphism.md)
