---
layer: 02_data_science
type: concept
status: seed
tags: [data, tabular]
created: 2026-05-31
---

# Data Representation

## Problem Context

Every ML pipeline begins with a choice of how to represent the world as data. The representation determines what information is available to the model, what preprocessing is required, and which algorithms are applicable. Poor representation choices cascade through the entire pipeline.

## Analytical Goal

Select and design data representations that (a) preserve task-relevant information, (b) are compatible with the intended model family, and (c) can be maintained consistently between training and serving.

## Data Considerations

### Tabular Data

The most common format: rows are observations, columns are features. Key properties:
- **Mixed types**: numerical (continuous, discrete) and categorical (nominal, ordinal).
- **Sparsity**: high-cardinality one-hot encoded categoricals produce sparse matrices; prefer embedding layers or target encoding at high cardinality.
- **Schema**: the column set, types, and allowed value ranges form an implicit contract. Formalise it with Pandera or Great Expectations (see [[02_data_science/05_experimentation_and_validation/data_validation|Data Validation]]).

### Text Data

Raw text must be converted to fixed-dimensional vectors:
- **Bag-of-words / TF-IDF**: sparse, interpretable, loses word order.
- **Word embeddings** (Word2Vec, GloVe): dense, captures semantic similarity.
- **Contextual embeddings** (BERT, sentence-transformers): sequence-aware; use for production NLP tasks.

### Image Data

Typically represented as 3-D tensors `(H, W, C)` — height × width × channels (RGB=3). Normalise pixel values to `[0, 1]` or standard score per channel. Augmentation (flip, crop, colour jitter) is applied at training time only.

### Graph Data

Nodes, edges, and optional attributes. Represented as:
- **Adjacency matrix** $A \in \{0,1\}^{n \times n}$ plus feature matrix $X \in \mathbb{R}^{n \times d}$.
- **Edge list** for sparse graphs.
Graph Neural Networks (GNNs) operate directly on these structures.

### Time-Series Data

Ordered sequences indexed by time. Key representation decisions:
- **Fixed-length window vs variable length**: windowing extracts overlapping sub-sequences as tabular features.
- **Frequency**: daily, monthly, irregular? Irregular series require interpolation or event-based models.
- **Stationarity**: many models assume stationary series; differencing or log-transform may be required (see [[03_modeling/05_time_series/01_classical_forecasting/time_series_models|Time Series Models]]).

### Data Contracts

A data contract specifies:
1. **Schema**: column names, types, and cardinalities.
2. **Statistical ranges**: min/max, allowed categories.
3. **Missingness thresholds**: maximum permitted missing rate per column.
4. **Freshness SLA**: maximum lag between data generation and availability.

Contracts serve as a shared agreement between data producers and consumers, enabling automated validation at pipeline boundaries.

## Method / Approach

**Choosing a representation:**
1. Identify the raw information type (structured, text, image, graph, time-series, or mixed).
2. Map each information type to its canonical representation (see above).
3. Check model family compatibility: tree models work naturally with tabular; CNNs with images; RNNs/Transformers with sequences.
4. Design the schema and document it as a data contract.

**Encoding decisions to make before EDA:**
- How are missing values represented? (`NaN`, sentinel value, or absence of row?)
- Are dates stored as strings or timestamps? (Convert to timestamps early.)
- Are IDs stored as integers or strings? (Never use IDs as model features without explicit encoding.)

## Validation / Risks

- **Schema drift**: column types or allowed values change in production without notice → enforce contracts at ingestion.
- **Train/serve skew**: representation differences between training and serving pipelines are a leading cause of production failures → serialize transformers (e.g., sklearn Pipeline) and deploy them alongside the model.
- **Leakage via ID fields**: entity IDs that correlate with the target (e.g., customer IDs assigned in chronological order) can leak temporal information → remove raw IDs from feature matrices.

## Communication Notes

When presenting data to non-technical stakeholders, translate representation choices into business terms:
- "Each row represents one insurance policy at inception" — not "each observation is a feature vector."
- "We represent claims history as a 12-month rolling count" — not "we use a temporal aggregate feature."

## References

- Sculley et al. (2015). "Hidden Technical Debt in Machine Learning Systems." NeurIPS.
- Breck et al. (2019). "Data Validation for Machine Learning." SysML.

## Links

- [[02_data_science/05_experimentation_and_validation/data_validation|Data Validation]]
- [[02_data_science/03_exploratory_data_analysis/exploratory_data_analysis|Exploratory Data Analysis]]
- [[02_data_science/04_feature_engineering/feature_engineering|Feature Engineering]]
- [[03_modeling/05_time_series/01_classical_forecasting/time_series_models|Time Series Models]]
- [[01_foundations/01_linear_algebra/index|Linear Algebra — vector and matrix representations]]
