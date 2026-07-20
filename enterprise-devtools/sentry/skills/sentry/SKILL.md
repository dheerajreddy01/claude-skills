---
schema: "1.0"
name: sentry
version: "1.0.0"
description: Sentry error grouping/fingerprinting, source maps, release health, and alerting/PII hygiene practices
domain: technology
triggers:
  keywords:
    primary: [sentry, error tracking, exception monitoring, source map]
    secondary: [fingerprinting, breadcrumb, release health, issue ownership]
  context_boost: [crash reporting, frontend error, stack trace]
  context_penalty: [datadog, new relic, honeycomb]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Sentry

> An error you can't read the stack trace for, or that's buried in a mis-grouped issue, might as well not have been captured

## Applicable Scenarios

- Setting up error/exception capture and configuring fingerprinting rules
- Ensuring readable stack traces for minified/transpiled frontend code via source maps
- Designing alert rules and issue ownership/routing
- Reviewing what context data (breadcrumbs, extra context) gets captured for PII risk
- Tuning sampling rates for performance monitoring and error capture

## Core Knowledge

### Error Grouping (Fingerprinting)

- Sentry groups similar errors into one **issue** using a **fingerprint** — by default, derived from the stack trace and error type
- Default fingerprinting can either **over-group** (genuinely different bugs sharing a generic error type/location get merged into one issue, hiding distinct problems) or **under-group** (the same underlying bug, triggered from slightly different call sites or with dynamic data in the message, splits into many separate issues)
- Custom fingerprinting rules let you override default grouping for known problem patterns — this is a deliberate tuning exercise, not a set-once default

### Source Maps

- Production frontend code is typically minified/transpiled; a captured stack trace without a matching **source map** shows meaningless minified variable names and line numbers instead of the actual source location
- Source maps must be uploaded to Sentry (or made fetchable) at build/deploy time, tied to a specific **release** — a missing or mismatched source map for a given release leaves that release's errors effectively undebuggable from the trace alone

### Releases & Release Health

- A **release** ties captured errors/sessions to a specific deployed version, enabling regression detection ("this error started in release X") and release health metrics (crash-free session/user rate)
- Release health data is only meaningful if release tagging is actually wired into the deploy process consistently — a missing or inconsistent release identifier breaks the ability to correlate errors with specific deploys

### Sampling

- **Error sampling** and **performance/tracing sampling** are configured separately, and both trade completeness for cost/overhead — a low sample rate can miss rare-but-critical errors entirely if they don't happen to fall within the sampled fraction
- Sampling is often necessary at high traffic volume, but the rate should be a deliberate choice tied to actual event volume and cost, not a default left unexamined as traffic grows

## Best Practices

1. **Upload source maps as part of the deploy pipeline**, tied to the correct release identifier, for every frontend deploy
2. **Review and tune fingerprinting rules** for known problem patterns rather than accepting default grouping blindly for high-volume error types
3. **Tag releases consistently** in every environment so regression detection and release health are actually usable
4. **Set alert rules with thresholds/rate limits**, not one alert per raw error occurrence, to avoid the same fatigue pattern as other alerting tools
5. **Scrub or avoid capturing PII** in error messages, breadcrumbs, and extra context — treat this as a data-handling decision, not an afterthought
6. **Set sampling rates deliberately** based on actual traffic volume and cost tolerance, and revisit as traffic grows

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| No source maps uploaded for a production frontend deploy | Wire source map upload into the build/deploy pipeline, tied to the release |
| Default fingerprinting accepted for a high-volume, dynamic-message error type | Add custom fingerprinting rules to group meaningfully or split appropriately |
| Alerting on every single error occurrence | Set thresholds/rate-based alert rules; route low-priority errors away from paging channels |
| Capturing full request bodies or user objects as error context without scrubbing | Configure PII scrubbing/data filtering before sensitive fields ever reach Sentry |
| Sample rate left at a default that no longer matches current traffic volume | Review and adjust sampling rates deliberately as traffic and cost profile change |

## Sharp Edges

### SE-1: Fingerprinting Grouping Unrelated Errors Together
- **Severity**: high
- **Situation**: Two genuinely different bugs share a generic error type and similar stack trace shape (e.g., a common utility function throwing for different underlying reasons), and Sentry's default fingerprinting merges them into one issue — triage treats them as one problem, and one gets silently ignored because the "same" issue was already marked resolved
- **Cause**: Default fingerprinting groups primarily by stack trace and error type; it has no semantic understanding of whether two errors, though structurally similar, represent genuinely distinct root causes
- **Symptoms**:
  - An issue's occurrence count doesn't drop after a fix, because a different underlying bug is still contributing to the same fingerprinted group
  - Two clearly different bug reports from users trace back to the same Sentry issue
- **Solution**: Add custom fingerprinting rules for known problem patterns (e.g., fingerprint on a more specific error message component or a custom tag), and periodically review high-volume issues for whether they're actually one coherent problem or several conflated ones
- **Details**: → [extended/checklists.md#error-hygiene-checklist]

### SE-2: Missing Source Maps Leaving Stack Traces Useless
- **Severity**: high
- **Situation**: A production frontend error is captured, but without a matching uploaded source map for that release, the stack trace shows minified variable names and bundled line numbers instead of actual source locations — debugging requires manually reproducing the issue rather than reading the trace
- **Cause**: Source maps must be explicitly generated and uploaded (or made fetchable) at build/deploy time, correctly associated with the release identifier baked into the deployed bundle — this step is easy to omit or misconfigure, especially across build tooling changes
- **Symptoms**:
  - Stack traces in Sentry show unreadable minified identifiers instead of source file/function names
  - Debugging a production-only error requires manual reproduction rather than reading the captured trace
- **Solution**: Wire source map generation and upload into the CI/CD pipeline as a required step for every frontend deploy, verify the release identifier matches between the deployed bundle and the uploaded source maps, and treat a missing-source-map alert as a build pipeline defect to fix promptly

### SE-3: Alert Rule Noise Causing the Same Fatigue Pattern as Other Alerting Tools
- **Severity**: high
- **Situation**: An alert rule fires on every occurrence of a moderately common error, and the team starts ignoring Sentry notifications entirely — including when a genuinely new, severe error appears
- **Cause**: Without thresholds, rate limiting, or severity-based routing, alert volume scales directly with error occurrence volume rather than with actual signal worth acting on
- **Symptoms**:
  - Team feedback indicates Sentry notifications are muted or ignored
  - A genuinely new critical error goes unnoticed because it's one more entry in a noisy channel
- **Solution**: Configure alert rules with occurrence thresholds, rate limiting, or "new issue" vs. "regression" distinctions, and route by severity — reserving immediate paging for genuinely actionable, novel, or high-impact issues

### SE-4: PII Accidentally Captured in Error Context
- **Severity**: critical
- **Situation**: Error context (request bodies, user objects, breadcrumbs) captured automatically includes personally identifiable information — email addresses, auth tokens, payment details — creating a compliance and data-handling problem, since that data now lives in a third-party error-tracking system with its own access surface
- **Cause**: Default SDK integrations often capture request/response data and application state broadly for debugging usefulness, with no automatic awareness of which fields are sensitive
- **Symptoms**:
  - A data audit finds PII present in captured error events/breadcrumbs
  - Sentry issue details expose fields that should never leave the primary system unredacted
- **Solution**: Configure Sentry's data scrubbing (`beforeSend` hooks, built-in PII scrubbing rules, or server-side filtering) to redact or exclude sensitive fields before events are sent, and review SDK default data capture settings as part of initial setup rather than after a compliance review flags it

### SE-5: Sample Rate Misconfiguration Losing Critical Rare Events or Costing More Than Expected
- **Severity**: medium
- **Situation**: Either the error/performance sample rate is set too low and a rare-but-critical error is missed entirely because it fell outside the sampled fraction, or it's left at 100% as traffic scales and event volume/cost grows far beyond what was budgeted
- **Cause**: Sampling is a deliberate tradeoff between completeness and cost/overhead, but the rate is often set once during initial setup and not revisited as traffic patterns and business criticality evolve
- **Symptoms**:
  - A known-to-have-occurred critical error has no corresponding captured event
  - Sentry usage/billing shows costs growing disproportionately to actual investigative value
- **Solution**: Set sampling rates deliberately based on actual traffic volume and the criticality of not missing rare events (consider always-capture rules for known-critical error types regardless of sample rate), and review sampling configuration as traffic grows rather than leaving it at an initial default indefinitely

## Recommended Tools

| Category | Tools |
|------|------|
| SDKs | Sentry SDKs (per language/framework) |
| CI/CD integration | Sentry CLI/Webpack/Vite plugins for source map upload |
| Alerting | Sentry Alert Rules, Slack/PagerDuty integrations |
| Data hygiene | `beforeSend` hooks, Data Scrubbing settings |

## Error Hygiene

**Full checklist**: → [extended/checklists.md#error-hygiene-checklist]

## Related Resources

[Sentry Documentation](https://docs.sentry.io/) | [Sentry Source Maps Guide](https://docs.sentry.io/platforms/javascript/sourcemaps/) | [Sentry Data Scrubbing](https://docs.sentry.io/security-legal-pii/scrubbing/)

## Related Domains

[[datadog]] | [[new-relic]] | [[honeycomb]] | [[javascript]]
