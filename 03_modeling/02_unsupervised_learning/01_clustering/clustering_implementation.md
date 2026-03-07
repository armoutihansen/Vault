---
layer: 03_modeling
type: workflow
status: growing
tags: [algorithm, clustering]
created: 2026-03-06
---

# Clustering — Implementation

## Conceptual Counterpart

[[03_modeling/02_unsupervised_learning/01_clustering/clustering|Clustering]] — K-Means, DBSCAN, hierarchical clustering, GMM soft clustering

## Purpose

Practical implementation of clustering algorithms with scikit-learn. Covers K-Means (with elbow and silhouette selection), DBSCAN (with eps estimation), Agglomerative clustering, and GMM soft clustering.

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
from sklearn.metrics import silhouette_score

# Synthetic data
X, y_true = make_blobs(n_samples=500, centers=4, cluster_std=0.8, random_state=42)
X = StandardScaler().fit_transform(X)
```

#### K-Means: Elbow + Silhouette Selection

```python
from sklearn.cluster import KMeans

# Elbow method — WCSS vs k
inertias = []
K_range = range(2, 11)
for k in K_range:
    km = KMeans(n_clusters=k, n_init=10, random_state=42)
    km.fit(X)
    inertias.append(km.inertia_)

plt.figure(figsize=(8, 4))
plt.plot(K_range, inertias, marker='o')
plt.xlabel("k"); plt.ylabel("WCSS"); plt.title("Elbow Plot")
plt.tight_layout(); plt.show()

# Silhouette score — higher is better
sil_scores = []
for k in K_range:
    labels = KMeans(n_clusters=k, n_init=10, random_state=42).fit_predict(X)
    sil_scores.append(silhouette_score(X, labels))

best_k = K_range[int(np.argmax(sil_scores))]
print(f"Best k by silhouette: {best_k}  (score: {max(sil_scores):.3f})")

# Final model
km = KMeans(n_clusters=best_k, n_init=10, random_state=42)
labels_km = km.fit_predict(X)
print(f"Cluster centres shape: {km.cluster_centers_.shape}")
```

#### DBSCAN: eps Estimation via k-Distance Graph

```python
from sklearn.cluster import DBSCAN
from sklearn.neighbors import NearestNeighbors

# k-distance graph: sort distances to 5th nearest neighbour
MIN_SAMPLES = 5
nbrs = NearestNeighbors(n_neighbors=MIN_SAMPLES).fit(X)
dists, _ = nbrs.kneighbors(X)
kth_dist = np.sort(dists[:, -1])

plt.figure(figsize=(8, 4))
plt.plot(kth_dist); plt.ylabel(f"{MIN_SAMPLES}-NN distance"); plt.title("k-Distance Graph")
plt.xlabel("Points (sorted)"); plt.tight_layout(); plt.show()
# Pick eps at the visual elbow of this curve

db = DBSCAN(eps=0.5, min_samples=MIN_SAMPLES)
labels_db = db.fit_predict(X)
n_clusters = len(set(labels_db)) - (1 if -1 in labels_db else 0)
n_noise    = (labels_db == -1).sum()
print(f"DBSCAN — clusters: {n_clusters}, noise points: {n_noise}")
```

#### Agglomerative (Hierarchical) Clustering

```python
from sklearn.cluster import AgglomerativeClustering
from scipy.cluster.hierarchy import dendrogram, linkage

# Dendrogram (use a subset for legibility)
Z = linkage(X[:100], method='ward')
plt.figure(figsize=(10, 4))
dendrogram(Z, truncate_mode='level', p=4)
plt.title("Dendrogram (Ward linkage, first 100 points)"); plt.tight_layout(); plt.show()

# Fit with chosen k
agg = AgglomerativeClustering(n_clusters=4, linkage='ward')
labels_agg = agg.fit_predict(X)
print(f"Silhouette (agglomerative): {silhouette_score(X, labels_agg):.3f}")
```

#### GMM: Soft Clustering with BIC Selection

```python
from sklearn.mixture import GaussianMixture

# BIC model selection
bic_scores = []
for k in K_range:
    gmm = GaussianMixture(n_components=k, covariance_type='full',
                          n_init=3, random_state=42)
    gmm.fit(X)
    bic_scores.append(gmm.bic(X))

best_k_gmm = K_range[int(np.argmin(bic_scores))]
print(f"Best k by BIC: {best_k_gmm}")

gmm_final = GaussianMixture(n_components=best_k_gmm, covariance_type='full',
                             n_init=5, random_state=42)
gmm_final.fit(X)
labels_gmm  = gmm_final.predict(X)            # hard assignment
probs_gmm   = gmm_final.predict_proba(X)      # soft (n_samples, k)
log_lik     = gmm_final.score(X)              # mean log-likelihood per sample
print(f"GMM log-likelihood: {log_lik:.3f}")
```

#### Silhouette Analysis Plot

```python
from sklearn.metrics import silhouette_samples
import matplotlib.cm as cm

fig, ax = plt.subplots(figsize=(8, 5))
sample_sil = silhouette_samples(X, labels_km)
y_lower = 10
for i in range(best_k):
    cluster_sil = np.sort(sample_sil[labels_km == i])
    y_upper = y_lower + len(cluster_sil)
    color = cm.nipy_spectral(float(i) / best_k)
    ax.fill_betweenx(np.arange(y_lower, y_upper), 0, cluster_sil,
                     facecolor=color, alpha=0.7)
    y_lower = y_upper + 10
ax.axvline(x=np.mean(sample_sil), color="red", linestyle="--")
ax.set_xlabel("Silhouette coefficient"); ax.set_ylabel("Cluster")
ax.set_title(f"Silhouette plot (k={best_k})")
plt.tight_layout(); plt.show()
```

## Architecture

```
Raw features
  └── StandardScaler
        ├── K-Means ──────── inertia (elbow) + silhouette → choose k
        ├── DBSCAN ───────── k-distance graph → choose eps → labels (-1 = noise)
        ├── Agglomerative ── linkage matrix (Ward) → dendrogram → cut at k
        └── GMM ─────────── EM algorithm → BIC/AIC → soft probabilities
```

**Key hyperparameters:**

| Algorithm | Primary hyperparameter | Selection method |
|---|---|---|
| K-Means | `n_clusters` | Elbow / silhouette score |
| DBSCAN | `eps`, `min_samples` | k-distance graph, rule of thumb |
| Agglomerative | `n_clusters`, `linkage` | Dendrogram visual cut |
| GMM | `n_components`, `covariance_type` | BIC / AIC |

**Preprocessing:** always `StandardScaler` (or `RobustScaler`) before distance-based methods (K-Means, DBSCAN, Agglomerative). GMM is technically scale-sensitive too.

## Trade-offs

- K-Means is the fastest but assumes spherical, equal-size clusters; fails on ring or crescent shapes.
- DBSCAN handles arbitrary shapes and labels outliers explicitly; sensitive to `eps` and degrades in high dimensions.
- Agglomerative Ward clustering is reliable for small to medium datasets and gives the full dendrogram; $O(n^2 \log n)$ — avoid above $n \approx 10^4$.
- GMM supports soft assignments and ellipsoidal clusters; EM can converge to local optima (use `n_init > 1`).

## References

- sklearn clustering user guide: https://scikit-learn.org/stable/modules/clustering.html
- sklearn `GaussianMixture`: https://scikit-learn.org/stable/modules/mixture.html

## Links

- [[03_modeling/02_unsupervised_learning/01_clustering/clustering|Clustering (theory)]]
- [[03_modeling/02_unsupervised_learning/03_density_estimation/density_estimation_implementation|Density Estimation — Implementation (GMM, Isolation Forest, LOF)]]
- [[02_data_science/03_exploratory_data_analysis/exploratory_data_analysis|Exploratory Data Analysis]]
- [[03_modeling/07_evaluation_and_model_selection/evaluation_and_validation|Evaluation and Validation — silhouette, metrics]]
