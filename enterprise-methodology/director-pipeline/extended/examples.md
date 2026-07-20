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

## Deviations From Brief
- None — HLD/LLD covered the design decisions (fail-open, sliding window, Redis
  key pattern) concretely enough that no ad hoc decisions were needed during
  implementation

## Known Gaps
- The LLD didn't specify what happens if the `plans` table lookup itself fails
  (separately from the Redis counter) — currently falls through to a generic
  500, not addressed by any acceptance criterion

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

Both findings route to **Director**, not Dev — in both cases Dev built exactly what
the HLD/LLD specified (that's confirmed by AC-1 through AC-4 all passing), and what
surfaced during testing is a **design gap**, not an implementation bug: the LLD simply
never specified a contract for the `plans`-table failure path, and the HLD's fail-open
decision didn't come with an operational visibility requirement. Because Dev's log
showed zero deviations from a concrete LLD, Tester could tell immediately that these
findings belong to Director (amend the design) rather than Dev (fix the code) — that
clarity is the direct payoff of Director having done a real HLD/LLD pass instead of
handing Dev a vague brief and letting these decisions get made ad hoc mid-implementation.

This is the pipeline working as intended: a real gap surfaced, got routed to the role
that should actually decide it, and shipping proceeded with the gap tracked instead
of either (a) blocking on a non-blocking issue, or (b) silently ignoring it.
