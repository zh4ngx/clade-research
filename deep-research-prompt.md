# Deep Research Prompt for NotebookLM

## Instructions
Copy this prompt into NotebookLM's Deep Research feature after uploading the source files from `~/dev/clade/sources/`.

---

## Prompt

You are a research assistant helping to synthesize knowledge for Project CLADE, an exploration of post-POSIX computing architectures. You have access to curated sources I've uploaded, and you can also search the web for additional context.

### Primary Goals

1. **Synthesize the uploaded sources** - Identify key themes, contradictions, and gaps across all the curated research
2. **Ground your analysis in the sources** - Always cite which source(s) support your claims
3. **Supplement with web research** - When sources are incomplete or you identify gaps, search the web and clearly indicate what came from uploaded sources vs. web research

### Key Questions to Address

**Architecture & Design:**
- What are the core principles that distinguish post-POSIX systems from traditional operating systems?
- How do WebAssembly, the Component Model, and WASI together enable new computing paradigms?
- What can embedded systems (TockOS, FreeRTOS, Embassy) teach us about minimal, secure system design?
- How do capability-based security models differ from POSIX permissions, and what are their tradeoffs?

**Agents & Memory:**
- What are the dominant architectures for long-running AI agents with persistent memory?
- How does MemGPT's approach to memory management relate to operating system memory models?
- What patterns emerge for agent communication and coordination?

**Hardware & Implementation:**
- What role do FPGAs and reconfigurable hardware play in post-POSIX visions?
- How close are we to practical implementations of these ideas?
- What are the minimal hardware requirements for running WASM-based component systems?

**Synthesis:**
- What are the 3-5 most important open problems in this space?
- Which approaches are most mature and ready for experimentation?
- What fundamental tensions or tradeoffs exist between different approaches?

### Output Format

Structure your response as:

1. **Executive Summary** (2-3 paragraphs on the most important insights)
2. **Thematic Analysis** (group findings by topic)
3. **Key Contradictions & Tensions** (where sources disagree or tensions exist)
4. **Gaps & Open Questions** (what's missing from the research)
5. **Recommendations** (concrete next steps for exploration)
6. **Source Citations** (which uploaded sources informed each section)

### Constraints

- Prioritize insights that connect multiple sources
- Be explicit about confidence levels
- When web research contradicts uploaded sources, highlight this
- Focus on actionable insights over general background

---

## After Running Deep Research

Once you have the output:
1. Save it to `~/dev/clade/synthesis/deep-research-output.md`
2. Identify any sources that were underutilized or missing key content
3. Note any topics that need additional research

## Refinement Notes

**Confirmed source files (7 total, well under 50 source limit):**
- [x] 01-wasm-and-runtimes.md (7 sources)
- [x] 02-fpga-and-hardware.md (9 sources)
- [x] 03-agents-and-memory.md (9 sources)
- [x] 04-embedded-systems.md (10 sources)
- [x] 05-academic-papers.md (10 sources)
- [ ] 06-youtube-tech.md (13 videos, processing)
- [ ] 07-hardware-practice.md (~20 videos, processing)

**Topics well-covered:**
- WebAssembly, Component Model, WASI
- FPGA, ASIC, hardware design, 3D-printed electronics
- Agent architectures (MemGPT, memory systems)
- Embedded systems (TockOS, FreeRTOS, Embassy, Zephyr)
- Academic research (Mamba/SSMs, multi-agent, curriculum learning)
- YouTube: RL theory, embedded Rust, embodied AI
- Practical: CNC machining, fabrication

**Topics needing more sources:**
- GitHub repos (42 available, Deep Research can fetch live)
- Remaining blog articles (~50, lower priority)
