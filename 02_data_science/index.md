---
layer: 02_data_science
type: index
status: evergreen
tags: []
created: 2026-05-31
---

# 02 — Data Science

Practical analytical reasoning from data: framing problems, understanding data, transforming it, and communicating findings. Answers: *how do we understand data well enough to make sound modelling decisions?*

> **Guiding question:** What does the data tell us, and how do we prepare it for sound modelling?

This layer does NOT cover: mathematical derivations (→ `01_foundations`), specific model families and training objectives (→ `03_modeling`), production data pipelines (→ `05_ml_engineering`), or business deployment decisions (→ `06_applications`).

## Sublayers

### [[02_data_science/01_problem_framing/index|01 — Problem Framing]]
Translating a real-world task into a well-specified ML problem: output types, loss choice, baselines, and success criteria.

### [[02_data_science/02_data_representation/index|02 — Data Representation]]
How data is structured, encoded, and typed — tabular, text, image, and graph representations; schema design and data contracts.

### [[02_data_science/03_exploratory_data_analysis/index|03 — Exploratory Data Analysis]]
Univariate and bivariate analysis, missing value diagnosis, class imbalance, outlier detection, and distribution shape.

### [[02_data_science/04_feature_engineering/index|04 — Feature Engineering]]
Constructing new input features from raw data: numeric transforms, temporal encoding, categorical aggregates, target encoding, and leakage prevention.

### [[02_data_science/05_experimentation_and_validation/index|05 — Experimentation and Validation]]
Data preprocessing pipelines, data validation (schema checks, drift tests, PSI), and experiment integrity.

### [[02_data_science/06_interpretability_and_communication/index|06 — Interpretability and Communication]]
Model interpretability taxonomy, fairness metrics, and communicating analytical results to stakeholders.

### [[02_data_science/07_decision_analysis_and_business_metrics/index|07 — Decision Analysis and Business Metrics]]
Translating model outputs into business decisions: expected value, cost-benefit analysis, decision thresholds, and metric alignment.

## Relationship to Other Layers

- **← [[01_foundations/index|01 Foundations]]**: probability, statistics, and linear algebra underpin every technique in this layer.
- **→ [[03_modeling/index|03 Modeling]]**: data prepared here feeds into model training and evaluation there.
- **→ [[05_ml_engineering/index|05 ML Engineering]]**: production versions of data pipelines, feature stores, and validation gating live there.
