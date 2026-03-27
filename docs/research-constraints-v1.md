# CLADE Engineering Constraints v1

## Module A: Hardware & Physical Compute

*   **The physical and memory bandwidth limitations of FPGA LUTs for packed 64-bit XNOR operations:**
    *   *Physical Routing Congestion:* Modern 6-input LUTs (LUT6) act as 64-bit SRAM truth tables and are highly efficient for mapping multi-input parity functions, reducing a 16-input XNOR from 5 LUTs (in older LUT4 architectures) to just 3 LUTs. However, LUT6 structures require 50% more input pins per node, which dramatically increases routing fabric congestion and places severe strain on place-and-route tools.
    *   *Memory Bandwidth Bottlenecks:* While 1-bit quantization reduces parameter memory footprints by 32x, it dramatically shifts the bottleneck to the computation-to-communication (CTC) ratio. Sustaining peak throughput requires feeding the XNOR lattice extremely fast; modern high-throughput logic can require up to 250 operations per byte of off-chip bandwidth to prevent logic starvation. Without highly distributed, parallel on-chip Block RAMs (BRAM), the accelerator will become strictly memory-bound during high-speed streaming.

*   **How data flow in these discrete lattices differs from standard FP16 tensor cores:**
    *   *Distributed Buffers vs. Centralized Registers:* Standard FP16 Tensor Cores rely on an instruction-scheduled (SIMT) "dot-product" execution style that is fundamentally bottlenecked by centralized register file (RF) bandwidth and hierarchical data movement (Global Memory $\rightarrow$ L2 $\rightarrow$ Shared Memory). In contrast, our BNN systolic array must use a "distributed buffer model" where bit-packed SRAMs (weight streamers) are physically adjacent to the processing element (PE) grid.
    *   *Arithmetic Intensity and Routing:* Data movement in the XNOR lattice is rigid, rhythmic, and exclusively neighbor-to-neighbor. This streaming dataflow pipeline maximizes local reuse, yielding an arithmetic intensity of 400–600 operations per byte—an order of magnitude higher than the 15–20 Ops/Byte seen in standard GPUs.

*   **The specific thermal or joule-per-inference constraints we must account for when running these logic gates at maximum clock speed:**
    *   *Thermal Throttling Penalties:* Running millions of bitwise operations at maximum clock speeds concentrates heat generation. If the chip exceeds thermal limits (e.g., $T_{chip} \le 50^\circ$C), hardware throttling initiates a sharp, discontinuous drop in throughput, destroying deterministic real-time latency guarantees. We must embed thermal telemetry directly into our deployment algorithms to restrict the maximum allowable power consumption ($P_{max}$), which establishes a hard upper limit on the number of concurrently active LUTs ($N_{cell-max}$).
    *   *Joule-per-Inference Targets:* Because FPGA dynamic power consumption scales linearly with active logic and memory switching, we must target extreme energy efficiency metrics. Highly optimized custom BNN implementations currently reach down to 21.6 femtojoules (fJ) per operation and can achieve over 200 TOPS/W. To stay within a strict sub-25W thermal envelope while processing millions of inferences per second, we must aggressively fold normalization into threshold logic or utilize dynamic voltage and clock scaling—such as slowing clocks to 250 kHz to pull chip power down to 0.2W for low-duty cycle edge tasks.

## Module B: Temporal Topology & Time

*   **The structural requirements for implementing Recurrent architectures (like Liquid State Machines or Leaky Integrate-and-Fire) in a strict 1-bit environment:**
    *   **Discrete Event-Driven Nodes:** Neurons must be engineered as **Leaky Integrate-and-Fire (LIF) units** that continuously accumulate potential and emit discrete **1-bit spikes** upon breaching a strict threshold, directly aligning with binary/ternary quantization models.
    *   **Fixed Recurrent Reservoirs:** To avoid the catastrophic vanishing and exploding gradient problems inherent in 1-bit networks, the recurrent core (the "liquid") must remain **fixed and randomly connected**, bypassing the biological implausibility and noise of continuous multi-layer weight tuning.
    *   **Bitwise Operation Hardware:** The physical design must replace high-precision floating-point Multiply-Accumulate (MAC) units with **bitwise logic operators (e.g., XNOR and popcount)** processed within Compute-In-Memory (CIM) or neuromorphic substrates.
    *   **Excitatory-Inhibitory (E/I) Balance:** The architecture demands a strictly tuned **E/I synaptic balance** paired with moderate connectivity (e.g., $K=4$) to sustain fading memory near the "edge of chaos," maximizing the computational power of the discrete binary state.
    *   **Sparse Input Encoding:** Continuous real-world signals must pass through **sparse encoding mechanisms** to be mapped into the spatio-temporal spike trains required to drive the 1-bit reservoir successfully.

*   **The mathematical isomorphic mapping between non-uniform Cellular Automata and Directed Cyclic Graphs:**
    *   **Functional Threshold Equivalence:** There is a formal, **mathematical equivalence between the local transition functions (rule tables) of non-uniform cellular automata (NUCA) and the threshold logic gates of binarized neural networks (BNNs)**, allowing CA models to be mapped directly to hardware logic. 
    *   **Topological Fluidity:** Moving beyond the rigid, geometric grids of classical cellular automata, mapping NUCAs onto **Directed Cyclic Graphs (DCGs) accommodates arbitrary network connectivity**, including self-loops and complex, heterogeneous feedback cycles. 
    *   **Basin of Attraction Categorization:** This topological freedom allows the state-space of the graph to be partitioned into **complex basins of attraction**, enabling the discrete network to function as an advanced decision-making system situated optimally between order and chaos.
    *   **Dynamic Morphogenetic Memory:** The mapping supports **Self-Organizing Dynamic Graph** structures where the physical substrate remains static, but the functional topology reorganizes itself on the fly via attention-modulated edge formation.

*   **How the network physically utilizes asynchronous clock cycles over time to replace spatial reasoning (e.g., 'thinking tokens'):**
    *   **Temporal Unrolling vs. Spatial Expansion:** Instead of exhausting a finite context window by verbalizing explicit "thinking tokens" across high-precision spatial layers, the network employs **depth-recurrence in a latent space**, applying a shared-weight block iteratively over successive clock cycles.
    *   **Dynamic Computational Depth:** Clock cycles substitute for tokens by providing **variable-depth processing**; the network can dynamically allocate more recurrent cycles (time) to resolve harder, non-Markovian problems without generating data transfer overhead or increasing parameter count.
    *   **Epistemic Updates for Information Sufficiency:** In 1-bit systems where information stagnates within a single forward pass, each recurrent clock cycle functions as an **"epistemic update."** This iterative refinement continuously reduces uncertainty and tracks long-term dependencies within the network's compact hidden state.
    *   **Power Conservation via Asynchrony:** By utilizing an **asynchronous, event-driven medium** where neurons fire only when required by threshold breaches, the energy cost of a recurrent clock cycle is reduced to practically zero compared to the massive power consumed by spatial token generation in von Neumann architectures.

## Module C: Curriculum Learning & Local Rules

* **Mathematical formulas and approaches for local learning in binary spaces (STDP & Evolutionary):**
    * *Stabilized Supervised STDP (S2-STDP):* Uses a temporal error modulated rule: $\Delta w_{i,j} = e_j \times A_{\pm} \times \exp(-\beta \frac{w_{ij} - w_{min}}{w_{max} - w_{min}})$, where $e_j$ is the temporal error between actual and desired firing times. Paired Competing Neurons (PCN) add lateral inhibition to stabilize spikes locally without global gradients.
    * *STDP-inspired Temporal Local Learning Rule (S-TLLR):* A highly scalable three-factor rule utilizing causal/non-causal relationships modulated by a top-down signal. Achieves $\mathcal{O}(n)$ memory complexity, bypassing the massive memory limits of standard BPTT.
    * *Boolean Variation (BOLD Framework):* Replaces floating-point latent weights with a discrete chain rule operating solely on logic gates (e.g., flipping binary states via NOT operations when XNOR conditions are met).
    * *CMA-ES with Margin (LB+IC):* Modifies canonical Covariance Matrix Adaptation Evolution Strategies by adding a diagonal matrix into the mutation distribution. This enforces a lower bound on marginal probabilities, preventing premature stagnation in discrete plateau spaces.
    * *Cellular-NEAT (PCELL):* Separates the search space into discrete interacting cells to parallelize structural exploration and evolve network topology, escaping local optima.

* **Architecture for a 'Semantic Projection Layer' (Bridging continuous SLMs to discrete hardware):**
    * *Probabilistic Constraint Mapping:* Translates continuous dense embeddings into exact probabilities of a binary spike occurring ($P(x=1) \equiv \tilde{x}$). The discrete hardware target state is then stochastically sampled directly from this distribution.
    * *Euclidean Distance Embedding Projection:* Projects continuous latent outputs against a learned binary/class embedding matrix ($W_{Embed}$). The dense representation maps to a discrete state by finding the minimum squared Euclidean distance.
    * *Variance-Scaled Boolean Interfaces:* Analytically scales incoming continuous signals by a closed-form factor (e.g., $\sqrt{2/m}$ for linear projections of size $m$) before processing through XNOR gates to prevent vanishing updates and maintain signal variance.

* **Concrete strategies for curriculum learning (Bootstrapping from opcodes to async hardware):**
    * *Tiered Abstraction Bootstrapping:* Initialize the curriculum by having the agent generate deterministic, stack-based Wasm bytecodes (e.g., `i32.add`) to safely ground it in core logic primitives within an isolated sandbox.
    * *High-Level Synthesis (HLS) Bridge:* Transition the agent to HLS (C/C++ to RTL) to learn parallelization and structural instantiation (e.g., applying `ARRAY_PARTITION` pragmas to resolve memory bottlenecks).
    * *Signal-Aware Process Feedback:* Move to concurrent hardware languages (Verilog). Since binary pass/fail testbenches are inadequate due to sparse rewards, implement Signal-Aware Learning to extract Abstract Syntax Tree (AST) rewards from functionally correct individual signal transitions.
    * *Asynchronous Hazard Mitigation:* Shift from synchronous operation to event-driven circuits. Task agents with resolving non-deterministic physical constraints using logic redundancy (glitch prevention), synchronizer flops (metastability), and Gray-code assignments (race conditions).
    * *Adaptive Environmental Sequencing:* Manage this trajectory using an Evolutionary Selector (estimating difficulty via inverse relative fitness) and Bayesian Knowledge Tracing (BKT) to track latent skills, ensuring an optimal difficulty gradient.

## Module D: Distributed OS Routing & Local-First State

* **Architectural constraints for routing raw binary payloads asynchronously across a component boundary:**
    * *Native Async Primitives:* The host environment must support non-blocking I/O at the ABI level (e.g., native streams and futures) to prevent the hypervisor from stalling while waiting for discrete lattice evaluations.
    * *Zero-Copy Memory Boundaries:* To maintain high-throughput bitbanging, the OS must route raw binary streams by directly mapping linear memory to kernel buffers, eliminating intermediate guest-side buffering or serialization overhead.
    * *Host-Driven Concurrency & Backpressure:* Payloads must progress via host-managed, nondeterministic scheduling. The system must natively implement backpressure (e.g., suspending component execution when exclusive locks are held) to prevent OOM conditions during massive asynchronous payload spikes.

* **Mathematical requirements for state synchronization across a heterogeneous, local-first cluster:**
    * *Delta-State Convergence:* The system must utilize Delta-state CRDTs (Conflict-free Replicated Data Types) to synchronize the swarm. By transmitting only recent state changes (deltas) rather than full state objects, we minimize bandwidth across constrained networks while ensuring all nodes mathematically converge without central coordination.
    * *Hardware-Agnostic Acceleration:* The host OS must be capable of runtime feature detection to execute architecture-specific SIMD instructions (for intensive CRDT merge phases) without the agentic payload needing to know the underlying hardware architecture.
    * *Epoch-Based Yielding:* To prevent a heavy state-merge on a high-power node from freezing a low-power edge device, the merge computation must utilize epoch interruption to yield control back to the host's async executor, ensuring local-first responsiveness.

* **Conceptual trade-offs between low-level compilation and formal verification:**
    * *Memory & Runtime Overhead:* Direct low-level compilation (relying on compile-time ownership) offers zero-cost abstractions with minimal baseline memory overhead. Formally verified extraction pipelines inevitably inject runtime overhead (e.g., managing an acyclic heap via reference counting), establishing a higher baseline memory footprint per agent.
    * *FFI Marshaling Latency:* While verified runtimes can use destructive updates to approximate imperative speeds, they suffer from marshaling latency when translating specialized mathematical representations across the Foreign Function Interface (FFI) to standard byte arrays.
    * *Assurance vs. Iteration Velocity:* Bootstrapping with direct compilation allows for rapid, iterative discovery of the emergent physics. Introducing formal verification too early imposes an extreme "proof tax," where minor structural evolution breaks rigid mathematical proofs. Verification should be applied to stabilize proven topologies, not to discover them.
