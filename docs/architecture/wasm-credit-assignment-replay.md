# WASM-Based Counterfactual Credit Assignment

Related: [surprise-driven-bootstrapping](surprise-driven-bootstrapping.md), [heterogeneous-cognitive-architecture](heterogeneous-cognitive-architecture.md), [snn-gate-and-memory-design](snn-gate-and-memory-design.md), [pulse-feedback](pulse-feedback.md), [tpo-target-policy-optimization](tpo-target-policy-optimization.md)

Deterministic branching replay via WASM solves the cross-timescale credit assignment problem without end-to-end gradients. This is the bridge between the heterogeneous neural architecture and the CLADE agentic OS.

Source: CC deep-dive on credit assignment, prompted by WASM VM discussion.

## The Problem

When the heterogeneous system fails a task, three components could be at fault (SSM encoding, gate timing, memory retrieval) — each learning at different timescales. End-to-end backprop would solve this but is abandoned at SNN/memory boundaries.

## Deterministic Branching Replay

With WASM snapshots, you fork at each decision point and override one component:

```
Episode trace:
  t=1  SSM observes token → gate fires "generate" → output token
  t=5  SSM observes token → gate fires "retrieve" → memory returns m₁ → ...
  t=12 SSM observes token → gate fires "generate" → WRONG OUTPUT
```

| Branch | Override | Result | Credit |
|--------|----------|--------|--------|
| A | t=5: force "idle" instead of "retrieve" | Still wrong | Memory wasn't the problem |
| B | t=5: force "retrieve" but inject correct memory | Correct | Memory retrieval was wrong |
| C | t=12: force "retrieve" instead of "generate" | Correct | Gate decision was wrong |
| D | t=5: correct memory + t=12: force "generate" | Correct | Both contributed |

The branch that improves outcome the most identifies the failing component. No gradients needed across paradigm boundaries.

## Why WASM Specifically

Three properties make this work:

1. **Deterministic by specification.** Given identical linear memory + imports, WASM execution is byte-identical across all implementations. No ambient randomness, no syscalls, no hidden state. Replay is guaranteed to reproduce the same trace.

2. **Snapshot is memcpy.** WASM linear memory is a contiguous byte array. A full system snapshot is copying that buffer — no serializing Python objects or PyTorch states.

```
// Pseudocode: branching replay
let base_snapshot = vm.snapshot(); // copy ~100MB linear memory

let mut branch = base_snapshot.fork();
branch.inject_gate_decision(t=5, "idle");
branch.replay_from(t=5);
let outcome = branch.evaluate();

if outcome > original_outcome {
    credit(t=5, gate_component, positive);
}
```

3. **Sandboxed side effects.** WASM modules only interact with the host through explicit imports. Branches can't mutate the real world — no file writes, no network calls, no GPU dispatches during replay. Each branch is hermetic.

## The MCTS Connection

This is Monte Carlo Tree Search where "moves" are internal component decisions:

```
                  ┌─────────────────┐
                  │  Root snapshot   │  (episode start)
                  └────────┬────────┘
                           │
            ┌──────────────┼──────────────┐
            │              │              │
      gate=generate   gate=retrieve   gate=write
            │              │              │
       [replay fwd]   [replay fwd]   [replay fwd]
            │              │              │
       outcome=bad    outcome=good   outcome=neutral
            │
       [expand]
       ┌───┼───┐
    mem=m₁ mem=m₂ mem=m₃
       │     │     │
     bad   good   bad
```

Each node is a WASM snapshot. Each edge is a component decision override. Leaf evaluations are task outcomes. The tree structure directly encodes credit: if expanding "gate=retrieve" at t=5 leads to good outcomes regardless of downstream decisions, credit goes to the gate's t=5 decision.

## WASM as Agentic OS Kernel

| Role | WASM Mechanism |
|------|---------------|
| Process isolation | Each component (SSM, memory, gate) is a WASM module |
| IPC | Shared linear memory or message passing via host |
| Checkpoint | Snapshot all module memories |
| Time travel | Fork snapshot, inject modification, replay |
| GPU access | Host import — WASM module calls into GPU runtime |
| Side-effect control | Branches run without host imports (pure replay) |

```
┌─────────────────────────────────────────────┐
│              WASM AGENTIC VM                │
│                                             │
│  ┌─────────┐  ┌──────────┐  ┌───────────┐ │
│  │ SSM     │  │ Memory   │  │ SNN Gate  │ │
│  │ module  │  │ module   │  │ module    │ │
│  │ (WASM)  │  │ (WASM)   │  │ (WASM)   │ │
│  └────┬────┘  └────┬─────┘  └─────┬─────┘ │
│       │            │              │         │
│       └────────────┼──────────────┘         │
│                    │                        │
│              ┌─────▼──────┐                 │
│              │  Snapshot   │                 │
│              │  Scheduler  │                 │
│              │  (host)     │                 │
│              └─────┬──────┘                 │
│                    │                        │
│     ┌──────────────┼──────────────┐         │
│     │              │              │         │
│  [inference]   [checkpoint]   [replay]     │
│     │              │              │         │
│  GPU dispatch  memcpy(mem)  fork+inject     │
└─────────────────────────────────────────────┘
```

## Practical Constraints

**Snapshot granularity:** Snapshot at SNN gate decision points only (every N tokens), not every token. A 3B SSM's hidden state is ~50 MB per layer. At 10 gate decisions per episode, ~500 MB of snapshots. Manageable in RAM.

**GPU interaction:** SSM runs on GPU (Vulkan), not in WASM. The WASM module holds hidden state and dispatches inference to host. During replay, host re-runs SSM forward pass from branch point — the expensive part. Each branch requires a real GPU forward pass.

**Branching factor:** 5 gate outputs × ~10 decision points = 5^10 ≈ 10M full tree. Use MCTS pruning: expand top-k branches by UCT score. 50-100 branches per episode is enough for credit assignment.

**When to replay:** Offline, after episode completion. During inference, just record the trace. Replay happens between episodes — like sleep-phase consolidation. The hippocampus replays the day's events during sleep to consolidate memories and assign credit.

## The Novel Research Claim

Deterministic counterfactual credit assignment across non-differentiable component boundaries. Nobody does this because:

1. Transformer people have end-to-end gradients (no need)
2. Modular/neurosymbolic people don't have deterministic replay substrates
3. WASM-for-ML people haven't connected snapshotting to credit assignment

**Testable claim:** A heterogeneous neural architecture with non-differentiable components can learn effectively using WASM-based branching replay for credit assignment, achieving comparable or better sample efficiency than end-to-end training on long-horizon agentic tasks.

## Connection to WASI Stack

- **WASI-NN** for SSM inference host import
- **WASI-futures** or custom host imports for async GPU dispatch
- **Component Model** for typed interfaces between SSM/memory/gate modules
- SSM runs on RX 7900 XTX via Vulkan, WASM orchestrator on CPU, snapshots in RAM

The agentic OS becomes a portable WASM binary with a GPU host interface.

---
#clade #wasm #credit-assignment #replay #mcts #agentic-os #architecture
