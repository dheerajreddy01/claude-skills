# Splunk — Extended Checklists

## Search Performance Checklist

- [ ] Every search has an explicit, bounded time range — no reliance on "All Time" defaults
- [ ] `index=`/`sourcetype=` filters appear at the very start of the search, before any `eval`/`stats`/transforming commands
- [ ] Search Job Inspector checked for scan-count vs result-count ratio on any search that feels slow
- [ ] `join`/`transaction` usage reviewed — replaced with `stats` or a lookup where the correlation allows it
- [ ] Frequently-run expensive searches use summary indexing or accelerated data models (`tstats`)
- [ ] Real-time searches limited to genuinely sub-minute-latency needs; everything else uses scheduled search
- [ ] Field extraction strategy (index-time vs search-time) reviewed for high-frequency fields

## License & Access Checklist

- [ ] Top ingesting sourcetypes/hosts reviewed regularly against actual search/alerting value
- [ ] DEBUG-level or otherwise verbose logging is opt-in, not onboarded by default
- [ ] Duplicate ingestion paths (same data via two forwarders/pipelines) checked for and eliminated
- [ ] Indexes split by retention and access-control needs, not bundled by convenience
- [ ] Role-to-index mappings (`srchIndexesAllowed`) audited so no role has broader index access than its users need
- [ ] Sensitive fields (PII, credentials-adjacent data) are masked or redacted at index time where full access isn't required
- [ ] Alert throttling configured to prevent duplicate notifications for the same underlying condition
