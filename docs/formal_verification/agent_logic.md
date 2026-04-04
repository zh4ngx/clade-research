# Agent Logic: Lean 4 for Agentic Verification

> The "Brain" half of the CLADE Brain-Body split.

## Rationale

CLADE agents require formal verification of their intent. Traditional testing is insufficient when agents autonomously negotiate, bid on tasks, and manage hardware resources. Lean 4 provides dependent types and the Curry-Howard correspondence — proofs as programs, types as propositions.

## Intuitionistic Logic for Agentic Intent

Classical logic assumes every proposition is true or false (law of excluded middle). Intuitionistic (constructive) logic requires a *witness* — a proof must construct the evidence. This is stronger:

- **Protocol compliance:** "Agent A correctly bid on task T" requires constructing the bid according to protocol rules, not just asserting it happened
- **State machine correctness:** Agent state transitions are proven to satisfy their specification, not just tested against examples
- **Resource negotiation:** When agents negotiate for GPU time, the negotiation protocol is a type that can only be satisfied by valid negotiation traces

### What We Can Prove

| Property | Lean 4 Proof Technique |
|----------|----------------------|
| Agent protocol compliance | Dependent types encoding protocol state machines |
| No deadlocks in task bidding | Progress and preservation theorems |
| CRDT merge correctness | Prove merge is commutative, associative, idempotent |
| CA rule safety | Prove attractor basin invariants hold under all inputs |
| Capability non-escalation | Prove agent cannot exceed granted capability set |

## From Haskell to Lean 4

Haskell provides strong static types but no formal proof capability. Key differences:

| | Haskell | Lean 4 |
|---|---|---|
| Types | Hindley-Milner + GADTs | Dependent types |
| Correctness | Type-level + testing | Mathematical proof |
| Memory | GC | Perseus (region-based) |
| Compilation | Native via GHC | C/C++, then to Wasm |
| Proof | Property testing (QuickCheck) | Full formal verification |

## The WASI Semantic Boundary

The interface between Lean 4 agent logic and Rust kernel/hardware is mediated by Wasm Component Model `.wit` files:

```
Lean 4 Agent (verified)
    │
    ▼ wit-bindgen (auto-generated bindings)
    │
Wasm Isolation Layer (capability boundary)
    │
    ▼ wit-bindgen (auto-generated bindings)
    │
Rust Kernel/HDL (safe execution)
```

**Critical rule:** All bindings must be generated via `wit-bindgen` (or Lean equivalent). Manual binding code is forbidden — AI coding agents must act as compiler orchestrators, not binding authors. This prevents hallucination-driven interface mismatches.

## CA Rule Verification

The cellular automata lattice (see `insights.md` Theme 2) requires verified transition rules. Lean 4 provides:

1. **Specify** the attractor basin invariants as Lean 4 types
2. **Prove** that CA transition rules preserve these invariants
3. **Extract** verified rules to Rust-HDL for FPGA synthesis

This pipeline means the BSCA fabric runs mathematically proven rules, not empirically tuned parameters.

### Verification Targets

- E/I balance maintenance under perturbation (Theme 4, noise-tolerant dynamics)
- Latent weight accumulation thresholds preserve E/I ratio
- STDP learning rules don't destabilize fixed recurrent reservoir
- Attractor basins are deep enough to survive thermal bit flips (SRAM noise)

## Toolchain (Nix Flake)

```nix
# In the CLADE flake.nix
lean-toolchain = pkgs.stdenv.mkDerivation {
  # elan (Lean 4 version manager)
  # lake (Lean 4 build system)
  # wit-bindgen (Wasm Component Model bindings)
  # wasmtime (Wasm runtime for testing)
};
```

Reproducible environment ensures AI agents operate in a stable, sandboxed execution context.

## Open Questions

- Lean 4 → C → Wasm compilation path maturity and overhead?
- wit-bindgen for Lean 4: exists or needs building?
- Can Lean 4 verify temporal properties (WCET) or only functional correctness?
- Proof effort vs testing for CA rules — what's the crossover point?
- How does Perseus memory management interact with Wasm linear memory?

## References

- `architecture.md` — CLADE system design
- `insights.md` — FPGA substrate, CA dynamics, BSCA paradigm
- Curry-Howard correspondence — proofs as programs, types as propositions
- WASI Component Model — `.wit` interface definitions
