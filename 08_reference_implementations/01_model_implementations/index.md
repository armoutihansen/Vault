---
layer: 08_reference_implementations
type: index
status: growing
tags: []
created: 2026-03-05
---

# Model Implementations

Runnable training, evaluation, and serialisation code for each model family.

> How do I implement this model family from data to prediction in code?

Each note provides complete, verified code using the canonical Python library for that model class. Notes synthesise theory from `03_modeling/` with practical scikit-learn, PyTorch, XGBoost, statsmodels, and Hugging Face APIs.

## Notes

- [[linear_models_implementation|Linear Models and GLMs]] — OLS, Ridge/Lasso, logistic regression, Tweedie GLMs with statsmodels and sklearn
- [[tree_ensembles_implementation|Tree Ensembles]] — Random Forest, XGBoost, LightGBM with Optuna tuning
- [[kernel_methods_implementation|Kernel Methods]] — SVC, SVR, kernel selection, GridSearchCV
- [[probabilistic_models_implementation|Probabilistic Models]] — GaussianNB, GMM + BIC, BayesianRidge
- [[unsupervised_learning_implementation|Unsupervised Learning]] — K-means, DBSCAN, PCA, t-SNE, UMAP, Isolation Forest
- [[time_series_implementation|Time Series]] — Auto-ARIMA, SARIMAX, Holt-Winters, walk-forward CV
- [[neural_network_implementation|Neural Networks (PyTorch)]] — MLP, CNN, RNN training loops with Accelerate
- [[transformer_finetuning_implementation|Transformer Fine-tuning]] — BERT/LLaMA fine-tuning with PEFT LoRA and SFTTrainer
- [[graphical_model_implementation|Graphical Models]] — Bayesian networks and PGM inference with pgmpy
- [[interpretability_implementation|Model Interpretability]] — SHAP values, PDP/ICE, permutation importance

## Links
- [[08_reference_implementations/index|08 — Reference Implementations]]
- [[08_reference_implementations/02_system_patterns/index|System Patterns]]
- [[03_modeling/index|03 — Modeling]]
