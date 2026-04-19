# Self-Distillation: Compressing Context into Weights

Related: [pulse-feedback](pulse-feedback.md), [bsca-fabric](bsca-fabric.md), [clade-insights](clade-insights.md), [tpo-target-policy-optimization](tpo-target-policy-optimization.md), [wasm-credit-assignment-replay](wasm-credit-assignment-replay.md)

## Core Idea

Standard LLM interaction accumulates context in the context window — each turn appends to an ever-growing sequence. The model's "memory" of the conversation is *external*: it lives in the prompt, not in the model.

Self-Distillation Preference Optimization (SDPO) proposes an alternative: **compress interaction history into model weights** rather than appending to context. The model repeatedly attempts a hard question, generates an answer, receives feedback, and *updates its weights* instead of extending the prompt.

Source: [arxiv 2601.20802](https://arxiv.org/abs/2601.20802) | [Project page](https://self-distillation.github.io/SDPO.html)

## The Loop

```
for each attempt:
    model πθ receives question x (+ compressed context)
    generates answer y
    receives feedback f (tokenized: runtime errors, judge evals, etc.)
    re-evaluate y under πθ(·|x,y,f) — the "self-teacher"
    distill self-teacher's corrected token predictions into πθ
    updates weights θt → θt+1

Result: πθ(·|x,c) encoded directly into πθ'(·|x)
```

The self-teacher is the *same model* conditioned on feedback. It retrospectively identifies its own mistakes in-context. The distillation step compares student vs self-teacher next-token distributions — dense per-token credit, not scalar outcome reward.

## Dense Credit Assignment

SDPO's key mechanism: instead of a scalar reward per attempt, it produces a per-token advantage by comparing:
- Student: $\pi_\theta(\cdot | x, y_{<t})$ — what the policy would do
- Self-teacher: $\pi_\theta(\cdot | x, y_{<t}, y, f)$ — what the policy would do *with feedback*

Where the self-teacher becomes more confident → positive advantage. Less confident → negative advantage. This is dense, targeted supervision along the entire attempt.

Even with scalar-only rewards (no textual feedback), SDPO uses successful rollouts as implicit feedback for failed attempts — learning from its own best generations without external demonstrations.

## Key Results (Updated from Project Page)

| Finding | Detail |
|---------|--------|
| **Sample efficiency** | Reaches GRPO's accuracy 6x faster (wall-clock) on Chemistry tasks (Olmo3-7B) |
| **Final accuracy** | Substantially higher than GRPO across scientific reasoning, tool use, competitive programming |
| **Reasoning efficiency** | Traces 11x shorter than GRPO — effective reasoning need not be verbose; GRPO enters logical loops |
| **Test-time acceleration** | On hard LiveCodeBench problems (pass@64 < 0.03), achieves same discovery probability as best-of-k with 3x fewer attempts |
| **Scales with model size** | Gap over GRPO widens from 0.6B → 8B (Qwen3). Better retrospection emerges with scale |
| **Scalar-only setting** | Still outperforms GRPO even without rich feedback — successful rollouts serve as implicit teachers |

## Why This Matters for CLADE

**Connection to [pulse-feedback](pulse-feedback.md):**
Feedback in CLADE is not a scalar reward but a structured temporal signal. SDPO provides a mechanism for *absorbing* that signal into weights rather than buffering it. The pulse sequence doesn't accumulate — it modifies. SDPO's dense per-token advantages are closer to pulse-sequence feedback than scalar rewards — structured, not just magnitude.

**Connection to [bsca-fabric](bsca-fabric.md):**
The BSCA lattice already does something analogous via STDP: pairwise causal events accumulate in a latent counter (BRAM), and eventually flip a binary weight. This is a 1-bit form of self-distillation — feedback is not stored as history but compressed into a weight change. SDPO generalizes this: the "latent counter" is the full weight matrix, and the "flip threshold" is replaced by a continuous distillation gradient.

**Connection to Temporal Scaling ([clade-insights](clade-insights.md) Theme 4):**
The system scales by iterating in time, not by expanding in space. SDPO is the same principle applied to learning: iterate on the same question, accumulate signal into parameters, rather than growing the context window (spatial expansion). Test-time self-distillation is literally this — repeated attempts on a single hard problem, each distillation step compressing the feedback into weights.

**Connection to [wasm-credit-assignment-replay](wasm-credit-assignment-replay.md):**
SDPO addresses credit assignment *within* a single trajectory by using the self-teacher for dense per-token supervision. WASM replay addresses credit assignment *across* components by forking at decision boundaries. They're complementary: SDPO tells you *where* in the token sequence things went wrong; WASM replay tells you *which component* caused it.

**Connection to [tpo-target-policy-optimization](tpo-target-policy-optimization.md):**
Both address the sparse credit assignment bottleneck in RLVR. TPO constructs an explicit target distribution $q$ over candidates — a between-trajectory signal. SDPO constructs a dense per-token signal *within* a trajectory using the self-teacher. TPO is "which trajectory was better"; SDPO is "which tokens within a trajectory were wrong." Composable: TPO for between-candidate selection, SDPO for within-candidate credit.

**Connection to [surprise-driven-bootstrapping](surprise-driven-bootstrapping.md):**
SDPO's self-teacher is a retrospective error signal — the model looks back at its own attempt with feedback and identifies where it went wrong. Surprise-driven retrieval is the forward analog — detecting prediction error in real-time to trigger memory. SDPO shows that retrospective self-correction is a powerful training signal; the same principle could drive offline replay in CLADE's sleep-phase consolidation.

## The Deeper Pattern

| Approach | Memory Location | Cost | Capacity |
|----------|----------------|------|----------|
| **Context window** | External (prompt tokens) | O(n) per turn | Bounded by window size |
| **Self-distillation** | Internal (model weights) | Fixed per update | Unbounded (weights encode indefinitely) |

The tradeoff: distillation is lossy. Not every detail survives compression into weights. But the capacity is structurally different — weights don't overflow. They generalize.

## Scales with Model Size — Implication for CLADE

SDPO's gap over GRPO widens at larger models. For CLADE's heterogeneous architecture:
- The **SLM interface** (8B Transformer) would benefit substantially from SDPO — strong retrospection at this scale
- The **BSCA fabric** operates at a fundamentally different scale (1-bit weights, no in-context learning). SDPO doesn't apply directly, but the *principle* — dense feedback from retrospective self-evaluation — could manifest as STDP events driven by replay-triggered surprise rather than online spike timing.

## Open Questions

- What is the minimum viable weight-update mechanism for a BSCA lattice? (STDP is one answer — are there others?)
- Can attractor basins serve as the "compressed representation" — distillation into basin topology rather than weight values?
- How does distillation quality scale with the number of iteration cycles?
- Does the K=4 edge-of-chaos regime constrain how much information can be distilled before destabilizing dynamics?
- Can SDPO's self-teacher pattern be adapted for non-language modalities — e.g., a BSCA fabric "replaying" its own spike patterns with a success/failure signal?
- Test-time self-distillation on a single hard problem ≈ online learning at inference. How does this interact with CLADE's goal of stable, persistent agents?

---
#clade #self-distillation #context-compression #learning
