# Project CLADE

## Overview
Post-POSIX Computing - exploring computing architectures beyond the POSIX process model.

## Current Phase
Transitioning from research synthesis to design.

## Key Questions
- What replaces the process as the fundamental unit of computation?
- How do capability-based systems differ from permission-based systems?
- What can embedded systems teach us about minimal, secure design?
- How do WebAssembly and the Component Model enable new paradigms?

## Research Foundation
See: https://github.com/zh4ngx/clade-research
- 69 curated sources across WebAssembly, FPGA, agents, embedded systems, academic papers
- YouTube talks from Sutton, LeCun, Silver, embedded Rust community
- Synthesis available in Deep Research output (NotebookLM)

## Working Documents
- `docs/insights.md` - Key findings from research
- `docs/architecture.md` - Evolving system design

## Commands
- Build: (tbd)
- Test: (tbd)
- Run: (tbd)

## Specialist Model Escalation

This repo spans code and research. Choose models accordingly:

- `cg` (GLM 5.1) — default. Cheap iteration, code exploration, first-pass thinking. ~95% of daily work.
- `co` (Opus 4.7) — architecture decisions, structural problems, code review, planning. Escalate when GLM's output is shallow or when the problem benefits from deeper reasoning.
- A specialist reasoning model (GPT-5 Pro, Gemini Deep Think, or whatever's currently SOTA on math benchmarks) — hard math, symbolic manipulation, proof verification. Manual escalation only — don't automate, don't route silently.

### When to escalate to a specialist reasoning model

Only when you'd normally hedge ("this seems right," "I think this works") on a math-heavy or proof question. Flag uncertainty explicitly: "I'm not confident about this step. Worth verifying with a reasoning-specialized model." Then the user decides whether to query manually.

### Anti-patterns

- Don't fabricate math answers. An honest "I don't know" is more useful than a confident wrong proof step.
- Don't escalate routine questions. Most code and structural work doesn't need specialist reasoning.
- Don't use specialist models for code. Claude handles code well across the board.
- Don't encode specific model names as durable facts. The SOTA shifts — the principle ("use a reasoning-specialized model") is more durable than "GPT-5 Pro."

### The real GLM-vs-Opus boundary

It's quota pressure and depth, not a clean capability line. Default to `cg`; escalate to `co` when you hit shallow analysis or the problem is architectural. There's no strict "GLM can't do X" rule.
