---
layer: 03_modeling
type: workflow
status: growing
tags: [algorithm, anomaly-detection]
created: 2026-06-01
---

# Density Estimation — Implementation

## Conceptual Counterpart

[[03_modeling/02_unsupervised_learning/03_density_estimation/density_estimation|Density Estimation]] — KDE, GMM with EM, model selection, anomaly detection

## Purpose

Practical implementation of density estimation and anomaly detection with scikit-learn and scipy. Covers Kernel Density Estimation (KDE), Gaussian Mixture Models (GMM) with BIC/AIC selection, Isolation Forest for high-dimensional anomaly detection, and Local Outlier Factor (LOF) for density-based local anomaly scoring.

### Examples

#### Setup

```bash
pip install scikit-learn numpy matplotlib scipy
```

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.preprocessing import StandardScaler
from sklearn.datasets import make_blobs

# Normal data with 5% anomalies injected
X_normal, _ = make_blobs(n_samples=475, centers=2, cluster_std=0.5, random_state=42)
X_anomaly   = np.random.default_rng(42).uniform(-6, 6, size=(25, 2))
X = np.vstack([X_normal, X_anomaly])
y_true = np.array([1] * 475 + [-1] * 25)   # 1=inlier, -1=outlier
X = StandardScaler().fit_transform(X)
```

#### Kernel Density Estimation (scipy)

```python
from scipy.stats import gaussian_kde
import numpy as np

# gaussian_kde expects (n_features, n_samples) — transpose from sklearn convention
kde = gaussian_kde(X.T, bw_method='scott')   # Scott's rule: n^(-1/(d+4))

# Evaluate log-density at each point
log_density = np.log(kde(X.T))               # shape (n_samples,)

# Anomaly threshold: bottom 5% of density
threshold = np.percentile(log_density, 5)
anomaly_mask = log_density < threshold
print(f"KDE anomalies flagged: {anomaly_mask.sum()}")

# 2D density surface for visualisation
xx, yy = np.meshgrid(np.linspace(-4, 4, 100), np.linspace(-4, 4, 100))
grid = np.vstack([xx.ravel(), yy.ravel()])
zz = kde(grid).reshape(xx.shape)

plt.figure(figsize=(8, 5))
plt.contourf(xx, yy, zz, levels=20, cmap='viridis', alpha=0.7)
plt.scatter(X[:, 0], X[:, 1], c=y_true, cmap='coolwarm', s=10, alpha=0.6, zorder=2)
plt.colorbar(label='density'); plt.title("KDE density surface"); plt.tight_layout(); plt.show()
```

**Bandwidth selection:**
- `bw_method='scott'` (default): $h = n^{-1/(d+4)} \hat\sigma$ — good for near-Gaussian data.
- `bw_method='silverman'`: $h = (n(d+2)/4)^{-1/(d+4)} \hat\sigma$ — slightly wider.
- Pass a float to override: `bw_method=0.2`.
- Cross-validated: use `sklearn.neighbors.KernelDensity` with `GridSearchCV` on the `bandwidth` parameter.

#### GMM — EM Fitting and BIC Model Selection

```python
from sklearn.mixture import GaussianMixture

# BIC/AIC over k — lower is better
k_range = range(1, 9)
bic_scores, aic_scores = [], []

for k in k_range:
    gmm = GaussianMixture(n_components=k, covariance_type='full',
                          n_init=5, random_state=42)
    gmm.fit(X)
    bic_scores.append(gmm.bic(X))
    aic_scores.append(gmm.aic(X))

fig, ax = plt.subplots(figsize=(8, 4))
ax.plot(k_range, bic_scores, 'o-', label='BIC')
ax.plot(k_range, aic_scores, 's--', label='AIC')
ax.set_xlabel("k"); ax.set_ylabel("Score"); ax.legend()
ax.set_title("GMM model selection"); plt.tight_layout(); plt.show()

best_k = k_range[int(np.argmin(bic_scores))]
print(f"Optimal k (BIC): {best_k}")

# Final model
gmm = GaussianMixture(n_components=best_k, covariance_type='full',
                      n_init=10, random_state=42)
gmm.fit(X)

# Log-likelihood per sample (lower → more anomalous)
log_probs   = gmm.score_samples(X)          # shape (n_samples,)
labels      = gmm.predict(X)                 # hard cluster assignment
soft_assign = gmm.predict_proba(X)           # (n_samples, k)

# Anomaly detection via low-density threshold
threshold = np.percentile(log_probs, 5)
anomaly_mask_gmm = log_probs < threshold
print(f"GMM anomalies (bottom 5% log-prob): {anomaly_mask_gmm.sum()}")
```

**`covariance_type` comparison:**

| Type | Constraint | Parameters per component | Use case |
|---|---|---|---|
| `'full'` | Free $\Sigma_k$ | $d(d+1)/2$ | Most flexible; default for small $d$ |
| `'tied'` | Shared $\Sigma$ | $d(d+1)/2$ total | Similar shapes, different means |
| `'diag'` | Diagonal $\Sigma_k$ | $d$ | Uncorrelated features; scalable |
| `'spherical'` | $\sigma_k^2 I$ | $1$ | Simplest; like soft K-Means |

#### Isolation Forest

Isolation Forest isolates anomalies by random recursive binary splits. Anomalies require fewer splits to isolate than normal points. Efficient for high-dimensional data; does not require density estimation.

```python
from sklearn.ensemble import IsolationForest

iso = IsolationForest(
    n_estimators=100,        # number of trees (default 100)
    contamination=0.05,      # expected fraction of anomalies; sets decision threshold
    max_samples='auto',      # subsample size per tree (default: min(256, n))
    random_state=42
)
iso.fit(X)

preds_iso = iso.predict(X)          # 1 = inlier, -1 = anomaly
scores_iso = iso.score_samples(X)   # lower (more negative) → more anomalous
                                    # Note: score_samples returns the anomaly score
                                    #       (negative average path length, normalised)
print(f"Isolation Forest anomalies: {(preds_iso == -1).sum()}")

# Plot anomaly scores
plt.figure(figsize=(8, 5))
sc = plt.scatter(X[:, 0], X[:, 1], c=scores_iso, cmap='RdYlGn', s=15, alpha=0.8)
plt.colorbar(sc, label='Anomaly score (higher = more normal)')
plt.title("Isolation Forest scores"); plt.tight_layout(); plt.show()
```

**Tuning:**
- `contamination`: set to the expected anomaly rate if known; otherwise use 0.05–0.10 and inspect the score distribution.
- `n_estimators=100` is usually sufficient; beyond 200 gives diminishing returns.
- `max_samples='auto'` uses `min(256, n_samples)` — a small subsample per tree is by design (improves isolation).

#### Local Outlier Factor (LOF)

LOF compares a point's local density to that of its neighbours. Points with much lower local density than their neighbours get high LOF scores.

```python
from sklearn.neighbors import LocalOutlierFactor

lof = LocalOutlierFactor(
    n_neighbors=20,           # neighbourhood size; larger is more global
    contamination=0.05,       # fraction flagged as outliers
    novelty=False             # False = fit_predict on training data only
)
preds_lof = lof.fit_predict(X)      # 1 = inlier, -1 = outlier
scores_lof = lof.negative_outlier_factor_  # lower (more negative) → more anomalous

print(f"LOF anomalies: {(preds_lof == -1).sum()}")
```

**`novelty=True` for new data scoring:**

```python
lof_novelty = LocalOutlierFactor(n_neighbors=20, novelty=True)
lof_novelty.fit(X)                   # fit on training data
preds_new = lof_novelty.predict(X_new)
scores_new = lof_novelty.score_samples(X_new)
```

#### Method Comparison on Test Data

```python
from sklearn.metrics import roc_auc_score

# Evaluate each method via ROC-AUC (higher = better)
# (y_true: 1=inlier, -1=outlier as defined above)
print("ROC-AUC comparison (anomaly detection):")
print(f"  KDE (log density):         {roc_auc_score(y_true, log_density):.3f}")
print(f"  GMM (log prob):            {roc_auc_score(y_true, log_probs):.3f}")
print(f"  Isolation Forest (scores): {roc_auc_score(y_true, scores_iso):.3f}")
print(f"  LOF (negative LOF):        {roc_auc_score(y_true, scores_lof):.3f}")
```

## Architecture

```
Input data X (n_samples, n_features)
  │
  ├── KDE (scipy.stats.gaussian_kde)
  │     └── Non-parametric; bandwidth (Scott/Silverman/CV) → log p̂(x)
  │
  ├── GMM (GaussianMixture)
  │     └── EM algorithm → BIC/AIC k selection → log p(x) via score_samples()
  │
  ├── Isolation Forest
  │     └── Random recursive splits → path length → anomaly score
  │         Best for: high-d data, large n, no density assumption needed
  │
  └── LOF
        └── k-NN local density ratio → outlier factor
            Best for: clusters with varying density, low-d data
```

**When to use each:**

| Method | Strengths | Weaknesses | Best for |
|---|---|---|---|
| KDE | No parametric assumption; exact | Slow, degrades in high-d | Low-d, exploratory analysis |
| GMM | Probabilistic; soft anomaly scores; interpretable | Assumes Gaussian components, EM local optima | Moderate-d, when structure is Gaussian-like |
| Isolation Forest | Fast, scalable, high-d robust | Less interpretable | High-d, production anomaly detection |
| LOF | Detects local density anomalies | Slow at inference, tuning-sensitive | Low-d, clusters with varying density |

## References

- sklearn IsolationForest docs: https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.IsolationForest.html
- sklearn GaussianMixture docs: https://scikit-learn.org/stable/modules/mixture.html
- sklearn LOF docs: https://scikit-learn.org/stable/modules/generated/sklearn.neighbors.LocalOutlierFactor.html
- Liu, F.T., Ting, K.M. & Zhou, Z-H. (2008). "Isolation Forest." *ICDM*. — original Isolation Forest paper.

## Links

- [[03_modeling/02_unsupervised_learning/03_density_estimation/index|Density Estimation Sublayer Index]]
- [[03_modeling/02_unsupervised_learning/03_density_estimation/density_estimation|Density Estimation (theory)]]
- [[03_modeling/03_probabilistic_models/02_latent_variable_models/latent_variable_models|Latent Variable Models — GMM as latent variable model]]
- [[03_modeling/02_unsupervised_learning/01_clustering/clustering_implementation|Clustering — Implementation (GMM as soft clustering)]]
