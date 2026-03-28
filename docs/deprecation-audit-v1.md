# Deprecation Audit: Legacy Docs vs. v1 Constraints

> Status: Active. Items listed here are concepts in `architecture.md` and `insights.md` that
> **contradict or are superseded by** the physics and mathematical constraints established in
> `research-constraints-v1.md`. Each item must be resolved before architecture.md is promoted
> past draft status.
>
> This is a tracking document, not a decree. Each item needs discussion before deprecation.

---

## CONTRADICTION 1: Vulkan backend assumption vs. discrete lattice compute model

**Location:** `architecture.md` → "Local Cognitive Nodes" → "Inference Stack"
**Legacy claim:** Run quantized 8B–14B models locally via **Vulkan** backend on consumer AMD GPUs.

**v1 constraint conflict:**
Module A establishes that the compute substrate is a **1-bit XNOR-popcount systolic array** with distributed SRAM buffers, not a GPU SIMT architecture. The arithmetic intensity target is 400–600 Ops/Byte via neighbor-to-neighbor streaming dataflow. Standard GPU tensor cores operate at 15–20 Ops/Byte with centralized register files and hierarchical memory movement — an entirely different physical regime.

The Vulkan backend is a pragmatic path for *running existing quantized models on consumer GPUs*. It is not the compute substrate for the discrete lattice. These are two different things, and architecture.md conflates them.

**Resolution needed:** Clarify whether CLADE has two tiers of compute (GPU for SLM inference, FPGA/lattice for agent evaluation) or one unified substrate. If two tiers, the Vulkan backend is not deprecated — it's correctly scoped to the SLM tier. If one tier, Vulkan is deprecated in favor of the lattice model.

**Deprecation action:** Pending discussion.

---

## CONTRADICTION 2: "LLM inference per node" vs. Binary Neural Network execution model

**Location:** `architecture.md` → "Local Cognitive Nodes"
**Legacy claim:** Each node runs a local LLM (Qwen 2.5 Coder 14B, Llama 3 8B) for micro-decisions.

**v1 constraint conflict:**
Module B establishes that computation must occur via **Leaky Integrate-and-Fire (LIF) units** with 1-bit spikes, fixed recurrent reservoirs, and XNOR-popcount accumulation — not floating-point matrix multiplications. Module A establishes sub-25W thermal envelopes and femtojoule-per-operation targets.

Running a 14B-parameter model (even at Q4 quantization) requires ~7-8GB VRAM and draws 50-150W during inference. This violates the thermal envelope. The v1 architecture calls for **1-bit BNNs** operating at 200+ TOPS/W, not FP16/Q4 transformers.

However: the SLM Semantic Gateway (idea-frontier-v1.md §1) explicitly uses a quantized SLM as the front-end that feeds the projection layer. The SLM is the *input parser*, not the *execution engine*. This may be a valid two-stage architecture (SLM parses English → projection layer binarizes → lattice executes), not a contradiction.

**Resolution needed:** Explicitly separate the "English understanding" tier (SLM, higher power, sparse invocation) from the "agent evaluation" tier (BNN lattice, low power, continuous operation). Architecture.md currently reads as if the LLM *is* the agent's brain.

**Deprecation action:** Pending discussion. At minimum, the language needs scoping.

---

## CONTRADICTION 3: "Near-zero cold start times" for Wasm agents vs. Lattice initialization reality

**Location:** `architecture.md` → "Wasm Execution Layer" → "Agility"
**Legacy claim:** Wasm provides "near-zero cold start times" for spawning thousands of ephemeral micro-agents.

**v1 constraint conflict:**
Module A establishes that the discrete lattice has a **physical routing congestion** problem. LUT6 structures require 50% more input pins per node, placing severe strain on place-and-route tools. The systolic array operates via rigid, rhythmic, neighbor-to-neighbor dataflow.

"Near-zero cold start" is a software abstraction. The physical reality is that routing a new agent's logic onto the lattice requires place-and-route compilation, which is not near-zero. The distributed buffer model requires that bit-packed SRAM weight streamers be physically adjacent to the PE grid. You cannot dynamically spawn "thousands" of agents onto a fixed lattice without physical reconfiguration.

The Wasm cold-start claim may hold for the *software sandbox* (Wasmtime instantiation). It does not hold for the *hardware execution* (lattice mapping). These are different operations.

**Resolution needed:** Distinguish Wasm module instantiation (fast, software) from lattice deployment (slow, physical). Architecture.md must not conflate the two.

**Deprecation action:** Pending discussion.

---

## CONTRADICTION 4: "Massive KV cache" framing vs. the actual memory model

**Location:** `insights.md` → "Memory as Active Computation"
**Legacy claim:** The systolic array paper proposes using "CPU + smart indexing instead of massive KV cache in transformers."

**v1 constraint conflict:**
This insight was written before the v1 constraints crystallized the compute model. It frames the problem as "how to optimize transformer inference" — but the v1 architecture doesn't use transformers for agent execution at all. The BNN lattice doesn't have a KV cache. It has fixed recurrent reservoirs with LIF units and fading memory near the edge of chaos.

The *principle* (memory as active computation, not passive cache) is correct and aligns with Module B's temporal unrolling model. But the *specifics* (KV cache, indexed attention, O(n²) bottleneck) are transformer-domain concerns that don't apply to the discrete lattice.

**Resolution needed:** Reframe this insight in terms of the actual substrate: SRAM weight streamers as active memory, PE grid as spatial compute, and the distributed buffer model as the memory hierarchy. The "smart indexing" becomes "sparse input encoding into spatio-temporal spike trains" (Module B).

**Deprecation action:** The KV cache / transformer-specific language should be deprecated. The principle (active memory) should be preserved and reframed.

---

## CONTRADICTION 5: "Aider/Claude Code style" monolithic agent as the discarded alternative

**Location:** `architecture.md` → "Core Paradigm Shift"
**Legacy claim:** "Discard: The 'Single Brain' monolithic agent (Aider/Claude Code style) running complex, full-repo reasoning across a fractured network."

**v1 constraint conflict:**
This is not a mathematical or physical contradiction — it's a framing issue. The v1 constraints don't mention "full-repo reasoning" or "Aider/Claude Code." They describe a system where 1-bit BNNs perform agent evaluation on discrete lattices. The monolithic agent is discarded not because of WAN latency (the legacy reasoning) but because the **physics of the substrate** (sub-25W envelope, femtojoule-per-op, distributed buffer model) make monolithic computation physically impossible.

The legacy reasoning (WAN latency → can't pool VRAM → can't run big model) is correct but incomplete. The deeper reason is that the *target hardware* physically cannot execute a monolithic model, regardless of network topology.

**Resolution needed:** Update the rationale. The monolithic agent is discarded because the discrete lattice substrate demands spatial distribution of computation, not merely because WAN latency makes VRAM pooling unviable.

**Deprecation action:** Pending discussion. The discard decision stands; the rationale needs upgrading.

---

## CONTRADICTION 6: SSM/Mamba comparison vs. fixed recurrent reservoir model

**Location:** `insights.md` → "Memory as Active Computation" → "Connection to SSMs"
**Legacy claim:** SSMs (Mamba) represent a path toward breaking the O(n²) attention bottleneck by "replacing attention with recurrent state compression."

**v1 constraint conflict:**
Module B specifies the recurrent architecture as **Liquid State Machines (LSM) with fixed, randomly connected recurrent cores** — not learned state-space models. LSMs bypass the vanishing/exploding gradient problem by keeping the recurrent reservoir fixed (no gradient flows through it; only the readout layer is trained). Mamba/SSMs are learned recurrent models with trained state-space parameters. These are fundamentally different:

- **LSM (v1):** Random fixed reservoir, readout trained via STDP or evolutionary methods, $\mathcal{O}(n)$ memory (S-TLLR rule).
- **SSM/Mamba:** Learned state transitions, trained via BPTT through the recurrent state, differentiable selection mechanism.

The LSM approach is chosen specifically because **1-bit networks cannot propagate meaningful gradients through recurrent connections** (Module B constraint). SSMs assume continuous-valued hidden states amenable to gradient flow.

**Resolution needed:** The SSM/Mamba reference in insights.md should be marked as an alternative path that was evaluated and rejected in favor of the LSM/LIF model, with the reason being the discrete 1-bit constraint.

**Deprecation action:** Pending discussion.

---

## CONTRADICTION 7: "Post-POSIX" framing of memory management

**Location:** `architecture.md` → "Fundamental Questions" → "How is memory managed?"
**Legacy claim:** Memory management is "TBD — see insights on active vs passive memory, model-managed load/store."

**v1 constraint conflict:**
The v1 constraints now specify the memory model precisely:
- **Distributed SRAM buffers** physically adjacent to PE grids (Module A)
- **Zero-copy memory boundaries** mapping linear memory directly to kernel buffers (Module D)
- **Epoch-based yielding** for merge computation to maintain local-first responsiveness (Module D)

"TBD" is no longer accurate. The memory model is: Wasm linear memory for agent state, distributed SRAM for lattice weights, zero-copy host boundaries for async I/O, and CRDT-delta propagation for cross-node state.

**Resolution needed:** Replace "TBD" with the v1 memory model.

**Deprecation action:** Straightforward update once other contradictions are resolved.

---

## DEPRECATED CONCEPTS (Confirmed, no discussion needed)

These items from the legacy docs are confirmed deprecated by the v1 constraints with no ambiguity:

1. **`llama.cpp` layer-by-layer RPC offloading** (architecture.md) — Explicitly rejected in the architecture doc itself. No change needed, already discarded.

2. **ROCm as a backend option** (architecture.md) — Already decided against ("ROCm too brittle"). No change needed.

3. **"Full repository context" for local models** (architecture.md, "Local Cognitive Nodes") — The v1 BNN lattice has no concept of repository context. Agents process binary vectors, not codebases. This framing (from the "coding assistant" mental model) is permanently deprecated.

---

## ITEMS NEEDING DISCUSSION (Not Yet Classified)

These are ambiguous items that may or may not contradict v1 constraints — they need your judgment:

1. **"8B-14B quantized models per node"** — Is the SLM a permanent fixture (English parser), or a transitional crutch until the projection layer can be trained directly on raw sensor/spike input? If transitional, there's a deprecation path for the SLM entirely.

2. **"WebGPU standard" for agent-hardware interaction** (architecture.md) — Module A's compute model is XNOR-popcount on LUT6 fabric, not GPU compute shaders. WebGPU may still apply to the SLM inference tier, but it's not the agent execution interface. Needs scoping.

3. **"The agent as von Neumann machine" (insights.md)** — The v1 constraints describe agents as patterns in a discrete lattice, not as von Neumann machines with fetch-decode-execute cycles. The von Neumann framing may still be useful as a *logical* abstraction (the agent issues requests, waits for responses), but it's not the *physical* reality. Is the von Neumann metaphor load-bearing, or should it be replaced with the "data-flow-wavefront" model from the CRDT topology?

---

*Last updated: 2026-03-27*
*Next action: Walk through each item and resolve deprecation decisions.*
