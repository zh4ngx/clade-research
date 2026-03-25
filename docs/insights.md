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

### 3. Agent as Von Neumann Machine

**Core Insight (2026-03-25):**
What if the agent *is* a von Neumann machine that faces the von Neumann bottleneck? The agent is essentially a CPU (or emulated CPU) that interacts with GPUs, FPGAs, and memory asynchronously—across multiple clock cycles—and must wait for responses.

**Connection to CLADE:**
This reframes the agent not as a stateless function but as a compute unit with its own bottleneck dynamics:
- Agent issues requests to accelerators (GPU/FPGA)
- Agent waits across clock cycles for async responses
- Agent manages its own memory hierarchy (load/store across slow and fast tiers)

**Connection to Systolic/Energy-Efficient ML:**
Incorporates insights from systolic array research on energy-efficient machine learning. Systolic arrays were designed to *reduce* the von Neumann bottleneck by keeping data flowing through compute elements. If agents are von Neumann machines, maybe the solution isn't to make them faster—but to make them *systolic* (spatial, data-flow-driven).

**Implication:**
The "process" abstraction hides the von Neumann bottleneck. A post-POSIX agent abstraction might expose it intentionally—making latency and data movement first-class concerns.

## Open Questions

- What would a "memory load/store" instruction set look like for an LLM?
- Does smart indexing change the O(n²) attention cost structure?
- Can agents be embedded directly inside FPGAs or systolic array trays?
- If computation is spatial rather than temporal, what replaces "process" as the unit of agency?
- If an agent is a von Neumann machine, can it become systolic instead?
- Should agent abstractions expose latency and data movement as first-class concerns?

## Tensions and Tradeoffs

-

## Sources

- Bjarke Hammersholt Roune, "Designing AI Chip Hardware and Software" (2026) - Former Google TPU/Nvidia GPU engineer. Key insight: use indexed attention + SSD instead of massive HBM for KV cache.
- Systolic Array / Energy-Efficient ML research - Systolic arrays reduce von Neumann bottleneck via spatial dataflow; agents as von Neumann machines might benefit from similar architectural thinking.

## Next Steps

-
