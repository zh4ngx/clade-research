# Heterogeneous Cognitive Architecture

> Different compute paradigms for different roles. The brain doesn't use transformers everywhere.

## Core Idea

Most AI systems use one compute paradigm everywhere (transformers). The brain doesn't. A heterogeneous architecture assigns specialized paradigms to specialized roles:

| Layer | Paradigm | Brain Region | Role |
|-------|----------|--------------|------|
| **Sensory** | SSM (Mamba/RWKV) | Language cortex | Streaming perception, O(1) inference |
| **Memory** | External store (differentiable or KV) | Hippocampus | Persistent storage, retrieval, consolidation |
| **CNS** | CA / BNN / SNN | Prefrontal cortex / basal ganglia | Routing, gating, decision-making |

## Why Each Piece

**SSM for language:** Well-validated. Linear-time inference, continuous state compression, no KV cache explosion. The hidden state is the compressed perception of the input. Fatal weakness: lossy, irreversible compression. Once the state forgets, it's gone.

**External memory:** Solves the SSM recall deficit. The SSM handles semantics, memory handles hard facts. Key design questions: addressing mechanism (content-addressable? learned routing?), write policy (when to commit vs let state expire?), read policy (CNS-driven or automatic associative recall?).

**CNS layer:** This is the BSCA fabric. The novel part. SNNs provide event-driven gating (spike → retrieve, spike → generate, spike → act). CAs provide self-organizing dynamics and homeostasis. BNNs track uncertainty. No single paradigm covers all three.

## The Hard Problems

### 1. The Interface Problem

SSM output is continuous (high-dim vectors). CNS input is discrete (spikes, cell states, binary weights). The translation layer is unsolved.

- SSM → Memory: Query generation (standard, works)
- Memory → CNS: How do retrieved memories become spikes? (open)
- CNS → SSM: How do routing decisions modulate state? (open)

### 2. The Gradient Problem

End-to-end training across differentiable → non-differentiable boundaries:

| Component | Differentiable? | Training Signal |
|-----------|----------------|-----------------|
| SSM | Yes | Supervised (language) |
| Memory (soft attention) | Yes | Supervised |
| BNN | Variational (hard) | Uncertainty |
| SNN | No (surrogate gradients) | RL / STDP |
| CA | No | Evolutionary / Hebbian |

Likely requires multi-stage training: pretrain separately, fine-tune jointly. Or abandon end-to-end and use structured pulse-sequence feedback (pulse-feedback.md) between layers.

### 3. Hardware Mismatch

SSMs exploit GPU SRAM parallel scans. SNNs/CAs want neuromorphic hardware. The FPGA substrate (insights.md Theme 2) is the answer: SSM on GPU, CNS on FPGA fabric, memory on host. Different hardware for different paradigms.

## Multi-Agent Consultation Results

Five independent models (CC, QC, GC, Kimi, Qwen) reviewed this architecture. Consensus:

**Agreement:**
- Legitimate research direction
- SSM + external memory is well-grounded
- CNS layer is where novelty and difficulty live
- Training across paradigms is the primary challenge
- Start small: SSM + memory + one CNS component

**CNS preference:**
- CC: BNN (practical, differentiable)
- QC: SNN (natural event-driven gating — strongest for CNS role)
- GC: BNN orchestrator (easiest start), CA for world model
- Kimi: SNN for action selection, BNN for uncertainty, CA for homeostasis

**Recommended starting point:** SSM + external memory + trained retrieval gate. Add SNN/CNS once the interface is characterized.

**Qwen's key addition:** SNNs fail at language specifically (dense symbolic vs sparse event-driven mismatch). The heterogeneity is load-bearing — SSM for dense perception, SNN for sparse gating, reconstructive memory. Don't spike where you should attend.

## Connection to Existing Architecture

**BSCA Fabric (insights.md Theme 2):** The CNS layer. LUTs as Boolean logic, heterogeneous graph topologies, hardware-native recurrent memory, attractor basins for decision-making.

**Temporal Scaling (insights.md Theme 4):** CNS iterates in time on FPGA while SSM processes streaming input. Different timescales for different paradigms. The CNS doesn't need to keep up with token rate — it operates on compressed state snapshots.

**Dual-Process Routing:** BSCA = System 1 (fast, reflexive, always-on), SSM = System 2 (deliberative, expensive, episodic). The heterogeneous architecture is dual-process made concrete.

**Pulse Feedback (pulse-feedback.md):** Inter-layer communication. Structured temporal signals between SSM, memory, and CNS. The feedback spectrum (1-bit → token sequences) maps to the interface complexity gradient.

**Self-Distillation (self-distillation-context-compression.md):** How the CNS layer learns. Feedback absorbed into weights rather than growing context. STDP on the BSCA lattice is a 1-bit form of distillation.

**CRDT State Topology (idea-frontier-v1.md §3):** External memory as CRDT-managed state. The CNS layer operates on CRDT deltas — binary payloads that propagate state changes as structured patterns.

## Why Not All SNN

If the brain is spiking, why not make the whole system an SNN? Because language is the wrong fit for spikes.

Language requires dense symbolic representations (4096-dim continuous embeddings), graduated similarity, soft attention (weighted sums over tokens), and massive dense parallel computation. SNNs communicate via sparse binary spikes — a single spike conveys very little. They can't naturally compute softmax or weighted sums.

Empirically, SNNs for NLP consistently achieve 20-30% of dense net accuracy at the same parameter count, even on simple tasks. The gap on language generation is catastrophic.

SNNs win at event detection, temporal pattern recognition, sensorimotor control, and ultra-low-power inference on neuromorphic chips — when the problem is sparse and event-driven. Language is neither.

**The right split:** SSM for perception (dense, continuous, high-bandwidth), SNN for gating (sparse, event-driven, discrete decisions). Don't spike where you should attend.

## Biological Memory Is Reconstructive

External memory shouldn't be a clean database query. Biological memory is:

- **Content-addressable:** Retrieval via associative cue, not address lookup. A smell triggers a childhood memory.
- **Reconstructive:** The hippocampus stores context (when, where, emotional state), not the event itself. Reassembly on recall produces interference, confabulation, and drift.
- **Plastic:** Memories change through interference. Storage is not archival.

| Biological Memory | Computational Equivalent |
|-------------------|------------------------|
| Short-term (sustained prefrontal firing) | SSM hidden state — active, decaying, lossy |
| Long-term semantic (synaptic weight changes) | Model weights (SSM parameters) |
| Long-term episodic (hippocampal contextual binding) | External associative store with contextual tags |

A vector DB with cosine similarity is closer to biological fidelity than a KV store. Imperfect recall forces the SSM to be robust — feature, not bug.

## The Core Tension

The brain evolved over 500M years for real-time embodied navigation on 12 watts — optimized for survival, not language. Language is a 50K-year cultural invention running on top of a navigation/object/social substrate. The architectural principles (sensory → memory → motor with gating) are sound even when binary spikes aren't computationally appropriate for language. The heterogeneity is the point — match paradigm to problem.

## Open Questions

- What representation flows from SSM → CNS? Raw hidden states? Bottlenecked projection? At what granularity per token, per N tokens, or on event triggers?
- Can pulse-sequence feedback serve as the SSM → CNS translation layer?
- Minimum viable CNS: single SNN gate (retrieve / generate / act) or full CA lattice?
- Does K=4 edge-of-chaos constrain the SSM → CNS interface bandwidth?
- Evaluation: what benchmark shows this beats a well-tuned transformer of similar size? Long-context recall? Multi-turn agentic planning? Knowledge persistence across sessions?
