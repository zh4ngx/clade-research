# Spike-Vector Bridge

**Status:** v0.1 — first-pass synthesis from GLM + Gemini v2 lit maps; awaiting Claude.ai third source for v0.2.

Related: [wasm-vm-world-model-thinking](../../notes/wasm-vm-world-model-thinking.md), [wasm-determinism-replay](wasm-determinism-replay.md), [bsca-fabric](bsca-fabric.md), [clade-architecture](../architecture.md), [self-distillation-context-compression](self-distillation-context-compression.md), [tpo-target-policy-optimization](tpo-target-policy-optimization.md), [llms-as-computers](llms-as-computers.md), [language-oriented-programming](language-oriented-programming.md), [agent-protocol-stack](agent-protocol-stack.md), [pulse-feedback](pulse-feedback.md), [surprise-driven-bootstrapping](surprise-driven-bootstrapping.md)

Source lit maps: [spike-vector-bridge-research-glm](../../notes/spike-vector-bridge-research-glm.md), [spike-vector-bridge-research-gemini-v2](../../notes/spike-vector-bridge-research-gemini-v2.md)

## The Question

Project CLADE has carried an apparent tension since inception. The original meaning was **L+DE** — Latent Directed Emergence — pointing at continuous high-dimensional vectors as the substrate of agentic understanding. Over time the project has evolved toward the **BSCA fabric** (binarized spiking cellular automata) as the hardware substrate, where state is binary spikes on a 1-bit lattice. Adjacent threads in the vault — [wasm-determinism-replay](wasm-determinism-replay.md) (deterministic VM state), [llms-as-computers](llms-as-computers.md) (Percepta's interpreter-in-weights), [bsca-fabric](bsca-fabric.md) (1-bit STDP) — all point at the same axis from different angles.

The question this note answers: **are these representations of the same computational reality, and if so, what's the unified math?**

The short answer, after this lit review: **yes, and the math is partly worked out, with two specific 2025 papers as the load-bearing pieces.**

## Four Representations of the Same State

Captured during an on-the-way-back-from-Sutro insight ([wasm-vm-world-model-thinking](../../notes/wasm-vm-world-model-thinking.md)):

| # | Representation | Math | Domain | Concretely |
|---|----------------|------|--------|-------------|
| 1 | Wasm VM state | σ(t) ∈ Σ (large finite) | Discrete | Stack, locals, linear memory, IP, call stack |
| 2 | Latent vector | h(t) ∈ R^d | Continuous | SSM hidden state predicting σ(t+1) |
| 3 | Spike train | s(t) ∈ {0,1}^N over time | Discrete + temporal | LIF neuron outputs |
| 4 | Serialized bytes / Morse | bits unrolled over clock cycles | Discrete + temporal | Voltage transitions on a wire |

**The Morse-code observation:** representations (3) and (4) are not just two encodings of the same info — at the limit, they are isomorphic. A vector v ∈ R^d at b-bit precision is d×b bits; serialized over time, it IS a binary signal on a wire, which IS a spike train (high voltage = spike, low = silence). This is literally how every digital signal works.

But naive serialization wastes the spike substrate's expressive power. The interesting question is: **what's the optimal coding scheme C: R^d → {0,1}^N such that the SSM transition `f` commutes with C?** I.e., C(f(h)) ≈ T(C(h)) where T is the spike-substrate dynamics.

That commutation question turns out to be exactly the open math the field has been circling.

## The Existing Bridges

Two cross-source lit reviews (GLM + Gemini-cli v2 with verified citations) converged on roughly the same set of formal frameworks. Sorted by relevance to CLADE:

### Tier 1 — Already-load-bearing for CLADE

**Neural Engineering Framework (NEF) / Semantic Pointer Architecture (SPA)** — Eliasmith group, Waterloo Centre for Theoretical Neuroscience.
- **NEF Representation Theorem:** for any LTI system x_{t+1} = Ax_t + Bu_t, there exists a spiking neural network with N neurons such that decoded population activity approximates the LTI system with error O(1/√N). This IS a constructive proof that linear SSMs can be implemented in spikes. The encoding/decoding matrices ARE the coding scheme C and its inverse.
- **SPAUN** (2.5M spiking neurons performing 8 cognitive tasks; Eliasmith et al., 2012) is the proof-of-concept at scale.
- Limitation pre-2025: works for linear f. Mamba-style selective SSMs have nonlinear f.
- Source: GLM lit map; "How to Build a Brain" (Eliasmith 2013).

**Vector Symbolic Architectures (VSA) / Hyperdimensional Computing (HDC)** — Kanerva (Berkeley Redwood), Kleyko (Luleå/Redwood), Plate (Otago), Furlong (Waterloo).
- High-dimensional binary or bipolar vectors that behave statistically like continuous vectors. Algebraic operations (binding via XOR, bundling via majority, permutation) implement compositionality.
- Same algebra works on continuous vectors AND binary spike patterns — the bridge layer.
- Recent: VSA on neuromorphic hardware (Loihi, Kleyko et al.) directly maps VSA operations to spike-based circuits.

**Reservoir Computing / Liquid State Machines** — Maass, Jaeger, Markram.
- LSM's liquid IS a continuous high-dim vector emerging from spike dynamics. Echo State Property = formal stability condition equivalent to SSM stability.
- An ESN is an SSM with fixed (random) transition and learnable output. **Mamba = ESN where the transition is also learned.** This connects modern SSMs directly to spike-based reservoirs.

### Tier 2 — Mechanism-level connections

**Stochastic Computing / Sigma-Delta Modulation** — Gaines (1969), Alaghi & Hayes (2013).
- Bit streams represent real numbers via firing probability. Multiplication = AND, addition = MUX.
- **An integrate-and-fire neuron IS a first-order Σ-Δ modulator.** Not analogy — same circuit. Membrane potential integrates input; threshold-cross fires a spike; reset = the ΣΔ loop.
- This makes the user's Morse-code intuition mathematically precise: the LIF neuron is the canonical continuous-to-spike encoder.

**Predictive Coding** — Rao & Ballard (1999), Friston (Free Energy Principle), Salvatori (spiking PC).
- Cortex implements predictive coding: each layer predicts the layer below; only prediction errors propagate upward.
- **Prediction error IS a spike signal.** Brendel et al. (2020) implements PC in integrate-and-fire neurons — spikes ∝ prediction error.
- Connects to [surprise-driven-bootstrapping](surprise-driven-bootstrapping.md): surprise IS the spike, the spike IS the gating event.

**Vector Quantization (VQ-VAE)** — van den Oord et al. (2017).
- Continuous latents → discrete codebook indices. Commitment loss attracts continuous states to nearest codebook entry — analogous to attractor basins in [bsca-fabric](bsca-fabric.md).
- Map "codebook entry" → "spike pattern" and you have a *learnable* coding scheme C.

### Tier 3 — Adjacent / supporting

**Liquid Time-Constant Networks** (Hasani et al. 2021/2022, MIT CSAIL) — continuous-time RNNs with time-varying dynamics. In the limit of hard activation, becomes a spiking model. Bridges SSMs and spiking via ODE formulation.

**Koopman operator theory** — lifts nonlinear dynamics to (potentially infinite-dim) linear systems via the Koopman embedding. If you can Koopman-linearize a nonlinear SSM, you can then NEF-implement the linear system in spikes. Two-step composition error analysis is unworked.

## The 2025 Breakthroughs

Two papers from the Gemini v2 lit map (verified — both real, both correctly cited) substantially close the math:

### Shaw, Furlong, Anderson, Orchard (Jan 2025) — Category Theory for VSAs

[arXiv:2501.05368](https://arxiv.org/abs/2501.05368) — *"Developing a Foundation of Vector Symbolic Architectures Using Category Theory"*

- **Authors are at the Waterloo Centre for Theoretical Neuroscience** (Eliasmith's lab). Furlong has worked with Eliasmith on NEF; Orchard is a Waterloo CTN researcher. So this is a direct extension of the NEF/SPA tradition into formal category-theoretic foundations.
- Uses **right Kan extensions of the external tensor product** to generalize VSA operations.
- Frames VSAs as alternatives to neural networks that "explicitly model compositionality" while remaining "compatible with neural networks and gradient-based learning."
- **Why this matters:** category theory is the formal language for "X commutes with Y" — functoriality is exactly the property we're asking for. If C is a functor between representation categories, then C(f(h)) = T(C(h)) holds *by definition of functoriality*. The Shaw et al. paper provides the categorical machinery that could turn the commutation question from "does it work?" into "what category should we choose?"

### Fabre, Dudchenko, Bouhadjar, Neftci (Jun 2025) — SiLIF

[arXiv:2506.06374](https://arxiv.org/abs/2506.06374) — *"SiLIF: Structured State Space Model Dynamics and Parametrization for Spiking Neural Networks"*

- Neftci is one of the leading SNN training researchers (surrogate gradients, neuromorphic).
- Two LIF variants: (1) two-state neuron with learnable discretization timestep + logarithmic reparametrization; (2) complex-state for oscillatory dynamics.
- **SOTA among spiking neuron models on speech recognition** — empirically demonstrates that SSM-style structured dynamics work in spiking neurons.
- **Why this matters:** the gap GLM identified ("extend NEF from linear to nonlinear dynamics") is *partially closed*. SiLIF DOES the extension — SSM dynamics in LIF neurons, learnable, with results. The SiLIF parametrization addresses the gradient-propagation instabilities that previously blocked this.

**Together, Shaw et al. + Fabre et al. answer different halves:**
- **SiLIF**: empirical bridge — SSM dynamics CAN be implemented in spiking neurons with SOTA results
- **Shaw et al.**: theoretical framework — category theory provides the language to PROVE which mappings commute and why

## The Unified Picture (v0.1)

Three layers, each with a candidate formal grounding:

| Layer | What it does | Existing math | CLADE-relevant work |
|-------|--------------|---------------|---------------------|
| **Algebra** | Operations on representations (binding, bundling, permutation) | VSA / HDC | Shaw et al. (CT formalization) |
| **Dynamics** | How state evolves | SSMs (continuous) ↔ LIF (spiking) | SiLIF (Fabre et al.) |
| **Substrate** | Physical implementation | Standard floating-point ↔ 1-bit FPGA / spike hardware | [bsca-fabric](bsca-fabric.md) |

The **CLADE-specific contribution** is a fourth axis these layers don't yet address:

| Layer | What it does | Status |
|-------|--------------|--------|
| **Anchor** | Concrete deterministic computational target | **wasm VM as the reference frame — novel for CLADE** |

Most of the spike-vs-vector literature works on abstract MDPs or biological neurons. CLADE's anchoring in a deterministic VM (per [wasm-determinism-replay](wasm-determinism-replay.md)) gives a concrete substrate to test commutation against — you can take a wasm program, generate execution traces, train both an SSM world model and a SiLIF SNN world model, and formally compare them via a categorical coding scheme.

## What CLADE Adds Beyond SiLIF + Shaw et al.

1. **Wasm VM as the deterministic-state reference frame.** SiLIF works on speech recognition (continuous, noisy input). Shaw et al. works on abstract VSA operations. Neither anchors to a deterministic computational substrate. CLADE's framing — predict the next σ(t+1) given σ(t) for an actual wasm program — provides a clean test bed where ground-truth dynamics are known exactly.

2. **The world-model angle.** Per Luke Wagner's WASM-IO observation ([wasm-determinism-replay](wasm-determinism-replay.md)): "deterministic, bounded, replayable execution is good for learning world model." CLADE proposes the world model is the spike-vector-bridged thing: a continuous SSM world model that can also be deployed as a spiking BSCA fabric, with formal guarantees of commutation.

3. **Hardware realization on the BSCA fabric.** SiLIF demonstrates SSM dynamics in (presumably floating-point) LIF neurons. The [bsca-fabric](bsca-fabric.md) is 1-bit weights with binary STDP. The harder question — "can SiLIF-style dynamics survive 1-bit binarization?" — is open and CLADE-specific. The stochastic STDP trick from [bsca-fabric](bsca-fabric.md) (probabilistic flip on causal spike, statistically mimics smooth STDP) is the candidate mechanism.

4. **L+DE as the continuous-emergent half of the same system.** Original CLADE meaning (Latent Directed Emergence) points at continuous latent vectors as the substrate of agentic understanding. The current BSCA framing points at discrete spike substrates. The synthesis: they're the same system at different levels of abstraction — L+DE describes the CONTINUOUS view (what the agent's "understanding" looks like in R^d), BSCA describes the SUBSTRATE view (what the FPGA fabric does), and SiLIF + Shaw et al. provide the math for moving between them.

5. **Pulse-feedback as structured temporal signal** ([pulse-feedback](pulse-feedback.md)) — feedback isn't a scalar reward but a temporal pattern. This connects directly to predictive coding: prediction errors propagate as spike trains, not as scalars. The pulse-feedback thesis becomes "feedback should be in the same representation as the state — a spike train, since spikes ARE prediction errors per predictive coding."

## Open Mathematical Questions (refined from GLM's list with the 2025 papers)

1. **Does Shaw et al.'s CT framework formally prove commutation for VSA operations + spike substrate?** Need to read the paper carefully — the abstract suggests yes, but the specific theorem statements need to be checked against our exact commutation requirement.

2. **Can SiLIF be extended to predict deterministic VM-state transitions?** SiLIF was tested on speech recognition. The wasm-VM world-model task is structurally different — discrete states, deterministic transitions, exact ground truth available. The training procedure (surrogate gradients) should work in principle, but the experimental work hasn't been done.

3. **What's the right Category for "wasm executions"?** The wasm transition function T: Σ × Instruction → Σ defines a category-theoretic structure. Probably a monoidal category (sequential composition of instructions = monoidal product). Mapping this to Shaw et al.'s VSA category should yield the formal commutation result.

4. **Does the L+DE attractor-basin framing admit a CT description?** Attractor basins in [bsca-fabric](bsca-fabric.md) are continuous-emergent structures over discrete spike dynamics. CT machinery for attractor dynamics exists (e.g., dynamical systems as coalgebras) but the connection to VSA categories isn't worked out.

5. **How does binary STDP (BSCA) relate to gradient-based SiLIF training?** SiLIF uses surrogate gradients (continuous shadow weights, snap to binary for forward pass). BSCA proposes binary STDP with hidden accumulators OR stochastic flips. Are these equivalent in the limit? Is there a categorical morphism from "continuous gradient training" to "discrete STDP" that preserves the learned dynamics?

6. **Capacity bound for spike-encoded SSMs.** An SSM with hidden dimension d models dynamics with d degrees of freedom. How many spikes per time step (N) preserve this capacity? Shannon-style bound exists implicitly but hasn't been stated for the specific case of structured (Mamba-style) SSMs.

## Experimental Program

Concrete next steps to test the unified picture, in order:

**Phase 1 — Wasm trace generation.** Pick small deterministic wasm programs: Fibonacci, sort, simple stack-machine interpreter. Generate execution traces with full VM state at each step.

**Phase 2 — SSM world model.** Train Mamba/S4 to predict next VM state from current state. Establish baseline accuracy and continuous latent representation.

**Phase 3 — SiLIF world model.** Train SiLIF SNN on the same traces using surrogate gradients (per Fabre et al.'s recipe). Establish baseline spiking accuracy.

**Phase 4 — Commutation test.** Define a coding scheme C inspired by NEF/SPA + Shaw et al.'s VSA functorial framework. Test: does C(SSM prediction) match SNN prediction within bounded error?

**Phase 5 — BSCA implementation.** Implement the SiLIF dynamics with 1-bit weights using stochastic STDP per [bsca-fabric](bsca-fabric.md). Measure how much fidelity is lost and whether the commutation result still holds approximately.

**Phase 6 — CT formalization.** Write the categorical version of the CLADE architecture: wasm executions as a category, SSM world model as a functor, spiking implementation as a functor, coding scheme as natural transformation. This is GPT-5.4 Pro consultation territory.

## Practical Impact

If this synthesis holds:

- **Agents can be trained as continuous SSMs and deployed as spiking BSCA fabrics** with formal guarantees of behavioral preservation. Training cost low (gradient descent on continuous), inference cost low (1-bit FPGA).
- **The deterministic wasm substrate** gives a concrete benchmark for measuring agent behavior — previously this was abstract.
- **The L+DE / BSCA / SNN tension dissolves** — they're the same system at different levels.
- **A roadmap to formal verification** of agent behavior emerges via category theory + Lean 4 ([lean4-clade](lean4-clade.md)) — if commutation can be proven categorically, it can be verified in Lean.
- **Connection to [llms-as-computers](llms-as-computers.md) (Percepta)** sharpens — the "interpreter in transformer weights" thesis is a continuous-substrate version of what CLADE proposes for spiking. Same idea, different layer.
- **Connection to [agent-protocol-stack](agent-protocol-stack.md)** — if "tool calls are wasm bin," and the agent's understanding of those tool calls is in the same VSA category as the wasm executions, then composition becomes formally clean.

## Single-Source Caveats (v0.1)

This synthesis draws primarily from:
- GLM lit map (Tier-1 framing, Eliasmith/Maass/Friston/Hasani identified)
- Gemini-cli v2 lit map (the two 2025 papers — Shaw et al., Fabre et al. — are the load-bearing pieces)

Pending sources to incorporate in v0.2:
- Claude.ai Research (deferred to tomorrow when work email available)
- Possibly a successful Gemini Web DR retry
- Direct reading of Shaw et al. (2501.05368) and Fabre et al. (2506.06374) — currently relying on abstract + verification. Need to read the actual papers before claiming the math works.

The risk: I'm anchoring on two specific 2025 papers that perfectly fit the question. That's exactly where confirmation bias concentrates. Read the papers in full before treating the unification as established.

## TODO

- [ ] **Read Shaw et al. (2501.05368) in depth** — verify the categorical machinery actually proves commutation as suggested. Likely the highest-value read.
- [ ] **Read Fabre et al. SiLIF (2506.06374) in depth** — confirm the SSM-LIF bridge mechanism and assess whether it extends to non-speech domains.
- [ ] Add Claude.ai Research as third lit-map source for v0.2.
- [ ] Synthesize v0.2 incorporating the third source + direct paper readings.
- [ ] **GPT-5.4 Pro consultation** on the unified-math question, given v0.2: "Does Shaw et al.'s CT framework + SiLIF's empirical bridge formally close the commutation question for SSM ↔ spiking representations? What's still missing for a wasm-VM-anchored proof?"
- [ ] Sketch Phase 1 (wasm trace generation) — probably a separate project note when ready.
- [ ] Cross-link this note from [bsca-fabric](bsca-fabric.md), [clade-architecture](../architecture.md), [wasm-determinism-replay](wasm-determinism-replay.md) open-questions sections (v0.2 cleanup pass).
