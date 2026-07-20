---
schema: "1.0"
name: vector-databases
version: "1.0.0"
description: Vector database (Pinecone/Weaviate/pgvector) embedding search, ANN indexing, and RAG chunking practices
domain: technology
triggers:
  keywords:
    primary: [vector database, pinecone, weaviate, pgvector, embeddings]
    secondary: [similarity search, ANN index, HNSW, RAG, chunking]
  context_boost: [retrieval augmented generation, semantic search, LLM application]
  context_penalty: [relational query, keyword search]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Vector Databases (Pinecone / Weaviate / pgvector)

> Similarity search is only as good as the embedding model consistency and chunking strategy behind it

## Applicable Scenarios

- Designing a retrieval-augmented generation (RAG) pipeline's storage and search layer
- Choosing between a dedicated vector database and `pgvector` in an existing Postgres instance
- Tuning approximate nearest neighbor (ANN) index parameters for the recall/speed tradeoff
- Designing chunking strategy for documents fed into a vector store
- Diagnosing poor search relevance that isn't surfacing as an obvious error

## Core Knowledge

### Embeddings & Similarity

- An **embedding** is a high-dimensional vector representation of text (or other content) produced by a specific model — semantically similar content produces vectors that are close together by some distance metric
- Common distance metrics: **cosine similarity** (angle between vectors, ignores magnitude), **dot product**, **Euclidean distance** — the metric must match what the embedding model was trained/normalized for, or similarity scores become meaningless
- Embeddings from *different models* (or different versions of the same model) are **not comparable** — there's no shared coordinate space across models

### Approximate Nearest Neighbor (ANN) Indexes

| Index Type | Tradeoff |
|------|------|
| **HNSW** (Hierarchical Navigable Small World) | Fast, high recall, more memory-intensive — common default in Pinecone/Weaviate/pgvector |
| **IVF** (Inverted File Index) | Lower memory, tunable recall via number of probes, generally faster to build |

ANN indexes trade exact correctness for speed — "approximate" means a query might miss the true nearest neighbor in exchange for sub-linear search time at scale. Recall (how often the true nearest neighbors are actually returned) is a tunable parameter, not a fixed guarantee.

### Metadata Filtering

- Real-world queries usually combine vector similarity with structured filters (e.g., "similar documents, but only from this tenant, published after this date")
- How filtering interacts with the ANN index matters a lot: **pre-filtering** (filter first, then search only the filtered subset) vs **post-filtering** (search broadly, then discard non-matching results) — post-filtering with a highly selective filter can return far fewer results than requested, or effectively degrade to a full scan

### RAG Chunking

- Documents fed into a vector store for RAG must be split into **chunks** small enough to embed meaningfully and fit within a retrieval budget, but large enough to preserve context
- Chunk size is a direct lever on retrieval quality: too large and irrelevant content dilutes the embedding's specificity; too small and necessary surrounding context is lost, even if the exact matching sentence is retrieved
- Overlapping chunks (a sliding window with overlap between consecutive chunks) mitigate the "context cut off at a chunk boundary" problem at the cost of redundant storage

### pgvector vs. Dedicated Vector Databases

| Factor | pgvector (in Postgres) | Dedicated (Pinecone/Weaviate) |
|------|------|------|
| **Operational simplicity** | One less system to run if already using Postgres | Additional infrastructure to operate/manage |
| **Scale** | Fine up to moderate vector counts; scales with Postgres itself | Purpose-built for very large-scale vector search |
| **Joins with relational data** | Native, in the same query | Requires application-level joining across systems |

## Best Practices

1. **Never mix embeddings from different models/versions** in the same index — reindex fully when upgrading the embedding model
2. **Choose the distance metric that matches the embedding model's training** (most modern text embedding models are optimized for cosine similarity)
3. **Tune and test ANN recall at production data scale**, not just on a small development dataset
4. **Design chunk size deliberately** based on the content type and retrieval use case, and evaluate retrieval quality, not just "it returns something"
5. **Understand pre- vs post-filtering behavior** of the chosen vector database when combining metadata filters with similarity search
6. **Start with `pgvector`** if already running Postgres and scale is moderate — only move to a dedicated vector database when the operational tradeoff is justified by actual scale needs

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Querying an index with vectors from a different embedding model than what indexed the data | Reindex fully whenever the embedding model changes; never mix model versions in one index |
| Chunking documents by a fixed character count with no regard for content structure | Chunk along semantic boundaries (paragraphs, sections) with deliberate size/overlap tuning |
| Assuming ANN search recall is exact/guaranteed | Tune and test recall explicitly at realistic data scale; understand it's a tunable tradeoff |
| Applying a highly selective metadata filter without understanding pre- vs post-filter behavior | Verify how the chosen vector database handles filtered search, and test with realistic filter selectivity |
| No evaluation process for retrieval quality | Build a small evaluation set of queries with known-relevant results and measure retrieval quality changes over time |

## Sharp Edges

### SE-1: Embedding Model Mismatch Silently Degrading Search Quality
- **Severity**: critical
- **Situation**: A vector index built with embeddings from one model (or model version) is queried using vectors from a different model — the search still returns results and doesn't error, but relevance is silently poor
- **Cause**: Embeddings from different models occupy entirely different, incompatible vector spaces — there's no shared meaning to "distance" between a vector from model A and one from model B, but the database has no way to detect or reject this mismatch
- **Symptoms**:
  - Search results are consistently irrelevant or oddly random, with no errors logged
  - The issue coincides with an embedding model upgrade where old data wasn't reindexed
- **Solution**: Treat embedding model version as part of the index's identity — reindex fully (not incrementally) whenever the embedding model changes, and consider versioning index names/namespaces to make a stale-model mismatch structurally impossible rather than just a process discipline
- **Details**: → [extended/checklists.md#rag-quality-checklist]

### SE-2: Chunking Strategy Degrading RAG Quality Without Obvious Errors
- **Severity**: high
- **Situation**: A RAG system's retrieval "works" (returns chunks, no errors) but generated answers are frequently incomplete or subtly wrong, and the root cause turns out to be chunk boundaries cutting through the exact context needed to answer correctly
- **Cause**: Chunking is a lossy operation — the chunk size and boundary choice determines what context is available for embedding and retrieval, and a poor choice degrades answer quality in a way that looks like a generation problem, not a retrieval problem
- **Symptoms**:
  - LLM-generated answers are inconsistent in quality for questions that should have clear answers in the source documents
  - Manually inspecting retrieved chunks shows relevant information split across chunk boundaries
- **Solution**: Design chunking around the content's natural structure (paragraphs, sections) rather than arbitrary character counts, use overlapping chunks to reduce boundary loss, and build a retrieval evaluation set (known queries with known-relevant source passages) to measure the actual impact of chunking changes

### SE-3: ANN Recall Degrading Only at Production Scale
- **Severity**: high
- **Situation**: Search quality looks fine during development on a small dataset, but once the index grows to production data volume, relevant results start getting missed — because the ANN index's recall characteristics were never actually tested at that scale
- **Cause**: ANN index parameters (e.g., HNSW's `ef_search`, IVF's number of probes) trade recall for speed, and the tradeoff curve isn't linear — a configuration that gives near-perfect recall on a small index can have meaningfully worse recall on a much larger one without any error or warning
- **Symptoms**:
  - Known-relevant documents stop appearing in top-K search results as the index grows, with no corresponding code change
  - Recall only becomes visible as a problem through user-reported "the AI didn't know about X" complaints, not through monitoring
- **Solution**: Test and tune ANN parameters against a dataset sized close to production scale, monitor a retrieval quality metric in production (not just latency/error rate), and re-evaluate index parameters as data volume grows significantly

### SE-4: Metadata Filter and ANN Index Interaction Returning Too Few Results
- **Severity**: medium
- **Situation**: A query combining vector similarity with a highly selective metadata filter (e.g., a specific tenant with very few documents) returns far fewer results than requested, or none, even though matching documents exist
- **Cause**: Depending on the vector database's filtering implementation, a post-filter approach (search broadly, then discard non-matching) can exhaust the ANN search's candidate pool before finding enough post-filter matches, especially when the filter is highly selective relative to the whole index
- **Symptoms**:
  - Filtered queries against a small tenant/segment return suspiciously few or zero results despite matching data existing
  - Increasing the search's candidate pool size (if configurable) fixes the symptom, confirming a filtering interaction issue
- **Solution**: Understand whether the chosen vector database does pre-filtering or post-filtering (and whether it's configurable), increase candidate pool size for highly selective filters if using post-filtering, or consider a pre-filtering-capable database/index structure when filter selectivity is a known, common pattern

### SE-5: Re-Embedding Cost and Downtime Not Planned for Model Upgrades
- **Severity**: medium
- **Situation**: A decision to upgrade the embedding model (for better quality) turns into an unplanned, expensive, and disruptive project because nobody accounted for the cost and time of re-embedding the entire existing corpus and the downtime/dual-write complexity of cutting over
- **Cause**: Re-embedding an entire corpus means re-running every document through the new model (real compute/API cost, proportional to corpus size) and then reindexing — this is often orders of magnitude more work than the initial indexing, and the cutover requires either downtime or a dual-index strategy
- **Symptoms**:
  - An embedding model upgrade that seemed like a simple config change turns into a multi-day migration project
  - Search quality temporarily degrades during a botched in-place migration that mixed old and new embeddings
- **Solution**: Plan embedding model upgrades as explicit migration projects — estimate re-embedding cost/time upfront, use a blue-green approach (build the new index fully before cutting over reads) rather than in-place migration, and budget for this recurring cost when choosing how often to chase embedding model improvements

## Recommended Tools

| Category | Tools |
|------|------|
| Dedicated vector databases | Pinecone, Weaviate, Qdrant, Milvus |
| Postgres-native | pgvector |
| Embedding evaluation | RAGAS, custom retrieval eval harnesses |
| Orchestration | LangChain, LlamaIndex |

## RAG Quality

**Full checklist**: → [extended/checklists.md#rag-quality-checklist]

## Related Resources

[pgvector Docs](https://github.com/pgvector/pgvector) | [Pinecone Learning Center](https://www.pinecone.io/learn/) | [Weaviate Docs](https://weaviate.io/developers/weaviate)

## Related Domains

[[postgresql]] | [[python]] | [[elasticsearch]]
