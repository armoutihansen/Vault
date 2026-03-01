# Vault 2026

A personal knowledge management (PKM) system for professional work in machine learning, statistics, and software engineering. Built in [Obsidian](https://obsidian.md).

---

## Purpose

This vault is a structured, long-lived knowledge base — not a note dump. It is organized by **epistemic layer and abstraction level**: each layer answers a distinct question, from *why does this work mathematically?* down to *why are we solving this problem?*

The goal is to accumulate knowledge that compounds over time: timeless theory in foundations, reusable models and algorithms in the modeling layer, proven engineering patterns in the engineering layers, and grounded domain knowledge in applications.

---

## Structure

```
Vault_2026/
├── 00_meta/              # Vault governance: conventions, templates, dashboard, glossary
├── 01_foundations/       # Timeless mathematical and theoretical foundations
├── 02_modeling/          # Model classes and modeling strategies (framework-agnostic)
├── 03_software_engineering/  # Programming, system design, tooling
├── 04_ml_engineering/    # ML productionization: pipelines, deployment, monitoring
├── 05_ai_engineering/    # LLM systems: RAG, fine-tuning, inference, LLMOps
├── 06_applications/      # Domain knowledge: insurance, business context, constraints
├── 07_projects/          # Time-bound execution instances
│   ├── _active/
│   ├── _completed/
│   └── _experimental/
├── 08_reading/           # Structured intake: papers, books, articles, reports
├── 09_logs/              # Operational memory: daily notes, reviews, brain dumps
└── 99_archive/           # Retired content
```

---

## Layer Guide

| Layer | Question it answers | Examples |
|---|---|---|
| `01_foundations` | Why does this work mathematically? | Calculus, linear algebra, probability theory |
| `02_modeling` | How do we model this problem? | GLMs, neural networks, Bayesian inference |
| `03_software_engineering` | How do we build robust software? | System design, Python, Docker, APIs |
| `04_ml_engineering` | How do we productionize ML? | Feature stores, MLflow, CI/CD for ML |
| `05_ai_engineering` | How do we operate LLM systems? | RAG, fine-tuning pipelines, quantization |
| `06_applications` | Why are we solving this problem? | Insurance pricing, fraud detection |
| `07_projects` | What are we executing right now? | Active work, experiments |
| `08_reading` | What have we learned from external sources? | Paper notes, book summaries |
| `09_logs` | What happened today/this week? | Daily notes, reflections |

---

## Foundations (`01_foundations`)

The most developed layer. Contains timeless theory organized into three domains:

### `math/`

**Calculus** (`calculus/`)
- Core: complex numbers, limits, trigonometric functions, natural logarithm, growth/decay, vectors
- `differentiation/` — definition of derivative, differentiation rules (power/product/quotient/sum), chain rule, common derivatives (exp/log/trig), partial derivatives, centered differences
- `integration/` — integrals (definite/indefinite), fundamental theorem of calculus, substitution, integration by parts, partial fractions, trigonometric integrals
- `series/` — sequences and series, geometric series, harmonic and p-series, ratio test, power series, Taylor series, L'Hôpital's rule
- `differential_equations/` — first-order (linear, separable), second-order (homogeneous, characteristic roots, inhomogeneous), superposition principle, Wronskian, Laplace transform, Heaviside/Dirac, ODE systems, numerical methods (Euler, Runge-Kutta)

**Linear Algebra** (`linear_algebra/`)
- `matrices/` — scalars/vectors/matrices, matrix operations, inverse, special matrices, inner/outer products
- `linear_systems/` — Gaussian elimination, RREF, elementary matrices, LU decomposition
- `vector_spaces/` — vector spaces, span/basis/dimension, linear independence, column/null/row spaces, rank, norms, orthogonal projections, Gram-Schmidt, least squares
- `eigenvalues/` — determinants, eigenvalues and eigenvectors, matrix diagonalization

### `deep_learning_theory/` and `statistical_learning_theory/`
Theoretical foundations for deep learning and statistical learning (in development).

---

## Note Types

Notes follow one of two templates (see `00_meta/templates/`):

**Concept note** (`type: concept`):
```
Definition → Intuition → Formal Description → Applications → Trade-offs → Links
```

**Proof note** (`type: proof`):
```
Statement → Assumptions → Proof Sketch → Full Proof → Notes / Intuition → Links
```

All notes carry frontmatter:
```yaml
layer: <top-level folder>
type: concept | proof
status: seed | growing | evergreen
tags: [...]
created: YYYY-MM-DD
```

---

## Conventions

Full conventions are in [`00_meta/conventions.md`](00_meta/conventions.md). Key rules:

- **File names**: lowercase with underscores (`chain_rule.md`, not `ChainRule.md`)
- **Top-level structure is stable** — changes require updating `conventions.md`
- **No duplicate concepts** — if overlap exists, merge
- **Cross-linking**: modeling notes link to foundations; application notes link to modeling and engineering
- **Knowledge flows**: Reading → Logs → Projects → Core layers (foundations/modeling/engineering are stable)

---

## Getting Started

- Start at [`00_meta/dashboard.md`](00_meta/dashboard.md) for an overview of active work
- Browse a layer's `index.md` for entry points into that layer
- See [`00_meta/glossary.md`](00_meta/glossary.md) for term definitions
- Use [`00_meta/conventions.md`](00_meta/conventions.md) before adding new notes
