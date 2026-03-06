---
layer: 02_modeling
type: index
status: evergreen
tags: []
created: 2026-03-02
---

# Modeling

Conceptual discipline of model design, selection, training, and evaluation.

> How should we model this problem, and how do we justify that choice?

This layer does NOT cover: mathematical proofs (→ 01_foundations), production pipelines (→ 04_ml_engineering), or foundation-model system design (→ 05_ai_engineering).

## Sublayers

### [[02_modeling/01_problem_formulation/index|01 — Problem Formulation]]
Framing ML problems: output types, loss choice, baselines, and success criteria.

### [[02_modeling/02_data_science/index|02 — Data Science]]
EDA, preprocessing, feature engineering, data validation, and visualization.

### [[02_modeling/03_model_families/index|03 — Model Families]]
Algorithmic toolkits by model class.
- [[02_modeling/03_model_families/01_linear_and_glm/index|Linear Models & GLMs]] — OLS, Ridge/Lasso, logistic regression, GLM families
- [[02_modeling/03_model_families/02_tree_ensembles/index|Tree Ensembles]] — decision trees, random forests, XGBoost, LightGBM
- [[02_modeling/03_model_families/03_probabilistic_models/index|Probabilistic Models]] — Naive Bayes, GMM, EM algorithm
- [[02_modeling/03_model_families/04_neural_networks/index|Neural Networks]] — MLP, CNN, RNN/LSTM, face recognition, object detection
- [[02_modeling/03_model_families/05_kernel_methods/index|Kernel Methods]] — SVM, kernel trick, SVR
- [[02_modeling/03_model_families/06_unsupervised_learning/index|Unsupervised Learning]] — k-means, DBSCAN, PCA, t-SNE/UMAP
- [[02_modeling/03_model_families/07_transformers/index|Transformers]] — attention, transformer architecture, seq2seq, word embeddings
- [[02_modeling/03_model_families/08_graphical_models/index|Graphical Models]] — Bayesian networks, MRF, HMM, belief propagation
- [[02_modeling/03_model_families/09_time_series_models/index|Time Series Models]] — ARIMA, ETS, state-space, VAR, neural forecasting

### [[02_modeling/04_training_dynamics/index|04 — Training Dynamics]]
Loss functions, optimization algorithms, regularization, hyperparameter tuning.

### [[02_modeling/05_evaluation_and_validation/index|05 — Evaluation and Validation]]
K-fold CV, classification/regression metrics, calibration, model selection.

### [[02_modeling/06_interpretability/index|06 — Interpretability & Fairness]]
SHAP, PDP/ICE, fairness metrics, model risk management.

## Links
- [[01_foundations/index|Foundations]]
- [[04_ml_engineering/index|ML Engineering]]
