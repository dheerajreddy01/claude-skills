---
schema: "1.0"
name: elasticsearch
version: "1.0.0"
description: Elasticsearch mapping design, sharding strategy, and query DSL performance practices
domain: technology
triggers:
  keywords:
    primary: [elasticsearch, elastic, opensearch, kibana, lucene]
    secondary: [mapping, shard, index, aggregation, analyzer, query DSL]
  context_boost: [full-text search, log search, search index]
  context_penalty: [splunk, postgresql, mongodb]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Elasticsearch

> Mappings are mostly permanent and shard count is a day-one decision — plan before the first document lands

## Applicable Scenarios

- Designing index mappings and choosing analyzers for search fields
- Deciding shard count and sizing strategy for a new index
- Writing or optimizing Query DSL queries and aggregations
- Diagnosing yellow/red cluster health or slow queries
- Handling deep pagination or large result set retrieval

## Core Knowledge

### The Inverted Index

- Elasticsearch (built on Lucene) indexes text by breaking it into terms via an **analyzer** (tokenizer + filters) and building an inverted index — term → list of documents containing it — which is what makes full-text search fast
- **Mapping** defines each field's type and how it's analyzed; unlike a relational schema, mapping is largely set at index-creation/first-document time — most field type changes require reindexing into a new index, not an in-place `ALTER`

### Sharding

- An index is split into **primary shards** (fixed at index creation in most versions — cannot be changed without reindexing) and **replica shards** (copies for HA and read scaling, changeable anytime)
- Shard count is a capacity-planning decision made once: too few shards limits horizontal scaling and can make shards unmanageably large; too many creates per-shard overhead (cluster state size, file handles, per-shard query overhead) that degrades cluster performance even with little data
- A good starting heuristic: size shards to roughly 10-50GB each based on expected index size, not a round number picked arbitrarily

### Query DSL: Query Context vs. Filter Context

| Context | Purpose | Caching | Scoring |
|------|------|------|------|
| **Query context** | "How well does this match?" | Not cached | Computes a relevance score |
| **Filter context** | "Does this match, yes/no?" | Cached by Elasticsearch | No scoring — cheaper |

Using query-context clauses (e.g., unwrapped `match`) for what's actually a binary filter (exact term match, date range) wastes scoring computation and misses out on filter caching — wrap exact-match conditions in a `bool.filter` clause instead of `bool.must`.

### Aggregations

- Aggregations compute over the *matched* document set (after query/filter), similar conceptually to SQL `GROUP BY` — but computed in a distributed way across shards, which means very high-cardinality aggregations (e.g., aggregating on a unique ID field) are expensive and memory-intensive
- `terms` aggregations return approximate results by default in some multi-shard configurations if not enough data is fetched per shard (controlled by `shard_size`) — accuracy vs. cost is a real tradeoff, not automatic

### Pagination

- `from`/`size` pagination becomes expensive and eventually blocked past a default limit (`index.max_result_window`, default 10,000) because deep pagination requires sorting and discarding all preceding results on every shard
- `search_after` (cursor-based, using the last result's sort values) or the Point-in-Time + `search_after` combination scales to arbitrary depth without the same overhead

## Best Practices

1. **Define explicit mappings** rather than relying on dynamic mapping for anything beyond quick prototyping — dynamic mapping can silently create unexpected fields and types
2. **Size shards deliberately** (roughly 10-50GB target) based on expected data volume, not a default or arbitrary round number
3. **Use filter context for exact-match/range conditions**, query context only for actual relevance-scored full-text matching
4. **Use `search_after` for deep pagination**, not `from`/`size` past a shallow depth
5. **Set an index lifecycle policy** (ILM) for time-series/log data so old indices roll over, get downgraded to cheaper storage tiers, or get deleted automatically
6. **Monitor cluster health status** (green/yellow/red) and unassigned shard count as standard operational metrics

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Relying on dynamic mapping for production indices | Define explicit mappings, disable or restrict dynamic mapping (`dynamic: strict` or `false`) where field shape should be controlled |
| Using `bool.must` with `match` for an exact-value filter | Use `bool.filter` for exact matches/ranges — cached, no scoring overhead |
| `from`/`size` pagination going deep into results | Use `search_after` with a Point-in-Time for deep pagination |
| One shard per index regardless of expected data size | Size shards to the expected data volume (~10-50GB per shard as a starting point) |
| No ILM policy on time-series/log indices | Configure ILM to roll over, tier, and delete old indices automatically |

## Sharp Edges

### SE-1: Dynamic Mapping Explosion
- **Severity**: critical
- **Situation**: An index with dynamic mapping enabled ingests documents with highly variable or attacker-influenced field names (e.g., using a user-controlled key as a JSON field name), and the mapping accumulates thousands of unique fields, destabilizing the cluster
- **Cause**: Dynamic mapping automatically creates a new mapped field for every previously-unseen field name — there's no default cap, and cluster state (which must be replicated to every node) grows with total mapped field count across all indices
- **Symptoms**:
  - Cluster state size grows abnormally large; cluster state updates become slow, affecting the whole cluster's responsiveness
  - `mapper_parsing_exception` or `illegal_argument_exception: Limit of total fields ... has been exceeded` once the field cap (default 1000) is hit
- **Solution**: Set `dynamic: strict` or `false` on indices where field shape should be controlled, use explicit mappings for known fields, and if arbitrary key-value data must be stored, use a `flattened` field type (indexes as a single field, not one mapped field per key) instead of relying on dynamic object mapping
- **Details**: → [extended/checklists.md#index-design-checklist]

### SE-2: Over-Sharding Degrading Cluster Performance
- **Severity**: high
- **Situation**: Many small indices (e.g., a new index created per day/tenant with default shard settings) accumulate thousands of tiny shards across the cluster, and cluster-wide operations (cluster state updates, shard allocation) become slow even though total data volume is modest
- **Cause**: Every shard has fixed per-shard overhead (file handles, memory for shard-level data structures, participation in cluster state) — this overhead is roughly constant per shard regardless of how much data it actually holds, so many small shards accumulate overhead disproportionate to the data they store
- **Symptoms**:
  - Cluster health checks and shard allocation operations become slow
  - `too_many_requests` / circuit breaker exceptions during normal operation despite modest total data size
- **Solution**: Right-size shard count for expected data volume (10-50GB per shard target), use index templates with a rollover policy to control shard proliferation, and periodically use `_shrink`/reindex to consolidate over-sharded indices

### SE-3: Deep Pagination Exhausting Memory
- **Severity**: high
- **Situation**: An application paginating deep into search results using `from`/`size` (e.g., `from: 50000, size: 10`) causes high memory usage and latency, and eventually hits the default 10,000-result window limit
- **Cause**: `from`/`size` pagination requires each shard to sort and return `from + size` results to the coordinating node, which then merges and discards everything before `from` — the cost of retrieving "page 5000" is proportional to `from`, not to the page size
- **Symptoms**:
  - Query latency increases the further into results pagination goes
  - `Result window is too large` errors once `from + size` exceeds `index.max_result_window`
- **Solution**: Use `search_after` (with a stable sort and, for point-in-time consistency, a PIT ID) for any pagination that needs to go beyond a shallow depth — it scales with page size, not absolute offset

### SE-4: Immutable Field Types Requiring Full Reindex
- **Severity**: medium
- **Situation**: A field mapped as `text` needs to become `keyword` (or any other type change) for a new query pattern, and there's no in-place way to do it — the only path is creating a new index with the corrected mapping and reindexing all existing data into it
- **Cause**: Lucene segments are immutable once written; changing a field's fundamental type would require rewriting every existing segment, which Elasticsearch doesn't support as an in-place operation
- **Symptoms**:
  - `PUT` mapping request for an existing field's type change fails with a mapper conflict error
  - A schema fix requires standing up a parallel index and a reindex/cutover process rather than a quick change
- **Solution**: Plan mappings carefully upfront (use `keyword` for exact-match/aggregation fields, `text` for full-text search, and `multi-fields` to index the same value both ways when both access patterns are needed), and use an alias pointing at the current index so reindex-and-cutover doesn't require an application-level change

### SE-5: Query Context Used Where Filter Context Belongs
- **Severity**: medium
- **Situation**: A query using `bool.must` with an exact-match `term`/`match` clause for something that's really a binary filter (e.g., `status: "active"`) computes and sums a relevance score unnecessarily, and misses out on Elasticsearch's filter result caching, on every single request
- **Cause**: Query context always computes a `_score`; filter context is cached and skips scoring — but nothing prevents writing an exact-match condition inside `must` (query context) when it should be in `filter`
- **Symptoms**:
  - Aggregate query latency across a cluster is higher than expected for the query complexity involved
  - Filter-like clauses (exact term matches, ranges) appear inside `bool.must` rather than `bool.filter` in query review
- **Solution**: Move exact-match and range conditions that don't need relevance scoring into `bool.filter`, reserving `bool.must`/`should` for genuinely relevance-scored full-text matching

## Recommended Tools

| Category | Tools |
|------|------|
| Query analysis | `_explain` API, Kibana Dev Tools, Profile API |
| Cluster management | `_cluster/health`, `_cat/shards`, ILM policies |
| Ingestion | Logstash, Beats, ingest pipelines |
| Client libraries | Official Elasticsearch clients per language |

## Index Design

**Full checklist**: → [extended/checklists.md#index-design-checklist]

## Related Resources

[Elasticsearch Docs](https://www.elastic.co/docs) | [Elasticsearch: The Definitive Guide](https://www.elastic.co/guide/en/elasticsearch/guide/current/index.html) | [Sizing Elasticsearch](https://www.elastic.co/blog/how-many-shards-should-i-have-in-my-elasticsearch-cluster)

## Related Domains

[[splunk]] | [[kafka]] | [[python]] | [[java]]
