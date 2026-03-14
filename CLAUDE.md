# Project CLADE: Context for Claude Code

## Project Overview
CLADE (Post-POSIX Computing) is a research project exploring next-generation computing architectures beyond POSIX. The project involves embedded systems, WebAssembly, agent architectures, FPGAs, and systems programming.

## Directory Structure
```
~/dev/clade/
  links/
    all-links-clean.txt    # Master list of all extracted URLs (260)
    youtube-links.txt      # YouTube videos (16)
    academic-links.txt     # arxiv/papers (43)
    github-links.txt       # GitHub repos (42)
    other-links.txt        # Blogs/articles (99)
  sources/                 # Extracted content, batched by topic
  transcripts/             # YouTube transcripts (after manual extraction)
```

## Link Extraction Status
- Total unique links: ~200
- Source: Google Doc "Project CLADE" Section 5 (Research Index)
- Links extracted and categorized: DONE

## Processing Strategy (Avoid Token Limits)
1. Process links in batches of 10-15
2. Use subagents to fetch/summarize - they write directly to files
3. Main context stays clean - never keep fetched content in context
4. Batch summaries into topic files for NotebookLM (~15-20 files total)

## NotebookLM Integration
- 50 source limit per notebook
- Batch by topic: each .md file = 1 source
- Target: ~20-25 sources total (well under limit)

## Output Format (for each extracted link)
```markdown
## [Title]

**URL:** https://...
**Type:** blog/paper/github/video

### Summary
[2-3 paragraph synthesis]

### Key Points
- Point 1
- Point 2
- Point 3

### Relevance to CLADE
[Why this matters for the project]
```

## Migration Path
1. Now: NotebookLM + Deep Research for synthesis
2. Next: Obsidian (markdown vault) for local access
3. Later: Git-tracked repo for version control and collaboration

## Key Topics in Links
- WebAssembly runtimes (wasmtime, wasmcloud, wasmedge)
- Embedded systems (TockOS, FreeRTOS, Zephyr)
- FPGA/hardware design
- Agent architectures (MemGPT, long-term memory)
- Systems programming (Rust, lean-lang)
