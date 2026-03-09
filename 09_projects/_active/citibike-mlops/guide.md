---
layer: 09_projects
type: workflow
status: growing
owner: jesper
created: 2026-03-09
tags: [mlops, ml-engineering, fastapi, mlflow, airflow, pydantic]
---

# CitiBike MLOps — Step-by-Step Implementation Guide

Companion to [[09_projects/_active/citibike-mlops/overview|Overview]].

Each phase builds on the previous. Phases 0–2 produce the data and feature layer; Phases 3–5 produce trained, registered models; Phases 6–9 wrap them in a production-style service.

---

## Phase 0 — Problem Framing & Environment Setup

### 0.1 Clarify success metrics

Before writing a line of code, write down (in a comment in `train.py` or a `DECISIONS.md`):

- **Primary metric (risk model):** Poisson deviance on held-out stations
- **Primary metric (net flow model):** MAE on a temporally held-out period (last 90 days)
- **Serving SLA:** p95 latency < 200ms, error rate < 0.1%
- **Drift threshold:** alert when PSI > 0.2 on any input feature or prediction distribution

Having explicit thresholds before training prevents post-hoc metric selection.

### 0.2 Repository setup

```bash
mkdir citibike-mlops && cd citibike-mlops
git init
uv init
uv add fastapi uvicorn pydantic mlflow airflow apache-airflow \
       lightgbm xgboost catboost scikit-learn duckdb pyarrow \
       evidently python-dotenv pytest httpx
```

Recommended project structure:

```
citibike-mlops/
├── dags/
│   ├── ingest_dag.py
│   └── retrain_dag.py
├── src/
│   ├── features/
│   │   ├── schemas.py        # Pydantic feature schemas
│   │   └── engineer.py       # Feature engineering functions
│   ├── models/
│   │   ├── risk.py           # Risk model (Poisson regression / XGBoost)
│   │   └── netflow.py        # Net flow regression model
│   ├── train.py              # MLflow training entry point
│   └── evaluate.py           # Offline evaluation metrics
├── api/
│   ├── main.py               # FastAPI app
│   ├── schemas.py            # Request/response Pydantic models
│   └── model_loader.py       # MLflow model loading
├── monitoring/
│   └── drift.py              # Evidently drift reports
├── tests/
│   ├── test_features.py
│   ├── test_models.py
│   └── test_api.py
├── data/
│   ├── bronze/               # Raw Parquet (no transformation)
│   ├── silver/               # Cleaned, validated
│   └── gold/                 # Feature-engineered, model-ready
├── docker-compose.yml
└── pyproject.toml
```

### 0.3 Docker Compose stack

Write `docker-compose.yml` to run:
- **MLflow tracking server** — `mlflow server --host 0.0.0.0 --port 5000 --backend-store-uri sqlite:///mlflow.db --artifact-root ./mlruns`
- **Airflow** — official `apache/airflow:2.x` image, LocalExecutor, SQLite backend (for local dev)
- **FastAPI** — `uvicorn api.main:app --reload`

```yaml
services:
  mlflow:
    image: ghcr.io/mlflow/mlflow:latest
    ports: ["5000:5000"]
    command: mlflow server --host 0.0.0.0 --backend-store-uri sqlite:///mlflow.db --artifact-root /mlruns
    volumes: ["./mlruns:/mlruns"]

  airflow:
    image: apache/airflow:2.9.1
    environment:
      AIRFLOW__CORE__EXECUTOR: LocalExecutor
      AIRFLOW__DATABASE__SQL_ALCHEMY_CONN: sqlite:////opt/airflow/airflow.db
    volumes: ["./dags:/opt/airflow/dags", "./data:/data"]
    ports: ["8080:8080"]
    command: standalone

  api:
    build: .
    ports: ["8000:8000"]
    environment:
      MLFLOW_TRACKING_URI: http://mlflow:5000
    command: uvicorn api.main:app --host 0.0.0.0 --port 8000
    depends_on: [mlflow]
```

---

## Phase 1 — Data Pipeline (Airflow)

### 1.1 Bronze layer: raw ingestion DAG

`dags/ingest_dag.py` — runs on schedule (weekly for CitiBike, monthly for NYPD).

```python
from airflow.decorators import dag, task
from datetime import datetime

@dag(schedule="@weekly", start_date=datetime(2024, 1, 1), catchup=False)
def ingest_dag():

    @task
    def download_citibike(month: str) -> str:
        """Download one month of CitiBike CSVs from S3."""
        # Use the existing src/download_citibike.py logic
        ...

    @task
    def convert_to_parquet(csv_path: str) -> str:
        """Convert raw CSV to Parquet (bronze layer). No transformation."""
        import duckdb
        out = csv_path.replace(".csv", ".parquet").replace("/raw/", "/bronze/")
        duckdb.sql(f"COPY (SELECT * FROM read_csv_auto('{csv_path}')) TO '{out}' (FORMAT PARQUET)")
        return out

    @task
    def clean_to_silver(parquet_path: str) -> str:
        """Validate schema, standardise columns, filter invalid rows → silver."""
        ...

    csv = download_citibike("{{ ds[:7] }}")
    parquet = convert_to_parquet(csv)
    clean_to_silver(parquet)

ingest_dag()
```

**Key tasks in silver cleaning:**
- Standardise column names (`starttime` → `started_at`)
- Parse datetimes with timezone normalisation
- Filter: drop rows with null station IDs, negative durations, distances > 50km
- Merge collision data: left join on (station ≈ nearest start/end station, date)

### 1.2 DuckDB for bronze → silver transformation

Use DuckDB for all aggregations — it reads Parquet directly and avoids pandas memory issues (lesson from original DSC analysis):

```python
import duckdb

conn = duckdb.connect()

station_day = conn.execute("""
    SELECT
        start_station_id,
        date_trunc('day', started_at) AS day,
        rideable_type,
        COUNT(*) AS total_trips,
        SUM(CASE WHEN rideable_type = 'electric_bike' THEN 1 ELSE 0 END) AS electric_trips,
        SUM(arrivals) - SUM(departures) AS net_flow
    FROM read_parquet('data/silver/citibike/*.parquet')
    GROUP BY 1, 2, 3
""").df()
```

### 1.3 Gold layer: feature-ready tables

Write station-day and station-hour aggregates to `data/gold/` as Parquet. These are the model input tables.

---

## Phase 2 — Feature Engineering & Schema Validation (Pydantic)

### 2.1 Feature engineering functions

`src/features/engineer.py`:

```python
import numpy as np
import pandas as pd

def cyclical_encode(series: pd.Series, period: int) -> tuple[pd.Series, pd.Series]:
    """Encode a periodic variable as (sin, cos) components."""
    angle = 2 * np.pi * series / period
    return np.sin(angle), np.cos(angle)

def add_time_features(df: pd.DataFrame) -> pd.DataFrame:
    df = df.copy()
    df["hour_sin"], df["hour_cos"] = cyclical_encode(df["hour"], 24)
    df["dow_sin"],  df["dow_cos"]  = cyclical_encode(df["dayofweek"], 7)
    df["month_sin"], df["month_cos"] = cyclical_encode(df["month"], 12)
    df["doy_sin"],  df["doy_cos"]  = cyclical_encode(df["dayofyear"], 365)
    return df

def add_lag_features(df: pd.DataFrame, target_col: str) -> pd.DataFrame:
    """Add lag-1 and lag-7 features. Sort by station, date first."""
    df = df.sort_values(["station_id", "date"])
    df[f"{target_col}_lag1"] = df.groupby("station_id")[target_col].shift(1)
    df[f"{target_col}_lag7"] = df.groupby("station_id")[target_col].shift(7)
    df[f"{target_col}_roll7"] = (
        df.groupby("station_id")[target_col]
        .transform(lambda x: x.shift(1).rolling(7).mean())
    )
    return df
```

### 2.2 Pydantic schemas for feature validation

`src/features/schemas.py` — validate every feature row before it enters the model:

```python
from pydantic import BaseModel, field_validator, model_validator
from enum import Enum

class BikeType(str, Enum):
    electric = "electric_bike"
    classic  = "classic_bike"

class RiskFeatures(BaseModel):
    station_id: str
    hour_sin: float
    hour_cos: float
    dow_sin: float
    dow_cos: float
    month_sin: float
    month_cos: float
    bike_type: BikeType
    log_exposure: float       # log(trip count) — offset term for Poisson model
    rolling_crash_7d: float

    @field_validator("log_exposure")
    @classmethod
    def exposure_positive(cls, v: float) -> float:
        if v < 0:
            raise ValueError("log_exposure must be non-negative")
        return v

class NetFlowFeatures(BaseModel):
    station_id: str
    net_flow_lag1: float
    net_flow_lag7: float
    net_flow_roll7: float
    dow_sin: float
    dow_cos: float
    doy_sin: float
    doy_cos: float
    station_cluster: int      # k-means cluster (0–9)
    is_holiday: bool
```

**Why Pydantic here (not just pandas dtype checks):** Pydantic coerces types at the boundary (JSON → Python), generates a JSON Schema (used by FastAPI for OpenAPI docs), and provides field-level validators with clear error messages. Use it at every system boundary — API input, feature pipeline output, model registry payload.

---

## Phase 3 — Experimentation (MLflow)

### 3.1 MLflow run structure

Every model variant is a **named MLflow run** inside a named experiment. Use consistent parameter and metric keys so runs are comparable.

```python
import mlflow
import mlflow.sklearn

mlflow.set_tracking_uri("http://localhost:5000")
mlflow.set_experiment("citibike-risk")

with mlflow.start_run(run_name="xgboost-poisson"):
    # Log parameters
    mlflow.log_params({
        "model_type": "xgboost",
        "objective": "count:poisson",
        "n_estimators": 500,
        "max_depth": 6,
        "learning_rate": 0.05,
        "feature_set": "v1",
    })

    # Train model
    model = xgb.XGBRegressor(objective="count:poisson", ...)
    model.fit(X_train, y_train)

    # Log metrics
    y_pred = model.predict(X_val)
    mlflow.log_metrics({
        "val_poisson_deviance": poisson_deviance(y_val, y_pred),
        "val_rmse": mean_squared_error(y_val, y_pred, squared=False),
        "val_mae": mean_absolute_error(y_val, y_pred),
    })

    # Log model + feature importance plot
    mlflow.sklearn.log_model(model, artifact_path="model")
    mlflow.log_figure(plot_feature_importance(model), "feature_importance.png")
```

### 3.2 Baseline runs (always log these first)

For the risk model:
- **Naive baseline**: always predict the dataset mean crash rate → sets a floor
- **EB formula baseline**: replicate the original DSC Empirical Bayes formula as a Python function, log its Poisson deviance as a reference point

For the net flow model:
- **Persistence baseline**: predict yesterday's net flow
- **Rolling mean baseline**: 7-day rolling mean

These provide honest reference points when interpreting experiment results.

### 3.3 Hyperparameter search

Use MLflow's `mlflow.autolog()` with scikit-learn or log manually. For systematic search:

```python
from itertools import product

param_grid = {
    "n_estimators": [200, 500],
    "max_depth": [4, 6, 8],
    "learning_rate": [0.05, 0.1],
}

for params in [dict(zip(param_grid, v)) for v in product(*param_grid.values())]:
    with mlflow.start_run(run_name=f"xgb-{params['n_estimators']}-d{params['max_depth']}"):
        mlflow.log_params(params)
        model = xgb.XGBRegressor(objective="count:poisson", **params)
        ...
```

---

## Phase 4 — Offline Evaluation & Model Selection

### 4.1 Time-series cross-validation

**Never use random splits on time series data.** Use `TimeSeriesSplit` (walk-forward validation):

```python
from sklearn.model_selection import TimeSeriesSplit

tscv = TimeSeriesSplit(n_splits=5, gap=7)  # 7-day gap between train/val
scores = []

for fold, (train_idx, val_idx) in enumerate(tscv.split(X)):
    X_train, X_val = X.iloc[train_idx], X.iloc[val_idx]
    y_train, y_val = y.iloc[train_idx], y.iloc[val_idx]
    model.fit(X_train, y_train)
    y_pred = model.predict(X_val)
    scores.append(poisson_deviance(y_val, y_pred))
    mlflow.log_metric("cv_poisson_deviance", np.mean(scores), step=fold)
```

Also evaluate **spatial holdout**: train on all but 20% of stations, evaluate on held-out stations. This tests generalisation to unseen station contexts.

### 4.2 Calibration check (risk model)

A risk model used for insurance pricing must be calibrated — predicted rates must match observed rates in aggregate.

```python
import matplotlib.pyplot as plt

# Bucket predictions into deciles, compare mean predicted vs mean observed
quantiles = pd.qcut(y_pred, q=10)
calibration = pd.DataFrame({"pred": y_pred, "obs": y_val})
calibration["bucket"] = quantiles
cal_df = calibration.groupby("bucket").mean()

fig, ax = plt.subplots()
ax.plot(cal_df["pred"], cal_df["obs"], "o-")
ax.plot([0, max(y_pred)], [0, max(y_pred)], "--", label="perfect calibration")
mlflow.log_figure(fig, "calibration.png")
```

### 4.3 Choosing the production model

Select the model with the best **mean TimeSeriesSplit Poisson deviance** (risk) or **mean MAE** (net flow) on the final evaluation. Log the selection decision in MLflow as a tag:

```python
mlflow.set_tag("selection_reason", "best CV Poisson deviance, well-calibrated on decile plot")
```

---

## Phase 5 — Model Packaging & Registry

### 5.1 Register the selected model

```python
import mlflow.sklearn

# After selecting the best run, register it
run_id = "abc123..."
mlflow.register_model(
    model_uri=f"runs:/{run_id}/model",
    name="citibike-risk-model"
)
```

### 5.2 Transition to production

Use the MLflow client to promote from `Staging` → `Production`:

```python
from mlflow.tracking import MlflowClient

client = MlflowClient()
client.set_registered_model_alias(
    name="citibike-risk-model",
    alias="production",
    version="3"
)
```

Use **aliases** (`production`, `staging`, `champion`) rather than mutable stage labels (deprecated in MLflow 2.x).

### 5.3 Model card in MLflow

Log a model card as a text artifact:

```
Model: CatBoost Poisson regression (crash rate per 1 000 trips)
Training data: CitiBike + NYPD 2023-01-01 to 2024-12-31
Held-out period: 2025-01-01 to 2025-03-01
Val Poisson deviance: 0.42 (baseline formula: 0.61)
Calibration: mean absolute calibration error 0.003 (10-decile)
Known limitations: sparse data for low-volume stations (< 50 trips/day) — EB smoothing not replicated
```

---

## Phase 6 — Serving API (FastAPI + Pydantic)

### 6.1 Request/response schemas

`api/schemas.py`:

```python
from pydantic import BaseModel, Field
from datetime import datetime
from enum import Enum

class BikeType(str, Enum):
    electric = "electric_bike"
    classic  = "classic_bike"

class RiskTier(str, Enum):
    low       = "Low"
    medium    = "Medium"
    high      = "High"
    very_high = "Very High"

class RiskRequest(BaseModel):
    station_id: str = Field(..., example="6926.10")
    datetime: datetime = Field(..., example="2025-06-15T17:30:00")
    bike_type: BikeType = Field(..., example="electric_bike")

class RiskResponse(BaseModel):
    station_id: str
    risk_score: float = Field(..., ge=0, description="Predicted crash rate per 1 000 trips")
    risk_tier: RiskTier
    model_version: str

class NetFlowRequest(BaseModel):
    station_id: str
    date: str = Field(..., example="2025-06-15")

class NetFlowResponse(BaseModel):
    station_id: str
    net_flow: float
    imbalance_class: str   # "importer" | "balanced" | "exporter"
    model_version: str
```

### 6.2 Model loading

`api/model_loader.py` — load once at startup, not per request:

```python
import mlflow.pyfunc
import os

MLFLOW_URI  = os.getenv("MLFLOW_TRACKING_URI", "http://localhost:5000")
RISK_MODEL_ALIAS    = "models:/citibike-risk-model@production"
NETFLOW_MODEL_ALIAS = "models:/citibike-netflow-model@production"

mlflow.set_tracking_uri(MLFLOW_URI)

def load_models():
    risk_model    = mlflow.pyfunc.load_model(RISK_MODEL_ALIAS)
    netflow_model = mlflow.pyfunc.load_model(NETFLOW_MODEL_ALIAS)
    return risk_model, netflow_model
```

### 6.3 FastAPI app

`api/main.py`:

```python
from fastapi import FastAPI, HTTPException
from contextlib import asynccontextmanager
from .schemas import RiskRequest, RiskResponse, NetFlowRequest, NetFlowResponse
from .model_loader import load_models
from src.features.engineer import build_risk_features, build_netflow_features

models = {}

@asynccontextmanager
async def lifespan(app: FastAPI):
    models["risk"], models["netflow"] = load_models()
    yield
    models.clear()

app = FastAPI(title="CitiBike MLOps API", lifespan=lifespan)

@app.get("/health")
def health():
    return {"status": "ok", "models_loaded": len(models) == 2}

@app.get("/model/info")
def model_info():
    return {
        "risk_model": RISK_MODEL_ALIAS,
        "netflow_model": NETFLOW_MODEL_ALIAS,
    }

@app.post("/predict/risk", response_model=RiskResponse)
def predict_risk(request: RiskRequest):
    features = build_risk_features(request.station_id, request.datetime, request.bike_type)
    score = float(models["risk"].predict([features])[0])
    tier = _risk_tier(score)
    return RiskResponse(
        station_id=request.station_id,
        risk_score=round(score, 4),
        risk_tier=tier,
        model_version=RISK_MODEL_ALIAS,
    )

def _risk_tier(score: float) -> str:
    if score < 0.5:   return "Low"
    if score < 1.5:   return "Medium"
    if score < 3.0:   return "High"
    return "Very High"
```

### 6.4 Test the API manually

```bash
curl -X POST http://localhost:8000/predict/risk \
  -H "Content-Type: application/json" \
  -d '{"station_id": "6926.10", "datetime": "2025-06-15T17:30:00", "bike_type": "electric_bike"}'
```

---

## Phase 7 — Monitoring & Drift Detection (Evidently)

### 7.1 Log predictions to a reference store

Every prediction is written to `data/monitoring/predictions.parquet`. At regular intervals, compare the recent distribution to the training distribution.

```python
import pandas as pd
from pathlib import Path

def log_prediction(request: dict, response: dict):
    row = {**request, **response, "logged_at": pd.Timestamp.now()}
    path = Path("data/monitoring/predictions.parquet")
    existing = pd.read_parquet(path) if path.exists() else pd.DataFrame()
    pd.concat([existing, pd.DataFrame([row])]).to_parquet(path, index=False)
```

### 7.2 Evidently drift report

`monitoring/drift.py`:

```python
from evidently.report import Report
from evidently.metrics import DataDriftPreset, TargetDriftPreset
import pandas as pd

def run_drift_report(reference_path: str, current_path: str, output_path: str):
    reference = pd.read_parquet(reference_path)
    current   = pd.read_parquet(current_path)

    report = Report(metrics=[DataDriftPreset(), TargetDriftPreset()])
    report.run(reference_data=reference, current_data=current)
    report.save_html(output_path)

    # Extract PSI for alerting
    result = report.as_dict()
    drifted_features = [
        m["result"]["column_name"]
        for m in result["metrics"]
        if m.get("result", {}).get("drift_detected")
    ]
    return drifted_features
```

Run this as an Airflow task in the `retrain_dag` or as a standalone scheduled task. If `len(drifted_features) > 0`, trigger a retraining run.

---

## Phase 8 — Retraining DAG (Airflow)

`dags/retrain_dag.py` — scheduled weekly; also triggerable manually via Airflow UI:

```python
from airflow.decorators import dag, task
from datetime import datetime

@dag(schedule="@weekly", start_date=datetime(2024, 1, 1), catchup=False)
def retrain_dag():

    @task
    def check_drift() -> bool:
        from monitoring.drift import run_drift_report
        drifted = run_drift_report(
            reference_path="data/monitoring/reference.parquet",
            current_path="data/monitoring/predictions.parquet",
            output_path="data/monitoring/drift_report.html"
        )
        return len(drifted) > 0

    @task
    def build_features():
        """Re-run feature engineering over updated silver tables."""
        ...

    @task
    def train_model(drift_detected: bool):
        """Run MLflow training script. Always retrain on schedule; log drift flag."""
        import subprocess
        subprocess.run(["python", "src/train.py", "--experiment", "citibike-risk"])

    @task
    def promote_if_better():
        """Compare new model vs current production model. Promote if better."""
        from mlflow.tracking import MlflowClient
        client = MlflowClient()
        # Get metrics of new candidate and current production
        # Promote new model to @production alias only if CV deviance improves by > 1%
        ...

    drift = check_drift()
    features = build_features()
    trained = train_model(drift)
    promote_if_better()

retrain_dag()
```

**Key principle:** the DAG always retrains on schedule, but only promotes if the new model strictly improves on the held-out evaluation. This prevents regressions from auto-deploying.

---

## Phase 9 — Testing

### 9.1 Unit tests

`tests/test_features.py`:

```python
import pandas as pd
import numpy as np
from src.features.engineer import cyclical_encode, add_lag_features

def test_cyclical_encode_range():
    s = pd.Series(range(24))
    sin_enc, cos_enc = cyclical_encode(s, 24)
    assert sin_enc.between(-1, 1).all()
    assert cos_enc.between(-1, 1).all()

def test_lag_features_no_leakage():
    """Lag features must not use future values."""
    df = pd.DataFrame({"station_id": ["A"]*10, "date": pd.date_range("2024-01-01", periods=10), "net_flow": range(10)})
    df = add_lag_features(df, "net_flow")
    # First row should have NaN lag (no prior value)
    assert pd.isna(df["net_flow_lag1"].iloc[0])
```

`tests/test_api.py` — FastAPI integration tests using `httpx.AsyncClient`:

```python
import pytest
from httpx import AsyncClient, ASGITransport
from api.main import app

@pytest.mark.asyncio
async def test_health():
    async with AsyncClient(transport=ASGITransport(app=app), base_url="http://test") as client:
        r = await client.get("/health")
    assert r.status_code == 200
    assert r.json()["status"] == "ok"

@pytest.mark.asyncio
async def test_predict_risk_schema():
    """Response must match RiskResponse schema."""
    async with AsyncClient(transport=ASGITransport(app=app), base_url="http://test") as client:
        r = await client.post("/predict/risk", json={
            "station_id": "6926.10",
            "datetime": "2025-06-15T17:30:00",
            "bike_type": "electric_bike"
        })
    assert r.status_code == 200
    body = r.json()
    assert "risk_score" in body
    assert body["risk_score"] >= 0
    assert body["risk_tier"] in ["Low", "Medium", "High", "Very High"]
```

### 9.2 Run the test suite

```bash
uv run pytest tests/ -v
```

---

## Summary Checklist

Use this as a progress tracker as you work through the project.

| Phase | Task | Done |
|---|---|---|
| 0 | Write success metrics before coding | ☐ |
| 0 | Create repo + uv environment | ☐ |
| 0 | Docker Compose stack running locally | ☐ |
| 1 | `ingest_dag` downloads bronze Parquet | ☐ |
| 1 | Silver cleaning pipeline (DuckDB) | ☐ |
| 1 | Gold feature tables written | ☐ |
| 2 | Cyclical + lag feature functions | ☐ |
| 2 | `RiskFeatures` and `NetFlowFeatures` Pydantic schemas | ☐ |
| 3 | MLflow tracking server running | ☐ |
| 3 | Baseline runs logged (formula, persistence) | ☐ |
| 3 | All model variants logged with consistent params/metrics | ☐ |
| 4 | TimeSeriesSplit CV implemented | ☐ |
| 4 | Spatial holdout evaluation | ☐ |
| 4 | Calibration plot for risk model | ☐ |
| 5 | Best model registered in MLflow model registry | ☐ |
| 5 | Model promoted to `@production` alias | ☐ |
| 5 | Model card artifact logged | ☐ |
| 6 | Pydantic request/response schemas | ☐ |
| 6 | FastAPI `lifespan` model loading | ☐ |
| 6 | `/predict/risk` endpoint working | ☐ |
| 6 | `/predict/netflow` endpoint working | ☐ |
| 7 | Prediction logging to Parquet | ☐ |
| 7 | Evidently drift report runs | ☐ |
| 8 | `retrain_dag` orchestrates full pipeline | ☐ |
| 8 | Auto-promotion only if metrics improve | ☐ |
| 9 | Unit tests for feature engineering | ☐ |
| 9 | API integration tests pass | ☐ |

---

## References

## Links

- [[09_projects/_active/citibike-mlops/overview|CitiBike MLOps Overview]]
- [[05_ml_engineering/01_principles_and_lifecycle/ml_lifecycle|ML Lifecycle]]
- [[05_ml_engineering/02_data_engineering/data_pipeline_patterns|Data Pipeline Patterns]]
- [[05_ml_engineering/05_model_development/experiment_tracking|Experiment Tracking]]
- [[05_ml_engineering/06_deployment_and_serving/serving_patterns|Serving Patterns]]
- [[05_ml_engineering/07_monitoring_and_observability/drift_detection|Drift Detection]]
- [[05_ml_engineering/08_continual_learning/retraining_strategies|Retraining Strategies]]
- [[09_projects/_completed/citibike-nyc-axa/overview|CitiBike NYC AXA (Source Analysis)]]
