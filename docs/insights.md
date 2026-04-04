# Research Insights

> Distilled findings from NotebookLM Deep Research synthesis.
> Paste the synthesis output here after running Deep Research.

## Executive Summary

(To be filled)

## Key Themes

### 1. Memory as Active Computation (not passive cache)

**Systolic Array Insight (2026-03-18):**
Paper author proposes using CPU + smart indexing instead of massive KV cache in transformers. This challenges the assumption that inference requires materializing all context into a giant passive cache.

**Connection to CLADE:**
Compare to the idea of models managing their own memory load/store operations. Both approaches share a common thread:
- *Traditional*: Memory as passive substrate (cache sits there, model reads)
- *Emerging*: Memory as active, indexed, agent-controlled resource

This aligns with capability-based thinking - the model has explicit capabilities for memory access rather than implicit access to everything.

**Connection to SSMs:**
This indexed/sparse attention approach is part of a broader trend moving away from O(n²) attention mechanisms. State Space Models (SSMs like Mamba) represent another path—both share the goal of breaking the quadratic bottleneck. Indexed attention does it by being selective about what enters the cache; SSMs do it by replacing attention with recurrent state compression. Both move memory from passive to active/compressed.

### 2. FPGA as Native Boolean Substrate: Beyond Matrix-Multiplication Emulation (2026-03-28)

**The Problem with XNOR-Popcount:**
Standard binarized neural networks use XNOR and popcount as their fundamental arithmetic. This reduces each neuron to a Linear Threshold Gate (LTG) — it can only compute linearly separable functions. This is a compromise inherited from the matrix-multiplication paradigm: the BNN is trying to emulate a dot product in binary, so it uses the binary equivalent of MAC operations. The expressivity ceiling is real.

**LUTs Are Not Fast XNOR Gates — They Are Universal Boolean Evaluator:**
The fundamental building block of an FPGA is the Look-Up Table (LUT): a $K$-input SRAM-backed truth table that can implement *any* of the $2^{2^K}$ possible Boolean functions for its inputs. A LUT6 (6-input, standard on modern FPGAs) can evaluate any 64-entry truth table. XNOR is one entry in that space. By mapping binarized neurons directly onto LUTs rather than using LUTs to emulate matrix multipliers, each node regains the full functional expressivity of a classical CA cell — no linear-separability constraint.

This is not an incremental improvement. It is a category change: from "binary accelerator for linear operations" to "native Boolean logic synthesis engine."

**Topological Freedom via Programmable Routing:**
Classical CAs are constrained to rigid, homogeneous spatial grids. FPGA switch matrices and programmable routing networks interconnect logic blocks into arbitrary topologies. This allows LUTs to be wired as Directed Cyclic Graphs (DCGs) — supporting long-range connections, self-loops, and heterogeneous feedback cycles. Because each LUT holds a distinct truth table, the system operates as a Non-Uniform CA (NUCA) or Random Boolean Network (RBN), where every node performs a specialized, distinct logic function. This is the physical realization of the Module B NUCA↔DCG isomorphism.

**Hardware-Native Memory for Temporal Recurrence:**
Inside the FPGA, every LUT is paired with a local output register (D Flip-Flop) to form a Basic Logic Element (BLE). Distributed Block RAM (BRAM) provides additional dense storage. These give the network native state retention across clock cycles — the "fading memory" property required for reservoir computing and non-Markovian temporal reasoning. No external memory hierarchy. No off-chip bandwidth bottleneck for state. The memory is the logic.

**The Synthesis:**
When the FPGA is treated as a sea of parameterized Boolean rules — not as a von Neumann accelerator — we get:
- Maximal local logical expressivity (any Boolean function per node)
- Heterogeneous graph topologies (arbitrary DCGs, not grids)
- Hardware-native recurrent memory (BLE registers + BRAM)
- Learnable basins of attraction (the state space of the NUCA organizes into attractor basins)

This answers the CLADE post-POSIX question: the agent is not a process. It is a pattern of Boolean dynamics resident in the FPGA fabric — a self-organizing attractor in a field of LUTs.

**Connection to Theme 4 (Temporal Scaling):**
The BLE registers are what make temporal iteration physically possible. Each clock cycle updates the DFF state from the LUT output. The recurrent reservoir iterates in-place — same fabric, same routing, same truth tables — with each cycle refining the attractor state. Temporal scaling is not just economically preferable here; it is the *native operational mode* of the hardware.

**Caveat on XNOR-Popcount:**
XNOR-popcount is not deprecated — it is a special case. For dense, uniform operations where the full truth table is unnecessary (e.g., bulk similarity matching in the systolic array), XNOR-popcount remains the right tool. But it is no longer the ceiling of per-node expressivity. The lattice should default to full LUT evaluation and fall back to XNOR-popcount only where the operation is provably linear.

### 3. Agent as Von Neumann Machine

**Core Insight (2026-03-25):**
What if the agent *is* a von Neumann machine that faces the von Neumann bottleneck? The agent is essentially a CPU (or emulated CPU) that interacts with GPUs, FPGAs, and memory asynchronously—across multiple clock cycles—and must wait for responses.

**Connection to CLADE:**
This reframes the agent not as a stateless function but as a compute unit with its own bottleneck dynamics:
- Agent issues requests to accelerators (GPU/FPGA)
- Agent waits across clock cycles for async responses
- Agent manages its own memory hierarchy (load/store across slow and fast tiers)

**Connection to Systolic/Energy-Efficient ML:**
Incorporates insights from systolic array research on energy-efficient machine learning. Systolic arrays were designed to *reduce* the von Neumann bottleneck by keeping data flowing through compute elements. If agents are von Neumann machines, maybe the solution isn't to make them faster—but to make them *systolic* (spatial, data-flow-driven).

**Implication:**
The "process" abstraction hides the von Neumann bottleneck. A post-POSIX agent abstraction might expose it intentionally—making latency and data movement first-class concerns.

### 4. Temporal Scaling Over Spatial Scaling (2026-03-28)

**Core Principle:**
Clock cycles are cheap. Data movement is expensive. Therefore: scale by iterating over time, not by expanding over space.

**The K=4 Edge-of-Chaos Result:**
For a discrete recurrent network (cellular automata or BNN reservoir) operating near the boundary between order and chaos, the optimal per-node connectivity is $K \approx 4$. Below $K=4$, the dynamics are too ordered — the reservoir can't sustain enough fading memory to be computationally useful. Above $K=4$, the dynamics become chaotic — small perturbations amplify and destroy information. $K=4$ is the critical connectivity that maximizes the computational capacity of the reservoir.

This is not a tuning parameter. It is a structural constraint derived from the phase transition dynamics of discrete recurrent systems. The lattice must be wired at $K=4$.

**Why Temporal, Not Spatial:**
The transformer scaling paradigm adds depth (more layers) and width (more parameters) to gain capability. This is *spatial scaling* — it physically requires more silicon, more memory bandwidth, more data movement. In our substrate, data movement is the bottleneck: the CTC ratio demands 250 ops per byte of off-chip bandwidth to prevent logic starvation (Module A). Every additional spatial layer requires shipping activations across the memory hierarchy.

Temporal scaling instead re-applies the *same* fixed recurrent block across successive clock cycles. No additional parameters. No additional data movement. The same weights, the same SRAM buffers, the same PE grid — just iterated. Each clock cycle is a refinement step (an "epistemic update") that reduces uncertainty in the compact hidden state. The network "thinks longer" rather than "thinking bigger."

**The Physical Economics:**
- One clock cycle: ~21.6 fJ per operation at the XNOR-popcount level (Module A).
- One byte of off-chip data movement: orders of magnitude more energy, plus latency measured in *many* clock cycles.
- An inactive neuron in an async event-driven medium: ~zero power (Module B).

The math is unambiguous. Adding a recurrent clock cycle is nearly free. Adding a spatial layer (with its attendant data movement) is the expensive path.

**Connection to Theme 3 (Von Neumann Machine):**
The agent as a von Neumann machine faces the bottleneck: data must move between compute and memory. Temporal scaling is the systolic answer — keep data in place and iterate the computation. The agent doesn't fetch more data to scale; it fetches once and *thinks longer*.

**Implication for CLADE:**
This principle applies at every level of the stack:
- **Lattice level:** Recurrent reservoir iterates at K=4, no spatial expansion.
- **Agent level:** Wasm agents refine their output across async clock cycles rather than requesting more context.
- **Swarm level:** CRDT deltas converge over time rather than requiring synchronous spatial consensus (idea-frontier-v1.md §3.6: "time substitutes for coordination").

Stop scaling spatially. Go temporal.

**Design Corollary: Noise-Tolerant CA Dynamics**
Operating at K=4 near the edge of chaos maximizes computational capacity — but it also places the system one perturbation away from chaos. The CA transition rules must be structurally tolerant of precision loss and noise from the physical substrate: thermal bit flips in SRAM, electromagnetic interference on routing traces, clock skew in async circuits, and the stochastic sampling noise inherent in the Bernoulli projection layer (idea-frontier-v1.md §1.3, Stage 2).

This is not an error-handling layer added on top. It is a structural property of the basin-of-attraction landscape:
- **Deep basins:** Small perturbations (bit flips on 1–2 nodes) must not eject the global state from its current basin. The attractor must pull the perturbed state back.
- **Redundant encoding:** Critical state transitions should be encoded across multiple nodes, not single bits. A single-bit corruption changes nothing about the basin classification.
- **Temporal self-correction:** Because we scale temporally, a noise-induced error in clock cycle $t$ is naturally overwritten by the recurrent dynamics of cycles $t+1, t+2, \ldots$ — the reservoir's fading memory discards transient perturbations while preserving structural state. This is temporal scaling's secondary benefit: not just cheap computation, but automatic denoising.

The CA must treat noise as an expected input, not an exception. Design for the perturbed case, not the ideal case.

**Design Corollary: Hebbian Learning in 1-Bit — Latent Accumulation and Inhibitory Regulation**

Classic Hebbian learning strengthens connections between co-firing neurons: $\Delta w \propto x \cdot y$. In a 1-bit network where weights are $\{-1, +1\}$, you cannot incrementally "strengthen" a binary weight. There is no gradient to climb.

**How it actually works: Latent weight accumulation → threshold flip.**
Each binary weight $w_j \in \{-1, +1\}$ is backed by a latent accumulator $\tilde{w}_j \in \mathbb{R}$. The Hebbian co-occurrence signal ($\text{pre}_i$ fires and $\text{post}_j$ fires) increments $\tilde{w}_j$. Anti-Hebbian events ($\text{pre}_i$ fires, $\text{post}_j$ silent) decrement it. When $\tilde{w}_j$ crosses a threshold, the binary weight flips via a NOT operation. This is the BOLD framework from Module C constraints: discrete chain rule operating solely on logic gates — flipping binary states when XNOR conditions are met. No floating-point arithmetic exposed to the execution path. The continuous variable exists only to accumulate evidence; the hardware only sees the flip.

This maps directly onto the FPGA substrate: the latent accumulator lives in a BRAM counter, the threshold comparison is a LUT truth table lookup, and the binary weight flip is a single clock-cycle state transition on the DFF.

**The runaway problem and why inhibition is load-bearing.**
Pure excitatory Hebbian learning is unstable. Neurons that fire together strengthen their mutual connections → they fire together more → runaway positive feedback → seizure dynamics (all nodes saturate at +1). The basin of attraction collapses into a single degenerate fixed point.

Inhibitory neurons prevent this. When excitatory activity in a local region exceeds a threshold, inhibitory nodes fire and suppress the excitatory population. This is negative feedback: high activity → inhibition → reduced activity → inhibition released → activity recovers. The system oscillates around a stable operating point rather than spiraling to saturation.

This is exactly the **Excitatory/Inhibitory (E/I) balance** at $K=4$ from Module B constraints:
- Too much excitation → dynamics collapse into a single attractor (ordered, computationally dead).
- Too much inhibition → no attractors form (chaotic, no useful computation).
- Balanced E/I → complex, maximally informative dynamics at the edge of chaos.

**The architectural implication: fixed reservoir, plastic readout.**
Module B specifies that the recurrent core (the "liquid") is fixed and randomly connected. This is not just a vanishing-gradient workaround — it is a stability guarantee. If the recurrent connections were Hebbian-plastic, the E/I balance would be a moving target: learning would continuously shift the balance, potentially crossing the critical boundary into order or chaos. By fixing the recurrent topology and restricting plasticity to the readout layer, the E/I balance is structurally locked. Learning cannot destabilize the reservoir dynamics.

The inhibitory population acts as the hardware's immune system against Hebbian runaway. Design the lattice with a tuned ratio of excitatory to inhibitory LUTs (studies suggest ~80/20 E/I split in biological cortical circuits). The inhibitory truth tables are fixed — they implement hard suppression rules, not learned ones. Only the excitatory readout weights are plastic via latent-accumulation Hebbian updates.

### 5. Training Methods for Binarized Systems (2026-04-04)

Standard backpropagation fails on binary weights (non-differentiable). Two paradigms:

**Approach A: Gradient Approximation (STE)**
Straight-Through Estimator executes forward passes in strict binary but pretends the activation was smooth during backward pass. **Catch:** requires maintaining high-precision shadow weights during training, usually needing a GPU. Only binarized for deployment.

**Approach B: Gradient-Free Learning (Pure Binary)**
Severs dependency on backpropagation entirely. No floating-point shadow weights:
- **Evolutionary / Genetic Algorithms:** heterogeneous binary grid as digital DNA
- **Evolution Strategies (ES):** estimates pseudo-gradient by randomly perturbing binary weights
- **Gradient-Free RL / Q-Learning:** grid as agent interacting with environment
- **Hebbian / STDP:** biologically inspired, purely local (see Theme 4 corollary)

**Implication for CLADE:** The BSCA lattice uses STDP (Approach B). But the initial rule set (LUT truth tables, attractor basin topology) could be seeded via STE on a GPU (Approach A), then handed off to on-device STDP for continuous adaptation. "Compiled by GPU, refined by silicon."

### 6. FPGA Modification Levels (2026-04-04)

Three levels of hardware modification for continuous learning and context-switching:

| Level | Method | Speed | Use Case |
|-------|--------|-------|----------|
| **System 1** | Full reconfiguration ("firmware flash") | ms–sec, stops system | Initial deployment |
| **System 2** | Dynamic Partial Reconfiguration (DPR / "hot swap") | μs–ms, rest of chip runs | Macro context-switching (swap vision→audio agent) |
| **System 3** | LUTRAM / BRAM weight updates | 1 clock cycle (ns) | Micro-level STDP learning |

**The CLADE Synthesis:** Combine System 2 and System 3. LUTRAM for real-time STDP within an active agent. DPR to hot-swap entire agent blocks as the OS demands. This is the hardware equivalent of Wasm module instantiation — fast load, fast swap, fast adaptation.

### 7. Implementation Starting Points (2026-04-04)

**Software Simulation (PyTorch/snnTorch):**
2D CA block as neuromorphic processor. Leaky Integrate-and-Fire (LIF) neurons in heterogeneous grid. No STE — pure STDP learning based on temporal firing difference between neighbors. Visualize sparse spiking activity over time.

**Hardware (Verilog/SpinalHDL):**
Parallel asynchronous SNN block for FPGA. LIF neuron using only addition (membrane potential accumulation) and comparator (firing threshold). Localized STDP circuit — no global clock-synchronized backprop. Tile into 2D grid with 1-bit spike wires between neighbors.

**Binary STDP Implementation:**
Two approaches for learning with 1-bit weights:
- **Hidden Accumulator (Trick A):** 3-4 bit counter per weight. STDP increments counter; 1-bit weight flips on overflow. Forward pass stays binary.
- **Stochastic STDP (Trick B):** No counter. Causal spikes trigger small random probability (~5%) to flip. Statistically mimics smooth STDP over millions of spikes. Minimum memory footprint.

## Open Questions

- What would a "memory load/store" instruction set look like for an LLM?
- Does smart indexing change the O(n²) attention cost structure?
- Can agents be embedded directly inside FPGAs or systolic array trays?
- If computation is spatial rather than temporal, what replaces "process" as the unit of agency?
- If an agent is a von Neumann machine, can it become systolic instead?
- Should agent abstractions expose latency and data movement as first-class concerns?

## Clarifying Notes

**WASI Host as Capability Harness (2026-04-01)**
The WASI host (Wasmtime) is not a neutral execution environment — it is the **capability grantor**. Agents receive only the capabilities (file handles, network sockets, memory regions, device interfaces) that the host explicitly passes to them at instantiation. No ambient authority. An agent cannot escalate beyond what the host provided. This makes the host the security boundary and the agent a constrained guest. This principle applies uniformly regardless of what the agent computes or where its logic executes (lattice, CPU, FPGA). The capability token set $\mathcal{C}_k(t)$ in the CRDT state model (idea-frontier-v1.md §3.2) is the agent's live view of what the host has granted — it is not a wishlist, it is a receipt.

## Tensions and Tradeoffs

-

## Sources

- Bjarke Hammersholt Roune, "Designing AI Chip Hardware and Software" (2026) - Former Google TPU/Nvidia GPU engineer. Key insight: use indexed attention + SSD instead of massive HBM for KV cache.
- Systolic Array / Energy-Efficient ML research - Systolic arrays reduce von Neumann bottleneck via spatial dataflow; agents as von Neumann machines might benefit from similar architectural thinking.
- **cax** (caxing/jax) - JAX-based framework for gradient-descent optimization of cellular automata. Relevant as a potential training tool for the LUT truth tables in the NUCA lattice (Theme 2). Raises an open tension: if CA rules are differentiable via straight-through or relaxed surrogates, the "fixed reservoir" constraint (Theme 4, Hebbian corollary) may be overly conservative — gradient-optimized CA rules could be both stable AND expressive. Needs evaluation against the E/I balance stability guarantee before committing.

## Next Steps

-
