# Self-Distillation: Compressing Context into Weights

> Feedback absorbed, not accumulated.

## Core Insight

Standard LLM interaction accumulates context in the context window — each turn appends to an ever-growing token sequence. The model's "memory" of the conversation is external: it lives in the prompt, not in the model.

Self-Distillation Preference Optimization (SDPO) proposes an alternative: compress interaction history into model weights rather than extending the prompt. The model repeatedly attempts a hard question, generates an answer, receives feedback, and updates its weights via SDPO (batch size 1). The interaction history is distilled into the policy: πθ(·|x,c) → πθ'(·|x).

Source: [arxiv 2601.20802](https://arxiv.org/abs/2601.20802)

## The Loop

```
for each attempt:
    model πθ receives question x
    generates answer y
    receives feedback f
    updates weights θt → θt+1 via SDPO

Result: context c encoded into weights, no longer needed in prompt
```

## Connection to Existing Architecture

**STDP / BSCA (insights.md Theme 4 corollary, Theme 7):**
The BSCA lattice already performs a 1-bit analog of self-distillation via STDP. Pairwise causal events accumulate in a latent counter (BRAM), then flip a binary weight. Feedback is not stored as history — it is compressed into a weight change. SDPO generalizes this: instead of flipping one binary weight, update a structured parameter set.

**Pulse Feedback (pulse-feedback.md):**
Feedback in CLADE propagates as temporal pulse sequences, not scalar rewards. SDPO provides the absorption mechanism: pulse sequences modify weights rather than accumulating in a buffer. The sequence is consumed, not retained.

**Temporal Scaling (insights.md Theme 4):**
The system scales by iterating in time. SDPO applies the same principle to learning: iterate on the same question, accumulate signal into parameters, rather than growing the context window (spatial expansion). Distillation is temporal scaling applied to memory.

**Signal-Aware Learning (idea-frontier-v1.md §2):**
AST-derived reward already proposes richer-than-scalar feedback. Self-distillation answers the question: what does the system *do* with that signal? It absorbs it into weights, not buffers.

## The Deeper Pattern

| Approach | Memory Location | Cost | Capacity |
|----------|----------------|------|----------|
| Context window | External (prompt tokens) | O(n) per turn | Bounded by window |
| Self-distillation | Internal (model weights) | Fixed per update | Unbounded (weights generalize) |

Distillation is lossy — not every detail survives compression. But weights don't overflow. They generalize.

## Open Questions

- Minimum viable weight-update mechanism for the BSCA lattice? (STDP is one candidate — are there others?)
- Can attractor basin topology serve as the compressed representation?
- Does the K=4 edge-of-chaos regime constrain distillation capacity?
- Tradeoff between distillation lossiness and attractor stability?
