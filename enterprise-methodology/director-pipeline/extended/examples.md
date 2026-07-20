# Director Pipeline — Worked Example

## Task: "Add a rate limiter to our public API"

### Stage 1 — Director

`director-brief.md`:

```markdown
# Director Brief: API rate limiter

## Restated Request
Add rate limiting to the public-facing API so that no single client can
overwhelm the service, while legitimate traffic is unaffected.

## In Scope
- Per-API-key rate limiting on all public endpoints
- Configurable limit (requests per minute) per API key tier
- 429 response with a Retry-After header when the limit is exceeded

## Out of Scope (Non-Goals)
- Rate limiting internal service-to-service calls
- A billing/quota UI for customers to see their own usage
- Distributed rate limiting across multiple regions (single-region only for now)

## Acceptance Criteria
- [ ] AC-1: Given a client under its rate limit, when it makes a request, then the request succeeds normally
- [ ] AC-2: Given a client at its rate limit, when it makes one more request, then it receives a 429 with a Retry-After header
- [ ] AC-3: Given a client that was rate-limited, when the window resets, then subsequent requests succeed again
- [ ] AC-4: Given two different API keys, when one is rate-limited, then the other is unaffected

## Edge Cases & Failure Modes
- What happens if the rate-limit store (e.g. Redis) is unreachable? Must not fail the whole API.
- Requests with no API key at all (should they be limited more strictly, or rejected outright?)

## Constraints
- Must not add more than ~5ms p50 latency to requests
- Must work across our existing horizontally-scaled API instances (not per-instance-local state)

## Open Questions for a Human
- What's the default limit per tier, and where do tier definitions live?
- Requests with no API key: reject outright, or apply the strictest tier?
```

*(Director flagged both open questions rather than guessing — a human answers:
default tiers come from the existing `plans` table; unauthenticated requests
get the strictest tier, not rejected outright.)*

### Stage 2 — Dev

`dev-log.md`:

```markdown
# Dev Log: API rate limiter

## Implementation Summary
Added a Redis-backed sliding-window rate limiter as middleware, applied to all
routes under `/api/v1/*`. Limit tiers pulled from the existing `plans` table per AC brief update.

## Mapping to Acceptance Criteria
- AC-1/AC-2/AC-3: implemented in `middleware/rate_limit.go`, sliding window via Redis sorted sets
- AC-4: keys scoped per API key in Redis, verified isolated

## Interpretations Made
- "Retry-After header" interpreted as seconds until the current window fully resets,
  not until the next single slot opens (brief didn't specify which)

## Deviations From Brief
- Redis-unreachable fallback: brief said "must not fail the whole API" but didn't
  specify fail-open vs fail-closed. Implemented fail-open (allow the request through)
  since the constraint emphasized availability over strict enforcement. Flagging
  this as a should-confirm item, not silently assumed correct.

## Known Gaps
- No fail-closed configuration option yet — fail-open is hardcoded

## Notes for Tester
- To simulate Redis unreachable, stop the local redis container and hit any endpoint
- Tier limits are seeded in `plans` table fixture `test/fixtures/plans.sql`
```

### Stage 3 — Tester

`test-report.md`:

```markdown
# Test Report: API rate limiter

## Acceptance Criteria Results
- [x] AC-1: PASS
- [x] AC-2: PASS — verified 429 + Retry-After header present and correctly valued
- [x] AC-3: PASS
- [x] AC-4: PASS

## Edge Cases Tested
- Redis unreachable → requests succeed (fail-open), consistent with dev log's flagged interpretation
- No API key → strictest tier applied correctly per Director's answered open question
- Two instances behind the load balancer simultaneously rate-limiting the same key → correctly shared via Redis, no per-instance drift

## Findings
| # | Finding | Severity | Routes To | Notes |
|---|---------|----------|-----------|-------|
| 1 | Fail-open on Redis outage was Dev's interpretation, not explicitly in the brief | should-fix | Director | Needs an explicit decision: is fail-open actually the desired behavior long-term, or just an acceptable interim default? |
| 2 | No metric/alert fires when fail-open kicks in | should-fix | Director | Silent fail-open could mask a Redis outage for a long time — worth scoping as a fast follow |

## Overall Verdict
- [ ] Ready to ship with accepted follow-ups: findings #1 and #2 accepted by Director
      as tracked follow-up work (add monitoring + confirm fail-open policy), not
      blocking this ship since core rate-limiting behavior (AC-1 through AC-4) is correct.
```

### The Loop, In Practice

Finding #1 routes to **Director**, not Dev — Dev made a reasonable interpretation of
an underspecified constraint, but whether fail-open is actually correct long-term is
a product/reliability decision, not an implementation detail. Director confirms
fail-open is acceptable for now and creates a follow-up brief for the monitoring gap
(finding #2) rather than blocking this ship on it.

This is the pipeline working as intended: a real gap surfaced, got routed to the role
that should actually decide it, and shipping proceeded with the gap tracked instead
of either (a) blocking on a non-blocking issue, or (b) silently ignoring it.
