---
layer: 03_modeling
type: concept
status: seed
tags: [algorithm, tabular]
created: 2026-05-31
---

# Dimensionality Reduction

## Core Idea

Dimensionality reduction maps data from a high-dimensional space $\mathbb{R}^d$ to a lower-dimensional representation $\mathbb{R}^k$ ($k \ll d$) while preserving as much task-relevant structure as possible. It is used to remove noise, reduce computational cost, enable visualisation, and mitigate the curse of dimensionality.

Two families: **linear methods** (PCA, SVD, random projections) that find global linear subspaces, and **non-linear methods** (t-SNE, UMAP) that preserve local neighbourhood structure.

## Mathematical Formulation

### Principal Component Analysis (PCA)

PCA finds the orthogonal directions of maximum variance in the data.

**Covariance matrix:** $\Sigma = \frac{1}{n} X^\top X$ (centred data).

**Eigendecomposition:** $\Sigma = V \Lambda V^\top$, where columns of $V$ are principal components (eigenvectors) and $\Lambda = \text{diag}(\lambda_1, \ldots, \lambda_d)$ with $\lambda_1 \geq \lambda_2 \geq \cdots$.

**Projection to $k$ dimensions:** $Z = X V_k$ where $V_k$ contains the top-$k$ eigenvectors.

**Equivalently via SVD:** $X = U \Sigma V^\top$; the top-$k$ right singular vectors form $V_k$.

**Explained variance ratio:** $\frac{\sum_{i=1}^k \lambda_i}{\sum_{i=1}^d \lambda_i}$ — choose $k$ such that this exceeds 0.95 (or use a scree plot elbow).

```python
from sklearn.decomposition import PCA

pca = PCA(n_components=0.95, svd_solver='full')  # retain 95% variance
X_reduced = pca.fit_transform(X_train)

print(f"Reduced to {pca.n_components_} dimensions")
print(f"Explained variance: {pca.explained_variance_ratio_.cumsum()[-1]:.3f}")
```

### t-SNE (t-Distributed Stochastic Neighbour Embedding)

Non-linear method designed for 2D/3D visualisation of high-dimensional data.

1. Computes pairwise similarities in original space using a Gaussian kernel centred on each point.
2. Computes pairwise similarities in low-dimensional space using a Student-t kernel.
3. Minimises KL divergence between the two similarity distributions via gradient descent.

**Key hyperparameter:** `perplexity` (default 30) — roughly the number of neighbours considered; controls the balance between local and global structure.

```python
from sklearn.manifold import TSNE

tsne = TSNE(n_components=2, perplexity=30, random_state=42)
X_2d = tsne.fit_transform(X)
```

**Caution:** t-SNE distances in the 2D plot do not reflect true distances. Cluster sizes and inter-cluster distances are not meaningful. Use only for visualisation.

### UMAP (Uniform Manifold Approximation and Projection)

Faster than t-SNE, better preserves global structure, and supports `transform()` on new data.

```python
import umap

reducer = umap.UMAP(n_components=2, n_neighbors=15, min_dist=0.1, random_state=42)
X_2d = reducer.fit_transform(X)
```

## Inductive Bias

- **PCA**: assumes the data lies on a linear subspace; maximises variance.
- **t-SNE/UMAP**: assumes the data lies on a non-linear low-dimensional manifold; preserves local topology.

## Training Objective

| Method | Objective |
|---|---|
| PCA | Maximise variance of projections $\Leftrightarrow$ minimise reconstruction error $\|X - XV_k V_k^\top\|^2_F$ |
| t-SNE | Minimise $KL(P \| Q)$ between high-d Gaussian and low-d t-distribution similarities |
| UMAP | Minimise cross-entropy between fuzzy topological representations |

## Strengths

- Removes noise dimensions, improving downstream model robustness.
- Enables visualisation of structure in $> 3$ dimensions.
- Reduces overfitting risk for high-dimensional inputs.
- PCA is fast and fully invertible; useful for compression.

## Weaknesses

- PCA only captures linear structure; fails on curved manifolds.
- t-SNE is slow ($O(n^2)$ naively), non-deterministic, and cannot be applied to new data.
- All methods can obscure structure if $k$ is chosen poorly.

## Variants

- **Kernel PCA**: applies PCA in a kernel-induced feature space for non-linear structure.
- **Sparse PCA**: enforces sparsity in loadings for interpretability.
- **Incremental PCA**: handles large datasets via minibatch updates.
- **Truncated SVD**: equivalent to PCA but works with sparse matrices (no mean centering).
- **Random Projections** (Johnson-Lindenstrauss): extremely fast, approximate dimensionality reduction with theoretical guarantees.

## References

- Jolliffe, I. (2002). *Principal Component Analysis* (2nd ed.). Springer.
- van der Maaten, L. & Hinton, G. (2008). "Visualizing data using t-SNE." *JMLR*.
- McInnes, L. et al. (2018). "UMAP: Uniform Manifold Approximation and Projection." *arXiv:1802.03426*.

## Links

- [[01_foundations/01_linear_algebra/03_eigenvalues/svd|SVD — mathematical foundation of PCA]]
- [[03_modeling/02_unsupervised_learning/01_clustering/unsupervised_learning|Clustering — dimensionality reduction as pre-processing step]]
- [[03_modeling/02_unsupervised_learning/04_representation_learning/representation_learning|Representation Learning — learned vs linear dimensionality reduction]]
- [[02_data_science/04_feature_engineering/feature_engineering|Feature Engineering — when to apply PCA as a preprocessing step]]
