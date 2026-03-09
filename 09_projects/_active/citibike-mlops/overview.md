---
layer: 09_projects
type: project
status: active
owner: jesper
created: 2026-03-09
tags: [mlops, ml-engineering, fastapi, mlflow, airflow, pydantic]
---

# CitiBike MLOps

A production-grade ML engineering project built on top of the [CitiBike NYC AXA analysis](09_projects/_completed/citibike-nyc-axa/overview.md). The goal is to go through the **entire ML engineer lifecycle** — from raw data ingestion to a monitored, retrain-able prediction API — using the tools and practices that define production ML systems.

This project is explicitly a **learning project**: every component is implemented manually and deliberately. The domain is CitiBike NYC; the real deliverable is the experience of operating each lifecycle phase end-to-end.

**Source analysis:** [[09_projects/_completed/citibike-nyc-axa/overview|CitiBike NYC AXA]]  
**Step-by-step guide:** [[09_projects/_active/citibike-mlops/guide|Implementation Guide]]

---

## Purpose

The original DSC analysis was exploratory: notebooks, formula-based risk scores, and a CatBoost classifier reaching Macro-F1 0.51 on a 3-class net-flow problem. This project takes the same data and context and asks: *what does it look like when you build this properly?*

That means:
- Replacing the formula-based risk score with a **learned model** that can be versioned, evaluated, and retrained
- Replacing random-split classification with **time-series cross-validation** and better features
- Wrapping predictions in a **typed API** with input validation
- Orchestrating the data and training pipeline with a **DAG scheduler**
- Tracking every experiment with **structured metrics and artifact logging**
- Monitoring the deployed model for **drift and degradation**

---

## Scope (In / Out)

**In:**
- Two prediction tasks (see Modeling section)
- Full Airflow-orchestrated data pipeline (download → clean → feature engineering → train → evaluate → register)
- MLflow experiment tracking and model registry
- FastAPI prediction service with Pydantic request/response schemas
- Evidently-based drift monitoring (data + prediction drift)
- Scheduled retraining DAG
- pytest suite (unit + API integration)

**Out:**
- Real-time streaming ingestion
- Multi-region deployment
- Cloud infrastructure (local Docker Compose only)
- Weather data integration (listed as optional extension)
- Fine-tuned deep learning models

---

## Deliverables

| Artefact | Description |
|---|---|
| `dags/ingest_dag.py` | Airflow DAG: download CitiBike CSVs → clean → Parquet |
| `dags/retrain_dag.py` | Airflow DAG: feature engineering → train → evaluate → register in MLflow |
| `src/features/` | Feature engineering functions + Pydantic feature schemas |
| `src/models/` | Model definitions (risk regression, net flow regression) |
| `src/train.py` | Training script (MLflow run logging) |
| `src/evaluate.py` | Offline evaluation: Poisson deviance, calibration, RMSE |
| `api/main.py` | FastAPI app |
| `api/schemas.py` | Pydantic request/response models |
| `api/model_loader.py` | MLflow model loading (production alias) |
| `monitoring/drift.py` | Evidently drift report generation |
| `tests/` | Unit tests + API integration tests |
| `docker-compose.yml` | MLflow server + Airflow + FastAPI stack |
| `pyproject.toml` | `uv`-managed dependencies |

---

## Data

Same sources as the original DSC analysis:

| Dataset | Source | Size |
|---|---|---|
| CitiBike trip data | [S3 bucket](https://s3.amazonaws.com/tripdata/index.html) | ~80M rows (2023–2025) |
| NYPD Motor Vehicle Collisions | [NYC Open Data](https://data.cityofnewyork.us/Public-Safety/Motor-Vehicle-Collisions-Crashes/h9gi-nx95/about_data) | ~2M rows (cyclist-filtered) |

Data flows through a **bronze → silver → gold** Parquet directory structure:
- `bronze/` — raw CSVs converted to Parquet (no transformation)
- `silver/` — cleaned, validated, standardised schema
- `gold/` — feature-engineered, model-ready tables

---

## Modeling

### Task 1 — Crash Rate Prediction (replaces formula risk score)

**Problem:** Given a station, time of day, and bike type, predict the expected crash rate per 1 000 trips.

The original DSC risk score is a smoothed empirical formula (Empirical Bayes credibility weighting). This is replaced with a learned model that accounts for confounders and enables uncertainty quantification.

**Target variable:**

$$
y = \frac{\text{crashes involving cyclists at station } s \text{ in period } t}{\text{trips departing station } s \text{ in period } t} \times 1000
$$

**Features:**
- Station identity (ordinal encoding or cluster embedding)
- `hour_sin`, `hour_cos` (cyclical encoding of hour of day)
- `dow_sin`, `dow_cos` (day of week)
- `month_sin`, `month_cos` (month)
- `bike_type` (electric / classic, binary)
- `log_exposure` (log of trip count, offset in Poisson model)
- `rolling_crash_7d` (7-day rolling crash count at station, causal)

**Model progression (all tracked in MLflow):**

| Model | Objective | Notes |
|---|---|---|
| Poisson GLM (baseline) | Poisson deviance | Interpretable; offset = log(exposure) |
| Ridge regression (log-transformed target) | MSE | Gaussian approximation |
| XGBoost (Poisson objective) | Poisson deviance | Non-linear interactions |
| LightGBM (Poisson objective) | Poisson deviance | Faster than XGBoost |
| CatBoost (RMSE on log target) | RMSE | Handles categoricals natively |

**Evaluation:** Poisson deviance on held-out stations (spatial), calibration plot, RMSE, MAE.

### Task 2 — Net Flow Regression (replaces 3-class classification)

**Problem:** Predict daily net flow (arrivals − departures) per station as a continuous value. The original ternary buckets (importer / balanced / exporter) can be recovered by thresholding, but regression is more informative for operational decision-making.

**Features (extended from DSC):**
- Lag features: `net_flow_lag1`, `net_flow_lag7` (causal)
- Rolling mean: `net_flow_roll7`
- Fourier decomposition: `sin/cos` for day-of-year (captures annual seasonality)
- Station spatial cluster (k-means on lat/lng, k=10)
- `is_holiday` (US federal holidays flag)
- Weather lags from prior DSC analysis (optional)

**Model progression:**
- Persistence baseline (yesterday's net flow)
- Ridge regression
- XGBoost regression (RMSE objective)
- LightGBM regression

**Evaluation:** MAE, RMSE, directional accuracy (sign of predicted vs actual net flow), TimeSeriesSplit (5-fold walk-forward).

---

## Stack

| Layer | Tool | Role |
|---|---|---|
| Orchestration | **Apache Airflow** | DAGs for ingestion, feature engineering, retraining |
| Experiment tracking | **MLflow** | Run logging, artifact store, model registry |
| Serving | **FastAPI** | Typed REST API for predictions |
| Schema validation | **Pydantic v2** | Request/response models, feature validation |
| Monitoring | **Evidently** | Data drift + prediction drift reports |
| Data layer | **DuckDB + Parquet** | Efficient columnar reads over bronze/silver/gold |
| Containerisation | **Docker Compose** | Local stack (Airflow, MLflow, FastAPI) |
| Testing | **pytest** | Unit + API integration tests |
| Env management | **uv** | Fast, reproducible Python environments |

---

## Architecture Diagram

```
CitiBike S3 / NYC OpenData
         │
         ▼
┌─────────────────┐
│  Airflow DAG    │  ingest_dag: download → bronze → silver Parquet
│  (ingest)       │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Feature Store  │  Pydantic-validated feature tables (gold Parquet)
│  (DuckDB)       │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Airflow DAG    │  retrain_dag: feature engineering → train → evaluate → register
│  (retrain)      │
└────────┬────────┘
         │           ┌──────────────┐
         ├──────────►│  MLflow      │  Experiment tracking + Model Registry
         │           │  (tracking   │  staging → production alias
         │           │   server)    │
         │           └──────────────┘
         ▼
┌─────────────────┐
│  FastAPI        │  /predict/risk
│  Service        │  /predict/netflow
│  (Uvicorn)      │  /model/info
└────────┬────────┘  /health
         │
         ▼
┌─────────────────┐
│  Evidently      │  Scheduled drift reports (data + prediction)
│  Monitoring     │  Alerts when drift exceeds threshold
└─────────────────┘
```

---

## API Endpoints

| Method | Endpoint | Input | Output |
|---|---|---|---|
| POST | `/predict/risk` | `station_id`, `datetime`, `bike_type` | `risk_score`, `risk_tier`, `model_version` |
| POST | `/predict/netflow` | `station_id`, `date` | `net_flow`, `imbalance_class`, `model_version` |
| GET | `/model/info` | — | model version, training date, key metrics |
| GET | `/health` | — | service status, model loaded |

---

## Timeline

Active (learning project — no deadline).

## References

- Sculley, D. et al. (2015). *Hidden Technical Debt in Machine Learning Systems.* NeurIPS.
- Huyen, C. (2022). *Designing Machine Learning Systems.* O'Reilly.

## Links

- [[09_projects/_active/citibike-mlops/guide|Step-by-Step Implementation Guide]]
- [[09_projects/_completed/citibike-nyc-axa/overview|CitiBike NYC AXA (Source Analysis)]]
- [[05_ml_engineering/01_principles_and_lifecycle/ml_lifecycle|ML Lifecycle]]
- [[05_ml_engineering/05_model_development/experiment_tracking|Experiment Tracking]]
- [[05_ml_engineering/06_deployment_and_serving/serving_patterns|Serving Patterns]]
- [[05_ml_engineering/07_monitoring_and_observability/drift_detection|Drift Detection]]
- [[05_ml_engineering/08_continual_learning/retraining_strategies|Retraining Strategies]]
- [[05_ml_engineering/02_data_engineering/data_pipeline_patterns|Data Pipeline Patterns]]
