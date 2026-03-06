---
layer: 08_reference_implementations
type: index
status: growing
tags: []
created: 2026-03-05
---

# 08 — Reference Implementations

> Guiding question: "How is this implemented in code or architecture?"

This layer contains **verified, runnable implementations** of models, system patterns, and end-to-end ML/AI architectures. It is the code counterpart to the conceptual layers:

- `03_modeling/` explains *what models are* → here we implement them
- `05_ml_engineering/` explains *how ML systems are designed* → here we show them in code
- `06_ai_engineering/` explains *how LLM systems work* → here we build them

This layer does **not** contain business problem framing (→ `07_applications`), theory (→ `01_foundations`), or project-specific work (→ `09_projects`).

---

## [[08_reference_implementations/01_model_implementations/index|01 — Model Implementations]]

Runnable training, evaluation, and serialisation code for each model family. Synthesises `03_modeling/` theory with scikit-learn, PyTorch, XGBoost, statsmodels, and related libraries.

- [[linear_models_implementation|Linear Models and GLMs]]
- [[tree_ensembles_implementation|Tree Ensembles]]
- [[kernel_methods_implementation|Kernel Methods]]
- [[probabilistic_models_implementation|Probabilistic Models]]
- [[unsupervised_learning_implementation|Unsupervised Learning]]
- [[time_series_implementation|Time Series Models]]
- [[neural_network_implementation|Neural Networks (PyTorch)]]
- [[transformer_finetuning_implementation|Transformer Fine-tuning]]
- [[graphical_model_implementation|Graphical Models]]
- [[interpretability_implementation|Model Interpretability (SHAP)]]

---

## [[08_reference_implementations/02_system_patterns/index|02 — System Patterns]]

Reusable production patterns for ML/AI systems: experiment tracking, feature stores, model serving, monitoring, RAG, agents, quantization, and more. Synthesises `05_ml_engineering/` and `06_ai_engineering/`.

---

## [[08_reference_implementations/03_end_to_end_examples/index|03 — End-to-End Examples]]

Complete ML/AI system walkthroughs combining multiple components across multiple layers. Each is a reference architecture adaptable to real projects.

---

## Links
- [[07_applications/index|07 — Applications]]
- [[03_modeling/index|03 — Modeling]]
- [[05_ml_engineering/index|05 — ML Engineering]]
- [[06_ai_engineering/index|06 — AI Engineering]]
