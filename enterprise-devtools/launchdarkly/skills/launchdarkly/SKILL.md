---
schema: "1.0"
name: launchdarkly
version: "1.0.0"
description: LaunchDarkly feature flag rollout strategy, targeting rules, and flag lifecycle hygiene practices
domain: technology
triggers:
  keywords:
    primary: [launchdarkly, feature flag, feature toggle, unleash]
    secondary: [progressive delivery, percentage rollout, kill switch, targeting rule]
  context_boost: [release management, gradual rollout, A/B testing infrastructure]
  context_penalty: [jira, ci/cd pipeline]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# LaunchDarkly (Feature Flags)

> A flag is a temporary rollout lever, not a permanent configuration system — until discipline slips and it becomes one

## Applicable Scenarios

- Designing a progressive rollout (percentage-based, targeted) for a risky change
- Wiring a kill switch into a feature before it ships to production
- Reviewing flag targeting rules and evaluation context design
- Auditing a codebase for stale flags and dead code paths
- Deciding whether a given use case actually warrants a feature flag

## Core Knowledge

### Flags as Progressive Delivery, Not Just On/Off

- A feature flag can be a simple boolean, but LaunchDarkly's real value is **percentage rollouts** (ship to 5% of traffic, watch metrics, increase gradually) and **targeting rules** (ship to internal users first, or a specific customer segment, before general availability)
- This separates **deploy** from **release** — code can be deployed to production dark (flag off) and released independently by flipping the flag, decoupling the two historically-conflated events

### Evaluation Context

- Flag evaluation depends on a **context** — attributes about the user/request (user ID, org ID, plan tier, custom attributes) that targeting rules match against
- The same flag can return different values for different contexts simultaneously — this is the mechanism behind both percentage rollouts (context attributes hashed into a rollout bucket) and targeted segments

### Kill Switches

- A flag wired around a risky code path (a new payment provider integration, a rewritten critical algorithm) provides an instant rollback lever that doesn't require a deploy — flip the flag off, the old code path resumes immediately
- This only works if the flag is actually wired in *before* the risk materializes — retrofitting a kill switch after an incident is too late for that incident

### Flag Lifecycle & Technical Debt

- Every flag that reaches 100% rollout and stays there indefinitely is a **stale flag** — dead code (the old path) plus a permanent conditional (the flag check) that nobody removes
- Flags multiply combinatorially — two independent flags means four code-path combinations to reason about (and potentially test), ten flags means over a thousand
- LaunchDarkly (and similar tools) can report flag evaluation data (is this flag still being evaluated as both true and false, or has it settled at one value) — this is the signal that a flag is ready for cleanup

## Best Practices

1. **Treat every flag as temporary** unless it's explicitly designed as a long-term operational toggle (and even then, review periodically)
2. **Wire a kill switch in before shipping risky changes**, not after something goes wrong
3. **Design evaluation context deliberately** — the attributes available at flag-check time determine what targeting is even possible later
4. **Clean up flags promptly after full rollout** — remove the flag check and the dead code path, don't let it linger
5. **Limit the number of simultaneously active flags** touching the same code path, to bound combinatorial testing complexity
6. **Restrict who can change targeting rules in production** for high-risk flags — a flag misconfiguration is effectively an unreviewed production change

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Flag left at 100%/0% indefinitely with the conditional still in code | Remove the flag and dead code path once the rollout is complete and stable |
| Using a flag as a long-term configuration mechanism | Use actual configuration (env vars, config service) for anything not meant to be temporary |
| Shipping a risky change with no flag/kill switch | Wire in a flag before the risk materializes, not reactively after an incident |
| Client-side SDK exposing flag values for unreleased features | Scope client-side flag delivery carefully; keep sensitive/unreleased flag data server-side only |
| Too many overlapping flags on the same code path | Limit concurrent flags per code path; retire flags before adding new ones in the same area |

## Sharp Edges

### SE-1: Stale Flags Accumulating as Permanent Technical Debt
- **Severity**: high
- **Situation**: A flag from a rollout completed a year ago is still in the codebase, still at 100%, and the "old" code path it was meant to replace is still present too — nobody remembers whether it's safe to remove, so nobody does
- **Cause**: Removing a flag requires a deliberate cleanup step (remove the conditional, delete the dead path, ship the change) that's easy to deprioritize once the flag has served its immediate purpose
- **Symptoms**:
  - Flag inventory shows many flags at 100%/0% with no recent evaluation-split data
  - New engineers can't tell which of several similar code paths is actually the "real" one
- **Solution**: Treat flag cleanup as part of the rollout's definition of done, not a separate future task; use flag evaluation data to identify flags that have settled at one value and are candidates for removal, and schedule regular flag debt review
- **Details**: → [extended/checklists.md#flag-lifecycle-checklist]

### SE-2: A Flag Becoming an Unplanned Configuration System
- **Severity**: medium
- **Situation**: A flag originally meant for a temporary rollout gets repurposed as a long-term on/off switch for a permanent business decision (e.g., "is feature X enabled for enterprise customers"), and now the flag system is load-bearing production configuration that nobody designed it to be
- **Cause**: It's easy and low-friction to reuse an existing flag mechanism for "just one more toggle," and the line between "temporary rollout" and "permanent configuration" isn't enforced by the tooling
- **Symptoms**:
  - Flags with names describing permanent business rules rather than a rollout in progress
  - Changing "configuration" requires going through the feature-flag UI/permissions rather than a proper config management process
- **Solution**: Explicitly distinguish rollout flags (temporary, cleaned up after full rollout) from operational toggles (long-term, intentionally permanent) — if a toggle is meant to be permanent, treat it as configuration with appropriate change management, not as a flag left in "rollout" limbo forever

### SE-3: Missing Kill Switch Discipline on Risky Changes
- **Severity**: high
- **Situation**: A risky change (new third-party integration, rewritten critical path) ships without a flag, it misbehaves in production, and the only rollback option is a full redeploy of the previous version — costing significantly more time during an active incident than flipping a flag would have
- **Cause**: Wiring a flag around a risky change requires anticipating the risk *before* shipping; without that upfront discipline, there's no fast lever available when something does go wrong
- **Symptoms**:
  - Incident postmortems show "we had to redeploy to roll back" as the resolution step, and MTTR is dominated by deploy pipeline time rather than the actual fix
- **Solution**: Establish a team norm that any change with meaningful blast-radius risk gets a flag/kill switch by default, reviewed as part of the change's design, not added reactively after an incident demonstrates the need

### SE-4: Targeting Misconfiguration Rolling Out to the Wrong Audience
- **Severity**: critical
- **Situation**: A percentage rollout or targeting rule is misconfigured (e.g., a rule intended for 1% of traffic actually evaluates to 100%, or a segment intended to exclude a customer accidentally includes them), and an unfinished/risky feature reaches the entire user base instead of the intended small cohort
- **Cause**: Targeting rule logic (rule ordering, percentage bucket configuration, context attribute matching) can be subtly wrong in ways that aren't obvious from a quick glance at the rule editor, especially with multiple overlapping rules
- **Symptoms**:
  - A feature meant for internal-only or small-percentage rollout is reported as visible to a much broader audience than intended
  - Metrics/error rates spike disproportionately to the intended rollout percentage
- **Solution**: Test targeting rule changes against representative evaluation contexts before applying to production (many flag tools support a rule preview/simulation), restrict who can modify targeting rules for high-risk flags, and monitor actual flag evaluation distribution against the intended rollout percentage as a sanity check

### SE-5: Client-Side Flag Exposure Leaking Unreleased Features
- **Severity**: medium
- **Situation**: A client-side (browser/mobile) flag SDK delivers flag values and targeting metadata to the client, and inspecting the client's network traffic or SDK payload reveals the existence and targeting logic of unreleased features before they're publicly announced
- **Cause**: Client-side flag evaluation often requires the client to receive some representation of available flags/rules to evaluate locally or report context — this can inadvertently expose more than intended if not scoped carefully
- **Symptoms**:
  - Unreleased feature names or targeting logic are discoverable by inspecting client-side network requests or bundled SDK payloads
- **Solution**: Scope client-side flag delivery to only what that client context needs (many providers support this), keep sensitive/embargoed flag data server-side and evaluated server-side, and treat client-visible flag payloads as public information from a security review perspective

## Recommended Tools

| Category | Tools |
|------|------|
| Feature flag platforms | LaunchDarkly, Unleash, Split, Flagsmith |
| Flag debt tracking | LaunchDarkly's code references / stale flag detection |
| Experimentation | LaunchDarkly Experimentation, Statsig |

## Flag Lifecycle

**Full checklist**: → [extended/checklists.md#flag-lifecycle-checklist]

## Related Resources

[LaunchDarkly Docs](https://docs.launchdarkly.com/) | [Progressive Delivery](https://launchdarkly.com/blog/what-is-progressive-delivery-all-about/) | [Unleash Docs](https://docs.getunleash.io/)

## Related Domains

[[github-actions]] | [[datadog]] | [[chaos-engineering]]
