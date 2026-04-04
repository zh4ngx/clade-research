# Kernel Boundaries: The Rust-Lean 4 Split

> Source of truth for the Brain-Body architecture boundary.

## The Split

CLADE's software architecture partitions into two logically distinct systems communicating through a Wasm isolation layer:

```
┌─────────────────────────────────────────────────┐
│                  Lean 4 "Brain"                  │
│   Agent logic, protocol verification, CA rules  │
│   Intuitionistic logic (constructive proofs)    │
│   Compiles to C/C++ → Wasm guest               │
└────────────────────┬────────────────────────────┘
                     │ .wit interface (WASI)
                     │ wit-bindgen (auto-generated)
┌────────────────────▼────────────────────────────┐
│                   Wasm Runtime                    │
│   Capability-based isolation boundary            │
│   Only explicitly granted capabilities accessible│
└────────────────────┬────────────────────────────┘
                     │ .wit interface
                     │ wit-bindgen (auto-generated)
┌────────────────────▼────────────────────────────┐
│                  Rust "Body"                      │
│   K23 microkernel, drivers, hardware access     │
│   Affine logic (ownership/borrowing)            │
│   Deterministic WCET for safety interrupts      │
└─────────────────────────────────────────────────┘
```

## Logic Mapping

| Concept | Lean 4 (Brain) | Rust (Body) |
|---------|-----------------|------------- |
| **Logic system** | Intuitionistic (constructive) | Affine (use at most once) |
| **What it guarantees** | Protocol intent correctness | Resource safety |
| **Proof mechanism** | Curry-Howard: types = theorems | Ownership/borrowing at compile time |
| **Memory model** | Perseus (region-based) | Ownership + borrow checker |
| **Error handling** | Proof elimination (impossible states unrepresentable) | Result types, no panics in kernel |
| **Compilation target** | C/C++ → Wasm guest | Native (kernel) or Rust-HDL (FPGA) |

## The WASI Contract

`.wit` files define the interface between Brain and Body. This is the only communication channel:

- **No direct function calls** between Lean 4 and Rust
- **No shared memory** — data passes through Wasm linear memory via the runtime
- **Capability tokens** — the Rust body grants specific capabilities to Lean 4 agents at instantiation
- **Auto-generated bindings** — `wit-bindgen` generates both sides; manual bindings forbidden

### Example Interface

```wit
interface agent-scheduler {
  /// Request a time slice on the local inference engine
  request-inference: func(model-id: string, tokens: list<u8>) -> result<inference-handle, schedule-error>;

  /// Bid on a distributed task via the swarm
  bid-task: func(task-id: string, capability: capability-set) -> result<bid-receipt, bid-error>;

  /// Report local state to the swarm
  report-state: func(state: node-state) -> result<_, sync-error>;
}
```

The Lean 4 agent implements this interface with verified logic. The Rust kernel provides the actual hardware-backed implementation.

## Rust Kernel: The K23 Microkernel

The "Body" runs on a minimal microkernel design:

- **Deterministic interrupts** — bounded WCET for all safety-critical paths
- **Driver isolation** — each hardware driver is a separate capability domain
- **No dynamic allocation in kernel path** — all memory pre-allocated, no heap in interrupt handlers
- **Affine types** — Rust's ownership system ensures each hardware resource (GPU, FPGA, network) is accessed by exactly one agent at a time

### Safety Architecture

```
┌──────────────────────┐
│   Safety-Critical    │ ← Hard real-time, verified WCET
│   (Interrupts, AFE)  │ ← No Wasm, no heap, no dynamic dispatch
├──────────────────────┤
│   Kernel Services    │ ← Capability-granted access
│   (Drivers, Memory)  │ ← Pre-allocated resources only
├──────────────────────┤
│   Wasm Runtime       │ ← Agent sandbox boundary
│   (Lean 4 agents)    │ ← All agent logic lives here
└──────────────────────┘
```

## CA Lattice → FPGA Pipeline

The cellular automata lattice connects Brain and Body through verified synthesis:

1. **Lean 4**: Specify and prove CA transition rules satisfy attractor basin invariants
2. **Lean 4 → C**: Extract verified rules as C code
3. **C → Rust-HDL**: Synthesize to FPGA bitstream via Amaranth or equivalent
4. **Rust kernel**: Load verified bitstream to FPGA, manage DMA access

The Rust kernel treats the FPGA as a hardware accelerator with verified logic. The Lean 4 proofs guarantee the logic is correct before it reaches hardware.

## Hardware Specifications (1-bit CA Lattice)

Updated to reflect Lean 4 verification requirements:

| Component | Specification | Verification |
|-----------|--------------|--------------|
| LUT6 truth tables | 6-input Boolean functions per cell | Lean 4: proven safe under all $2^6$ input combinations |
| BRAM latent weights | Integer accumulators for STDP | Lean 4: proven to maintain E/I balance threshold |
| Connectivity K=4 | Critical edge-of-chaos wiring | Lean 4: proven to avoid phase transition to order/chaos |
| Attractor basins | Deep basins for noise tolerance | Lean 4: proven to survive N-bit perturbations |
| DFF state registers | Recurrent memory per cell | Rust: verified DMA access, no concurrent modification |

## References

- `architecture.md` — CLADE system design
- `insights.md` — FPGA substrate, CA dynamics, temporal scaling
- `formal_verification/agent_logic.md` — Lean 4 verification details
