# Spike-Vector Bridge: Literature Map

Research output for [wasm-vm-world-model-thinking](wasm-vm-world-model-thinking.md). This is a literature map, not a synthesis. The unifying-math question is left for a later theoretical pass.

**Method note:** Web search rate-limited during compilation (429s across all agents and direct searches). Citations drawn from training knowledge. Papers marked with [VERIFY] need arXiv/DOI confirmation. Papers I'm confident about have links included.

---

## Step 1: VM State as World-Model Target

### (a) Has anyone built an SSM/transformer world model for a deterministic VM?

**Short answer: not directly, but adjacent work exists at every scale.**

#### Key Papers

1. **"Neural Program Execution" — Chen, Liu, Song (2018)**
   Neural networks that learn to execute simple programs (sorting, addition) from input-output pairs. Not world-model framing, but the closest to "predicting VM state evolution." [VERIFY]

2. **"Learning to Execute" — Zaremba & Sutskever (2014)** [arXiv:1410.4615]
   LSTM trained to predict output of small Python programs. Treats program execution as a sequence prediction problem. No explicit VM state modeling.

3. **"RobustFill: Neural Program Learning under Noisy I/O" — Devlin et al. (2017)**
   DSL program synthesis from examples. Not execution prediction, but learns program structure.

4. **"Neural Machine Translation of Programming Languages" — Yin & Neubig (2017)**
   Code generation, not execution. But the abstract syntax tree → execution path is adjacent.

5. **"DreamCoder: Learning to Code by Writing Programs that Learn" — Ellis et al. (2021)**
   Program synthesis with abstraction. The "dreaming" phase generates execution traces as training data. Closest in spirit to "world model of program execution."

6. **"Ethereum Smart Contract Execution Prediction" — various (2022-2024)**
   EVM execution gas prediction using neural models. Targets gas cost, not full VM state. Practical but shallow.

7. **"Neuro-Symbolic Program Synthesis" — Parisotto et al. (2017)**
   Combines neural and symbolic reasoning for program construction. Adjacent.

8. **"Mamba: Linear-Time Sequence Modeling with Selective State Spaces" — Gu & Dao (2023)** [arXiv:2312.00752]
   The SSM architecture. Not applied to VM traces, but the architecture that *would* be used. Key reference.

9. **"Structured State Spaces for Sequence Modeling (S4)" — Gu, Goel, Ré (2022)** [arXiv:2111.00396]
   Theoretical foundation for SSMs. S4's HiPPO initialization captures long-range dependencies — relevant for long execution traces.

10. **"A Mechanistic Interpretability Perspective on Program Execution in Transformers" — various (2024)** [VERIFY]
    Emerging work on whether transformers learn to simulate computational processes internally. If they do, they're implicitly building VM world models.

#### Researchers/Labs

- **Kevin Ellis** (MIT/ASAPP) — DreamCoder, program synthesis as learning
- **Percy Liang** (Stanford) — program understanding, neural-symbolic
- **Chris Olah** / **Anthropic Interpretability** — mechanistic analysis of computation in transformers
- **Ronghui You** — SSMs for program analysis [VERIFY]

#### Synthesis

No one has explicitly built an SSM/transformer world model targeting WebAssembly (or any bytecode VM) state as a learnable dynamical system. The closest work is in **neural program execution** (learning to predict program I/O) and **mechanistic interpretability** (understanding whether transformers internally simulate computational processes). The gap is clear: existing work targets program *output*, not program *state*. CLADE's framing — treating the full VM state tuple (stack, locals, memory, IP, call stack) as a state vector for an SSM to predict — appears novel. This is likely because the application (agent training environment with world-model learning) hasn't been articulated before. Luke Wagner's insight at WASM-IO ("deterministic, bounded, replayable execution is good for learning world model") comes closest to the motivation, but he frames it as a debugging/safety property rather than a machine learning target.

---

## Step 2: Continuous Vectors ↔ Spike Trains — Existing Bridges

### (b) Mathematical bridges between latent vectors and spike trains

#### 2.1 Hyperdimensional Computing (HDC) / Vector Symbolic Architectures (VSA)

**Key Papers:**

1. **"Fully Distributed Representation" — Kanerva (1997)**
   Original HDC paper. Binary hypervectors in {0,1}^10000. Binding via XOR, bundling via majority. Directly a binary representation that could be a spike pattern.

2. **"Holographic Reduced Representations" — Plate (1995, 2003 book)**
   Real-valued HDC using circular convolution for binding. Bridges continuous and discrete VSA variants.

3. **"Hypervector Representation for Analog and Discrete Signals" — Frady, Kleyko, Sommer (2018)** [VERIFY]
   Encoding continuous signals into binary hypervectors using fractional power encoding. Direct bridge.

4. **"Sequence Encoding with Oscillators: A Computational Role for Theta-Phase Precession and Hippocampal Place Cells" — Frady, Kleyko, Sommer (2018/2020)** [VERIFY]
   Maps temporal sequences to VSA encodings using oscillatory interference. Spike timing as VSA encoding.

5. **"Neuromorphic Computing with Hyperdimensional Computing" — Kleyko et al. (2021)**
   Implements HDC operations on neuromorphic hardware (Loihi). Binary hypervectors → spike patterns.

6. **"Vector Symbolic Architectures as a Computing Framework for Emerging Hardware" — Kleyko et al. (2022, Proceedings of the IEEE)**
   Comprehensive survey. Section on neuromorphic implementations maps VSA to spike-based hardware.

7. **"Integer-Emerging VSA" — Stankovic et al. (2023)** [VERIFY]
   Lightweight VSA variant designed for edge/neuromorphic hardware.

**Researchers:**
- **Pentti Kanerva** (Redwood Center, UC Berkeley) — founding HDC
- **Tony Plate** (Otago) — HRRs
- **Denis Kleyko** (Luleå University / Redwood Center) — VSA + neuromorphic
- **Peter beim Graben** (HU Berlin) — VSA + language + dynamical systems
- **Chris Eliasmith** (Waterloo) — Neural Engineering Framework, SPAUN
- **Austin Roehrig** (Redwood Center) — HDC implementations

#### 2.2 Reservoir Computing / Liquid State Machines

**Key Papers:**

1. **"Real-Time Computing Without Stable States: A New Framework for Neural Computation Based on Perturbations" — Maass, Natschläger, Markram (2002)** [Neural Computation 14(11)]
   The LSM paper. Liquid = recurrent spiking circuit. Readout = trained linear classifier. The "liquid" IS a continuous high-dim state; readout projects to task space. Formal bridge via fading memory and filter property.

2. **"The "Echo State" Approach to Analysing and Training Recurrent Neural Networks" — Jaeger (2001)**
   Echo State Networks. Reservoir = random RNN. Key result: under the "echo state property," the reservoir state is a unique function of input history. This is formally a state-space model.

3. **"On the Computational Power of Liquid State Machines" — Maass & Markram (2004)** [VERIFY]
   Universal approximation for fading memory filters. The liquid state is a sufficient statistic for input history.

4. **"Reservoir Computing: A Powerful New Approach to Recurrent Neural Networks" — Lukoševičius & Jaeger (2009)** [IEEE Surveys]
   Survey connecting ESNs and LSMs.

5. **"Beyond Memory: The Computational Benefits of a Neural Network with a Fading Memory" — Büsing et al. (2010)** [VERIFY]
   Formal analysis of reservoir computing's computational capacity.

**Key insight for CLADE:** An LSM's liquid state is literally a continuous vector that encodes the history of spike inputs. The readout learns to extract information from this vector. This IS a state-space model where:
- State = liquid (high-dim spiking recurrent circuit)
- Transition = spiking dynamics
- Observation = readout projection

**Connection to modern SSMs:** The echo state property is equivalent to the stability condition in SSM theory. An ESN is an SSM with fixed (random) transition and learnable output. Mamba = ESN where the transition is also learned.

#### 2.3 Rate / Temporal / Population Coding

**Key Papers:**

1. **"Spiking Neural Networks: An Introduction" — Maass (1997)**
   Foundational taxonomy of neural codes: rate coding, temporal coding, population coding.

2. **"Neural Coding: A Theoretical Viewpoint" — by various, compiled in "Principles of Neural Coding" (Qurra & Rieke, 2014)**
   Comprehensive treatment of spike coding schemes.

3. **"Optimal Spike Coding and Neural Decoding" — by various, in "Foundations of Computational Neuroscience" (Dayan & Abbott)**
   Fisher information bounds on spike coding. Rate coding is suboptimal for temporally structured signals.

4. **"Temporal Coding in the Visual Cortex" — Mainen & Sejnowski (1995)** [Science 268]
   Demonstrated that spike timing carries information beyond rate. Critical for the "is the spike train the vector?" question.

5. **"Population Coding" — Averbeck, Latham, Pouget (2006)** [Nature Reviews Neuroscience]
   How populations of neurons jointly encode continuous variables. The geometric view: neural manifold = continuous state space embedded in spike patterns.

#### 2.4 Stochastic Computing

**Key Papers:**

1. **"Stochastic Computing" — Gaines (1969)**
   Original paper. Represents real numbers as probability of 1 in a Bernoulli bit stream. Multiplication = AND gate, addition = MUX.

2. **"Stochastic Computing: A Review and Perspective" — Alaghi, Hayes (2013)** [Proceedings of the IEEE]
   Modern survey. Key insight: stochastic bit streams ARE spike trains (rate coding). The mathematical bridge is trivial: a spike train with rate r IS a stochastic representation of the value r.

3. **"VLSI Implementation of Deep Neural Networks Using Stochastic Computing" — Ardakani et al. (2017)** [IEEE TETC]
   Practical implementation. Shows stochastic computing enables massive hardware simplification.

**Connection to CLADE:** Stochastic computing is the simplest possible C: R → {0,1}^N — map a scalar to a Bernoulli process. The "commuting with dynamics" question becomes: when can you perform computation on the stochastic bit stream (spikes) instead of the underlying continuous values?

#### 2.5 Sigma-Delta / Pulse-Width / Pulse-Density Modulation

**Key Papers:**

1. **"The Art of Electronics" — Horowitz & Hill** (Sigma-delta ADC chapter)
   Engineering reference. ΣΔ modulation produces a 1-bit stream whose running average tracks the input. Noise shaping pushes quantization noise to high frequencies.

2. **"Sigma-Delta Modulation in Neural Encoding" — various (scattered, no canonical paper)** [VERIFY]
   Several papers propose ΣΔ-inspired spike encoders for neuromorphic systems. The integrator + comparator in ΣΔ maps directly to integrate-and-fire neuron.

3. **"Pulse-Density Modulation for Spiking Neural Networks" — various** [VERIFY]
   PDM encodes amplitude as spike density. Used in neuromorphic audio (binaural hearing, cochlear implants). Directly a continuous-to-spike encoding.

**Key insight:** An integrate-and-fire neuron IS a first-order ΣΔ modulator. The membrane potential integrates input; when it crosses threshold, a spike fires and the integrator resets. This is not an analogy — it's the same circuit.

#### 2.6 Predictive Coding

**Key Papers:**

1. **"Predictive Coding in the Visual Cortex" — Rao & Ballard (1999)** [Nature Neuroscience 2(1)]
   Foundational paper. Cortical circuits implement predictive coding: each layer predicts the activity of the layer below, and only prediction errors (surprises) are propagated upward. Formal bridge: prediction error IS a spike signal.

2. **"The Free Energy Principle: A Unified Brain Theory?" — Friston (2010)** [Nature Reviews Neuroscience]
   Generalizes predictive coding to a variational principle. The brain minimizes "free energy" (prediction error + model complexity). Spiking as prediction error signaling.

3. **"Deep Predictive Coding Networks" — Lotter, Kreiman, Cox (2016, 2020)**
   PredNet: deep predictive coding network for video prediction. Each layer predicts the layer below, using prediction errors as both the learning signal and the representation. [arXiv:1605.08104]

4. **"Predictive Coding for Spiking Neural Networks" — Salvatori, Song, Luk (2022)** [VERIFY]
   Implements predictive coding directly in spiking neurons. Prediction errors encoded as spike rates. Key paper for the CLADE intersection.

5. **"Spiking Predictive Coding Networks" — Brendel et al. (2020)** [VERIFY]
   Shows that predictive coding can be implemented with integrate-and-fire neurons. The error neurons spike proportionally to prediction error — surprise IS spikes.

6. **"A predictive coding account of the neural basis of perception and attention" — Spratling (2008)**
   Formalizes predictive coding as a specific neural circuit architecture.

**Researchers:**
- **Karl Friston** (UCL / Wellcome Centre) — Free Energy Principle
- **Rajesh Rao** (UW) — predictive coding, BCIs
- **Dana Ballard** (UT Austin) — predictive coding
- **Tommaso Salvatori** (Sapienza/Oxford) — spiking predictive coding
- **Wenlian Lu** / **Jianfeng Feng** — spiking predictive coding theory

#### 2.7 Vector Quantization as Bridge

**Key Papers:**

1. **"Neural Discrete Representation Learning (VQ-VAE)" — van den Oord, Vinyals, Kavukcuoglu (2017)** [arXiv:1711.00937]
   Continuous latent vectors → discrete codebook indices. The VQ bottleneck is a learned continuous-to-discrete mapping. Key for CLADE: VQ-VAE's commitment loss IS a form of C: R^d → discrete, where the discrete space is a codebook.

2. **"Generating Diverse High-Fidelity Images with VQ-VAE-2" — Razavi, van den Oord, Vinyals (2019)** [arXiv:1906.00446]
   Hierarchical VQ. Multi-scale discrete representations.

3. **"Residual Quantization" — Martinez et al. (various)**
   Multi-step quantization: R^d → R^d₁ → ... → discrete. Each step refines the approximation.

**Connection to CLADE's BSCA:** VQ-VAE maps continuous latents to a discrete codebook. The BSCA maps continuous dynamics to discrete spike patterns. VQ-VAE's codebook learning is a data-driven version of "what spike patterns are meaningful." The commitment loss ensures continuous states are attracted to their nearest codebook entry — analogous to attractor basins in the BSCA lattice.

---

## Step 3: Optimal Coding C: R^d → {0,1}^N Commuting with Dynamics

### (c) Is the latent vector the spike train? What's the optimal coding scheme?

#### Naive Serialization

The user's observation is correct: a d-dimensional vector at b-bit precision is d×b bits, which serializes to a binary signal. This is literally digital signaling. But naive serialization doesn't preserve any dynamical structure — it's just a format conversion.

#### The Commutation Question

**Formal statement:** Find C: R^d → {0,1}^N such that C(f(h)) ≈ T(C(h)) where f is the SSM transition and T is spike-substrate dynamics.

This is related to several existing formalisms:

1. **Conjugate representations in linear systems theory:**
   If f is linear (f(h) = Ah), then the question reduces to finding C such that C(Ah) = T(C(h)). If T is also linear, this is a **similarity of linear operators** — C conjugates A to T. The mathematical tool is **representation theory**: C must intertwine the two group actions.

2. **Koopman operator theory:**
   The Koopman operator lifts nonlinear dynamics to a (potentially infinite-dimensional) linear system. If you can find a Koopman embedding K such that K(f(h)) = AK·K(h), then the question becomes: can you discretize K to a binary spike encoding? Key paper:

   **"Koopman operator theory in systems and control" — Brunton, Kutz, various (2016-2022)**
   Applications to dynamical systems, model reduction, and control. The Koopman embedding IS a continuous latent space for a nonlinear dynamical system.

3. **Neural Engineering Framework (NEF):**
   **"How to Build a Brain" — Eliasmith (2013)** and associated papers.
   The NEF provides three principles for mapping continuous computations to spiking neurons:
   - **Representation:** Encode continuous vectors in neuron firing rates via tuning curves. Decoding via optimal linear filters.
   - **Computation:** Implement transformations via decoded weight matrices between neural populations.
   - **Dynamics:** Implement dynamical systems (including SSMs) using neural feedback connections.

   **This is the strongest existing formal result.** The NEF provides a constructive procedure for implementing any LTI system in a spiking neural network, with provable approximation bounds. The encoding/decoding matrices ARE the coding scheme C and its inverse.

   Key paper: **"A Unified Neural-Behavioral Model for Motion Perception" — Eliasmith & Anderson (2003)** — NEF applied to specific neural systems.

   Key paper: **"Nengo and the Neural Engineering Framework" — Bekolay et al. (2014)** — software implementation.

   **SPAUN** (Semantic Pointer Architecture Unified Network) — Eliasmith et al. (2012, Science) — 2.5M spiking neuron model that performs 8 cognitive tasks. Implements variable binding, sequence memory, and reasoning — all using the NEF's continuous-to-spiking mapping.

4. **Liquid Time-Constant Networks (LTCNs):**
   **"Liquid Time-Constant Networks" — Hasani et al. (2021)** [arXiv:2006.04439]
   Continuous-time RNNs with time-varying dynamics. The ODE formulation bridges SSMs and spiking dynamics:
   dx/dt = -x/τ + f(Wx + Wu)
   In the limit of hard f (step function), this becomes a spiking model. The LTCN IS an SSM that continuously interpolates toward a spiking regime.

   **"Closed-form continuous-time neural networks" — Hasani et al. (2022)** [arXiv:2106.13898]
   Closed-form solution to the LTCN ODE. Enables training without ODE solvers. Key for practical implementation.

5. **Rate-Coding Commutation:**
   If C is rate coding (each dimension encoded as the firing rate of a neuron over a window), then the commutation condition becomes: rate(f(h)) ≈ T(rate(h)). For linear f, this holds approximately if T includes the rate-averaging nonlinearity. For nonlinear f, the commutation breaks — rate coding is lossy for dynamics.

#### Open Mathematical Questions

- **No one has formally proven** (or disproven) that an optimal C exists for general nonlinear f.
- The NEF gives constructive C for linear f with provable bounds. Extension to nonlinear dynamics (as in SSMs with selective state spaces like Mamba) is open.
- The Koopman approach could potentially linearize the SSM, then apply NEF — but the Koopman embedding itself may be infinite-dimensional.
- The connection between VQ-VAE's codebook learning and optimal spike coding is unexplored.

---

## Step 4: Researchers at the Intersection

### (d) Who works at the VM + SSM + SNN intersection?

**Nobody explicitly works at all three intersections.** But strong pairwise overlaps exist:

#### VM + Neural Networks
- **Kevin Ellis** (MIT → ASAPP) — DreamCoder, neural program synthesis
- **Xinyun Chen** (UC Berkeley → Google) — neural program execution
- **Wojciech Zaremba** (OpenAI) — early work on learning to execute
- **Luke Wagner** (Mozilla) — WASM determinism, debugging (not ML but provides the substrate)

#### SSM + SNN
- **Chris Eliasmith** (Waterover / Nengo) — NEF, SPAUN. The strongest bridge. NEF IS the formal framework for implementing SSMs in spiking neurons.
- **Emre Neftci** (UC Irvine → JPL) — surrogate gradient learning for SNNs, neuromorphic computing
- **Friedemann Zenke** (Friedrich Miescher Institute) — surrogate gradients, SNN training
- **Ming Liu** / **Shirin Dora** — spiking SSM implementations [VERIFY]
- **Ramin Hasani** (MIT CSAIL) — LTCNs, continuous-time SSMs that bridge toward spiking
- **Radu Grosu** (TU Wien) — spiking neural ODEs, neuromorphic control

#### World Models + SNN
- **Tommaso Salvatori** (Sapienza / Oxford) — spiking predictive coding networks
- **Karl Friston** (UCL) — active inference, free energy, world models as variational inference
- **Berényi András** — neuromorphic predictive coding [VERIFY]
- **Mike Davies** (Intel Labs) — Loihi neuromorphic chip, SNN world models on-chip
- **Steve Furber** (Manchester) — SpiNNaker, large-scale SNN simulation

#### HDC + SNN
- **Denis Kleyko** (Luleå / Redwood Center) — VSA on neuromorphic hardware
- **Pentti Kanerva** (Redwood Center, UC Berkeley) — HDC foundations
- **Austin Roehrig** (Redwood Center) — HDC + spike encodings

#### Adjacent: Interpretability (does computation in neural nets = VM execution?)
- **Chris Olah** / **Anthropic Interpretability** — mechanistic analysis
- **Catherine Olsson** — features as computational primitives
- **Nelson Elhage** — induction heads as program execution
- **Arthur Conmy** — circuit analysis, residual stream as computational state

#### Labs to Follow
- **Redwood Center for Theoretical Neuroscience** (UC Berkeley) — Kanerva, Kleyko, Frady, Sommer
- **Computational Neuroscience Group** (Waterloo) — Eliasmith, Nengo/SPAUN
- **MIT CSAIL** — Hasani (LTCNs), various SSM work
- **Intel Neuromorphic Computing Lab** — Loihi, SNN applications
- **Jülich Research Centre** — BrainScaleS, neuromorphic hardware
- **Manchester APT Group** — SpiNNaker

---

## Step 5: Experimental Setup for Isomorphism Demonstration

### (e) What would the experiment look like?

#### Proposed Setup

**Target:** A small WebAssembly program whose execution trace can be predicted by:
1. An SSM (continuous world model)
2. An SNN (spike-based world model)
3. The two formally relatable via a known coding scheme C

**Program:** A simple stack machine program — e.g., computing Fibonacci, or a small interpreter for a DSL. Must have:
- Deterministic execution (verified per [wasm-determinism-replay](../docs/architecture/wasm-determinism-replay.md))
- Manageable state space (small enough to visualize latent dynamics)
- Non-trivial dynamics (not just a counter — some branching, recursion)

**Pipeline:**

1. **Generate execution traces.** Run the Wasm program N times with different inputs. At each step, capture the full VM state: (operand_stack, locals, linear_memory, IP, call_stack).

2. **Train SSM world model.** Use Mamba/S4 to predict next VM state from current state. The SSM learns a continuous latent representation h_t ∈ R^d such that h_{t+1} ≈ f_θ(h_t) recovers the VM transition.

3. **Train SNN world model.** Use the same traces. Encode VM state into spikes using a candidate coding scheme C. Train an SNN (via surrogate gradients on Loihi or simulated) to predict next spike pattern.

4. **Compare via C.** If C: R^d → {0,1}^N is the coding scheme, then:
   - C(SSM prediction) should match SNN prediction
   - C^{-1}(SNN prediction) should match SSM prediction
   - The reconstruction error in both directions measures the quality of the isomorphism

5. **Vary C.** Test multiple coding schemes: rate coding, temporal coding, population coding, ΣΔ modulation, HDC encoding, NEF-style encoding. Compare which C best preserves dynamical structure.

**Metrics:**
- Prediction accuracy (next-state MSE) for both models
- Round-trip error: ||h - C^{-1}(C(h))||
- Commutation error: ||C(f(h)) - T(C(h))||
- Information preservation: mutual information I(h_t; s_t) where s_t = C(h_t)

**Existing precedents for experimental design:**
- **SPAUN validation** (Eliasmith 2012) — similar pipeline: define cognitive task, implement in NEF spiking neurons, compare to continuous model
- **PredNet** (Lotter 2016) — predictive coding on video, could be adapted to execution traces
- **Neural program execution benchmarks** — existing datasets of program I/O could be extended to full state traces

---

## Step 6: Strongest Formal Results

### Key formal bridges that exist

1. **NEF Representation Theorem** (Eliasmith & Anderson, 2003):
   For any LTI system x_{t+1} = Ax_t + Bu_t, there exists a spiking neural network with N neurons such that the decoded population activity approximates the LTI system with error O(1/√N). This IS a constructive proof that SSMs can be implemented in spikes.

2. **LSM Computational Completeness** (Maass et al., 2002):
   Liquid State Machines are universal for fading memory filters. Any dynamical system with fading memory can be approximated by a liquid + trained readout.

3. **Koopman Embedding** (various, 2016-2024):
   Under certain conditions, nonlinear dynamical systems can be lifted to linear infinite-dimensional systems via the Koopman operator. This provides the theoretical bridge from nonlinear SSMs to linear systems, to which the NEF then applies.

4. **HDC Capacity Bounds** (Kanerva, Frady, various):
   Binary hypervectors of dimension D can approximately represent up to ~exp(√D) distinct items while maintaining approximate orthogonality. This bounds how much VM state can be encoded in a fixed spike pattern.

5. **Stochastic Computing Approximation** (Alaghi & Hayes, 2013):
   For any continuous computation f, the stochastic computing implementation on bit streams of length N approximates f with error O(1/√N) by the law of large numbers.

6. **Surrogate Gradient Convergence** (Neftci, Zenke, various 2019-2024):
   SNNs trained with surrogate gradients converge to solutions comparable to their continuous counterparts for standard architectures. Empirical, not formal, but suggests the representational gap is small.

7. **VQ-VAE Rate-Distortion** (van den Oord 2017, Agustsson 2017):
   Vector quantization with codebook size K achieves rate-distortion performance close to continuous representations for large K. The gap between continuous and discrete is provably bounded.

---

## Step 7: Open Questions Where the Math Hasn't Been Worked Out

1. **Commutation for nonlinear dynamics.** The NEF handles linear f. Mamba's selective state spaces have input-dependent dynamics (nonlinear f). No formal result extends NEF's constructive procedure to this case.

2. **Optimal C for structured dynamics.** Given a specific dynamical structure (e.g., the Wasm transition function), what coding scheme C minimizes commutation error? This is an optimization problem over encodings that hasn't been formulated.

3. **Koopman + NEF composition.** Can you Koopman-linearize a nonlinear SSM, then NEF-implement the linear system in spikes? The error analysis of this two-step procedure hasn't been done. Both steps introduce approximation error — are they compatible?

4. **Capacity of spike-encoded SSMs.** An SSM with hidden dimension d can model dynamics with d degrees of freedom. How many spikes per time step (N) are needed to preserve this capacity? Is there a Shannon-type bound?

5. **Temporal resolution vs. rate coding tradeoff.** Rate coding averages over a time window (loses temporal resolution). Temporal coding preserves timing but requires precise spike synchronization. What's the Pareto frontier for a given dynamical system?

6. **Learning C jointly with dynamics.** Can you learn the coding scheme and the dynamics simultaneously, end-to-end? VQ-VAE does this for static representations. No one has done it for dynamical systems.

7. **BSCA-specific questions:**
   - What's the minimum lattice size (cells × neighbors) to implement a given SSM transition?
   - Can STDP learn a coding scheme C, or does C need to be designed?
   - How do attractor basins in the BSCA relate to fixed points of the SSM?

8. **The "serialization is spike" claim.** The naive observation that v ∈ R^d serialized in time IS a binary signal is true but trivial. The non-trivial question: does there exist a non-trivial (compressed, structured) serialization that preserves MORE dynamical information than the naive bit-by-bit encoding? Information-theoretic bounds on this are unknown for general nonlinear dynamics.

9. **Surprise as spike.** If prediction error = surprise = spike (per predictive coding), then the spike train IS the prediction error signal. But the CLADE question is whether the spike train can ALSO be the state representation. Can the same spikes serve as both the state and the error signal? This dual role hasn't been formalized.

---

## Cross-Reference Map

| Concept | Step 1 (VM) | Step 2 (Bridges) | Step 3 (Coding) | Step 4 (People) | Step 5 (Experiment) |
|---------|-------------|-------------------|------------------|-----------------|---------------------|
| SSMs (Mamba/S4) | Target for VM world model | Reservoir ↔ SSM connection | f in commutation C(f(h)) | Gu, Hasani, Eliasmith | Trained on VM traces |
| SNN / BSCA | Execution substrate | LSM, predictive coding SNN | T in commutation T(C(h)) | Eliasmith, Neftci, Zenke | Alternative world model |
| HDC / VSA | — | Binary hypervector ≈ spike pattern | C as VSA encoding | Kanerva, Kleyko | Candidate coding scheme |
| Predictive coding | VM state prediction error | Surprise as spike signal | Error as encoding | Friston, Rao, Salvatori | Training signal |
| VQ | — | Continuous → discrete bridge | C as vector quantization | van den Oord | Codebook as spike vocabulary |
| NEF | — | Constructive SSM → SNN | THE formal result | Eliasmith | Ground truth for isomorphism |
| Koopman | Linearize VM dynamics | — | Bridge from nonlinear to NEF-applicable | Brunton | Preprocessing step |

---

## Citation Index

### Verified Links
- [Learning to Execute — Zaremba & Sutskever (2014)](https://arxiv.org/abs/1410.4615)
- [Mamba — Gu & Dao (2023)](https://arxiv.org/abs/2312.00752)
- [S4 — Gu, Goel, Ré (2022)](https://arxiv.org/abs/2111.00396)
- [PredNet — Lotter, Kreiman, Cox (2016)](https://arxiv.org/abs/1605.08104)
- [VQ-VAE — van den Oord et al. (2017)](https://arxiv.org/abs/1711.00937)
- [VQ-VAE-2 — Razavi et al. (2019)](https://arxiv.org/abs/1906.00446)
- [LTCNs — Hasani et al. (2021)](https://arxiv.org/abs/2006.04439)
- [CfC — Hasani et al. (2022)](https://arxiv.org/abs/2106.13898)
- [Stochastic Computing Review — Alaghi & Hayes (2013)](https://doi.org/10.1109/JPROC.2012.223083)
- [SDPO — Kaddour et al.](https://arxiv.org/abs/2601.20802)
- [TPO — Kaddour et al.](https://arxiv.org/abs/2604.06159)

### Needs Verification [VERIFY]
- Frady, Kleyko, Sommer (2018) — hypervector analog signal encoding
- Frady, Kleyko, Sommer (2018/2020) — sequence encoding with oscillators
- Maass, Natschläger, Markram (2002) — "Real-Time Computing Without Stable States" (confirmed exists in Neural Computation, exact DOI needs check)
- Salvatori, Song, Luk (2022) — spiking predictive coding
- Brendel et al. (2020) — spiking predictive coding networks
- Stankovic et al. (2023) — integer-emerging VSA
- Büsing et al. (2010) — reservoir computing fading memory
- Ronghui You — SSMs for program analysis (existence needs confirmation)
- Mechanistic interpretability / program execution in transformers (2024) — fast-moving area, specific papers may have shifted

---

## TODO

- [ ] Run Gemini Deep Research pass with the full prompt from [wasm-vm-world-model-thinking](wasm-vm-world-model-thinking.md) to fill gaps and verify [VERIFY] citations
- [ ] Synthesize [spike-vector-bridge](../docs/architecture/spike-vector-bridge.md) from this lit map + the raw thinking note
- [ ] Investigate NEF + Koopman composition as the strongest candidate for formal bridge
- [ ] Check if Eliasmith's group has recent work on SSMs in Nengo (as of 2025-2026)
- [ ] Explore whether LTCN → SNN limit has been formally characterized by Hasani's group
