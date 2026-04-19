# Program as Weights: Compiling Logic into Neural Networks

> Compiling programmatic behavior directly into the neural network's weights.

Companion to [self-distillation-context-compression.md](self-distillation-context-compression.md), [architecture.md](../architecture.md), and [heterogeneous-cognitive-architecture.md](heterogeneous-cognitive-architecture.md).

## The Concept

The "Program as Weights" paradigm (spearheaded by researchers like Yuntian Deng at Waterloo/Harvard, detailed at [programasweights.com](https://programasweights.com)) proposes a fundamental shift in how we build complex AI systems and agents. 

Instead of writing traditional, symbolic code to handle complex or "fuzzy" logic and relying on the LLM merely as a text-generation module, this paradigm advocates for **compiling programmatic behavior directly into the neural network's weights.**

## Key Tenets

1. **Programming Without Code:** Moving away from code as the intermediary. Developers define the desired behavior, constraints, and logic of a function, which is then internalized into the model's weights during a compilation (training/distillation) phase.
2. **Implicit Reasoning (Implicit CoT):** Instead of forcing a model to output its step-by-step reasoning (Chain-of-Thought) into the context window (which consumes tokens and slows down execution), the model learns to reason internally. The hidden states of the network carry the computational steps, allowing the model to "think" without speaking.
3. **Neural World Models (NeuralOS):** Extending this to system architecture, entire computing environments or operating system behaviors can be simulated via generative neural models rather than rigid code bases.

## Integration with CLADE

This concept bridges a critical gap in the CLADE architecture, specifically complementing the ideas in [self-distillation-context-compression.md](self-distillation-context-compression.md):

* **From Context to Weights:** While Self-Distillation Preference Optimization (SDPO) compresses *interaction history and context* into weights, "Program as Weights" compresses *logic, rules, and tool-use behaviors* into weights.
* **The Post-POSIX Agent:** In the context of CLADE's search for a post-POSIX abstraction (see [insights.md](../insights.md)), if an agent's core operating logic is compiled into its weights rather than running as an external Python/Rust script coordinating API calls, the agent *is* the program. 
* **Latency & The OODA Loop:** By internalizing reasoning (Implicit CoT), we drastically reduce the "Act" and "Observe" latency in the OODA loop (see [surprise-driven-bootstrapping.md](surprise-driven-bootstrapping.md)). The agent doesn't need to generate 500 tokens of reasoning before calling a tool; the reasoning happens in the forward pass of the hidden layers, outputting the tool call immediately.

## Open Questions for CLADE
- How do we update/patch a "neural program" when a tool API changes, without catastrophic forgetting?
- Does the dual memory architecture (episodic vs semantic) require different "compilation" strategies when converting programs to weights?
