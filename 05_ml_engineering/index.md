---
layer: 05_ml_engineering
type: index
status: evergreen
tags: []
created: 2026-03-05
---

# 05 — ML Engineering

Engineering discipline for designing, building, deploying, and operating production-grade ML systems — from raw data ingestion through to monitored, retraining-ready production deployments.

> Guiding question: "How do we productionize and operate ML systems?"

This layer does NOT cover: foundation-model-based systems (→ 06_ai_engineering), model family selection and statistical trade-offs (→ 03_modeling), or mathematical derivations (→ 01_foundations).

## Sublayers

### [[05_ml_engineering/01_principles_and_lifecycle/index|01 — Principles & Lifecycle]]
End-to-end ML system thinking: reliability contracts, scalability, and the iterative development philosophy.

### [[05_ml_engineering/02_data_engineering/index|02 — Data Engineering]]
Data pipelines, ETL/ELT, batch vs stream processing, feature stores.

### [[05_ml_engineering/03_training_data/index|03 — Training Data]]
Data labeling, class imbalance handling, augmentation, dataset versioning.

### [[05_ml_engineering/04_feature_engineering/index|04 — Feature Engineering]]
Numeric/categorical/temporal feature transforms, embeddings, data leakage prevention.

### [[05_ml_engineering/05_model_development/index|05 — Model Development]]
Experiment tracking, distributed training, offline evaluation, reproducibility.

### [[05_ml_engineering/06_deployment_and_serving/index|06 — Deployment & Serving]]
Batch/online/edge serving patterns, model compression, rollout strategies.

### [[05_ml_engineering/07_monitoring_and_observability/index|07 — Monitoring & Observability]]
Drift detection, operational metrics, alerting, incident response.

### [[05_ml_engineering/08_continual_learning/index|08 — Continual Learning]]
Retraining triggers, stateless vs stateful updates, test-in-production.

### [[05_ml_engineering/09_infrastructure_and_platform/index|09 — Infrastructure & Platform]]
Orchestration, ML platform architecture, environment and dependency management.

## Relationship to Other Layers

- **03_modeling** — model families and training dynamics; this layer handles productionizing them
- **04_software_engineering** — general software patterns applied to ML system concerns
- **06_ai_engineering** — foundation-model-specific workflows that build on top of this layer's infrastructure

## Links
- [[03_modeling/index|Modeling]]
- [[04_software_engineering/index|Software Engineering]]
- [[06_ai_engineering/index|AI Engineering]]
