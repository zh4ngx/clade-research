# Academic Papers for CLADE

*Last updated: 2026-03-13*

A curated collection of academic papers relevant to Project CLADE's focus on post-POSIX computing, agent-based systems, and next-generation architectures.

---

## State Space Models and Hybrid Architectures

### Mamba: Linear-Time Sequence Modeling with Selective State Spaces

**URL:** https://arxiv.org/abs/2312.00752
**Type:** arxiv (December 2023)

**Summary:**
This foundational paper introduces Mamba, a new neural network architecture based on selective state space models (SSMs). The authors identify that existing subquadratic-time architectures fail at content-based reasoning, a critical weakness. Their solution allows SSM parameters to be functions of input, enabling selective propagation or forgetting of information along sequence dimensions. The hardware-aware parallel algorithm achieves linear scaling in sequence length while maintaining the modeling capabilities previously exclusive to Transformers.

Mamba demonstrates 5x higher inference throughput than Transformers and performs well on sequences up to million-length. The Mamba-3B model outperforms same-sized Transformers and matches models twice its size on language tasks. This architecture has become a cornerstone for efficient long-context modeling and hybrid approaches that combine SSM efficiency with Transformer quality.

**Key Points:**
- Introduces selective state spaces where parameters are input-dependent
- Hardware-aware parallel algorithm in recurrent mode for efficient training
- Linear time complexity O(n) vs Transformer's quadratic O(n^2)
- 5x higher inference throughput than Transformers
- State-of-the-art on language, audio, and genomics modalities
- No attention mechanism or MLP blocks required

**Relevance to CLADE:** Mamba's linear scaling and efficient inference make it highly relevant for post-POSIX systems that need to process long contexts (logs, codebases, documents) without quadratic memory growth. The selective information retention mechanism provides a model for how CLADE agents might manage context windows.

---

### Transformers are SSMs: Generalized Models and Efficient Algorithms Through Structured State Space Duality

**URL:** https://arxiv.org/abs/2405.21060
**Type:** arxiv (ICML 2024)

**Summary:**
This paper establishes theoretical connections between Transformers and state space models through the lens of structured semiseparable matrices. The authors develop the State Space Duality (SSD) framework, showing that these seemingly different architectures are closely related. This unification enables cross-pollination of ideas between the two families of models and provides new algorithmic insights.

The practical outcome is Mamba-2, a refined architecture with a core layer that is 2-8x faster than the original Mamba while maintaining competitive language modeling performance. The SSD framework reveals that attention mechanisms can be viewed through a state space lens, opening new optimization opportunities for both architecture families.

**Key Points:**
- Establishes theoretical duality between Transformers and SSMs
- Introduces structured semiseparable matrix decompositions
- Mamba-2 achieves 2-8x speedup over original Mamba
- Provides unified framework for understanding attention and state spaces
- ICML 2024 publication with significant impact on hybrid architectures

**Relevance to CLADE:** The SSD framework provides theoretical grounding for hybrid architectures that CLADE could leverage. Understanding the mathematical relationship between attention and state spaces enables more principled design of efficient inference systems that don't sacrifice quality.

---

### Jamba: A Hybrid Transformer-Mamba Language Model

**URL:** https://arxiv.org/abs/2403.19887
**Type:** arxiv (March 2024)

**Summary:**
Jamba represents the first production-quality hybrid Transformer-Mamba model, combining blocks from both architectures with mixture-of-experts (MoE) layers. The interleaved design captures benefits of both families: Transformer quality on standard benchmarks and Mamba efficiency for long contexts. The specific configuration fits in a single 80GB GPU while achieving strong performance on 256K token contexts.

The paper provides extensive ablation studies on architectural decisions, including how to combine Transformer and Mamba layers and how to mix experts. These findings are crucial for practitioners building hybrid systems. Jamba's ability to handle extremely long contexts while maintaining reasonable memory footprint makes it practical for real-world deployment.

**Key Points:**
- First hybrid Transformer-Mamba MoE architecture
- Supports 256K token context length
- Fits in single 80GB GPU
- Interleaves Transformer and Mamba blocks strategically
- MoE increases capacity while managing active parameters
- Open weights released under permissive license

**Relevance to CLADE:** Jamba demonstrates that hybrid architectures are practically viable, combining the best of both worlds. For CLADE's agent systems that need both quality reasoning (attention) and efficient long-context handling (SSMs), hybrid approaches like Jamba provide a proven design pattern.

---

### Zamba: A Compact 7B SSM Hybrid Model

**URL:** https://arxiv.org/abs/2405.16712
**Type:** arxiv (May 2024)

**Summary:**
Zamba pioneers a unique hybrid architecture using Mamba as the backbone with a single shared attention module, achieving attention benefits at minimal parameter cost. Trained on 1T tokens, it achieves competitive performance against leading 7B models while being significantly faster at inference and using less memory for long sequence generation.

The two-phase training approach (web data pretraining followed by high-quality instruct/synthetic data annealing) with rapid learning rate decay provides insights into efficient training of hybrid models. Zamba demonstrates that selective attention placement can capture most benefits without the full computational cost of transformer-style attention throughout.

**Key Points:**
- Mamba backbone with single shared attention module
- Best non-transformer model at 7B scale
- Significantly faster inference than comparable transformers
- Lower memory for long sequence generation
- Two-phase training with annealing over quality data
- All checkpoints open-sourced

**Relevance to CLADE:** Zamba's minimal-attention design pattern shows that efficient inference doesn't require abandoning attention entirely. For CLADE systems running on constrained hardware or needing to process many streams concurrently, this architecture provides a practical template.

---

## Multi-Agent Systems and Latent Collaboration

### Latent Collaboration in Multi-Agent Systems

**URL:** https://arxiv.org/abs/2511.20639
**Type:** arxiv (November 2025)

**Summary:**
This paper introduces LatentMAS, a training-free framework enabling LLM agents to collaborate directly in continuous latent space rather than through text-based mediation. Each agent generates auto-regressive latent thoughts through last-layer hidden embeddings, with a shared latent working memory preserving and transferring internal representations for lossless information exchange.

The theoretical analysis establishes that latent collaboration achieves higher expressiveness with substantially lower complexity than text-based multi-agent systems. Empirical evaluation across 9 benchmarks shows up to 14.6% higher accuracy, 70.8%-83.7% reduction in token usage, and 4x-4.3x faster inference. This work fundamentally reconceives multi-agent collaboration by removing the bottleneck of textual communication.

**Key Points:**
- Training-free pure latent collaboration among LLM agents
- Auto-regressive latent thought generation via hidden embeddings
- Shared latent working memory for lossless information exchange
- Up to 14.6% accuracy improvement over text-based MAS
- 70.8%-83.7% reduction in output tokens
- 4x-4.3x faster end-to-end inference
- Open-sourced code and data

**Relevance to CLADE:** Latent collaboration is directly applicable to CLADE's multi-agent architecture. Removing text serialization between agents could dramatically improve coordination efficiency while preserving the rich information in agent representations. This approach aligns with post-POSIX thinking about avoiding unnecessary serialization boundaries.

---

## Training Methods and Curriculum Learning

### Self-Evolving Curriculum for LLM Reasoning

**URL:** https://arxiv.org/abs/2505.14970
**Type:** arxiv (May 2025)

**Summary:**
SEC addresses the challenge of training curriculum in RL-based LLM fine-tuning by formulating problem selection as a non-stationary Multi-Armed Bandit problem. Each problem category (difficulty level or problem type) is treated as an arm, with absolute advantage from policy gradient methods serving as a proxy for immediate learning gain.

The curriculum policy dynamically selects categories to maximize reward and updates using TD(0) method. Experiments across planning, inductive reasoning, and mathematics demonstrate significant improvements in reasoning capabilities and better generalization to harder out-of-distribution problems. SEC also achieves better skill balance when fine-tuning on multiple domains simultaneously.

**Key Points:**
- Formulates curriculum as non-stationary Multi-Armed Bandit
- Uses policy advantage as learning gain proxy
- TD(0) method for curriculum policy updates
- Improves generalization to out-of-distribution problems
- Better skill balance across multiple domains
- No manual curriculum design required

**Relevance to CLADE:** Self-evolving curricula could inform how CLADE agents develop capabilities autonomously. The bandit-based approach to skill acquisition provides a principled method for agents to determine what to learn next, supporting continuous self-improvement.

---

### DUMP: Automated Distribution-Level Curriculum Learning for RL-based LLM Post-training

**URL:** https://arxiv.org/abs/2504.09710
**Type:** arxiv (April 2025)

**Summary:**
DUMP provides a principled curriculum learning framework for RL-based LLM post-training that addresses heterogeneous training data. The core insight is that policy advantage magnitude reflects how much benefit remains from further training on a distribution. Using Upper Confidence Bound (UCB), the framework dynamically adjusts sampling probabilities to balance exploitation (high average advantage) and exploration (low sample count).

When instantiated with GRPO as the underlying RL algorithm, DUMP demonstrates significant improvements in convergence speed and final performance on logic reasoning datasets with varying difficulties and sources. This distribution-aware approach is particularly relevant for modern LLM training that mixes diverse data sources.

**Key Points:**
- Distribution-level learnability based on policy advantages
- UCB principle for dynamic sampling probability adjustment
- Balances exploitation (high advantage) and exploration (low count)
- Works with GRPO and other RL algorithms
- Significantly improves convergence and final performance
- Theoretically grounded training schedule

**Relevance to CLADE:** DUMP's distribution-aware curriculum approach could guide how CLADE systems manage diverse data streams and learning objectives. The UCB-based scheduling provides an elegant solution for prioritizing what to learn in complex multi-domain environments.

---

## Long-Context Processing

### Recursive Language Models

**URL:** https://arxiv.org/abs/2512.24601
**Type:** arxiv (December 2025)

**Summary:**
RLMs treat long prompts as external environments, allowing LLMs to programmatically examine, decompose, and recursively call themselves over prompt snippets. This inference-time scaling approach enables processing inputs two orders of magnitude beyond context windows while dramatically outperforming vanilla frontier LLMs and standard long-context scaffolds on diverse tasks.

The authors post-train RLM-Qwen3-8B, the first natively recursive language model, which outperforms the base model by 28.3% and approaches GPT-5 quality on long-context tasks. The recursive paradigm provides a general inference framework for arbitrary-length inputs at comparable cost to standard approaches.

**Key Points:**
- Treats long prompts as external environment
- Programmatic examination and decomposition
- Recursive self-calls over prompt snippets
- Processes 100x beyond context window limits
- 28.3% improvement over base model
- Approaches GPT-5 quality on long-context tasks
- Comparable cost to standard inference

**Relevance to CLADE:** Recursive processing is essential for CLADE systems handling large codebases, extensive logs, or multi-document contexts. The self-decomposition pattern aligns with hierarchical agent architectures and provides a scalable approach to arbitrary-length inputs.

---

## Data and Capability Shaping

### Shaping Capabilities with Token-Level Data Filtering

**URL:** https://arxiv.org/abs/2601.21571
**Type:** arxiv (January 2026)

**Summary:**
This paper demonstrates that shaping LLM capabilities during pretraining through data filtering is more effective, robust, and scalable than post-hoc approaches. On the proxy task of removing medical capabilities, token-level filtering achieves the same reduction in undesired capabilities at lower cost to benign ones compared to document-level filtering.

Critically, filtering becomes more effective with scale: for the largest models, token filtering creates a 7000x compute slowdown on the forget domain. The methodology uses sparse autoencoders for token labeling and distills cheap, high-quality classifiers. Filtering proves robust to noisy labels given sufficient pretraining compute.

**Key Points:**
- Token-level filtering more effective than document-level
- Filtering effectiveness increases with model scale
- 7000x compute slowdown on forget domain
- Sparse autoencoder methodology for token labeling
- Distilled classifiers for efficient filtering
- Robust to label noise with sufficient compute

**Relevance to CLADE:** Understanding how data shapes capabilities is crucial for CLADE systems that may need to control what agents learn. Token-level filtering provides fine-grained control over capability acquisition during training, enabling more precise system behavior.

---

## Hardware Acceleration

### LightMamba: Efficient Mamba Acceleration on FPGA with Quantization and Hardware Co-design

**URL:** https://arxiv.org/html/2502.15260v2
**Type:** arxiv (February 2025)

**Summary:**
LightMamba presents the first FPGA-based Mamba acceleration framework through co-design of quantization algorithms and accelerator architecture. The rotation-assisted post-training quantization handles scattered activation outliers unique to Mamba, enabling 4-bit quantization with minimal accuracy degradation. Power-of-two SSM quantization reduces re-quantization overhead.

The partially unrolled spatial architecture with computation reordering and fine-grained tiling achieves 4.65-6.06x higher energy efficiency than GPU baselines on Xilinx Versal VCK190. On Alveo U280, LightMamba reaches 93 tokens/s, 1.43x faster than GPU. This work demonstrates that SSM architectures can be efficiently accelerated on FPGAs.

**Key Points:**
- First FPGA-based Mamba acceleration framework
- Rotation-assisted PTQ for 4-bit quantization
- Power-of-two SSM quantization for efficiency
- Partially unrolled spatial architecture
- 4.65-6.06x energy efficiency over GPU
- 1.43x throughput improvement
- Computation reordering improves utilization 58% to 96%

**Relevance to CLADE:** Hardware acceleration is essential for efficient CLADE deployment, especially at the edge or in resource-constrained environments. FPGA-based acceleration provides energy-efficient inference that aligns with post-POSIX goals of minimizing unnecessary abstraction overhead.

---

## Synthesis: Key Themes

### 1. State Space Models as Transformer Alternative
The Mamba lineage (Mamba, Mamba-2, Jamba, Zamba) demonstrates that SSMs can match or exceed Transformer quality while providing linear scaling and efficient inference. Hybrid architectures combine the best of both worlds, using attention selectively for quality and SSMs for efficiency. This is directly relevant to CLADE's need for long-context processing without quadratic memory growth.

### 2. Latent-Space Communication
LatentMAS shows that agent collaboration need not be bottlenecked by text serialization. Direct latent-space communication achieves higher quality, lower cost, and faster execution. For CLADE's multi-agent architecture, this suggests moving away from POSIX-style text pipes toward direct representation sharing.

### 3. Recursive and Inference-Time Scaling
Recursive Language Models demonstrate that inference-time computation can extend context windows by two orders of magnitude. Combined with the insight that scaling laws favor inference compute, this suggests CLADE systems should invest in sophisticated inference strategies rather than just larger models.

### 4. Automated Curriculum and Capability Control
SEC and DUMP show that curriculum learning can be automated through bandit-based approaches and advantage signals. Token-level filtering demonstrates fine-grained control over capability acquisition. These techniques enable CLADE systems to develop and control agent capabilities systematically.

### 5. Hardware-Algorithm Co-Design
LightMamba exemplifies how quantization and architecture must be co-designed for efficient deployment. For CLADE systems targeting diverse hardware (cloud GPUs, edge FPGAs, etc.), algorithm-hardware co-design will be essential for efficiency.

---

## Future Directions for CLADE

Based on these papers, key research directions for CLADE include:

1. **Hybrid SSM-Attention Agents:** Building agents that use SSMs for context management and selective attention for quality-critical operations
2. **Latent-Space Agent Orchestration:** Replacing text-based agent communication with direct latent representation sharing
3. **Recursive Context Processing:** Implementing self-decomposing inference for arbitrary-length inputs
4. **Automated Skill Acquisition:** Using bandit-based curricula for continuous agent improvement
5. **Hardware-Aware Deployment:** Co-designing models and accelerators for efficient CLADE execution

---

*Papers processed: 12*
*Source: ~/dev/clade/links/academic-links.txt*
