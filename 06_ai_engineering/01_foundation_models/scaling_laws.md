---
layer: 06_ai_engineering
type: concept
status: growing
tags: [llm, theory]
created: 2026-03-07
---

# Scaling Laws

## Purpose
How model performance scales with compute, data, and parameters — and how to use this to make training decisions.

## Chinchilla Scaling Laws
[Hoffmann et al. 2022] showed that for compute-optimal training, tokens ≈ 20× parameters. 

Key formula: for a compute budget C (FLOPs), optimal N* ≈ (C / 6)^0.5 and D* ≈ 20 × N*.

Prior (Kaplan et al. 2020) over-parameterised models relative to data; Chinchilla corrected this.

## Power Law Relationships
Loss scales as a power law in:
- Model parameters N: L(N) ∝ N^{-α}
- Dataset tokens D: L(D) ∝ D^{-β}
- Compute C: L(C) ∝ C^{-γ}

Exponents (Chinchilla): α ≈ β ≈ 0.5, γ ≈ 0.5.

## Emergent Abilities
Certain capabilities (multi-step arithmetic, chain-of-thought, in-context learning) appear abruptly above threshold scales. Not predicted by smooth power laws — may reflect phase transitions or evaluation metric discontinuities.

## Inference-Time Compute Scaling
Recent work (OpenAI o1, DeepSeek-R1) shows scaling test-time compute via chain-of-thought/search can substitute for pre-training scale. Separate scaling curve: performance ∝ inference-time FLOPs.

## Practical Implications
- Choose N and D jointly for a compute budget — don't just maximise N
- Smaller models trained on more data often outperform larger undertrained models at equivalent FLOPs
- Training beyond Chinchilla-optimal improves inference efficiency (1× FLOPs, smaller model)

## References
- Hoffmann et al. (2022). Training Compute-Optimal Large Language Models. https://arxiv.org/abs/2203.15556
- Kaplan et al. (2020). Scaling Laws for Neural Language Models. https://arxiv.org/abs/2001.08361
- Wei et al. (2022). Emergent Abilities of Large Language Models. https://arxiv.org/abs/2206.07682

## Links
- [[06_ai_engineering/01_foundation_models/index|01 — Foundation Models]]
- [[01_foundations/06_deep_learning_theory/index|Deep Learning Theory]]
