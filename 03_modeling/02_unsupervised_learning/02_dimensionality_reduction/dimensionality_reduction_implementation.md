---
layer: 03_modeling
type: workflow
status: growing
tags: [algorithm, tabular]
created: 2026-06-01
---

# Dimensionality Reduction — Implementation

## Conceptual Counterpart

[[03_modeling/02_unsupervised_learning/02_dimensionality_reduction/dimensionality_reduction|Dimensionality Reduction]] — PCA, truncated SVD, t-SNE, UMAP, random projections, mathematical foundations

## Purpose

Practical implementation of linear and non-linear dimensionality reduction with scikit-learn, UMAP, and scipy. Covers PCA (dense data), Truncated SVD (sparse data), t-SNE and UMAP (2D/3D visualisation), and Sparse Random Projection (ultra-high-dimensional data).

### Examples

#### Setup

```bash
pip install scikit-learn numpy matplotlib scipy umap-learn
```

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.preprocessing import StandardScaler
from sklearn.datasets import load_digits

digits = load_digits()
X, y = digits.data, digits.target          # (1797, 64)
X_scaled = StandardScaler().fit_transform(X)
```

#### PCA — Principal Component Analysis

```python
from sklearn.decomposition import PCA

# Full PCA — inspect explained variance
pca_full = PCA(svd_solver='full')
pca_full.fit(X_scaled)

# Scree plot — cumulative explained variance
cum_var = np.cumsum(pca_full.explained_variance_ratio_)
plt.figure(figsize=(8, 4))
plt.plot(np.arange(1, len(cum_var) + 1), cum_var, marker='o', markersize=3)
plt.axhline(0.95, color='red', linestyle='--', label='95% threshold')
plt.xlabel("Number of components"); plt.ylabel("Cumulative explained variance")
plt.title("PCA Scree Plot"); plt.legend(); plt.tight_layout(); plt.show()

n_95 = int(np.argmax(cum_var >= 0.95)) + 1
print(f"Components for 95% variance: {n_95}")

# Reduce
pca = PCA(n_components=n_95, svd_solver='full')
X_pca = pca.fit_transform(X_scaled)       # (1797, n_95)
print(f"Shape after PCA: {X_pca.shape}")

# Whitening PCA — scales components to unit variance
pca_white = PCA(n_components=n_95, whiten=True)
X_white = pca_white.fit_transform(X_scaled)
# Useful before distance-based methods (e.g., K-Means on PCA features)

# 2D visualisation
pca_2d = PCA(n_components=2)
X_2d = pca_2d.fit_transform(X_scaled)
plt.figure(figsize=(8, 6))
scatter = plt.scatter(X_2d[:, 0], X_2d[:, 1], c=y, cmap='tab10', s=8, alpha=0.7)
plt.colorbar(scatter); plt.title("PCA 2D projection"); plt.tight_layout(); plt.show()
```

**Whitening (`whiten=True`):** divides each component by its standard deviation, giving uncorrelated, unit-variance features. Recommended before K-Means or neural network input if downstream method is sensitive to scale.

#### Truncated SVD — Sparse Matrices

`PCA` requires centering, which destroys sparsity. `TruncatedSVD` applies SVD directly on the sparse matrix without centering — equivalent to PCA on non-centered data. Standard for TF-IDF text features.

```python
from sklearn.decomposition import TruncatedSVD
from scipy.sparse import csr_matrix

X_sparse = csr_matrix(X)   # simulate sparse matrix (e.g., TF-IDF output)

svd = TruncatedSVD(n_components=40, n_iter=7, random_state=42)
X_svd = svd.fit_transform(X_sparse)      # (1797, 40)
print(f"Explained variance ratio (sum): {svd.explained_variance_ratio_.sum():.3f}")
```

For text: pipe `TfidfVectorizer` → `TruncatedSVD` (Latent Semantic Analysis / LSA).

#### t-SNE — 2D Visualisation

```python
from sklearn.manifold import TSNE

# Run on PCA-reduced data for speed (recommended for d > 50)
X_pca_50 = PCA(n_components=50).fit_transform(X_scaled)

tsne = TSNE(
    n_components=2,
    perplexity=30,          # 5–50; try 10, 30, 50
    learning_rate="auto",   # sklearn >= 1.2: 'auto' = max(N/early_exaggeration, 50)
    n_iter=1000,
    init="pca",             # PCA init is more stable than random
    random_state=42
)
X_tsne = tsne.fit_transform(X_pca_50)

plt.figure(figsize=(8, 6))
scatter = plt.scatter(X_tsne[:, 0], X_tsne[:, 1], c=y, cmap='tab10', s=8, alpha=0.7)
plt.colorbar(scatter); plt.title("t-SNE (perplexity=30)"); plt.tight_layout(); plt.show()
```

**Important caveats:**
- t-SNE is non-deterministic even with `random_state` across sklearn versions.
- Inter-cluster distances and cluster sizes in the 2D plot are **not meaningful** — only neighbourhood membership is preserved.
- Does **not** support `transform()` on new data (must refit for each new dataset).
- Use only for visualisation, never as features for downstream models.

#### UMAP — Faster Non-linear Reduction

```bash
pip install umap-learn
```

```python
import umap

reducer = umap.UMAP(
    n_components=2,
    n_neighbors=15,    # local neighbourhood size; smaller → local structure, larger → global
    min_dist=0.1,      # minimum distance in the embedded space; smaller → tighter clusters
    metric='euclidean',
    random_state=42
)
X_umap = reducer.fit_transform(X_scaled)

plt.figure(figsize=(8, 6))
scatter = plt.scatter(X_umap[:, 0], X_umap[:, 1], c=y, cmap='tab10', s=8, alpha=0.7)
plt.colorbar(scatter); plt.title("UMAP (n_neighbors=15, min_dist=0.1)"); plt.tight_layout(); plt.show()

# UMAP supports transform() on new data (unlike t-SNE)
X_new_umap = reducer.transform(X_scaled[:10])
```

**UMAP vs t-SNE:** UMAP is faster ($O(n)$ vs $O(n^2)$), supports `transform()`, and better preserves global structure. Both are primarily for visualisation, but UMAP features can sometimes improve downstream performance.

#### Sparse Random Projection — Very High Dimensional Data

For $d \gg 10^4$ (e.g., bag-of-words, genomics): the Johnson-Lindenstrauss lemma guarantees that distances are preserved to within $\epsilon$ with $k = O(\log n / \epsilon^2)$ dimensions, independent of $d$.

```python
from sklearn.random_projection import SparseRandomProjection
from scipy.sparse import random as sp_random

# Simulate high-dimensional sparse data (n=1000, d=100000)
X_highdim = sp_random(1000, 100_000, density=0.01, format='csr')

rp = SparseRandomProjection(n_components='auto', eps=0.1, random_state=42)
# eps=0.1 → preserve distances within 10%; n_components set automatically via J-L bound
X_projected = rp.fit_transform(X_highdim)
print(f"Original: {X_highdim.shape}, Projected: {X_projected.shape}")
```

`n_components='auto'` uses the Johnson-Lindenstrauss bound to compute the minimum required dimension automatically. `eps` controls the distortion tolerance.

## Architecture

```
High-dimensional input X (n_samples, d_features)
  │
  ├── Dense data (d moderate)
  │     └── StandardScaler → PCA (linear, variance-optimal, invertible)
  │           └── whiten=True if downstream is scale-sensitive
  │
  ├── Sparse data (TF-IDF, genomics)
  │     └── TruncatedSVD (no centering, preserves sparsity)
  │
  ├── Very high-dimensional (d >> 10^4)
  │     └── SparseRandomProjection (J-L, approximate distance preservation)
  │
  └── Visualisation only (2D/3D)
        ├── PCA → t-SNE (for d > 50, pre-reduce with PCA first)
        └── UMAP (faster, supports transform(), better global structure)
```

**Decision rule:**
1. Start with PCA for linear structure and preprocessing.
2. Use Truncated SVD for sparse inputs.
3. Use Random Projection for speed when $d > 50{,}000$.
4. Use t-SNE / UMAP for exploratory 2D visualisation.

## Trade-offs

| Method | Linear | Invertible | transform() | Speed | Use case |
|---|---|---|---|---|---|
| PCA | Yes | Yes | Yes | Fast | Dense features, preprocessing |
| Truncated SVD | Yes | Approx. | Yes | Fast | Sparse matrices, text |
| Random Projection | Yes (random) | No | Yes | Very fast | $d \gg 10^4$, sketching |
| t-SNE | No | No | No | Slow $O(n^2)$ | 2D visualisation only |
| UMAP | No | No | Yes | Fast $O(n)$ | Visualisation, sometimes features |

## References

- sklearn PCA docs: https://scikit-learn.org/stable/modules/decomposition.html#pca
- sklearn random projections: https://scikit-learn.org/stable/modules/random_projection.html
- van der Maaten, L. & Hinton, G. (2008). "Visualizing data using t-SNE." *JMLR*.
- McInnes, L. et al. (2018). "UMAP: Uniform Manifold Approximation and Projection." *arXiv:1802.03426*.

## Links

- [[03_modeling/02_unsupervised_learning/02_dimensionality_reduction/index|Dimensionality Reduction Sublayer Index]]
- [[03_modeling/02_unsupervised_learning/02_dimensionality_reduction/dimensionality_reduction|Dimensionality Reduction (theory)]]
- [[01_foundations/01_linear_algebra/03_eigenvalues/svd|SVD — mathematical foundation of PCA]]
- [[01_foundations/01_linear_algebra/03_eigenvalues/eigenvalues_and_eigenvectors|Eigenvalues and Eigenvectors]]
- [[03_modeling/02_unsupervised_learning/01_clustering/clustering_implementation|Clustering — Implementation (PCA as preprocessing step)]]
