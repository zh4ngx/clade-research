---
date: 2026-04-18
tags: [ml, reinforcement-learning, policy-optimization, credit-assignment, sparse-reward, LLM]
aliases: [TPO, Target Policy Optimization]
---

# Target Policy Optimization (TPO)

Related: [wasm-credit-assignment-replay](wasm-credit-assignment-replay.md), [surprise-driven-bootstrapping](surprise-driven-bootstrapping.md), [pulse-feedback](pulse-feedback.md), [bsca-fabric](bsca-fabric.md), [self-distillation-context-compression](self-distillation-context-compression.md)

Source: [Kaddour et al., arXiv 2604.06159](https://arxiv.org/abs/2604.06159)

## Core Idea

Decouple "what should gain probability" from "how do we move parameters to achieve that." Given K scored candidates per prompt:

1. Build a target distribution $q_i \propto p_i^{\text{old}} \exp(u_i)$ where $u_i$ are standardized scores
2. Fit the policy to $q$ by cross-entropy
3. Gradient vanishes when policy matches target: $\nabla_\ell \mathcal{L} = p^\theta - q$

This is ~10 lines of code (JAX or PyTorch). Replaces the clipped surrogate in GRPO/PPO entirely.

## Why It Works

**Self-extinguishing gradient.** TPO's gradient goes to zero as the policy converges to the target. GRPO's gradient persists even after error plateaus — the policy keeps churning.

**Concentrates signal on informative groups.** Under sparse reward, ~90% of groups have all-failing candidates at initialization. TPO treats these as neutral ($q = p^{\text{old}}$, zero loss). Only groups with actual signal get update budget. GRPO wastes gradient on noise.

**Allocates budget to hard contexts.** On the multi-context bandit, TPO's update weight $\beta_{\text{TPO}}(p_n)$ stays large even when $p_n$ is small. GRPO and DG both vanish as $p_n \to 0$ — they spend budget on already-easy contexts. TPO is closer to the optimal cross-entropy oracle that weights all contexts equally.

**Stable multi-epoch reuse.** TPO's fixed target supports multiple gradient epochs on the same batch without trust-region issues. DG diverges with >1 epoch. GRPO is sensitive to epoch count (non-monotonic: 2 epochs worse than 1 or 4).

## Key Results

| Setting | TPO vs Baselines |
|---------|-----------------|
| Dense reward (MNIST bandit) | Matches baselines, slightly faster convergence |
| Token reversal (dense) | Matches GRPO at small vocab, 2-6x faster at large vocab |
| Sequential reward | GRPO and PPO fail to converge; only TPO and DG converge |
| Terminal reward (sparse) | TPO dominates — GRPO/PPO degrade steeply with sequence length, TPO degrades gracefully |
| LLM RLVR (1.5-1.7B) | Matches GRPO on GSM8K; GRPO fails entirely on graph coloring while TPO reaches 0.96 |

The pattern: TPO matches on easy tasks and substantially outperforms on sparse reward. Sparse reward is exactly the regime that matters for long-horizon agentic tasks.

## Relationship to CLADE

### Credit Assignment (Direct Connection)

The central problem in [wasm-credit-assignment-replay](wasm-credit-assignment-replay.md) — "when the system fails, which component gets the blame?" — is a sparse credit assignment problem. TPO's insight that constructing an explicit target and fitting to it outperforms scalar-weighted updates has a direct structural parallel:

- **Current CLADE approach:** Counterfactual replay via WASM snapshots — fork at decision points, override components, compare outcomes. This is MCTS over component decisions.
- **TPO's structural insight:** Instead of MCTS (which is expensive — $5^{10}$ branches), construct a target distribution over what the correct component activation should have been, then fit components to that target.

**Possible synthesis:** Use TPO-style target matching *within* each component's update, while keeping the WASM branching replay for *between*-component credit assignment. The replay identifies which component failed; TPO updates that component's policy more efficiently than scalar REINFORCE.

### Surprise-Driven Bootstrapping

[surprise-driven-bootstrapping](surprise-driven-bootstrapping.md) uses prediction error (surprise) as both the retrieval trigger and the write signal. TPO's score standardization is doing something analogous — it normalizes reward signals so the update depends on relative performance, not absolute magnitude.

The bootstrapping curriculum (Phase 0: no memory → Phase 3: write decisions) creates increasingly sparse reward. Early phases have dense feedback (supervised labels); later phases rely on delayed, sparse signals (task completion). TPO's advantage grows precisely as reward becomes sparser — it may be the right update rule for later bootstrapping phases.

### Pulse Feedback

[pulse-feedback](pulse-feedback.md) argues feedback should be token sequences, not scalars. TPO's target distribution $q$ is richer than a scalar reward — it's a full distribution over candidates that encodes not just "which is better" but "how much better, relative to the old policy." This is a step toward structured feedback: $q$ is a *message* about desired behavior, not just a magnitude.

### BSCA Fabric

The BSCA [bsca-fabric](bsca-fabric.md) uses STDP for learning — a local, pairwise, 1-bit update rule. TPO is a global, group-level update. They operate at different scales. But the principle is shared: both construct a target (STDP's causality window → strengthen/weaken; TPO's exponential tilt → target distribution) and move toward it.

## Practical Takeaways

- **If training CLADE components with RL:** Use TPO instead of GRPO/PPO for the gate policy. The sparse reward regime (task completion after many decisions) is exactly where TPO shines.
- **Temperature is robust:** $\eta = 1$ works across a 16x range. No tuning needed.
- **Group size matters:** Larger K (more candidates per prompt) helps TPO monotonically. For the BSCA, "candidates" could be different gate activation patterns sampled from the lattice.
- **No critic needed:** TPO doesn't require a value function. This is important for the BSCA fabric — there's no obvious way to train a critic on spike patterns, but TPO only needs scores.

## Limitations Noted by Authors

- Can only redistribute over sampled candidates — low-diversity samples yield uninformative targets
- Score standardization amplifies tiny differences in low-variance groups (same difficulty-bias as GRPO)
- Only tested up to 1.7B parameters; unclear if gains persist at 7B+
- Same rollout cost as GRPO (K rollouts per prompt) — better signal extraction, not cheaper computation

---
#ml #reinforcement-learning #policy-optimization #credit-assignment
