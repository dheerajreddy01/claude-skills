# Playwright / Cypress — Extended Checklists

## Test Reliability Checklist

- [ ] No hard-coded waits (`waitForTimeout`, `cy.wait(ms)`) — every wait is an assertion on real state
- [ ] Every test seeds its own required state; no test depends on another test having run first
- [ ] Selectors use `data-testid` or accessible role/label, not CSS classes or DOM structure
- [ ] Network mocking limited to genuinely flaky/third-party dependencies; first-party APIs exercised for real
- [ ] Trace/video/screenshot capture enabled on failure (and ideally on first retry) in CI
- [ ] Parallel worker count sized to actual CI runner resources, not maximized blindly
- [ ] Flaky tests are quarantined and fixed promptly, not silently retried into passing

## CI Integration Checklist

- [ ] Headless mode used in CI with the same browser versions pinned as local development
- [ ] Test sharding across multiple CI runners considered before over-packing parallelism on one runner
- [ ] Visual regression baselines reviewed and approved deliberately, not auto-accepted
- [ ] Test run artifacts (traces, videos, HTML reports) retained and linked from CI failure notifications
- [ ] Retry-on-failure configured as a safety net, not as a substitute for fixing root-cause flakiness
- [ ] E2E suite run time monitored — growing suite duration investigated before it becomes a CI bottleneck
