# Pulse Feedback: From 1-Bit to Arbitrary Token Sequences

> Feedback as temporal structure, not scalar magnitude.

## Core Insight

Classical ML propagates feedback as a scalar reward — a single floating-point number. The BSCA fabric constrains this to 1-bit (fire / don't fire), propagated pairwise via STDP. But the substrate generalizes naturally: feedback can be an **arbitrary sequence of tokens** — a temporal pulse pattern where meaning is encoded in sequence structure, not signal magnitude.

## The Feedback Spectrum

| Level | Signal | Information Content | Example |
|-------|--------|---------------------|---------|
| **1-bit** | Fire / don't fire | Binary presence | BSCA STDP pairwise causality |
| **n-bit burst** | Rate-coded pattern | Pulse frequency in a window | Spike rate encodes confidence |
| **Token sequence** | Ordered pulse pattern | Arbitrary structured feedback | Sequence encodes "try again, different region of state space" |

## Why Sequences Over Scalars

Scalar reward collapses feedback to magnitude. Biological feedback is richer:

- **When** the feedback arrives (timing relative to action)
- **What pattern** it follows (burst vs single pulse)
- **From where** it originates (which upstream region)

A pulse sequence preserves all three. The feedback is not "good" or "bad" — it's a structured temporal message that the receiving circuit interprets through its own dynamics.

## Connection to Existing Architecture

**BSCA / STDP (insights.md Theme 4, Theme 7):**
The 1-bit STDP pairwise signal is the hardware floor — cheapest possible feedback. The shadow weight bridge (latent accumulator in BRAM) already introduces temporal structure: multiple STDP events accumulate before triggering a weight flip. This is implicitly burst-counting. The extension: instead of counting to a threshold, interpret the pulse *pattern* as a structured signal.

**Temporal Scaling (insights.md Theme 4):**
If the system *computes* in time, it should *learn* in time. Pulse-sequence feedback is the natural complement to temporal iteration.

**Signal-Aware Learning (idea-frontier-v1.md §2):**
AST-derived reward from signal traces already proposes richer-than-scalar feedback. Pulse sequences formalize this: the reward signal itself is a first-class temporal object.

**CRDT State Topology (idea-frontier-v1.md §3):**
Binary delta payloads already propagate state changes as structured bit patterns. Pulse feedback uses the same channel — the CA lattice itself carries both state updates and learning signals.

## Open Questions

- Minimal token vocabulary for useful structured feedback?
- Can attractor basins serve as feedback tokens (state-as-signal)?
- Latency tradeoff: sequence length vs real-time constraint?
- Does this subsume or complement the AST-derived reward signal from idea-frontier-v1.md?
