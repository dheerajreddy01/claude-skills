# Redis — Extended Checklists

## Reliability Checklist

- [ ] `maxmemory` set explicitly, with an eviction policy matched to actual intent (`allkeys-lru`/`lfu` for pure cache, `noeviction` only when write-rejection is the correct failure mode)
- [ ] Every cache key has an explicit TTL unless deliberately meant to be permanent
- [ ] TTLs on bulk-populated/high-traffic keys include random jitter to avoid synchronized expiry stampedes
- [ ] Persistence mode (none/RDB/AOF/both) chosen deliberately based on what data loss on restart would actually mean
- [ ] No production code path uses `KEYS *`; `SCAN` used for any keyspace iteration
- [ ] Check-then-set logic uses atomic primitives (`SET NX`, `INCR`, or Lua scripts) instead of separate `GET`+`SET` calls
- [ ] Redis Cluster or Sentinel configured for any deployment where a single-node outage is unacceptable
- [ ] Memory usage and `DBSIZE` monitored over time to catch unbounded key growth early
- [ ] Slow log (`SLOWLOG`) reviewed periodically for commands with unexpectedly high latency
