---
layer: 09_projects
type: project
status: completed
owner: jesper
created: 2026-03-09
tags: [retrieval, rag, nlp, search, multimodal]
---

# RAG Search Engine

A hybrid search engine for movie data implementing **BM25 keyword search**, **semantic embedding search**, and **CLIP multimodal search**, fused with Reciprocal Rank Fusion (RRF) or weighted combination. Layered on top is a full AI pipeline: Gemini-powered query enhancement, three reranking strategies, and a Retrieval-Augmented Generation (RAG) module for question answering and summarisation.

**Repository:** https://github.com/armoutihansen/rag-search-engine

## Goal

Build a production-grade RAG search pipeline from scratch — implementing each component (inverted index, embedding search, query rewriting, reranking, generation) manually to understand the trade-offs between approaches rather than relying on a single end-to-end framework.

## Scope (In / Out)

**In:**
- BM25 inverted index with Porter stemming and stopword removal
- Sentence-transformer semantic search (`all-MiniLM-L6-v2`) with chunked documents
- CLIP (`ViT-B-32`) multimodal image-based search
- Hybrid fusion: weighted α-combination and Reciprocal Rank Fusion (RRF)
- AI query enhancement: spell correction, rewrite, expansion, and image-to-query (Gemini 2.5 Flash)
- Three reranking strategies: individual scoring, batch ranking, cross-encoder
- RAG module: Q&A, summarisation, and citation-based answer generation
- Evaluation framework: Precision@k, Recall@k, F1 against golden dataset
- Persistent cache (index + embeddings to disk)

**Out:**
- Live crawling or document ingestion pipeline
- Multi-turn conversational search
- Fine-tuned retrieval models
- Production deployment (no API server)
- Datasets beyond movies

## Deliverables

| Artefact | Description |
|---|---|
| `cli/lib/keyword_search.py` | BM25 inverted index with stemming, stopwords, score normalisation |
| `cli/lib/semantic_search.py` | Chunked sentence-transformer search with embedding caching |
| `cli/lib/multimodal_search.py` | CLIP image embedding search |
| `cli/lib/hybrid_search.py` | RRF and weighted fusion; Gemini query enhancement and reranking |
| `cli/lib/augmented_generation.py` | RAG — Q&A, summaries, citations from retrieved context |
| `cli/*_cli.py` | CLI entry points for each search mode |
| `cli/evaluation_cli.py` | Evaluation harness — Precision@k, Recall@k, F1 |
| `data/golden_dataset.json` | Hand-curated test cases for evaluation |
| `tests/` | pytest suite covering all core modules |

## Data

| Dataset | Description |
|---|---|
| `data/movies.json` | Movie corpus: titles, descriptions, genres, poster URLs |
| `data/golden_dataset.json` | Curated query–relevant-document pairs for evaluation |

The movie dataset serves as a controlled, semantically rich domain for testing retrieval strategies. Poster URLs enable multimodal image-based search.

## Architecture

### Retrieval Layer

| Component | Method | Model / Params |
|---|---|---|
| Keyword | BM25 | k1=1.5, b=0.75, Porter stemmer |
| Semantic | Dense retrieval | `all-MiniLM-L6-v2` (384-dim), chunked docs |
| Multimodal | Image embedding | CLIP `ViT-B-32` |
| Fusion | RRF | $\sum_r \frac{1}{k + \text{rank}_r(d)}$, k=60 |
| Fusion | Weighted | $\alpha \cdot s_{\text{BM25}} + (1-\alpha) \cdot s_{\text{sem}}$ |

### AI Enhancement Pipeline

| Stage | Method | Notes |
|---|---|---|
| Query enhancement | Gemini 2.5 Flash | spell correction, rewrite, expansion |
| Image-to-query | Gemini 2.5 Flash + CLIP | describe image, embed description |
| Reranking (individual) | Gemini 2.5 Flash | 1 API call per document, 0–10 score |
| Reranking (batch) | Gemini 2.5 Flash | 1 API call per page, ranked list output |
| Reranking (cross-encoder) | `cross-encoder/ms-marco-MiniLM-L6-v2` | Local inference, no API cost |
| RAG generation | Gemini 2.5 Flash | Q&A, summaries, citation anchors |

### Evaluation

Metrics computed against `golden_dataset.json`:
- Precision@k: fraction of top-k results that are relevant
- Recall@k: fraction of relevant documents found in top-k
- F1@k: harmonic mean of precision and recall at k

## Engineering

| Concern | Choice |
|---|---|
| Language | Python 3.13 |
| LLM | Gemini 2.5 Flash (`google-genai>=1.65.0`) |
| Semantic model | `sentence-transformers>=5.2.3` |
| Multimodal model | CLIP via `sentence-transformers` |
| Text processing | `nltk==3.9.1` (Porter stemmer, stopwords) |
| Numerics | `numpy>=2.4.2` |
| Env management | `uv` + `.env` via `python-dotenv` |
| Caching | `cache/*.pkl` — persists BM25 index + embeddings |
| Testing | `pytest>=8.0.0` |
| Expansion factor | 500× initial retrieval before reranking |

## Timeline

Completed (learning / portfolio project).

## References

- Robertson, S. E., & Zaragoza, H. (2009). *The Probabilistic Relevance Framework: BM25 and Beyond.* Foundations and Trends in Information Retrieval.
- Lewis, P., et al. (2020). *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks.* NeurIPS.
- Radford, A., et al. (2021). *Learning Transferable Visual Models From Natural Language Supervision (CLIP).* ICML.

## Links

- [[09_projects/_completed/rag-search-engine/lessons_learned|Lessons Learned]]
- [[06_ai_engineering/03_retrieval_and_rag/index|RAG]]
- [[06_ai_engineering/01_foundation_models/index|Foundation Models]]
- [[03_modeling/02_unsupervised_learning/04_representation_learning/representation_learning|Representation Learning]]
