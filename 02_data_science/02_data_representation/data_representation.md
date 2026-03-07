---
layer: 02_data_science
type: concept
status: growing
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
- **Missing value representation**: distinguish between MCAR (missing completely at random — safe to impute), MAR (missing at random — missingness depends on observed variables), and MNAR (missing not at random — the fact of missingness carries signal). When using mean or median imputation, always add a binary indicator column (`col_was_missing`) to preserve the MNAR signal; models can then learn that "missing" is itself informative.
- **Feature scaling**: standardisation (subtract mean, divide by std — required for distance-based models such as k-NN, SVM and gradient-based models such as neural networks and logistic regression with regularisation); min-max scaling to `[0, 1]` preserves distribution shape but is sensitive to outliers; robust scaling (subtract median, divide by IQR) is preferred when outliers are present. Tree-based models (decision trees, random forests, gradient boosting) are invariant to any monotone feature transformation and do not require scaling.
- **Categorical encoding**: strategy depends on cardinality of the feature.
  - *Low cardinality (<10 unique values)*: one-hot encoding — produces a binary column per category; sparse and interpretable.
  - *Medium cardinality (10–50)*: ordinal encoding (when there is a natural order) or target encoding (replace category with the mean of the target within that category, using cross-validation folds to prevent leakage).
  - *High cardinality (>50)*: entity embeddings (learned dense vectors during neural network training) or mean-target encoding with out-of-fold estimation to prevent leakage into training data.
  - *Binary categories*: a single integer column (0/1) suffices; no expansion needed.

### Text Data

Raw text must be converted to fixed-dimensional vectors:
- **Bag-of-words / TF-IDF**: sparse, interpretable, loses word order.
- **Word embeddings** (Word2Vec, GloVe): dense, captures semantic similarity.
- **Contextual embeddings** (BERT, sentence-transformers): sequence-aware; use for production NLP tasks.

### Image Data

Typically represented as 3-D tensors `(H, W, C)` — height × width × channels (RGB=3). Normalise pixel values to `[0, 1]` or standard score per channel. Augmentation (flip, crop, colour jitter) is applied at training time only.

### Audio Data

Raw audio is a 1-D waveform `(T,)` sampled at a fixed rate — typically 16 kHz for speech tasks or 44.1 kHz for music. Key representation choices:
- **Spectrograms**: apply the short-time Fourier transform (STFT) to convert the waveform to a 2-D array of frequency × time; the resulting magnitude spectrogram can be processed with standard CNNs.
- **Mel-frequency cepstral coefficients (MFCCs)**: map spectrogram bins onto the mel scale (perceptually uniform) and apply a DCT to decorrelate coefficients. Produces a compact 13–40 dimensional representation per time frame; standard baseline for speech recognition and audio classification.
- **Raw waveform models**: modern architectures (Wav2Vec 2.0, Whisper) learn representations directly from the raw signal, bypassing hand-crafted features. Preferred when sufficient labelled or unlabelled audio is available.
- **Preprocessing**: normalise amplitude to `[-1, 1]` before any transform. Augmentation at training time includes time-stretching, pitch-shifting, additive noise, and SpecAugment (masking contiguous frequency bands and time steps).

### Multimodal Data

Combining multiple modalities (e.g., text + image, tabular + text, audio + video) requires an explicit fusion strategy:
- **Early fusion**: concatenate the raw or lightly processed representations from all modalities into a single vector before any shared model layers. Simple to implement but may lose modality-specific structure and can be dominated by the highest-dimensional modality.
- **Late fusion**: train a separate model per modality and combine their predictions (averaging, stacking, or learned weighting). Easier to debug and retrain independently; loses cross-modal interactions.
- **Attention-based fusion**: use cross-attention layers so each modality can query the representations of others (the approach used in multimodal transformers such as CLIP, Flamingo, and BLIP-2). Most expressive; requires more data and compute.

Additional considerations:
- **Alignment**: representations must correspond to the same entity or event. Misaligned joins (e.g., attaching the wrong image to a text description) introduce label noise that is difficult to detect.
- **Missing modalities at inference time**: design a fallback strategy before deployment — common options are a zero vector of the expected embedding dimension, the corpus mean embedding, or a unimodal model that operates without the missing modality.

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

### Sparse vs Dense Representations

- **Sparse matrices** (e.g., `scipy.sparse.csr_matrix`, `csc_matrix`): store only the non-zero entries; appropriate when most values are zero, as occurs with one-hot encoded text (bag-of-words), TF-IDF matrices, and user–item interaction matrices. Storage complexity is O(nnz) where nnz is the number of non-zero elements. Scikit-learn linear models, tree ensembles (XGBoost, LightGBM), and most scikit-learn transformers accept sparse input natively — pass sparse matrices directly to avoid unnecessary densification.
- **Dense arrays** (NumPy `ndarray`, PyTorch `Tensor`): required for neural networks and GPU operations. Converting a large sparse matrix to dense with `.toarray()` or `.todense()` can cause an OOM error when the matrix is very wide; benchmark memory usage first.
- **Decision rule**: if feature sparsity exceeds ~90 % (i.e., >90 % of values are zero), use a sparse format. Use `scipy.sparse.issparse(X)` to check format at runtime; use `1 - X.nnz / (X.shape[0] * X.shape[1])` to compute sparsity ratio for CSR/CSC matrices.
- **Memory layout**: NumPy defaults to row-major order (C-order), meaning rows are contiguous in memory — efficient when iterating over rows (common in batch processing). Column-major (Fortran-order, `order='F'`) is efficient when iterating over columns. PyTorch tensors default to contiguous row-major; use `.contiguous()` after transpose operations to restore layout.

### Feature Stores

A **feature store** is a shared, versioned repository of pre-computed features consumed by both training pipelines and real-time serving — the standard architectural solution to the train/serve skew problem.

Two complementary views:
- **Offline store** (batch): stores historical feature values for training and backfilling. Typically implemented as a columnar format (Parquet files, BigQuery, Redshift). Supports time-range queries.
- **Online store** (low-latency): stores the most recent feature values for real-time inference. Implemented as a key-value database (Redis, DynamoDB, Cassandra) to achieve sub-millisecond lookup by entity key.

Key design consideration — **point-in-time joins**: when constructing training data, the offline store must join feature values at the exact timestamp of each prediction event (not the current value). Without this, features computed after the prediction time leak future information into training, causing optimistic offline metrics that do not hold in production.

Examples of feature store platforms: **Feast** (open-source, self-hosted), **Hopsworks** (open-source with managed option), **Tecton** (managed), **Vertex AI Feature Store** (GCP-managed).

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
- Kenton & Toutanova (2019). "BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding." NAACL. (contextual text embeddings)
- Zheng & Casari (2018). *Feature Engineering for Machine Learning*. O'Reilly. (categorical encoding and feature scaling)

## Links

- [[02_data_science/05_experimentation_and_validation/data_validation|Data Validation]]
- [[02_data_science/03_exploratory_data_analysis/exploratory_data_analysis|Exploratory Data Analysis]]
- [[02_data_science/04_feature_engineering/feature_engineering|Feature Engineering]]
- [[03_modeling/05_time_series/01_classical_forecasting/time_series_models|Time Series Models]]
- [[01_foundations/01_linear_algebra/index|Linear Algebra — vector and matrix representations]]
