# WASM-Based Counterfactual Credit Assignment

> Deterministic branching replay solves credit assignment without end-to-end gradients.

Companion to [surprise-driven-bootstrapping.md](surprise-driven-bootstrapping.md), [heterogeneous-cognitive-architecture.md](heterogeneous-cognitive-architecture.md), and [snn-gate-and-memory-design.md](snn-gate-and-memory-design.md). Source: CC deep-dive on WASM VM-based credit assignment.

## The Problem

When the heterogeneous system fails a task, three components could be at fault (SSM, gate, memory) — each learning at different timescales. End-to-end backprop is abandoned at SNN/memory boundaries.

## Deterministic Branching Replay

Fork WASM snapshots at each decision point, override one component, replay:

| Branch | Override | Result | Credit |
|--------|----------|--------|--------|
| A | Force "idle" at t=5 | Still wrong | Memory not the problem |
| B | Force "retrieve" + correct memory | Correct | Memory retrieval was wrong |
| C | Force "retrieve" at t=12 | Correct | Gate decision was wrong |
| D | Correct memory + force "generate" | Correct | Both contributed |

Branch that improves outcome most identifies the failing component. No gradients needed across paradigm boundaries.

## Why WASM

1. **Deterministic by specification** — identical linear memory + imports → byte-identical execution. No ambient randomness, no hidden state.
2. **Snapshot is memcpy** — linear memory is a contiguous byte array. Full snapshot = copying that buffer.
3. **Sandboxed side effects** — modules interact only through explicit imports. Branches can't mutate the real world.

```
let base_snapshot = vm.snapshot();
let mut branch = base_snapshot.fork();
branch.inject_gate_decision(t=5, "idle");
branch.replay_from(t=5);
let outcome = branch.evaluate();
if outcome > original_outcome {
    credit(t=5, gate_component, positive);
}
```

## MCTS Over Component Decisions

Monte Carlo Tree Search where "moves" are internal component decisions:

```
                  ┌─────────────────┐
                  │  Root snapshot   │
                  └────────┬────────┘
                           │
            ┌──────────────┼──────────────┐
      gate=generate   gate=retrieve   gate=write
            │              │              │
       outcome=bad    outcome=good   outcome=neutral
            │
       ┌───┼───┐
    mem=m₁ mem=m₂ mem=m₃
     bad   good   bad
```

Each node is a WASM snapshot. Edges are component decision overrides. Leaf evaluations are task outcomes. Tree structure directly encodes credit.

## WASM as Agentic OS Kernel

| Role | WASM Mechanism |
|------|---------------|
| Process isolation | Each component is a WASM module |
| IPC | Shared linear memory or host message passing |
| Checkpoint | Snapshot all module memories |
| Time travel | Fork snapshot, inject, replay |
| GPU access | Host import for GPU dispatch |
| Side-effect control | Branches run without host imports |

```
┌─────────────────────────────────────────────┐
│              WASM AGENTIC VM                │
│  ┌─────────┐  ┌──────────┐  ┌───────────┐ │
│  │ SSM     │  │ Memory   │  │ SNN Gate  │ │
│  │ (WASM)  │  │ (WASM)   │  │ (WASM)   │ │
│  └────┬────┘  └────┬─────┘  └─────┬─────┘ │
│       └────────────┼──────────────┘         │
│              ┌─────▼──────┐                 │
│              │  Snapshot   │                 │
│              │  Scheduler  │                 │
│              └─────┬──────┘                 │
│     ┌──────────────┼──────────────┐         │
│  [inference]   [checkpoint]   [replay]     │
│  GPU dispatch  memcpy(mem)  fork+inject     │
└─────────────────────────────────────────────┘
```

## Practical Constraints

- **Snapshot granularity:** At gate decision points only (every N tokens). ~500 MB per episode (10 decisions × 50 MB). Manageable in RAM.
- **GPU interaction:** SSM on GPU (Vulkan). WASM holds hidden state, dispatches inference. Each branch requires a real GPU forward pass.
- **Branching factor:** 5^10 full tree. MCTS pruning: 50-100 branches per episode sufficient.
- **When to replay:** Offline, between episodes. Sleep-phase consolidation — hippocampus replays daily events during sleep.

## Novel Research Claim

Deterministic counterfactual credit assignment across non-differentiable component boundaries. Nobody does this because:
1. Transformer people have end-to-end gradients
2. Modular/neurosymbolic people lack deterministic replay substrates
3. WASM-for-ML people haven't connected snapshotting to credit assignment

Testable: "A heterogeneous neural architecture with non-differentiable components learns effectively via WASM branching replay, achieving comparable or better sample efficiency than end-to-end training on long-horizon agentic tasks."

## WASI Stack

- WASI-NN for SSM inference host import
- WASI-futures for async GPU dispatch
- Component Model for typed interfaces between modules
- SSM on GPU via Vulkan, WASM orchestrator on CPU, snapshots in RAM
