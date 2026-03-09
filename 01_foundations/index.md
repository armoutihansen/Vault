---
layer: 01_foundations
type: index
status: evergreen
tags: []
created: 2026-03-02
---

# Foundations

Timeless theoretical foundations. Notes here answer: *why does this work mathematically?*

## Sublayers

### [[01_foundations/01_linear_algebra/index|01 — Linear Algebra]]

#### Vectors, matrices, linear systems, eigendecomposition

- [[01_foundations/01_linear_algebra/01_vector_spaces/index|Vector Spaces]] — vector space, span, basis, null/column/row space, norms, projections
- [[01_foundations/01_linear_algebra/02_matrices/index|Matrices]] — operations, inner/outer product, inverse, special matrices
- [[01_foundations/01_linear_algebra/03_eigenvalues/index|Eigenvalues]] — determinants, eigenvalues/eigenvectors, diagonalization
- [[01_foundations/01_linear_algebra/04_linear_systems/index|Linear Systems]] — Gaussian elimination, LU decomposition, inverses

### [[01_foundations/02_calculus_and_analysis/index|02 — Calculus & Analysis]]

#### Differentiation, integration, series, ODEs, vector calculus

- [[01_foundations/02_calculus_and_analysis/01_differentiation/index|Differentiation]] — limits, derivatives, chain rule, partial derivatives, Newton's method
- [[01_foundations/02_calculus_and_analysis/02_integration/index|Integration]] — definite/indefinite, FTC, techniques, numerical methods
- [[01_foundations/02_calculus_and_analysis/03_series/index|Series]] — sequences, convergence, power series, Taylor series
- [[01_foundations/02_calculus_and_analysis/04_differential_equations/index|Differential Equations]] — first/second order ODEs, Laplace transform, numerical methods
- [[01_foundations/02_calculus_and_analysis/05_vector_calculus/index|Vector Calculus]] — vectors, multivariable calculus

### [[01_foundations/03_probability_and_statistics/index|03 — Probability & Statistics]]

#### Probability theory, distributions, Bayesian inference, hypothesis testing

- [[01_foundations/03_probability_and_statistics/probability_theory|Probability Theory]] — axioms, conditional probability, Bayes' theorem, expectation
- [[01_foundations/03_probability_and_statistics/probability_distributions|Probability Distributions]] — Bernoulli, Poisson, Gaussian, Beta, Dirichlet, exponential family
- [[01_foundations/03_probability_and_statistics/bayesian_inference|Bayesian Inference]] — prior, posterior, MLE vs MAP, conjugate priors
- [[01_foundations/03_probability_and_statistics/hypothesis_testing|Hypothesis Testing]] — p-values, t-test, chi-squared, confidence intervals, multiple testing

### [[01_foundations/04_optimization/index|04 — Optimization]]

#### Convex optimization, gradient methods, constrained optimization

- [[01_foundations/04_optimization/convex_optimization|Convex Optimization]] — convex sets, functions, KKT conditions, duality
- [[01_foundations/04_optimization/gradient_descent_optimization|Gradient Descent and Variants]] — SGD, momentum, Adam, learning rate schedules
- [[01_foundations/04_optimization/lagrangian_and_constrained_optimization|Lagrangian and Constrained Optimization]] — Lagrange multipliers, dual problem, SVM

### [[01_foundations/05_statistical_learning_theory/index|05 — Statistical Learning Theory]]

#### Generalization, bias–variance, evaluation strategy, transfer learning

- [[01_foundations/05_statistical_learning_theory/supervised_learning|Supervised Learning]] — task setup, loss functions, generalization
- [[01_foundations/05_statistical_learning_theory/pac_learning|PAC Learning]] — PAC framework, sample complexity, agnostic PAC
- [[01_foundations/05_statistical_learning_theory/vc_dimension|VC Dimension]] — shattering, growth function, fundamental theorem
- [[01_foundations/05_statistical_learning_theory/generalization_bounds|Generalization Bounds]] — ERM bound, uniform convergence, Rademacher complexity
- [[01_foundations/05_statistical_learning_theory/bias_variance_analysis|Bias–Variance Analysis]] — decomposition, underfitting vs overfitting
- [[01_foundations/05_statistical_learning_theory/data_splits_and_distribution|Data Splits and Distribution]] — train/dev/test strategy, distribution mismatch
- [[01_foundations/05_statistical_learning_theory/error_analysis|Error Analysis]] — ceiling analysis, error categorization
- [[01_foundations/05_statistical_learning_theory/evaluation_metrics|Evaluation Metrics]] — precision, recall, F1, AUC-ROC
- [[01_foundations/05_statistical_learning_theory/multi_task_learning|Multi-Task Learning]] — shared representations, hard/soft parameter sharing
- [[01_foundations/05_statistical_learning_theory/orthogonalization|Orthogonalization]] — separating tuning concerns
- [[01_foundations/05_statistical_learning_theory/transfer_learning|Transfer Learning]] — pretrain/fine-tune, domain adaptation

### [[01_foundations/06_deep_learning_theory/index|06 — Deep Learning Theory]]

#### Neural network theory: backpropagation, optimization, regularization, architectures

- [[01_foundations/06_deep_learning_theory/neural_network_notation|Neural Network Notation]] — layers, activations, forward pass notation
- [[01_foundations/06_deep_learning_theory/activation_functions|Activation Functions]] — sigmoid, ReLU, GELU, Swish, universal approximation
- [[01_foundations/06_deep_learning_theory/weight_initialization|Weight Initialization]] — Xavier, Kaiming, symmetry breaking
- [[01_foundations/06_deep_learning_theory/backpropagation|Backpropagation]] — computational graph, chain rule, Jacobian chain
- [[01_foundations/06_deep_learning_theory/backpropagation_through_time|Backpropagation Through Time]] — unrolled RNNs, gradient truncation
- [[01_foundations/06_deep_learning_theory/gradient_descent|Gradient Descent]] — SGD, mini-batch, learning rate schedules
- [[01_foundations/06_deep_learning_theory/adaptive_optimizers|Adaptive Optimizers]] — Momentum, RMSProp, Adam, AdamW
- [[01_foundations/06_deep_learning_theory/gradient_checking|Gradient Checking]] — numerical gradient verification
- [[01_foundations/06_deep_learning_theory/dropout|Dropout]] — inverted dropout, MC dropout, ensemble interpretation
- [[01_foundations/06_deep_learning_theory/batch_normalization|Batch Normalization]] — normalise over batch, scale/shift, covariate shift
- [[01_foundations/06_deep_learning_theory/layer_normalization|Layer Normalization]] — normalise over features, Pre-LN, RMSNorm
- [[01_foundations/06_deep_learning_theory/residual_connections|Residual Connections]] — skip connections, gradient highway, ensemble theory
- [[01_foundations/06_deep_learning_theory/cross_entropy_loss|Cross-Entropy Loss]] — categorical and binary cross-entropy
- [[01_foundations/06_deep_learning_theory/triplet_loss|Triplet Loss]] — metric learning, anchor/positive/negative
- [[01_foundations/06_deep_learning_theory/vectorization|Vectorization and Broadcasting]] — batch operations, NumPy/PyTorch broadcasting
- [[01_foundations/06_deep_learning_theory/style_cost_function|Style Cost Function]] — neural style transfer loss

## Links
- [[03_modeling/index|Modeling]]

---

**Navigation:** [[01_foundations/01_linear_algebra/index|Start with Linear Algebra →]]
