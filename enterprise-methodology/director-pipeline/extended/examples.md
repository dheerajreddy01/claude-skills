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

## High-Level Design (HLD)

### Components
- **Rate limit middleware** (new): sits in front of all `/api/v1/*` routes, checks/increments usage before the request reaches the route handler
- **Shared counter store** (existing Redis cluster, reused): holds per-API-key request counts — must be shared, not per-instance, since the API runs on multiple horizontally-scaled instances
- **Plans table** (existing, unchanged): source of truth for each API key's tier and its rate limit

### Data Flow
Request arrives → middleware extracts API key → middleware looks up tier limit from Plans (cached) → middleware checks/increments count in Redis for that key → allow (forward to route handler) or reject (429 + Retry-After) → response returned

### Key Architectural Decisions
- **Redis, not per-instance memory**: required because the API is horizontally scaled and limits must be enforced globally across instances, not per-instance
- **Middleware, not per-route code**: applied once at the routing layer so no individual route handler can forget to enforce it
- **Fail-open on Redis unreachable**: availability constraint ("must not fail the whole API") takes priority over strict enforcement during a Redis outage

## Low-Level Design (LLD)

### Data Model / Schema
- No new persistent schema — tier limits read from the existing `plans` table (`plans.rate_limit_per_minute`, already present)
- Redis: sorted set per API key, key pattern `ratelimit:{api_key}`, members are request timestamps, used for a sliding-window count

### API / Function Contracts
- Middleware function: `checkRateLimit(apiKey string) (allowed bool, retryAfterSeconds int, err error)`
- On `allowed == false`: route layer returns `429 Too Many Requests` with header `Retry-After: {retryAfterSeconds}`
- On `err != nil` (Redis unreachable): treat as `allowed == true` (fail-open) and log the error

### Core Logic / Algorithms
- Sliding window: on each request, remove sorted-set entries older than the window, count remaining entries, compare to the key's tier limit, add current timestamp if under limit
- Requests with no API key: apply the strictest defined tier (per the answered open question below), not outright rejection

### Sequence for Key Flows
1. Request arrives at middleware
2. Extract API key (or flag as anonymous)
3. Fetch tier limit (cached lookup from `plans`, or strictest tier if anonymous)
4. Sliding-window check/update against Redis
5. If allowed: forward to route handler; if not: short-circuit with 429 + Retry-After
6. If Redis errors: log, fail-open, forward to route handler

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

## HLD/LLD Completeness Check
- [x] Pass — design was complete enough to implement without inventing structure

Walked the LLD's Data Model, API contract (`checkRateLimit`), and Sequence for Key
Flows against the actual codebase before starting. Redis sorted-set approach and
the `plans` table read both check out against existing infrastructure. HLD's
fail-open decision is directly implementable as specified. No components needed
that the HLD didn't already name. Check passed clean — see Design Gaps Found
below for what still slipped through despite that.

## Implementation Summary
Built exactly to the LLD: sliding-window rate limiter as middleware on all routes
under `/api/v1/*`, using Redis sorted sets keyed `ratelimit:{api_key}`, tier limits
read from the existing `plans` table, fail-open on Redis errors per the HLD's
architectural decision.

## Mapping to Acceptance Criteria
- AC-1/AC-2/AC-3: implemented in `middleware/rate_limit.go`, following the LLD's
  `checkRateLimit` contract and sliding-window algorithm exactly
- AC-4: keys scoped per API key in Redis, verified isolated

## Interpretations Made
- LLD's `retryAfterSeconds` wasn't fully specific about whether it counts to the
  full window reset or the next single slot opening — interpreted as "seconds
  until the full window resets," the simpler of the two to reason about correctly

## Design Gaps Found
- The LLD specified an error contract for the Redis dependency (fail-open) but
  never specified one for the `plans` table lookup — a separate dependency on
  the same request path. Not caught by the pre-implementation completeness
  check because the LLD's Data Flow only called out Redis as a failure-prone
  step; the `plans` lookup read as "just a DB read" and didn't visibly need
  its own error contract until actually building the error-handling branches.
  Currently falls through to a generic 500. Flagging rather than silently
  picking fail-open/fail-closed myself, since the HLD's fail-open rationale
  (availability over strict enforcement) may or may not extend to this
  dependency too — that's a call for Director, not an implementation detail.

## Deviations From Brief
- None — HLD/LLD covered the design decisions (fail-open, sliding window, Redis
  key pattern) concretely enough that no ad hoc decisions were needed during
  implementation

## Known Gaps
- None beyond the item logged in Design Gaps Found above

## Notes for Tester
- To simulate Redis unreachable, stop the local redis container and hit any endpoint
- Tier limits are seeded in `plans` table fixture `test/fixtures/plans.sql`
```

### Stage 3 — Tester

`test-report.md`:

```markdown
# Test Report: API rate limiter

## HLD/LLD Completeness Check
- [ ] Gaps found — see notes below

Traced the sliding-window flow end-to-end against the LLD's Sequence for Key
Flows: steps 1-5 match the implementation exactly. HLD's fail-open decision
verified directly in code (`middleware/rate_limit.go`, Redis error path),
not just asserted in the dev log. Independently confirms the gap Dev's own
dev-log already flagged: the LLD's Data Flow only covers the Redis path — it
never specifies what happens if the *Plans table* lookup (a different
dependency, also on the request path) fails. Dev's pre-implementation check
didn't catch this because the LLD only visibly called out Redis as
failure-prone; this check catches it independently by verifying every
dependency on the actual request path against the LLD, not just the ones the
LLD happened to already flag. This is a gap in the design itself, not
something Dev got wrong — same conclusion Dev already reached, now confirmed
by an independent pass rather than taken on the developer's word alone.

## Acceptance Criteria Results
- [x] AC-1: PASS
- [x] AC-2: PASS — verified 429 + Retry-After header present and correctly valued
- [x] AC-3: PASS
- [x] AC-4: PASS

## Edge Cases Tested
- Redis unreachable → requests succeed (fail-open), consistent with the HLD's architectural decision
- No API key → strictest tier applied correctly per Director's answered open question
- Two instances behind the load balancer simultaneously rate-limiting the same key → correctly shared via Redis, no per-instance drift
- `plans` table lookup failing (separate from Redis) → currently a generic 500, not covered by any AC (see finding #1)

## Findings
| # | Finding | Severity | Routes To | Notes |
|---|---------|----------|-----------|-------|
| 1 | LLD didn't specify behavior when the `plans` table lookup fails (as opposed to Redis) — this is a requirements gap, not a bug, since Dev correctly implemented everything the brief specified | should-fix | Director | LLD needs an explicit contract for this failure path (fail-open like Redis? Reject? Cache last-known tier?) before it's truly done |
| 2 | No metric/alert fires when fail-open kicks in on a Redis outage | should-fix | Director | Silent fail-open could mask a Redis outage for a long time — worth scoping as a fast follow |

## Overall Verdict
- [ ] Ready to ship with accepted follow-ups: findings #1 and #2 accepted by Director
      as tracked follow-up work (amend the LLD for the plans-table failure path,
      add monitoring for fail-open), not blocking this ship since all four
      acceptance criteria (AC-1 through AC-4) pass and the gaps are edge cases
      beyond what was originally scoped.
```

### The Loop, In Practice

Both findings route to **Director**, not Dev — and this example shows the check
working at both layers, not just Tester's. Dev ran the HLD/LLD Completeness Check
*before* writing any code and it passed clean on paper, but the `plans`-table gap
still slipped through: the LLD only visibly flagged Redis as failure-prone, so
nothing about a first pass over the design pointed at the Plans lookup needing
its own error contract. Dev only surfaced it while actually wiring up the error
handling — logged immediately as a Design Gap rather than silently picked
one way or the other. Tester's independent check then re-verified the same
conclusion from scratch (tracing the actual code, not trusting Dev's log), which
is exactly the point of running it twice: Dev's pre-implementation check catches
most gaps before a line of code exists, and Tester's post-implementation check
catches whatever slipped past that first pass, cross-checked against reality
instead of a developer's own account of their work.

Because both checks agree Dev built exactly what the LLD specified (also
corroborated by AC-1 through AC-4 all passing), Tester could confidently label
this a **design gap**, not an implementation bug — the LLD simply never specified
a contract for the `plans`-table failure path, and the HLD's fail-open decision
didn't come with an operational visibility requirement. Without either
completeness check, it would have been tempting to just hand both findings to
Dev as "bugs" — they'd have gotten patched ad hoc, and the underlying design gap
would still be undocumented the next time a similar case came up.

This is the pipeline working as intended: a real gap surfaced, got routed to the role
that should actually decide it, and shipping proceeded with the gap tracked instead
of either (a) blocking on a non-blocking issue, or (b) silently ignoring it.
