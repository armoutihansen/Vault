---
layer: 03_modeling
type: index
status: evergreen
tags: []
created: 2026-05-31
---

# 03 — Modeling

Model families, training objectives, inductive biases, and model selection strategies. Answers: *how do we choose and train a model that generalises?*

> **Guiding question:** Which model captures the structure of this problem, and how do we train it to generalise?

This layer does NOT cover: mathematical derivations (→ `01_foundations`), production pipelines or serving (→ `05_ml_engineering`), foundation-model system design (→ `06_ai_engineering`), or data preparation (→ `02_data_science`).

## Sublayers

### [[03_modeling/01_supervised_learning/index|01 — Supervised Learning]]
Linear and GLM models, tree ensembles, kernel methods, and instance-based methods.

### [[03_modeling/02_unsupervised_learning/index|02 — Unsupervised Learning]]
Clustering, dimensionality reduction, density estimation, and representation learning.

### [[03_modeling/03_probabilistic_models/index|03 — Probabilistic Models]]
Graphical models, latent variable models, and Bayesian methods.

### [[03_modeling/04_deep_learning/index|04 — Deep Learning]]
MLPs, CNNs, sequence models, transformers, and multimodal architectures.

### [[03_modeling/05_time_series/index|05 — Time Series]]
Classical forecasting (ARIMA, ETS), state-space models, and ML/DL approaches to temporal prediction.

### [[03_modeling/06_training_and_regularization/index|06 — Training and Regularization]]
Loss functions, optimisation algorithms, regularisation strategies, hyperparameter tuning, and early stopping.

### [[03_modeling/07_evaluation_and_model_selection/index|07 — Evaluation and Model Selection]]
Cross-validation, classification and regression metrics, calibration, SHAP, PDP/ICE, and model risk.

## Relationship to Other Layers

- **← [[01_foundations/index|01 Foundations]]**: mathematical prerequisites — linear algebra, calculus, probability, optimization theory.
- **← [[02_data_science/index|02 Data Science]]**: data preparation, feature engineering, and problem framing feed into model training.
- **→ [[05_ml_engineering/index|05 ML Engineering]]**: productionizing trained models: serving, monitoring, retraining.
- **→ [[06_ai_engineering/index|06 AI Engineering]]**: foundation models and LLM-based systems.
