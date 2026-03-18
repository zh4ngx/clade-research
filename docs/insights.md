# Research Insights

> Distilled findings from NotebookLM Deep Research synthesis.
> Paste the synthesis output here after running Deep Research.

## Executive Summary

(To be filled)

## Key Themes

### 1. Memory as Active Computation (not passive cache)

**Systolic Array Insight (2026-03-18):**
Paper author proposes using CPU + smart indexing instead of massive KV cache in transformers. This challenges the assumption that inference requires materializing all context into a giant passive cache.

**Connection to CLADE:**
Compare to the idea of models managing their own memory load/store operations. Both approaches share a common thread:
- *Traditional*: Memory as passive substrate (cache sits there, model reads)
- *Emerging*: Memory as active, indexed, agent-controlled resource

This aligns with capability-based thinking - the model has explicit capabilities for memory access rather than implicit access to everything.

**Connection to SSMs:**
This indexed/sparse attention approach is part of a broader trend moving away from O(n²) attention mechanisms. State Space Models (SSMs like Mamba) represent another path—both share the goal of breaking the quadratic bottleneck. Indexed attention does it by being selective about what enters the cache; SSMs do it by replacing attention with recurrent state compression. Both move memory from passive to active/compressed.

### 2. Hardware Substrate for Agents (FPGAs/Systolic Arrays)

**Open Query:**
Can agents be embedded directly inside FPGAs or systolic array trays? If memory becomes active and indexed (see Theme 1), and computation is spatial rather than temporal, what does an "agent" look like when it's not a process but a hardware-resident pattern?

This connects to:
- CLADE's post-POSIX question: what replaces the process?
- Embedded systems thinking: computation bound to physical substrate
- Systolic arrays as spatial computing (data flows through fixed computation)

### 3. [Theme]

## Open Questions

- What would a "memory load/store" instruction set look like for an LLM?
- Does smart indexing change the O(n²) attention cost structure?
- Can agents be embedded directly inside FPGAs or systolic array trays?
- If computation is spatial rather than temporal, what replaces "process" as the unit of agency?

## Tensions and Tradeoffs

-

## Sources

- Bjarke Hammersholt Roune, "Designing AI Chip Hardware and Software" (2026) - Former Google TPU/Nvidia GPU engineer. Key insight: use indexed attention + SSD instead of massive HBM for KV cache.

## Next Steps

-
