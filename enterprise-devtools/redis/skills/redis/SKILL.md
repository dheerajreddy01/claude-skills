---
schema: "1.0"
name: redis
version: "1.0.0"
description: Redis data structures, eviction/persistence strategy, and cache reliability practices
domain: technology
triggers:
  keywords:
    primary: [redis, cache, key-value, in-memory database]
    secondary: [TTL, eviction policy, pub/sub, sorted set, redis cluster]
  context_boost: [caching layer, session store, rate limiting]
  context_penalty: [postgresql, mongodb, relational database]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Redis

> Fast because it's in memory — which means every capacity and durability decision is explicit, not automatic

## Applicable Scenarios

- Designing a caching layer, session store, or rate limiter with Redis
- Choosing data structures (string, hash, list, set, sorted set) for a use case
- Configuring persistence (RDB/AOF) and eviction policy
- Diagnosing memory growth, latency spikes, or cache stampedes
- Setting up Redis Cluster or Sentinel for HA

## Core Knowledge

### Data Structures

| Structure | Use Case |
|------|------|
| **String** | Simple cache values, counters (`INCR`) |
| **Hash** | Object-like records (user profile fields) without separate keys per field |
| **List** | Queues, recent-activity feeds (ordered, push/pop from either end) |
| **Set** | Unique membership tests, tags |
| **Sorted Set (ZSet)** | Leaderboards, rate limiting windows, anything needing ranked/range queries |
| **Stream** | Append-only event log with consumer groups (lightweight message queue) |

Choosing the right structure avoids reinventing it in application code — e.g., a leaderboard is a sorted set with `ZADD`/`ZRANGE`, not a set of strings parsed and sorted in the app.

### Persistence Options

| Mode | Durability | Performance Cost |
|------|------|------|
| **None** | Pure cache — data lost on restart | None |
| **RDB** (snapshotting) | Point-in-time snapshots at intervals — can lose data since the last snapshot | Low, periodic |
| **AOF** (append-only file) | Logs every write — configurable fsync policy trades durability for throughput | Higher, continuous |
| **RDB + AOF** | Combines both for faster restarts with AOF's durability | Highest |

Redis defaults are tuned for cache use cases, not durable primary-datastore use — if Redis is the system of record for anything, persistence must be configured deliberately, not assumed.

### Eviction Policies

- When `maxmemory` is reached, Redis's behavior depends on the configured **eviction policy** (`noeviction`, `allkeys-lru`, `volatile-lru`, `allkeys-lfu`, etc.)
- `noeviction` (a common default) makes Redis **reject writes** once memory is full, rather than evicting anything — this is correct for some use cases (session data you never want silently dropped) and catastrophic for others (a cache that should just evict old entries)

### Expiration (TTL)

- `EXPIRE`/`SETEX` attach a TTL to a key; without one, a key lives forever unless explicitly deleted or evicted
- Expired keys are removed lazily (on access) and via a periodic active-expiration cycle — a key can technically still exist briefly past its logical expiration under specific timing

## Best Practices

1. **Set a `maxmemory` and an eviction policy deliberately** — don't run with unbounded memory growth as the implicit plan
2. **Always set a TTL on cache entries** unless a key is genuinely meant to be permanent
3. **Use the data structure that matches the access pattern** (sorted sets for ranked data, hashes for object-like records) instead of encoding everything as strings
4. **Never run `KEYS *` in production** — it's O(N) and blocks the single-threaded event loop; use `SCAN` for iteration
5. **Add jitter to TTLs** for high-traffic cache keys to avoid many keys expiring simultaneously (stampede)
6. **Configure persistence based on what Redis is actually storing** — pure cache vs. anything you'd be upset to lose

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| `KEYS *` used for iteration or debugging in production | Use `SCAN` (cursor-based, non-blocking) instead |
| No TTL set on cache entries | Set an explicit TTL matched to how stale the data is allowed to get |
| Treating Redis as a durable primary datastore with default (or no) persistence | Configure AOF (or RDB+AOF) explicitly if data loss on restart is unacceptable |
| Check-then-set logic done as two separate commands (`GET` then `SET`) | Use an atomic command (`SET key val NX`, `INCR`, or a Lua script) to avoid race conditions |
| All cache keys sharing the exact same TTL | Add random jitter to TTLs to spread out expiration and avoid a stampede |

## Sharp Edges

### SE-1: No Eviction Policy Set, Causing OOM Under Memory Pressure
- **Severity**: critical
- **Situation**: Redis is configured with `maxmemory` set but eviction policy left at the default `noeviction`, and once memory fills up, all further writes start failing with `OOM command not allowed`
- **Cause**: `noeviction` means Redis refuses new writes rather than silently dropping old data once at capacity — appropriate for some workloads, but a production outage if the intent was "cache, evict old stuff automatically"
- **Symptoms**:
  - Writes start failing with an explicit OOM error once memory hits the configured cap
  - The service using Redis as a cache starts throwing errors instead of gracefully falling back to the source of truth
- **Solution**: Choose an eviction policy matching actual intent — `allkeys-lru`/`allkeys-lfu` for a pure cache that should evict automatically, `noeviction` only when write-rejection is genuinely preferable to silent data loss (and the application handles that failure mode gracefully)
- **Details**: → [extended/checklists.md#reliability-checklist]

### SE-2: Cache Stampede on Synchronized Expiry
- **Severity**: high
- **Situation**: A popular cache key expires, and a burst of concurrent requests all miss the cache simultaneously and hit the backing datastore/expensive computation at once, spiking load and sometimes cascading into a broader outage
- **Cause**: Many cache entries populated at the same time (e.g., a bulk warm-up) with identical TTLs expire at the exact same moment, and nothing coordinates the resulting cache-miss requests
- **Symptoms**:
  - Periodic, synchronized spikes in backend database/service load correlated with cache TTL intervals
  - Latency spikes recur at a regular cadence matching the TTL
- **Solution**: Add random jitter to TTLs so keys don't expire in lockstep, and use a locking/single-flight pattern (only one request recomputes the value, others wait or serve stale) for expensive-to-recompute keys

### SE-3: `KEYS *` Blocking the Event Loop in Production
- **Severity**: high
- **Situation**: A debugging command or a poorly written maintenance script runs `KEYS *` (or a broad pattern) against a production Redis instance with millions of keys, and it blocks all other operations for the duration
- **Cause**: Redis is single-threaded for command execution, and `KEYS` is O(N) with no incremental yielding — it holds the event loop until it fully scans the keyspace
- **Symptoms**:
  - A sudden, sharp latency spike across all Redis-dependent services, correlated with a manual debugging session or script run
- **Solution**: Always use `SCAN` (which iterates incrementally without blocking) instead of `KEYS` for anything beyond a tiny, known-safe development instance

### SE-4: Non-Atomic Check-Then-Set Race Conditions
- **Severity**: medium
- **Situation**: Application code does a `GET` to check a value, then a separate `SET` to update it (e.g., implementing a simple lock or counter), and under concurrent access, two requests interleave and corrupt the intended logic
- **Cause**: `GET` and `SET` as two separate round-trips aren't atomic — another client can act in between them, defeating check-then-set logic that assumes no interleaving
- **Symptoms**:
  - A "lock" implemented this way is occasionally held by two clients simultaneously
  - Counters or rate limiters occasionally under/over-count under concurrent load
- **Solution**: Use Redis's atomic primitives for this — `SET key val NX` (set-if-not-exists) for locks, `INCR`/`INCRBY` for counters, or a Lua script (`EVAL`) for anything requiring multiple operations to be atomic together

### SE-5: Unbounded Key Growth From Missing TTLs
- **Severity**: medium
- **Situation**: A feature writes new keys continuously (e.g., per-user or per-session data) without ever setting a TTL, and memory usage grows indefinitely until it hits the eviction/OOM boundary
- **Cause**: Keys without an explicit TTL persist forever by default — nothing automatically expires data that isn't told to
- **Symptoms**:
  - Memory usage climbs steadily and doesn't correlate with active/recent usage — old, effectively-abandoned keys are still present
  - `DBSIZE` grows roughly linearly with total historical writes, not active working set
- **Solution**: Set a TTL on every key unless it's genuinely meant to be permanent (and even then, consider whether an explicit cleanup job is needed), and periodically audit key patterns for ones missing expiration

## Recommended Tools

| Category | Tools |
|------|------|
| Monitoring | `redis-cli --latency`, `INFO`, RedisInsight |
| Scaling/HA | Redis Cluster, Redis Sentinel |
| Scripting | Lua (`EVAL`), Redis Functions |
| Alternatives/complements | Redis Streams (lightweight queue), RediSearch (search on Redis) |

## Reliability & Memory Management

**Full checklist**: → [extended/checklists.md#reliability-checklist]

## Related Resources

[Redis Docs](https://redis.io/docs/latest/) | [Redis Data Types](https://redis.io/docs/latest/develop/data-types/) | [Redis University](https://redis.io/university/)

## Related Domains

[[postgresql]] | [[mongodb]] | [[python]] | [[go]]
