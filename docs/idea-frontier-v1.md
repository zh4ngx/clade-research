# CLADE Idea Frontier v1

> Active exploration areas and conceptual pipelines.
> Pre-formal. Nothing here is committed to architecture.md.
> All explorations bounded by the physics and routing constraints in research-constraints-v1.md.

---

## 1. The SLM Semantic Gateway: [English → Binary Vector] Pipeline

### 1.1 Problem Statement

The local cognitive node runs an 8B–14B quantized SLM (e.g., Qwen 2.5 Coder 14B in Q4_K_M). Its output vocabulary is continuous—dense floating-point embeddings in a high-dimensional latent space. The execution target is a discrete 1-bit lattice: an FPGA fabric of LUT6 cells executing XNOR-popcount operations at sustained arithmetic intensities of 400–600 Ops/Byte. The gateway must bridge these two regimes without violating the sub-25W thermal envelope or the CTC ratio constraint (250 ops per byte of off-chip bandwidth to prevent logic starvation).

The core question: **How do you lossily compress a natural language command into a deterministic binary vector that the hardware lattice can evaluate in a single rhythmic pass, while preserving semantic fidelity?**

### 1.2 Dataset Architecture: The [English → Binary Vector] Corpus

The training corpus is a supervised mapping $\mathcal{D} = \{(u_i, v_i)\}_{i=1}^{N}$ where:

- $u_i \in \Sigma^*$ is an English utterance (the natural language command, e.g., "Sync my tablet").
- $v_i \in \{0, 1\}^d$ is the target binary vector of dimensionality $d$, where each bit corresponds to a discrete hardware action, capability flag, or routing decision in the lattice.

**Target vector structure** (conceptual, not fixed):

```
v_i = [ op_code | capability_mask | routing_header | hazard_flags | pad ]
        ───────   ───────────────   ───────────────   ────────────   ────
        8 bits    16 bits           32 bits           8 bits         variable
```

- **op_code**: The discrete operation class (sync, spawn, migrate, evaluate, yield). Directly maps to the top-level branching in the systolic dataflow.
- **capability_mask**: Bitfield of required capabilities (network access, file I/O, GPU inference, CRDT merge). Maps to the capability-based security model.
- **routing_header**: Target node(s) or mesh topology decisions. Encodes Tailscale identity or broadcast semantics.
- **hazard_flags**: Pre-computed hazard mitigations (metastability risk, race condition probability, thermal throttle anticipation).
- **pad**: Reserved bits for lattice-specific alignment (must be a multiple of 64 for efficient LUT6 packing).

**Corpus generation pipeline (conceptual):**

1. **Schema-driven synthesis**: Define a formal grammar of valid operations. For each operation, enumerate the valid capability combinations, routing targets, and hazard profiles. This yields a combinatorial space of valid target vectors.
2. **Natural language templating**: For each valid $v_i$, generate a distribution of English utterances—varying syntax, ambiguity, and context depth. Include adversarial examples where the utterance is ambiguous between multiple target vectors.
3. **Negative samples**: Include utterances that map to $\mathbf{0}^d$ (null operation) or to intentionally invalid vectors, training the gateway to reject rather than misroute.

The corpus is not a static file. It is a **generative schema** that expands as the lattice's capability surface grows.

### 1.3 The Semantic Projection Layer: Continuous → Discrete

The SLM produces a dense embedding $h \in \mathbb{R}^m$ from its final hidden layer. The projection layer transforms $h \mapsto \hat{v} \in [0, 1]^d$ then samples $v \in \{0, 1\}^d$. This is a three-stage process:

**Stage 1: Variance-Scaled Linear Projection**

$$z = W_{proj} \cdot h, \quad W_{proj} \in \mathbb{R}^{d \times m}, \quad \text{scaled by } \sqrt{2/m}$$

This prevents vanishing updates when the output enters the XNOR-popcount chain. The scaling factor $\sqrt{2/m}$ is the closed-form variance-preserving initializer from Module B constraints—it ensures that the expected magnitude of $z$ remains stable regardless of embedding dimension $m$.

**Stage 2: Independent Bernoulli Sampling**

$$\hat{v}_j = \sigma(z_j), \quad v_j \sim \text{Bernoulli}(\hat{v}_j), \quad j = 1, \ldots, d$$

Each bit is sampled independently from the sigmoid-activated projection. This is the **probabilistic constraint mapping** from the Module C constraints: the continuous embedding is translated into exact probabilities of a binary spike occurring, and the discrete hardware target state is stochastically sampled from this distribution.

Critical implication: the same utterance can produce different target vectors across invocations. This is not noise—it is **semantic uncertainty propagation**. The lattice must be robust to slight variations in its input vector, which enforces a basin-of-attraction structure in the hardware state space (cf. Module B: NUCA→DCG basin categorization).

**Stage 3: Euclidean Distance Embedding (Fallback / Verification)**

For high-stakes operations where stochastic sampling is unacceptable (e.g., capability escalation, migration decisions), the system falls back to a deterministic path:

$$v^* = \arg\min_{c \in \mathcal{C}} \|h - W_{Embed} \cdot c\|_2^2$$

where $\mathcal{C}$ is the set of canonical binary codebook vectors and $W_{Embed}$ is the learned embedding matrix. This finds the nearest valid target by minimum squared Euclidean distance—a **nearest-neighbor lookup in the joint continuous-discrete space**.

### 1.4 Training Dynamics Under Hardware Constraints

The SLM is frozen after initial quantization. The projection layer is the only trainable component. Training uses:

- **Straight-Through Estimator (STE)**: Gradients flow through the Bernoulli sampling via identity override during backprop (since $\nabla v_j / \nabla \hat{v}_j$ is undefined for discrete samples).
- **CMA-ES with Margin (LB+IC)**: For the binary weight components of $W_{proj}$, evolutionary search with diagonal mutation matrices prevents premature stagnation in the discrete plateau landscape.
- **Thermal-aware batching**: Training batch size is dynamically constrained by $N_{cell-max}$, the maximum number of concurrently active LUTs permitted by the thermal envelope. If the chip approaches $T_{chip} = 50°C$, the training scheduler reduces batch size to lower concurrent switching activity.

### 1.5 Open Questions

- **Dimensionality of $d$**: Too small and the lattice can't express the operation space. Too large and we violate the CTC ratio (not enough bandwidth to feed the lattice). What is the critical $d$ where semantic compression loss becomes unacceptable?
- **Multi-utterance aggregation**: "Sync my tablet" may require two lattice evaluations (route to tablet, then execute sync protocol). Does the gateway emit a sequence of binary vectors, or a single composite vector that the lattice unfolds temporally via recurrence?
- **Adversarial compression**: Can a maliciously crafted utterance produce a target vector that, while valid, routes to an unintended capability? This is a binary-space adversarial attack surface that requires analysis.

---

## 2. The "v0" Curriculum Ladder: From Opcodes to Asynchronous Hazards

### 2.1 Design Philosophy

The curriculum ladder is the bootstrapping trajectory for a local-first agent that must eventually operate on asynchronous, heterogeneous hardware with non-deterministic timing. It follows the tiered abstraction bootstrapping from Module C constraints, but adds a crucial constraint: **the agent must never leave the Wasm sandbox at any tier**. All learning occurs within the capability boundary.

The environment at each tier is a progressively richer Wasm execution context. The agent does not "graduate" to a new language—it graduates to a new *interface surface* exposed via WIT (Wasm Interface Types).

### 2.2 Tier 0: Pre-Curriculum — Sandbox Calibration

**Environment:** Bare Wasmtime instance. No imports. No memory. Just the stack.

**Objective:** The agent demonstrates that it can generate syntactically valid Wasm bytecodes that parse, validate, and execute without trapping.

**Milestones:**
- Generate a valid module header (magic number + version).
- Emit a single `i32.add` instruction inside a function body.
- Construct a valid `start` function that returns a scalar.
- Pass the Wasm validation pass (type checking, control flow stack consistency).

**Reward signal:** Binary pass/fail on validation + execution. No semantic reward yet.

**Skill tracked by BKT:** $P(\text{wasm\_syntax} | \text{evidence})$

### 2.3 Tier 1: Deterministic Stack Machines

**Environment:** Wasmtime instance with linear memory (1 page = 64KB). Imports: `env.log`, `env.assert_eq`.

**Objective:** The agent generates bytecodes that perform correct deterministic computation: arithmetic, control flow, memory load/store.

**Milestones:**
- Implement `i32.factorial(n)` using `loop`/`br_if` control flow.
- Implement memory operations: write an array, read it back, compute a checksum.
- Implement `i32.sort` (bubble sort or similar) operating on linear memory.
- Pass deterministic property-based tests: $\forall n \in [0, 20], \text{factorial}(n) = n!$.

**Reward signal:** Property-based test pass rate. The environment generates random inputs and checks outputs against a reference implementation.

**Why this matters:** The agent learns that computation has *consequences*—its bytecodes produce observable state changes in linear memory. This is the grounding layer.

**Skill tracked by BKT:** $P(\text{det\_computation} | \text{evidence})$

### 2.4 Tier 2: Concurrent Composition (The HLS Bridge, Reimagined)

**Environment:** Wasmtime instance with multiple linear memories (via Wasm Component Model). Imports: `env.spawn`, `env.send`, `env.recv`.

**Objective:** The agent generates bytecodes that coordinate multiple concurrent computations. This replaces the HLS tier from the Module C constraints—rather than bridging to C/C++ HLS pragmas, the agent learns parallelism through Wasm-native concurrent composition.

**Milestones:**
- Spawn a child component that computes independently and returns a result.
- Implement a pipeline: component A produces a stream, component B consumes it.
- Resolve a data race: two components write to overlapping memory. Agent must insert synchronization (a simple mutex or channel protocol).
- Implement map-reduce: spawn N workers, distribute work, aggregate results.

**Reward signal:** Correctness under concurrency. Tests inject scheduling nondeterminism (random yields, delayed spawns). The agent must produce correct results regardless of interleaving.

**Why this matters:** This is where the agent first encounters nondeterminism—not in hardware timing, but in scheduling order. The environment forces the agent to discover that *the order of events matters*.

**Skill tracked by BKT:** $P(\text{concurrency} | \text{evidence})$

### 2.5 Tier 3: Signal-Aware Feedback (The Verilog Tier, Reimagined)

**Environment:** Wasmtime instance with a simulated hardware interface. Imports: `env.read_signal`, `env.write_signal`, `env.clock_tick`, `env.assert_timing`.

**Objective:** The agent generates bytecodes that interact with a cycle-accurate hardware simulation. The simulation models the discrete lattice: bit-packed SRAMs, XNOR-popcount processing elements, and the distributed buffer model.

**Milestones:**
- Read a bit-packed weight stream from simulated SRAM and route it to the correct PE.
- Implement the XNOR-popcount reduction for a single PE column.
- Detect and handle a simulated thermal event: when `env.read_signal("temp")` exceeds threshold, reduce clock frequency.
- Sustain a streaming inference for $N$ cycles without logic starvation (maintain the 400–600 Ops/Byte arithmetic intensity).

**Reward signal:** AST-derived reward from signal traces. This is the Signal-Aware Learning approach from Module C: instead of binary pass/fail, the environment extracts Abstract Syntax Tree rewards from functionally correct individual signal transitions. A partially correct dataflow still earns partial credit.

**Why this matters:** The agent learns that computation has *physical consequences*—timing, power, heat. The sandbox is no longer purely logical; it models the physics of the substrate.

**Skill tracked by BKT:** $P(\text{signal\_awareness} | \text{evidence})$

### 2.6 Tier 4: Asynchronous Hazard Mitigation

**Environment:** Full Wasm Component Model runtime with async host functions. Imports: `env.async_eval`, `env.yield`, `env.backpressure`, `env.migrate`.

**Objective:** The agent operates in a fully asynchronous environment where operations return futures, the host scheduler may preempt at any `yield` point, and hardware responses arrive nondeterministically.

**Milestones:**
- **Glitch prevention:** Generate logic that produces stable outputs despite input transition overlap. The agent must insert redundancy to prevent transient false outputs.
- **Metastability resolution:** Handle a simulated synchronizer flop scenario where an async input arrives during a clock edge. The agent must generate a two-flop synchronizer chain and wait for resolution.
- **Gray-code state transitions:** Implement a multi-bit state counter that transitions across the async boundary without race conditions. All multi-bit state changes use Gray encoding.
- **Epoch-based yielding under load:** During a heavy CRDT merge simulation, the agent must yield control back to the host executor within a bounded epoch, preventing local-first responsiveness violations.
- **Cross-substrate migration:** The agent serializes its execution state, the host transfers it to a simulated different architecture (ARM → x86), and the agent resumes correctly on the new substrate.

**Reward signal:** Multi-objective: correctness × latency × energy. The environment measures not just whether the computation succeeded, but how many joules it consumed and whether it violated any real-time deadline.

**Skill tracked by BKT:** $P(\text{async\_hazards} | \text{evidence})$

### 2.7 Curriculum Orchestration

The tier progression is managed by the interplay of two systems:

1. **Evolutionary Selector**: Estimates tier difficulty via inverse relative fitness. If the agent's fitness on Tier $k$ plateaus (fitness gain < $\epsilon$ over $G$ generations), the selector promotes to Tier $k+1$. If fitness drops below a floor on Tier $k+1$, it demotes back to Tier $k$ with increased difficulty scaling.

2. **Bayesian Knowledge Tracing (BKT)**: Maintains a latent skill model $\theta = (P(L_0), P(T), P(G), P(S))$ for each tier's tracked skill. The agent advances when $P(\text{skill} | \text{evidence}) > 0.95$ with high confidence (low posterior variance).

**Key constraint:** The agent never sees the tier boundaries. From its perspective, the environment simply becomes richer and less predictable over time. The curriculum is *implicit*.

### 2.8 Open Questions

- **Tier 2 parallelism model:** Should the agent learn shared-memory parallelism (threads) or message-passing (actors)? The former is more natural for Wasm linear memory; the latter is more aligned with the swarm architecture.
- **Simulated hardware fidelity at Tier 3:** How detailed must the thermal/physical simulation be? Full LUT-level thermal modeling is computationally expensive. Can we use a learned surrogate model for thermal behavior?
- **Tier 4 failure modes:** What happens when the agent generates bytecodes that deadlock the async executor? The host must have a watchdog timer—but what should the reward penalty be for a deadlock vs. a slow-but-correct solution?
- **Curriculum transfer:** If an agent masters Tier 4 on one hardware profile (e.g., FPGA-heavy), does it need to re-climb the ladder for a different profile (e.g., GPU-heavy), or does the Wasm abstraction provide sufficient transfer?

---

## 3. CRDT State Topology: Local-First Swarm Synchronization

### 3.1 Problem Statement

The CLADE swarm consists of $K$ physically distributed nodes, each running Wasmtime with local cognitive engines (8B–14B SLMs). Agents are Wasm modules that may migrate across nodes. Each agent carries mutable state. The constraint: **no central coordinator, no synchronous consensus, and the WAN mesh (Tailscale) introduces non-trivial, variable latency.**

The mathematical requirement from Module D: Delta-state CRDTs ensure convergence without coordination. But the question is not just "which CRDT?"—it is "what does a delta actually represent when the state being synchronized is a Wasm agent's execution context across heterogeneous hardware?"

### 3.2 The State Model: What Actually Travels

An agent's state at any node $n_k$ at logical time $t$ is a tuple:

$$S_k(t) = (\mathcal{M}_k(t), \mathcal{C}_k(t), \mathcal{W}_k(t), \mathcal{P}_k(t))$$

Where:
- $\mathcal{M}_k(t)$: Linear memory snapshot (the Wasm module's address space).
- $\mathcal{C}_k(t)$: Capability token set (which permissions the agent currently holds).
- $\mathcal{W}_k(t)$: Waiting futures (pending async operations the agent issued to the host).
- $\mathcal{P}_k(t)$: Projection layer parameters (if the agent is an SLM gateway instance, this is the trainable $W_{proj}$; for other agents, this is null).

A **delta** $\delta_{k \to j}$ from node $k$ to node $j$ is not the full state. It is the set of state changes since the last acknowledged sync:

$$\delta_{k \to j}(t) = S_k(t) \ominus S_k(t_{last})$$

where $\ominus$ is the CRDT-specific difference operator and $t_{last}$ is the logical timestamp of the last delta that node $j$ acknowledged from $k$.

### 3.3 Delta Structure: The Asynchronous Binary Payload

When a Wasm agent on an ARM tablet updates its state and needs to sync with the x86 desktop, the delta payload is a bit-packed binary blob structured as:

```
┌──────────────────────────────────────────────────────┐
│ Header (fixed, 64 bytes)                             │
│ ├─ agent_id:      [u8; 32]   SHA-256 of agent Wasm  │
│ ├─ origin_node:   [u8; 16]   Tailscale IPv6         │
│ ├─ vector_clock:  [u64; K]   per-node logical time  │
│ ├─ delta_type:    u8         {MEM, CAP, FUT, PROJ}  │
│ ├─ compression:   u8         {RAW, XNOR_PACKED, LZ4}│
│ └─ checksum:      u32        CRC32 of payload       │
├──────────────────────────────────────────────────────┤
│ Payload (variable length)                            │
│ ├─ For MEM:    bit-packed memory page deltas         │
│ ├─ For CAP:    capability add/remove delta-set       │
│ ├─ For FUT:    future resolution vectors             │
│ └─ For PROJ:   binarized weight update deltas        │
├──────────────────────────────────────────────────────┤
│ Merkle Proof (variable length)                       │
│ └─ Sparse Merkle Tree inclusion proof for the delta  │
│    against the agent's full state root hash          │
└──────────────────────────────────────────────────────┘
```

**Key design decisions:**

- **Delta type discrimination:** A single sync operation may carry multiple delta types in one payload. The header's `delta_type` is a bitfield, not an enum.
- **XNOR_PACKED compression:** For PROJ deltas (weight updates to the projection layer), the binary weight deltas are already bit-packed. They can be XNOR-compressed against a reference vector (the previous weights) for near-zero overhead compression—leveraging the same XNOR-popcount hardware that the lattice uses for inference.
- **Merkle inclusion proof:** Each delta carries a sparse Merkle tree proof against the agent's full state root. This allows the receiver to verify the delta's integrity without holding the full state—a critical property when a node receives deltas for agents it doesn't currently host.

### 3.4 The Merge Semantics: CRDTs Across Wasmtime Boundaries

The merge operation at node $j$ when receiving $\delta_{k \to j}$ must handle four sub-merges, one per state component:

**$\mathcal{M}$ (Linear Memory): Last-Writer-Wins Register per page.**

Linear memory is divided into 64KB pages (matching Wasm's natural page size). Each page has a version vector. On merge, the page with the later vector clock wins. This is a standard LWW-Register CRDT, but the granularity is page-level, not byte-level—because Wasm memory access patterns tend to be page-clustered (stack, heap, data segments are naturally page-aligned).

**$\mathcal{C}$ (Capabilities): Observed-Remove Set.**

Capabilities are additive-only in the normal case (agents acquire capabilities; they are not revoked). But revocation must be supported for security. The OR-Set semantics ensure that if node $k$ adds capability $c$ and node $j$ concurrently removes it, the add wins (because the removal can only remove capabilities it has observed). This matches the capability-based security model: capability grants are durable; revocations require global observation.

**$\mathcal{W}$ (Waiting Futures): G-Counter per future ID.**

Pending futures are tracked with a grow-only counter per future identifier. When a future resolves at node $k$, the resolution is a delta that increments the counter. If two nodes concurrently resolve the same future (impossible in correct operation, but possible under Byzantine conditions), the G-Counter merge takes the maximum—the later resolution wins. The host executor is responsible for delivering the resolution to the agent's `await` point.

**$\mathcal{P}$ (Projection Parameters): Delta-State Counter-RO-CKS.**

The projection layer weights are the most sensitive component. They are binarized ($W_{proj} \in \{0, 1\}^{d \times m}$), so updates are bit flips. The CRDT for this is a per-bit counter: each bit position has an increment/decrement counter. Merge takes the maximum counter value per bit. This is effectively a **Delta-State Counter-CRDT** operating at bit granularity. The XNOR_PACKED compression directly encodes these bit-level counters—zero bandwidth overhead.

### 3.5 The Cross-Substrate Problem: ARM ↔ x86

The Wasm abstraction guarantees that the agent's *logic* is hardware-agnostic. But the CRDT merge operation itself executes on the host (Wasmtime), which is substrate-specific. Module D constraints require:

1. **Hardware-agnostic acceleration:** The host performs runtime feature detection (x86: AVX-512 VNNI; ARM: SVE2 dot products) and dispatches to the optimal SIMD path for the merge-phase computation. The agentic payload (the delta) is unaware of this.

2. **Epoch-based yielding:** The merge is not atomic. On a high-power x86 desktop merging a large PROJ delta, the host processes the merge in epochs (e.g., 1024 bit-flips per epoch), yielding back to the async executor between epochs. This ensures that a low-power ARM tablet sharing the mesh isn't blocked waiting for the desktop's merge to complete.

3. **Endianness and alignment:** Wasm is little-endian. ARM can be bi-endian but is typically little-endian in practice. The delta payload is explicitly little-endian. Alignment within the payload is 8-byte (matching the LUT6 packing granularity). No conversion needed on the critical path.

### 3.6 State Topology: The "Idea of State"

The conceptual model: **state is not a snapshot, it is a wavefront.**

At any moment, the global state of an agent is not a single value—it is a distribution across the $K$ nodes, each holding a partially converged view. The "true" state is the fixed point of the CRDT merge semilattice: $\text{merge}(S_1, S_2, \ldots, S_K)$. This fixed point is mathematically guaranteed to exist and be unique (by the CRDT convergence proof). But it is never *observed* in full at any single node.

Instead, each node holds a **locally consistent projection** of the global state. The delta propagation is the mechanism by which these local projections converge. The rate of convergence is bounded by the WAN RTT across the Tailscale mesh.

The "Idea of State" moves across Wasmtime host boundaries not as a copy, but as a **delta wavefront**—a set of compressed binary differences propagating through the mesh, each delta carrying its own Merkle proof and vector clock. No node ever holds the full truth. Every node holds a consistent approximation that improves monotonically over time.

This has a direct analogy to the Module B temporal topology: just as the discrete lattice uses temporal unrolling (recurrent clock cycles) to replace spatial expansion, the CRDT swarm uses **temporal convergence (delta propagation over time)** to replace spatial consensus (synchronous agreement across all nodes). Time substitutes for coordination.

### 3.7 Open Questions

- **Conflict hotspots:** If two nodes concurrently modify the same linear memory page (e.g., two agents sharing a memory region), the LWW-Register merge will discard one writer's changes. How does the agent detect and recover from this? Is there a CRDT-aware memory allocator that prevents concurrent writes to the same page?
- **Delta ordering and causality:** The Tailscale mesh provides reliable, ordered delivery between any pair of nodes. But deltas from node $k$ to node $j$ may arrive out of order relative to deltas from node $l$ to node $j$. The vector clock in the delta header resolves this—but what is the memory cost of buffering out-of-order deltas pending merge?
- **Projection weight divergence:** If the PROJ deltas converge slowly (high WAN latency, low sync frequency), different nodes may have slightly different projection layer weights. This means the same utterance produces different target vectors on different nodes. Is this tolerable (the lattice handles semantic uncertainty via basin-of-attraction dynamics), or does it require a consensus protocol for weight updates?
- **Garbage collection of tombstones:** OR-Sets (for capabilities) and Counters (for projection weights) accumulate metadata over time. When can tombstones be garbage-collected? This requires a distributed GC protocol—which re-introduces coordination. Is there a coordination-free GC strategy?

---

## Appendix: Constraint Cross-Reference

| Frontier Area | Module A (Hardware) | Module B (Temporal) | Module C (Curriculum) | Module D (Routing) |
|---|---|---|---|---|
| SLM Semantic Gateway | LUT6 packing, CTC ratio, thermal batching | Variance scaling, basin of attraction | Semantic Projection Layer, CMA-ES, STE | — |
| v0 Curriculum Ladder | — | Temporal unrolling, async clock cycles | Tiered bootstrapping, BKT, AST rewards | Zero-copy boundaries, backpressure |
| CRDT State Topology | SIMD acceleration, BRAM adjacency | Temporal convergence ≈ spatial consensus | — | Delta-state CRDTs, epoch yielding, zero-copy |

---

*Document status: Active exploration. Next step: pressure-test each area against the Open Questions before promoting any decisions to architecture.md.*
