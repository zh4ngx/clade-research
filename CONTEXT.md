# Project CLADE

## Overview
CLADE (Post-POSIX Computing) is a research project exploring next-generation computing architectures beyond POSIX. Key areas include embedded systems, WebAssembly, agent architectures, FPGAs, and systems programming.

## Project Goals
- Research and document post-POSIX computing paradigms
- Curate a knowledge base of relevant sources
- Support deep research synthesis via NotebookLM and other AI tools
- Maintain a portable, version-controlled knowledge repository

## Directory Structure
```
links/           # Extracted URLs, categorized by type
  youtube-links.txt      (16 videos)
  academic-links.txt     (43 papers/arxiv)
  github-links.txt       (42 repositories)
  other-links.txt        (99 blogs/articles)

sources/         # Extracted content, batched by topic
transcripts/     # YouTube transcripts
```

## Data Sources
- Primary: Google Doc "Project CLADE" (~1600 pages)
- Section 5 "Research Index" contains curated links
- Embedded content includes email threads and AI conversations

## Processing Workflow
1. Links extracted and categorized
2. Content fetched and summarized in batches
3. Summaries organized by topic into markdown files
4. Ready for NotebookLM ingestion (staying under 50 source limit)

## Output Format
Each extracted source includes:
- Title and URL
- Content type (blog/paper/repo/video)
- Summary (2-3 paragraphs)
- Key points (bulleted)
- Relevance to CLADE project

## Toolchain
- **Claude Code**: Implementation and content extraction
- **NotebookLM + Deep Research**: Synthesis and Q&A
- **Obsidian**: Local markdown access (future)
- **Git**: Version control and collaboration (future)

## Key Research Topics
- WebAssembly runtimes and component model
- Embedded operating systems (TockOS, FreeRTOS, Zephyr)
- FPGA and hardware description
- AI agent memory architectures (MemGPT)
- Verified systems programming (Rust, Lean)

## Related Projects
- Google Anti-gravity: Project design and research (planned)
- This repo: Code execution and content management

---
*This file provides tool-agnostic context for any AI assistant or human working on this project.*
