---
layer: 03_modeling
type: workflow
status: growing
tags: [algorithm, time-series, training]
created: 2026-03-07
---

# ML and DL Forecasting — Implementation

## Conceptual Counterpart

[[03_modeling/05_time_series/03_ml_and_dl_forecasting/ml_forecasting|ML and DL Forecasting]]

## Setup

```python
import numpy as np
import pandas as pd
import lightgbm as lgb
from sklearn.metrics import mean_absolute_error, mean_squared_error
```

## LightGBM for Time Series (Lag Features)

```python
def make_lag_features(df: pd.DataFrame, target: str, lags: list[int]) -> pd.DataFrame:
    for lag in lags:
        df[f'{target}_lag_{lag}'] = df[target].shift(lag)
    return df.dropna()

# Rolling statistics
df['rolling_mean_7'] = df['value'].shift(1).rolling(7).mean()
df['rolling_std_7']  = df['value'].shift(1).rolling(7).std()

# Calendar features
df['dayofweek'] = df['ds'].dt.dayofweek
df['month']     = df['ds'].dt.month
df['quarter']   = df['ds'].dt.quarter

feature_cols = [c for c in df.columns if c not in ['ds', 'value']]
X, y = df[feature_cols], df['value']

train_size = int(len(df) * 0.8)
X_train, X_val = X.iloc[:train_size], X.iloc[train_size:]
y_train, y_val = y.iloc[:train_size], y.iloc[train_size:]

model = lgb.LGBMRegressor(n_estimators=500, learning_rate=0.05, num_leaves=31)
model.fit(X_train, y_train,
          eval_set=[(X_val, y_val)],
          callbacks=[lgb.early_stopping(50), lgb.log_evaluation(100)])
```

## LSTM Forecasting

```python
import torch
import torch.nn as nn

class LSTMForecaster(nn.Module):
    def __init__(self, input_size: int, hidden_size: int = 64, num_layers: int = 2):
        super().__init__()
        self.lstm = nn.LSTM(input_size, hidden_size, num_layers, batch_first=True, dropout=0.2)
        self.head = nn.Linear(hidden_size, 1)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        out, _ = self.lstm(x)           # (B, T, hidden_size)
        return self.head(out[:, -1, :]) # last timestep → (B, 1)

# Windowed dataset
def create_sequences(data: np.ndarray, seq_len: int) -> tuple[np.ndarray, np.ndarray]:
    X, y = [], []
    for i in range(len(data) - seq_len):
        X.append(data[i:i+seq_len])
        y.append(data[i+seq_len])
    return np.array(X), np.array(y)

seq_len = 30
X_seq, y_seq = create_sequences(series_scaled, seq_len)
```

## Walk-Forward Validation

```python
def walk_forward_validate(model, df: pd.DataFrame, n_splits: int = 5):
    split_size = len(df) // (n_splits + 1)
    errors = []
    for i in range(1, n_splits + 1):
        train = df.iloc[:i * split_size]
        val   = df.iloc[i * split_size:(i + 1) * split_size]
        # fit and predict logic depends on model type
        ...
    return np.mean(errors), np.std(errors)
```

## Trade-offs

| Model | Strengths | Weaknesses |
|---|---|---|
| LightGBM (lag features) | Fast, handles many exogenous features | Requires manual feature engineering |
| LSTM | Learns temporal patterns end-to-end | Slow to train, needs large data |
| N-BEATS | State-of-art univariate, interpretable | Univariate only in base form |
| TFT | Handles covariates, attention-based | Complex to tune |

## References

- Chen & Guestrin (2016). "XGBoost: A Scalable Tree Boosting System." KDD.
- Oreshkin et al. (2020). "N-BEATS: Neural Basis Expansion Analysis." ICLR.
- Lim et al. (2021). "Temporal Fusion Transformers for Interpretable Multi-horizon Time Series Forecasting." IJF.

## Links

- [[03_modeling/05_time_series/03_ml_and_dl_forecasting/ml_forecasting|ML and DL Forecasting]]
- [[03_modeling/05_time_series/03_ml_and_dl_forecasting/index|ML Forecasting Index]]
- [[03_modeling/05_time_series/01_classical_forecasting/time_series_implementation|Classical Time Series Implementation]]
