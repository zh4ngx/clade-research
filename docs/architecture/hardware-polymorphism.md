---
tags: [architecture, clade, hardware, wasi]
date: 2026-03-26
---

# Hardware Polymorphism in CLADE

**Definition:** The ability of a mathematically verified, stateless agent to execute identical logic across radically different physical substrates without recompilation or architectural awareness.

**Implementation:**
Achieved via the WebAssembly Component Model (WASI 0.3). The emergent agent is compiled to `wasm32-wasi`. It interfaces with its environment exclusively through abstract contracts (e.g., `wasi-lattice` for binary compute, `wasi-tiered-memory` for state).

The underlying host OS (the Rust hypervisor) dynamically maps these WASI calls to the available local hardware (AMD 6900XT via ROCm, bare-metal FPGAs, or local CPU emulation). The agent remains mathematically contained while the hardware execution is fully polymorphic.

**Optimization Metrics:**
* Joules per inference (Energy cost)
* Clock cycles per state resolution (Time cost)

---

Related: [clade-architecture](../architecture.md), [bsca-fabric](bsca-fabric.md)
