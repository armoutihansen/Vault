# ML Engineering

## Purpose of This Layer

This layer captures the engineering discipline required to design, build, deploy, and operate production-grade machine learning systems.

It focuses on structured ML systems where:

- You train and own the model
- You manage feature pipelines
- You deploy trained artifacts
- You monitor and retrain models over time

Guiding question:

> How do we build reliable, scalable ML systems in production?

This layer does NOT cover:

- Foundation-model-based system design (→ 05_ai_engineering)
- Model family selection and statistical trade-offs (→ 02_modeling)
- Mathematical derivations (→ 01_foundations)

---
## Structure

## 00_principles_and_lifecycle/

High-level system thinking.

Includes:
- ML systems overview
- Reliability, scalability, maintainability, adaptability
- Iterative development cycles

Focus:
Understanding the end-to-end ML lifecycle before implementation details.

---
### 01_data_engineering/

Data infrastructure for ML systems.

Includes:
- Data sources
- Data formats and data models
- OLTP vs OLAP systems
- ETL / ELT
- Batch vs stream processing
- Dataflow between services

Focus:
Reliable movement and transformation of raw data.

---
### 02_training_data/

Construction of model-ready datasets.

Includes:
- Sampling strategies
- Labeling strategies
- Weak and natural labels
- Class imbalance handling
- Data augmentation
- Dataset versioning and lineage

Focus:
Turning raw data into structured training data.

---
### 03_feature_engineering/

Designing model inputs.

Includes:
- Learned vs engineered features
- Handling missing values
- Scaling and encoding
- Feature crosses and embeddings
- Data leakage
- Feature stores

Focus:
Bridging raw data and model representation.

---
### 04_model_development_and_offline_eval/

Controlled model development.

Includes:
- Baselines and error analysis
- Training and regularization
- Model selection trade-offs
- Evaluation methods and calibration
- Experiment tracking (e.g., MLflow)
- Model versioning and reproducibility
- Distributed training

Focus:
Offline experimentation and model iteration.

---
### 05_deployment_and_serving/

Making models accessible in production.

Includes:
- Batch vs online prediction
- Serving patterns (API, streaming)
- Model optimization and compression
- Edge and browser inference
- Rollout strategies

Focus:
Operationalizing trained models.

---
### 06_monitoring_and_observability/

Operating ML systems safely.

Includes:
- Operational metrics
- ML-specific metrics (predictions, features, inputs)
- Drift detection (covariate, label, concept)
- Logging, tracing, dashboards, alerts
- Incident response playbooks

Focus:
Detecting failures and maintaining reliability.

---
### 07_continual_learning_and_testing_in_prod/

Adapting models over time.

Includes:
- Retraining strategies (stateless vs stateful)
- Data freshness considerations
- Test-in-production
- Shadow, A/B, canary, bandit testing
- Safe rollback and backfills

Focus:
Managing model evolution in dynamic environments.

---
### 08_infrastructure_and_platform/

Platform-level concerns.

Includes:
- Development environment standardization
- Containers
- Schedulers and orchestrators
- Workflow management systems
- ML platform reference architecture
- Model stores
- Build vs buy decisions

Focus:
The infrastructure foundation that enables ML engineering.

---
## Relationship to Other Layers

02_modeling:
Defines what models and evaluation strategies to use.

04_ml_engineering:
Implements, deploys, and operates those models in production.

05_ai_engineering:
Designs systems built around pretrained foundation models.

This layer treats ML models as production artifacts that require lifecycle management, infrastructure, and observability.

---
## Explicit Structure

```
04_ml_engineering/
├── index.md
├── 00_principles_and_lifecycle/
├── 01_data_engineering/
├── 02_training_data/
├── 03_feature_engineering/
├── 04_model_development_and_offline_eval/
├── 05_deployment_and_serving/
├── 06_monitoring_and_observability/
├── 07_continual_learning_and_testing_in_prod/
└── 08_infrastructure_and_platform/
```
