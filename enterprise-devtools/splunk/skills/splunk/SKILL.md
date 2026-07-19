---
schema: "1.0"
name: splunk
version: "1.0.0"
description: Splunk architecture, SPL search optimization, index design, and license/cost management
domain: technology
triggers:
  keywords:
    primary: [splunk, SPL, index, log search, forwarder]
    secondary: [props.conf, transforms.conf, search head, indexer, SIEM]
  context_boost: [logs, log analysis, security monitoring, observability]
  context_penalty: [datadog, new relic, grafana]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Splunk

> Fast, cost-aware log search — SPL that scales with the data, not against it

## Applicable Scenarios

- Writing or optimizing SPL searches, reports, and dashboards
- Designing index structure and data onboarding (props.conf/transforms.conf)
- Diagnosing slow searches or unexpected license/ingest overage
- Setting up alerts and scheduled searches
- Reviewing role-based access controls for sensitive log data

## Core Knowledge

### Architecture

| Component | Role |
|------|------|
| **Forwarder** | Lightweight agent that ships data from a source host to indexers |
| **Indexer** | Parses, indexes, and stores event data; handles search execution against its data |
| **Search Head** | Coordinates searches across indexers, renders dashboards/reports |
| **Deployment Server** | Distributes configuration to forwarders |

In distributed deployments, a search head dispatches the search to all relevant indexers in parallel (map) and combines results (reduce) — search performance depends heavily on how well the search can be pushed down to indexers before the reduce phase.

### SPL (Search Processing Language)

- Pipeline model: `search | command1 | command2 | ...` — each stage transforms the result set from the previous stage
- **Search-time vs index-time**: field extraction can happen at index time (faster search, more storage/CPU at ingest) or search time (flexible, but repeated cost per search) — most fields should stay search-time unless a field is used in nearly every search
- Common commands: `stats`, `eval`, `where`, `table`, `timechart`, `join` (expensive — avoid when `stats`/`lookup` can substitute)
- **Always filter early**: `index=`, `sourcetype=`, and time range should narrow the data before any `eval`/`stats`/`join` — filtering after expensive commands processes far more events than necessary

### Index Design

- One index per logical dataset with a distinct retention/access policy — mixing high-volume debug logs with low-volume audit logs in the same index forces a single retention/ACL tradeoff on both
- `props.conf`/`transforms.conf` control parsing (line breaking, timestamp extraction, field extraction) at index time — misconfigured line-breaking silently merges or splits events
- Retention is set per index (`frozenTimePeriodInSecs`) — data aged past retention is deleted or archived, not searchable

### License & Cost Model

- Splunk Enterprise licensing (in ingest-based tiers) is typically billed by daily ingest volume (GB/day) — verbose, unfiltered, or duplicate data directly drives cost
- License usage is enforced per-license-pool; exceeding the daily quota triggers warnings and, after repeated violations, search restrictions (license "violation" states)

## Best Practices

1. **Filter before transforming** — put `index=`, `sourcetype=`, and time range at the very start of the search
2. **Push work to indexers** — commands like `stats` distribute across indexers; `join`/`transaction` often don't and pull data to the search head
3. **Use summary indexing or data model acceleration** for frequently-run, expensive searches/dashboards
4. **Separate indexes by retention and access needs**, not just by source system
5. **Extract fields at index time only for high-frequency fields** — search-time extraction is more flexible for everything else
6. **Set alert throttling** to avoid duplicate/noisy notifications for the same underlying condition

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| `index=* sourcetype=* error` with no time range | Scope `index=`/`sourcetype=` and a bounded time range explicitly |
| Using `join` to combine two searches | Use `stats` with a common field, or a lookup table, where possible |
| One index for everything regardless of retention needs | Split indexes by retention/access policy |
| Ingesting verbose DEBUG-level logs in production by default | Filter at the source or via `transforms.conf` before indexing; keep DEBUG opt-in |
| Real-time searches left running indefinitely | Use scheduled searches with an appropriate interval instead of real-time where sub-second latency isn't required |

## Sharp Edges

### SE-1: Unbounded Time Range Searches
- **Severity**: high
- **Situation**: A search with no explicit time range (or "All Time") scans far more data than intended, consuming indexer resources and returning slowly
- **Cause**: Without an explicit bound, Splunk searches the full retained history of the matched index(es)
- **Symptoms**:
  - A search that "should be quick" runs for minutes and impacts other concurrent searches
  - Search job inspector shows a scan count orders of magnitude larger than the actual matching event count
- **Solution**: Always set an explicit, as-narrow-as-reasonable time range; use the Search Job Inspector to check scan-count vs result-count and tighten filters when the ratio is high
- **Details**: → [extended/checklists.md#search-performance-checklist]

### SE-2: License Overage From Verbose or Duplicate Ingestion
- **Severity**: high
- **Situation**: Daily ingest volume creeps past the licensed quota because a source was onboarded with DEBUG-level verbosity, or the same data is ingested twice (e.g., via two different forwarders/pipelines)
- **Cause**: License usage is metered by raw bytes ingested per day; nothing blocks a noisy or duplicate source from being onboarded
- **Symptoms**:
  - License usage reports show a sustained climb with no corresponding increase in useful search volume
  - Repeated license violation warnings, eventually restricting search
- **Solution**: Audit top ingesting sourcetypes/hosts regularly, filter unnecessary verbosity at the source or via `transforms.conf` null-queue routing, and check for duplicate ingestion paths before onboarding a new forwarder

### SE-3: Expensive `join`/`transaction` Commands at Scale
- **Severity**: medium
- **Situation**: A search using `join` or `transaction` to correlate two datasets becomes the slowest part of a dashboard as data volume grows, eventually timing out
- **Cause**: `join` (and to a lesser extent `transaction`) typically execute on the search head against a subset of results, rather than distributing across indexers like `stats` does — they don't scale the same way
- **Symptoms**:
  - Dashboard panel using `join` times out or is disproportionately slower than adjacent panels
  - `join`'s default result limit (subsearch limit) silently truncates correlated data
- **Solution**: Replace `join` with `stats` grouped by a shared field where possible, or use a lookup table for relatively static reference data

### SE-4: Role-Based Access Misconfiguration Exposing Sensitive Logs
- **Severity**: critical
- **Situation**: A role intended for general users can search an index containing sensitive data (PII, security incident details, credentials-adjacent logs) because index-level access wasn't restricted per role
- **Cause**: Splunk's role-based access control requires explicit index restrictions per role; a role with broad default index access can search anything unless scoped down
- **Symptoms**:
  - A permissions audit finds general users able to search a sensitive index
  - Sensitive data surfaces in a dashboard or report visible to a broader audience than intended
- **Solution**: Scope roles to the specific indexes they need (`srchIndexesAllowed`), use field-level or event-level data redaction (e.g., via `transforms.conf` SEDCMD or field masking) for partial sensitivity, and audit role-to-index mappings periodically

### SE-5: Real-Time Search Overload
- **Severity**: medium
- **Situation**: Multiple real-time searches (dashboards refreshing continuously) run concurrently and degrade indexer performance for everyone
- **Cause**: Real-time searches hold open a continuous pipeline against incoming data rather than running once against a bounded result set, consuming resources for as long as they're open
- **Symptoms**:
  - Indexer CPU/memory usage climbs correlated with the number of open real-time dashboards
  - Concurrent search quota gets consumed by long-running real-time searches, blocking scheduled searches
- **Solution**: Reserve real-time search for genuinely sub-minute-latency use cases; use scheduled searches with a short interval (e.g., every 1-5 minutes) for everything else

## Recommended Tools

| Category | Tools |
|------|------|
| Search optimization | Search Job Inspector, `tstats` (accelerated data models) |
| Data onboarding | Splunk Add-ons, Universal Forwarder |
| Dashboards | Splunk Dashboard Studio, Simple XML |
| Admin | Splunk Monitoring Console |

## Search Performance & Cost

**Full checklist**: → [extended/checklists.md#search-performance-checklist]

## Related Resources

[Splunk Docs](https://docs.splunk.com/) | [SPL Search Reference](https://docs.splunk.com/Documentation/Splunk/latest/SearchReference/) | [Splunk Search Best Practices](https://docs.splunk.com/Documentation/Splunk/latest/Search/Aboutsearchoptimization)

## Related Domains

[[new-relic]] | [[datadog]] | [[grafana-prometheus]] | [[aws]]
