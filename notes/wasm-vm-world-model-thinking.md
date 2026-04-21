# Wasm VM World Model — Four Representations of State

Raw thinking from a 2026-04-21 chat session, captured before it slips. Source for the eventual [spike-vector-bridge](../docs/architecture/spike-vector-bridge.md) synthesis.

## The Insight

Same computational state, four ways to represent it:

1. **Wasm VM state** — operand stack, locals, linear memory, IP, call stack. Discrete, finite, deterministic transition function. (Wasm determinism verified — see [wasm-determinism-replay](../docs/architecture/wasm-determinism-replay.md).)
2. **Latent vector** in R^d from an SSM / world model that predicts the VM's evolution. Continuous, low-dim relative to |Σ|.
3. **Spike train** s(t) ∈ {0,1}^N over time. Discrete + temporal. SNN-style encoding (rate / temporal / population codes).
4. **Morse / serialized bytes** — a vector unrolled into a temporal bit stream. Same as (3) at the limit but framed as digital signaling.

## The Key Claim

**(2) and (3) are not just two encodings of the same info — they are isomorphic when you think of the latent vector being "tapped out" over time.**

A vector v ∈ R^d at 32-bit precision is d × 32 bits. Serialize over time = a binary signal on a wire. That signal IS a spike train (high voltage = spike, low = silence). This is literally how every digital signal works.

Beyond naive serialization, biology + neuromorphic systems exploit sparse coding, population codes, temporal binding, phase coding — making the spike representation more efficient than serialized bits.

**Open theoretical question:** what's the optimal coding scheme C: R^d → {0,1}^N such that the SSM transition `f` commutes with C? I.e., C(f(h)) ≈ T(C(h)) where T is the spike-substrate dynamics?

## Why Wasm Specifically

Most spike-vs-vector literature works on abstract MDPs or biological neurons. Wasm gives a **concrete, deterministic, well-defined discrete state target** to ground the theory against. You can take a wasm program, generate execution traces, train an SSM world model on them, train an SNN world model on them, and formally compare.

## Connection to CLADE's L+DE Origin

Original CLADE = Latent Directed Emergence (continuous side). Current vault = BSCA + SNN substrate (discrete side). These aren't in opposition — they're the same computational reality in two representations. Wasm-as-substrate (per `wasm-determinism-replay.md`) is the discrete reference frame; the SSM is the continuous frame; the SNN is the spike frame; all three are isomorphic up to encoding/decoding.

A world model in any one of them is a world model in all three.

## The Bigger Pattern

This is the same axis as several other recent vault threads:
- [llms-as-computers](../docs/architecture/llms-as-computers.md) (Percepta) — substrate IS the interpreter; programs encoded in transformer weights
- [bsca-fabric](../docs/architecture/bsca-fabric.md) — substrate IS the computation; attractor dynamics encode behavior
- [language-oriented-programming](../docs/architecture/language-oriented-programming.md) — DSLs as compressed semantic surfaces
- The percepta thesis at three different scales: weight space, lattice space, spike space

## Deep Research Dispatch

Drafted prompt in chat for Gemini Deep Research (web). Phase plan:
1. **Gemini Deep Research** — lit map for HDC, reservoir computing, rate/temporal/population coding, stochastic computing, sigma-delta, predictive coding, VSA, plus VM-as-world-model literature
2. **GPT-5.4 Pro** — given the lit map, theoretical synthesis: is there a unified math? What is it?

### Gemini DR Plan (returned 2026-04-21, then stopped on "unusually high traffic")

Plan is well-structured, save for retry or for GLM dispatch:

1. Search academic databases for papers treating deterministic virtual machine states (WebAssembly, JVM, EVM, eBPF, CIL) as targets for state-space models, transformers, or world-model learning.
2. Survey formal mathematical literature bridging continuous high-dimensional latent vectors and timed binary spike trains, specifically analyzing Hyperdimensional Computing, Reservoir Computing, stochastic computing, and predictive coding.
3. Investigate theoretical frameworks that map continuous latent dynamics to spike trains while preserving dynamical structure, looking for coding schemes that commute with state-space model transition functions.
4. Identify specific researchers, academic groups, and laboratories working implicitly or explicitly at the intersection of virtual machine state modeling, continuous world models, and spiking neural network substrates.
5. Explore existing experimental methodologies for evaluating isomorphic representations of computation, focusing on setups that compare the predictive modeling of small program execution traces using both continuous models and spiking networks.
6. Synthesize the strongest formal mathematical results that unify discrete state machines, continuous vectors, and spike sequences, identifying specific open questions where the theoretical bridge remains unresolved.
7. Compile findings into a structured report covering literature review, key researchers, formal results, open questions, and precise citations with arXiv or DOI links.

## TODO

- [ ] Run the Gemini Deep Research prompt (in chat)
- [ ] Synthesize spike-vector-bridge.md from this raw note + Deep Research output + earlier session's framing
- [ ] Fold the Morse / serial-bytes insight into bsca-fabric's "Open Questions" or self-distillation note as a connecting hypothesis
