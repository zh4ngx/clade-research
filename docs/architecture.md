# CLADE Architecture

> Evolving design document. Starts with questions, becomes decisions.

## Core Principles

- **WAN latency is a feature, not a bug** — design forces resilient async protocols
- **Emergence over size** — intelligence from coordination, not parameter count
- **Local cognition, global coordination** — each node is autonomous, swarm is collective
- **Capability-based security** — explicit permissions, no implicit access
- **Hardware-agnostic via Wasm** — agents migrate freely across substrate

## Fundamental Questions

### What is the unit of computation?

- POSIX: Process
- CLADE: Wasm module / ephemeral agent (not process-bound, hardware-agnostic)

### How do components communicate?

- POSIX: Byte streams (pipes, sockets, files)
- CLADE: Asynchronous A2A negotiation over Tailscale mesh (WAN-latency-aware)

### How is security enforced?

- POSIX: Users, permissions, syscalls
- CLADE: Capability-based security via Wasm (explicit capabilities, not implicit access)

### How is memory managed?

- POSIX: Virtual memory, paging
- CLADE: TBD — see insights on active vs passive memory, model-managed load/store

## Design Sketches

### Distributed Swarm Architecture & WAN Topologies (2026-03-25)

#### Hardware & Network Constraints
- **Infrastructure:** 4x physically distributed machines, each equipped with consumer AMD GPUs (e.g., 9000 series) with 8-16GB VRAM.
- **Networking:** Connected over WAN via Tailscale mesh.
- **The Latency Reality:** Attempting to pool VRAM over WAN for a single large model (e.g., 70B parameter) using `llama.cpp` layer-by-layer RPC offloading is unviable. The physics of internet latency crush the throughput required for tensor calculations.

#### Core Paradigm Shift
- **Discard:** The "Single Brain" monolithic agent (Aider/Claude Code style) running complex, full-repo reasoning across a fractured network.
- **Adopt:** An emergent, multi-agent swarm architecture.
- **Project Scope & Nomenclature:** CLADE operates as an umbrella framework. Because CLADE represents multiple acronyms and encompasses several distinct concepts, the system architecture must inherently support this multiplicity. Rather than a single monolithic program, CLADE should be structured as an ecosystem where different agent clusters and nodes can embody these various definitions and interacting concepts.

#### Local Cognitive Nodes (The "Engines")
- **Role:** Each physical machine acts as an independent, high-speed cognitive engine for the agents currently residing on it.
- **Inference Stack:** Run heavily quantized 8B-14B models (e.g., Qwen 2.5 Coder 14B, Llama 3 8B) locally on each node. Use the **Vulkan** backend—it offers pragmatic, out-of-the-box stability for consumer AMD cards (bypassing ROCm brittleness), easily managed via declarative OS configurations like NixOS.
- **Function:** These local LLMs do not need full repository context. They process micro-decisions, evaluate local state, and format structured data for the transient agents passing through.

#### Wasm Execution Layer (The "Swarm")
- **Portability:** Compile individual agent logic, tools, and negotiation protocols into WebAssembly (Wasm) modules.
- **Agility:** Wasm provides near-zero cold start times and capability-based security. This allows for the rapid spawning of thousands of ephemeral micro-agents that can be easily migrated or executed across any of the Tailscale nodes, completely hardware-agnostic.
- **Integration:** Utilize WIT (Wasm Interface Type) and WebGPU standards to standardize how these portable agents interact with the underlying hardware and local inference engines on whatever node they land on.

#### Agent-to-Agent (A2A) Negotiation
- **WAN as a Feature:** The network latency of Tailscale over WAN forces the design of highly resilient, asynchronous communication protocols.
- **Behavior:** Agents must not rely on instantaneous, centralized compute. Instead, they communicate via the Tailscale mesh to:
  - Broadcast local state.
  - Bid on discrete tasks (e.g., "Node B has available VRAM, claiming the test-generation task").
  - Pass state and context securely across the network.
- **Emergence:** The true intelligence of CLADE won't come from a single massive parameter count, but from the complex, networked negotiation and emergent behavior of these smaller, highly specialized Wasm agents interacting across the topology.

## Decisions Log

| Date | Decision | Rationale |
|------|----------|-----------|
| 2026-03-25 | Swarm architecture over monolithic agent | WAN latency makes single-brain approach unviable; emergence from many small agents |
| 2026-03-25 | Wasm for agent execution layer | Near-zero cold start, capability-based security, hardware-agnostic portability |
| 2026-03-25 | Vulkan backend for AMD GPUs | ROCm too brittle; Vulkan offers pragmatic stability via NixOS |
| 2026-03-25 | 8B-14B quantized models per node | Local inference doesn't need full repo context; micro-decisions only |

## References

- `insights.md` - Research foundation
- [clade-research](https://github.com/zh4ngx/clade-research) - Raw sources
