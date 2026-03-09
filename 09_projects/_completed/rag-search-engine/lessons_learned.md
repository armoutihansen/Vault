---
layer: 09_projects
type: concept
status: evergreen
tags: [retrieval, rag, nlp, search]
created: 2026-03-09
---

# RAG Search Engine — Lessons Learned

## 1. BM25 Is Surprisingly Strong Baseline

BM25 with proper stemming (Porter) and stopword removal outperforms naive TF-IDF and performs competitively with dense retrieval on exact-match and keyword-heavy queries. The tuned default parameters (k1=1.5, b=0.75) generalise well; only tweak them when document length variance is high. The inverted index builds in seconds and fits in memory — deploy it before reaching for embeddings.

**Key insight:** BM25 handles rare, specific terms better than semantic search. A query like "1972 Godfather Coppola" retrieves by exact token overlap; semantic search may anchor on "family drama" and miss the specific film.

## 2. Reciprocal Rank Fusion (RRF) Outperforms Weighted Combination

RRF ($\sum_r \frac{1}{k + \text{rank}_r(d)}$, k=60) is more robust than weighted score combination because it is rank-invariant — scores from BM25 and semantic search live on incomparable scales. Score normalisation can correct this but introduces sensitivity to outliers. Use RRF as the default fusion strategy; only switch to weighted combination when you need to explicitly amplify one modality.

**Key insight:** The `k=60` constant dampens the influence of top-ranked documents, preventing one modality from dominating. Lower k gives more weight to top ranks; raise k to treat all retrieved results more uniformly.

## 3. Search Expansion Before Reranking Is Essential

Retrieving `limit * 500` candidates before reranking and returning the top-`limit` results is not premature over-retrieval — it is the correct architecture. Rerankers have high precision but cannot recover documents the first-stage retriever discarded. The expansion factor of 500 ensures the cross-encoder or LLM reranker has enough signal to improve the final ranking substantially.

**Key insight:** First-stage retrieval optimises for recall; reranking optimises for precision. Conflating the two (small retrieval + reranking) defeats the purpose.

## 4. Reranking Cost Trade-offs Are Significant

Three strategies with different accuracy/latency/cost profiles:

| Strategy | Latency | Cost | Accuracy |
|---|---|---|---|
| Individual (1 LLM call / doc) | High | High | Highest |
| Batch (1 LLM call / page) | Medium | Low | Good |
| Cross-encoder (local) | Low | Zero | Good (domain-specific) |

The cross-encoder (`ms-marco-MiniLM-L6-v2`) offers the best cost-accuracy trade-off for most retrieval tasks. Individual LLM scoring is reserved for high-value ranking decisions where quality trumps latency. Batch scoring is a practical middle ground when API budget matters.

## 5. Query Enhancement Has Asymmetric Value

- **Spell correction**: high ROI for short queries with typos; nearly free via LLM.
- **Query expansion**: adds recall for underspecified queries ("family movie" → adds "animated, Disney, Pixar, children"); may hurt precision for specific queries.
- **Query rewriting**: most valuable for conversational or poorly-structured queries; adds latency.
- **Image-to-query**: CLIP embeds the image directly; the LLM description route is useful when the image is ambiguous or needs domain knowledge to interpret.

**Key insight:** Apply spell correction always. Apply expansion only when initial results are sparse. Avoid rewriting when the original query is already precise.

## 6. Chunking Documents Matters for Semantic Search

Long documents violate the assumption that an embedding represents a single semantic unit. Chunking (splitting documents into fixed-length segments with overlap) ensures the embedding of each chunk captures a coherent idea. The best-matching chunk score is then used as the document score. Without chunking, long-document retrieval systematically degrades as the embedding averages over multiple topics.

**Key insight:** For this project, movie descriptions are short enough that chunking had limited impact. For long-form documents (research papers, books), chunking is non-negotiable.

## 7. Evaluation Requires a Golden Dataset Before You Build

Precision@k and Recall@k are meaningless without hand-curated query–relevant-document pairs. Building the golden dataset (`golden_dataset.json`) early provided a concrete signal throughout development, making it possible to compare BM25 vs semantic vs hybrid vs reranked results objectively. Without it, all qualitative comparisons are anecdotal.

**Key insight:** Invest in even a small golden dataset (20–50 queries) before comparing retrieval strategies. The investment pays off immediately in directional guidance.

## 8. Caching Retrieval Artefacts Is Mandatory

Cold-start latency for semantic search (encoding all documents on first run) is prohibitive. Caching BM25 index (`cache/index.pkl`) and embedding matrices (`cache/*.pkl`) to disk reduces startup from minutes to seconds. Always separate the index-build step from the query step.

## References

## Links

- [[09_projects/_completed/rag-search-engine/overview|RAG Search Engine — Overview]]
- [[06_ai_engineering/03_retrieval_and_rag/index|RAG]]
