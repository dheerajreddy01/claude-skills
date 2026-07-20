# Vector Databases — Extended Checklists

## RAG Quality Checklist

- [ ] Embedding model version tracked as part of index identity; full reindex performed on any model change
- [ ] Distance metric matches what the embedding model was trained/normalized for (usually cosine for text models)
- [ ] Chunking strategy aligned with content structure (paragraphs/sections), not arbitrary character counts
- [ ] Overlapping chunks used where boundary-context loss is a known risk
- [ ] A retrieval evaluation set (known queries + known-relevant passages) exists and is run when chunking/model/index changes are made
- [ ] ANN index parameters tuned and tested at production-scale data volume, not just development scale
- [ ] Pre- vs post-filter behavior of the chosen vector database understood and tested with realistic filter selectivity
- [ ] Retrieval quality monitored in production (not just latency/error rate)

## Migration & Operations Checklist

- [ ] Embedding model upgrades planned as explicit migration projects with cost/time estimated upfront
- [ ] Blue-green reindexing strategy used for model upgrades (build new index fully before cutover), avoiding in-place mixed-embedding states
- [ ] Index namespace/versioning scheme makes an embedding-model mismatch structurally hard to introduce accidentally
- [ ] pgvector vs. dedicated vector database choice reviewed against actual scale needs, not defaulted without consideration
- [ ] Backup/restore or index-rebuild process tested for the chosen vector store
- [ ] Cost monitoring in place for API-based embedding generation at re-embedding time (proportional to corpus size)
