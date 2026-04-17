# Hierarchical Control Architecture (Kimi Synthesis)

> Not pipeline, not coupled oscillator — hierarchical control with inhibitory gating.

Third architectural view, from Kimi gap analysis. Companion to [heterogeneous-cognitive-architecture.md](heterogeneous-cognitive-architecture.md), [coupled-dynamical-architecture.md](coupled-dynamical-architecture.md), [surprise-driven-bootstrapping.md](surprise-driven-bootstrapping.md), and [wasm-credit-assignment-replay.md](wasm-credit-assignment-replay.md).

## 1. Inhibitory Gating, Not Excitatory

Basal ganglia default to "do nothing" (globus pallidus tonic inhibition). Pathways release inhibition to allow action.

```
Wrong:  Neuron A fires  → generate
Right:  All neurons suppress → Neuron A stops → release generate
```

More robust: failure = do nothing, not random action. Maps to BSCA attractor dynamics. Eliminates priority-ordering hack.

## 2. Three Memory Systems in One Lattice

| Type | Content | BSCA Encoding |
|------|---------|--------------|
| Episodic | Events | Transient attractor basins (decay) |
| Semantic | Facts | Stable fixed points (STDP-consolidated) |
| Procedural | Skills | Spatiotemporal patterns (tool sequences) |

Procedural memory can't be retrieved as a vector. Tool execution = collapse into tool-use attractor basin. BSCA fabric IS the procedural substrate.

## 3. Modulation Budget

SNN modulation of SSM parameters bounded: `|Δ_t - Δ_0| < ε`. Budget regenerates slowly (homeostasis). SSM degrades gracefully if BSCA enters epileptic oscillation.

## 4. Self-Supervised Surprise Mining

Training data from existing corpora:
1. Train base SSM on next-token prediction
2. Mask random spans → predict masked content
3. High error → synthetic "surprise"
4. Train gate: "would retrieval help?"
5. Memory: store masked spans, retrieve on similar context

No human labels. Surprise is synthetic.

## 5. The Amnesia Task (Evaluation)

100-step task. Info at steps 10/30/50/70. Query at step 80 about step 20. Baselines: Transformer 4K context (cheating), Transformer 128 context (fair). Success: match 4K accuracy at <5% KV cache on wall-clock time.

## 6. Soft VQ with Residual

Hard VQ wastes bits (13.3 bits from 12,288-bit state). Soft VQ-VAE: BSCA receives discrete code + continuous residual (uncertainty, valence).

## 7. WebGPU Sparse Ops

CA updates are sparse. WebGPU: sparse matrix formats, indirect dispatch, workgroup barriers. 4096 cells at 1% activity → 40 active cells → single workgroup. Same code for research (browser) and deployment (native). Higher abstraction wins here because sparse workloads match WebGPU's dispatch model better than raw Vulkan compute (dense workloads → close-to-metal wins, sparse workloads → indirect dispatch wins).

## Hybrid Architecture

```
SENSORY (GPU) → Soft VQ → BSCA FABRIC (memory + CNS + procedural)
                              ↑↓
                    Inhibitory gating, modulation budget
                              ↓
                    LEARNING (surprise + STDP + WASM replay)
```

## What's Still Open

- LTCN vs SSM for sensory layer
- Successor Representations vs surprise-driven credit assignment
- Hard VQ vs soft VQ
- WebGPU sparse vs Vulkan dense
