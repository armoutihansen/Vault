# Modeling

## Purpose of This Layer

This layer captures the conceptual discipline of model design and selection.

It focuses on:

- Framing predictive and inferential problems
- Designing features and representations
- Choosing appropriate model families
- Training and tuning models
- Designing evaluation protocols
- Reasoning about interpretability and fairness
- Applying modeling techniques to specific domains

Guiding question:

> How should we model this problem, and how do we justify that choice?

This layer does NOT cover:

- Mathematical proofs and derivations (→ 01_foundations)
- Production pipelines, deployment, and monitoring (→ 04_ml_engineering)
- Foundation-model system design (→ 05_ai_engineering)

---
## Structure

### 00_modeling_principles/

Core conceptual foundations of modeling.

Includes:
- Modeling workflow
- Problem types and objectives
- Bias–variance tradeoff
- Overfitting and underfitting
- Error analysis

Focus:
Building intuition for how models behave and fail.

---
### 01_problem_framing/

Formalizing real-world problems as ML tasks.

Includes:
- Prediction vs inference
- Target definition
- Data leakage
- Train/validation/test splitting
- Time-series splitting

Focus:
Translating business or research questions into well-posed modeling tasks.

---
### 02_data_and_features/

Representation design.

Includes:
- Feature engineering overview
- Handling missing values
- Categorical encoding
- Scaling and normalization
- Text vectorization
- Embeddings
- Class imbalance handling
- Data augmentation strategies

Focus:
Designing inputs that make learning possible and robust.

---
### 03_model_families/

Algorithmic toolkits.

Includes:

### Linear and GLM
- Linear regression
- Logistic regression
- Poisson, Gamma, Tweedie GLMs

### Trees and Boosting
- Random forest
- Gradient boosting
- XGBoost, LightGBM, CatBoost

### Probabilistic Models
- Naive Bayes
- Bayesian regression
- Hidden Markov models

### Deep Learning
- MLP
- CNN
- RNN / LSTM / GRU
- Transformer architectures (modeling perspective)

### Unsupervised Learning
- Clustering (K-means, GMM)
- Dimensionality reduction (PCA, UMAP)
- Anomaly detection

### Time Series Models
- ARIMA / SARIMA
- State-space / Kalman models
- ML-based forecasting

Focus:
Understanding strengths, weaknesses, assumptions, and trade-offs of model classes.

---
### 04_training_and_tuning/

Optimization and generalization control.

Includes:
- Loss functions
- Regularization
- Optimization algorithms
- Hyperparameter tuning
- Early stopping

Focus:
Controlling model capacity and convergence behavior.

---
### 05_evaluation/

Designing reliable offline evaluation.

Includes:
- Offline evaluation design
- Classification metrics
- Regression metrics
- Ranking metrics
- Calibration
- Uncertainty quantification
- Cross-validation
- Backtesting for time series

Focus:
Measuring performance before deployment.

---
### 06_interpretability_and_fairness/

Model transparency and risk.

Includes:
- Interpretability overview
- SHAP and feature attribution
- Partial dependence and ICE
- Fairness metrics
- Model risk considerations

Focus:
Understanding model behavior and societal implications.

---
### 07_domain_modeling/

Applying modeling principles to specific domains.

### Insurance Models
- Claims frequency modeling
- Claims severity modeling
- Pricing with GLMs and boosting
- Reserving overview
- Fraud modeling

### Experimentation
- A/B testing basics
- Uplift modeling
- Causal inference overview

Focus:
Contextualizing general modeling techniques within applied settings.

---

## Relationship to Other Layers

01_foundations:
Provides mathematical justification for modeling techniques.

02_modeling:
Defines which models, features, and evaluation strategies to use.

04_ml_engineering:
Implements, deploys, and operates chosen models in production.

05_ai_engineering:
Engineers systems built around pretrained foundation models.

This layer treats models as conceptual decision objects — not production artifacts.

---
## Explicit Structure

```
02_modeling/
├── index.md
├── 00_modeling_principles/
├── 01_problem_framing/
├── 02_data_and_features/
├── 03_model_families/
│   ├── 00_linear_and_glm/
│   ├── 01_trees_and_boosting/
│   ├── 02_probabilistic_models/
│   ├── 03_deep_learning/
│   ├── 04_nlp/
│   ├── 05_unsupervised_learning/
│   └── 06_time_series_models/
├── 04_training_and_tuning/
├── 05_evaluation/
├── 06_interpretability_and_fairness/
└── 07_domain_modeling/
    ├── 00_insurance_models/
    └── 01_experimentation/
```