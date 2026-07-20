---
schema: "1.0"
name: playwright-cypress
version: "1.0.0"
description: Playwright and Cypress end-to-end testing — automation model, flakiness, and CI reliability
domain: technology
triggers:
  keywords:
    primary: [playwright, cypress, e2e testing, end-to-end test, browser automation]
    secondary: [flaky test, selector, headless, visual regression, test isolation]
  context_boost: [frontend testing, integration test, ci pipeline]
  context_penalty: [unit test, jest, pytest]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Playwright / Cypress (E2E Testing)

> Tests that wait for real state, not for a clock — otherwise "flaky" is the default

## Applicable Scenarios

- Choosing between Playwright and Cypress for a new E2E suite
- Diagnosing flaky tests that pass locally but fail intermittently in CI
- Designing selector strategy that survives unrelated UI changes
- Structuring test isolation and fixtures for parallel execution
- Setting up visual regression testing or CI artifact collection (traces, videos)

## Core Knowledge

### Automation Model

| Tool | Execution Model | Browser Coverage |
|------|------|------|
| **Playwright** | Drives browsers out-of-process via CDP (Chromium) / WebDriver BiDi-style protocols | Chromium, Firefox, WebKit — true cross-browser |
| **Cypress** | Runs inside the browser alongside the app, in the same run loop | Chromium-family + Firefox (WebKit support limited) |

Cypress's in-browser model gives powerful DOM/network introspection but means it can't easily drive multiple tabs/origins the way Playwright can; Playwright's out-of-process model trades a bit of that introspection for broader browser and multi-context support.

### Auto-Waiting vs. Explicit Waits

- Both tools auto-wait on actionability (element visible, enabled, stable) before interacting — this is what makes them fundamentally more reliable than raw Selenium scripts with manual `sleep()`
- Auto-waiting only covers the specific action/assertion being made — it does **not** wait for unrelated async state (a toast that appears after an API call finishes) unless the test explicitly asserts on that state
- Reaching for a hard-coded wait (`cy.wait(2000)`, `page.waitForTimeout()`) defeats the auto-waiting model and reintroduces the exact flakiness it was designed to prevent

### Selectors

- Best practice ordering: accessible role/label (`getByRole`, `cy.findByRole`) > test-specific attribute (`data-testid`) > CSS class/structure (last resort)
- Selectors tied to CSS classes or DOM nesting break whenever a designer/developer changes styling or refactors markup, even though the actual user-facing behavior hasn't changed

### Test Isolation

- Each test should set up its own required state (via API calls, fixtures, or a clean seeded database) rather than depending on a previous test having run and left the app in a particular state
- Shared state between tests (a shared login session, a shared database row) makes tests **order-dependent** — passing individually but failing when run in a different order or in parallel

## Best Practices

1. **Assert on state, not time** — replace any hard-coded wait with an assertion that polls for the actual condition (element appears, network call resolves, text updates)
2. **Prefer accessible/test-ID selectors** over CSS classes or DOM structure
3. **Make every test independently runnable** — seed its own state, don't depend on execution order
4. **Mock at the network boundary sparingly** — mock third-party/flaky external dependencies, but keep first-party API calls real enough to catch integration bugs
5. **Capture traces/videos on failure in CI** — a failure with no artifact is nearly undebuggable after the fact
6. **Run tests in parallel with isolated environments/data** (separate test users, namespaced data) to avoid cross-test interference

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| `cy.wait(2000)` / `page.waitForTimeout()` to "give it time" | Assert on the actual condition the test is waiting for |
| Selecting elements by CSS class (`.btn-primary-2`) | Use `data-testid` or accessible role/label selectors |
| Test B assumes Test A already logged in / created a record | Seed each test's required state independently |
| Mocking every network call, including first-party APIs | Mock only flaky/third-party dependencies; exercise real first-party integration |
| No trace/video/screenshot captured on CI failure | Configure trace-on-retry or video capture so failures are debuggable after the fact |

## Sharp Edges

### SE-1: Flaky Tests From Implicit Timing Assumptions
- **Severity**: high
- **Situation**: A test passes consistently on a fast local machine but fails intermittently in CI, especially under parallel load, because it implicitly assumed an action (API response, animation, re-render) would complete within an ambient delay
- **Cause**: Auto-waiting covers actionability of the specific element being interacted with, but doesn't know to wait for unrelated async work (a debounced search, an animation transition) unless the test explicitly asserts on the resulting state
- **Symptoms**:
  - Test failure rate correlates with CI load/parallelism, not with any code change
  - Adding a `waitForTimeout`/`cy.wait(n)` "fixes" it until CI gets slower again
- **Solution**: Replace implicit timing assumptions with explicit assertions on the actual end state (`expect(locator).toBeVisible()`, `cy.get(...).should('contain', ...)`), which both tools will auto-retry until it passes or times out — this is fundamentally different from a fixed sleep because it succeeds as soon as the real condition is met
- **Details**: → [extended/checklists.md#test-reliability-checklist]

### SE-2: Test Interdependence From Shared State
- **Severity**: high
- **Situation**: Tests pass when run in file-declaration order but fail when run in parallel or in a different order, because a later test implicitly depends on state a previous test left behind (a logged-in session, a database row, a modified setting)
- **Cause**: Without explicit per-test setup/teardown, tests accumulate hidden dependencies on execution order — nothing in the test framework enforces isolation by default
- **Symptoms**:
  - CI failures that don't reproduce when running the single failing test in isolation
  - Test suite becomes unsafe to parallelize or reorder
- **Solution**: Give each test its own seeded state (via API setup calls or database fixtures scoped to that test), and treat any test that only passes in a specific order as a bug to fix immediately, not a known quirk to work around

### SE-3: Over-Mocking Masking Real Integration Bugs
- **Severity**: medium
- **Situation**: A suite mocks every network call, including first-party backend APIs, and stays green through a real backend contract change that breaks production — the mocked responses never got updated to reflect reality
- **Cause**: Mocking removes a dependency's flakiness but also removes its ability to catch real breakage; a mock that isn't kept in lockstep with the real API silently diverges from it over time
- **Symptoms**:
  - E2E suite passes while the actual integrated system is broken
  - A production incident traces back to an API contract change that no test caught
- **Solution**: Reserve mocking for genuinely flaky or slow third-party dependencies (payment gateways, external APIs with rate limits), and let first-party API calls hit a real (test/staging) backend so contract drift is caught

### SE-4: Selector Brittleness From CSS/DOM Coupling
- **Severity**: medium
- **Situation**: A purely cosmetic change (renaming a CSS class, restructuring a wrapper `<div>`) breaks a batch of unrelated E2E tests with no actual behavior change
- **Cause**: Selectors tied to implementation details (class names, DOM nesting) couple the test suite to styling/markup choices that have nothing to do with the behavior being tested
- **Symptoms**:
  - A design system refactor or CSS framework migration breaks dozens of tests simultaneously
  - Test maintenance burden feels disproportionate to actual behavior changes
- **Solution**: Use `data-testid` attributes or accessible-role-based selectors (`getByRole`, `getByLabelText`) that are stable across styling/structural changes and, as a side benefit, keep the test suite honest about actual accessibility

### SE-5: CI Flakiness From Resource-Starved Parallel Browsers
- **Severity**: medium
- **Situation**: A test suite that's reliable when run with low parallelism becomes flaky (timeouts, dropped frames, slow actions) when CI parallelism is increased, because too many browser instances compete for the same CPU/memory on one runner
- **Cause**: Headless browsers are resource-intensive; running more parallel instances than the runner's actual CPU/memory can support causes real slowdowns that look like application bugs but are actually infrastructure contention
- **Symptoms**:
  - Flakiness rate increases specifically when parallelism/worker count is raised, independent of any code change
  - Timeouts cluster on CI runners under heavier concurrent load
- **Solution**: Size parallel worker count to the runner's actual resources (not just "more is faster"), consider sharding across multiple runner machines instead of over-packing one, and monitor CI runner resource utilization when tuning parallelism

## Recommended Tools

| Category | Tools |
|------|------|
| Test runners | Playwright Test, Cypress |
| Visual regression | Playwright's built-in screenshot assertions, Percy, Chromatic |
| CI reporting | Playwright HTML report/trace viewer, Cypress Dashboard |
| Legacy/alternative | Selenium WebDriver (broader legacy browser support, more manual wait handling) |

## Test Reliability

**Full checklist**: → [extended/checklists.md#test-reliability-checklist]

## Related Resources

[Playwright Docs](https://playwright.dev/docs/intro) | [Cypress Docs](https://docs.cypress.io/) | [Testing Library Guiding Principles](https://testing-library.com/docs/guiding-principles/)

## Related Domains

[[javascript]] | [[github-actions]] | [[k6-jmeter]] | [[sentry]]
