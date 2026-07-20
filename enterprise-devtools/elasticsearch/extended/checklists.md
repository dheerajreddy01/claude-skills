# Elasticsearch — Extended Checklists

## Index Design Checklist

- [ ] Explicit mappings defined for production indices; `dynamic: strict`/`false` set where field shape should be controlled
- [ ] Arbitrary key-value data uses a `flattened` field type instead of relying on dynamic object mapping
- [ ] Shard count sized to expected index data volume (~10-50GB per shard as a starting heuristic), not left at defaults
- [ ] Fields intended for exact-match/aggregation use `keyword`, fields intended for full-text search use `text`; multi-fields used where both are needed
- [ ] Index aliases used so reindex-and-cutover for mapping changes doesn't require application changes
- [ ] Index Lifecycle Management (ILM) configured for time-series/log indices (rollover, tiering, deletion)
- [ ] `total_fields.limit` and other mapping guardrails reviewed for indices ingesting variable-shape documents

## Query Performance Checklist

- [ ] Exact-match/range conditions placed in `bool.filter`, not `bool.must`, when relevance scoring isn't needed
- [ ] Deep pagination uses `search_after` (with Point-in-Time for consistency), not `from`/`size` past a shallow depth
- [ ] High-cardinality aggregations reviewed for `shard_size`/accuracy tradeoffs, not assumed exact by default
- [ ] `_explain` or the Profile API used to diagnose any query suspected of being slow
- [ ] Cluster health (`_cluster/health`) and unassigned shard count monitored as standard operational metrics
- [ ] Circuit breaker trips investigated as capacity/query-shape signals, not just retried past
