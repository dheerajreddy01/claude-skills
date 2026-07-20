---
schema: "1.0"
name: slack
version: "1.0.0"
description: Slack workspace structure, app/bot OAuth scopes, and integration reliability practices
domain: technology
triggers:
  keywords:
    primary: [slack, slack bot, slack app, webhook]
    secondary: [workspace, channel, OAuth scope, events api, socket mode]
  context_boost: [notification, chatops, team communication]
  context_penalty: [confluence, jira, pagerduty]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Slack

> A workspace's usefulness degrades in direct proportion to how much noise its bots generate

## Applicable Scenarios

- Designing channel structure and workspace organization
- Building Slack apps/bots and scoping their OAuth permissions
- Choosing between webhooks, the Events API, and Socket Mode for integrations
- Designing notification/alert bots that don't get muted
- Setting message retention and compliance policies

## Core Knowledge

### Workspace Structure

- **Channels** are the primary organizational unit (public, private, or shared with external orgs); **threads** keep a specific conversation contained without flooding the channel; **DMs** are for genuinely one-to-one or small-group discussion
- **Enterprise Grid** (for larger orgs) layers multiple workspaces under one org, with cross-workspace channels and centralized admin — a different governance model than a single flat workspace
- Channel naming/archival conventions matter more as a workspace grows — an unmanaged channel list becomes as unnavigable as an unmanaged file system

### App & Bot Permissions (OAuth Scopes)

- Slack apps request **OAuth scopes** (e.g., `chat:write`, `channels:read`, `users:read.email`) — each scope grants a specific, auditable capability, and an app should request only what it actually uses
- **Bot tokens** vs **user tokens**: a bot token acts as the app's own bot identity; a user token acts *as* the installing user, which is a meaningfully broader and riskier grant if not actually needed
- Installed apps are visible and auditable per-workspace (admin can see every app's granted scopes) — this is the actual control surface for reviewing what integrations can do

### Integration Patterns

| Pattern | Use |
|------|------|
| **Incoming Webhook** | Simplest way to post into a fixed channel — a plain URL that anyone holding it can POST to |
| **Events API** | Subscribe to workspace events (messages, reactions, etc.) via an HTTP endpoint you host |
| **Socket Mode** | Events delivered over a persistent WebSocket instead of a public HTTP endpoint — useful when you can't expose a public URL |
| **Slash Commands / Interactive Components** | User-initiated actions (commands, buttons, modals) invoking your app |

An incoming webhook URL is a bearer credential — anyone who has it can post as that integration into that channel, with no further authentication.

### Retention & Compliance

- Message retention is configurable per workspace (and per channel type in Enterprise Grid) — default retention keeps messages indefinitely unless explicitly configured otherwise
- Compliance exports (via Slack's Discovery APIs / enterprise compliance tooling) are a separate, admin-level capability from what regular users can see — relevant for regulated industries

## Best Practices

1. **Request the minimum OAuth scopes an app actually needs** — review and prune scopes on every app update, not just at install time
2. **Treat webhook URLs as secrets** — never commit them to a repo or paste them somewhere broadly visible
3. **Design bot notifications to be low-noise and high-signal** — batch, threshold, or thread notifications rather than posting on every event
4. **Use threads for sub-conversations** to keep channels scannable
5. **Establish a channel naming/archival convention** as the workspace grows, and actually archive inactive channels
6. **Review installed apps periodically** — an admin-level audit of what's installed and what scopes each app holds

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Requesting broad OAuth scopes "just in case" during app setup | Request only the specific scopes the app's actual functionality needs |
| Committing a webhook URL to a public or shared repo | Treat webhook URLs as secrets, store them in a secret manager/environment variable |
| A bot posting on every single event with no batching/threshold | Design for signal — batch related notifications, use thresholds, or route to a dedicated low-priority channel |
| No channel archival policy as the workspace grows | Establish and enforce a naming/archival convention; archive channels with no recent activity |
| Using a user token when a bot token would suffice | Use bot tokens for app-identity actions; reserve user tokens for cases that genuinely require acting as the installing user |

## Sharp Edges

### SE-1: Overly Broad OAuth Scopes Granted "To Make It Work"
- **Severity**: high
- **Situation**: An app is granted a broad scope (e.g., a workspace-wide read scope) during initial setup to unblock development, and it's never revisited — the app now has far more access than its actual functionality requires
- **Cause**: It's faster to grant a broad scope than to identify the precise minimal set needed, and nothing forces a later review once the app is working
- **Symptoms**:
  - A workspace app audit shows an app's granted scopes far exceed what its integration actually does
  - A compromised or buggy app can read/post far beyond its intended function
- **Solution**: Scope every app to the minimum required permissions at setup, and periodically audit installed apps' granted scopes against their actual usage, tightening where possible
- **Details**: → [extended/checklists.md#security-checklist]

### SE-2: Leaked Webhook URLs Enabling Unauthorized Posting
- **Severity**: high
- **Situation**: An incoming webhook URL is committed to a public repository, pasted into a shared doc, or otherwise exposed, and anyone who finds it can post messages into that channel impersonating the integration — including phishing-style messages that look legitimate
- **Cause**: A Slack incoming webhook URL is itself the entire credential — no additional authentication is required to POST to it, so possession of the URL equals full posting capability
- **Symptoms**:
  - Unexpected or malicious-looking messages appear from a known integration's identity
  - A secret scan finds a webhook URL in a public commit history
- **Solution**: Treat webhook URLs as secrets from the start — store in environment variables/secret managers, never commit to version control, and rotate (regenerate) the webhook immediately if exposure is discovered

### SE-3: Notification Bots Generating Enough Noise to Get Muted
- **Severity**: high
- **Situation**: An alert/notification bot posts on every single underlying event without batching or thresholds, and the channel becomes noisy enough that the team mutes it or stops reading it — including when it eventually posts something that actually matters
- **Cause**: It's simplest to wire "on event X, post a message" with no aggregation logic, but event volume at real scale quickly overwhelms a channel's actual attention capacity
- **Symptoms**:
  - Channel members report muting or ignoring a specific bot's channel
  - A genuinely important notification goes unnoticed because it's buried in routine noise from the same bot
- **Solution**: Design notification logic with batching, thresholds, or severity-based routing (route low-priority events to a digest or a separate low-priority channel, reserve the main channel for things that need attention), mirroring the same alert-fatigue discipline used in dedicated alerting tools

### SE-4: Channel Sprawl With No Archival Policy
- **Severity**: medium
- **Situation**: Channels accumulate over years of ad hoc creation (one per project, one per incident, one per one-off discussion) with no archival process, and the channel list becomes so cluttered that finding the right channel — or the right historical conversation — becomes impractical
- **Cause**: Channel creation has essentially no friction and no default lifecycle, so nothing prompts cleanup once a channel's purpose has ended
- **Symptoms**:
  - Search results return many stale, inactive channels alongside the currently-relevant one
  - New members can't tell which of several similarly-named channels is actually active
- **Solution**: Establish a naming convention and an archival cadence (e.g., auto-flag channels with no activity in N months for review), and actually archive channels whose purpose has concluded rather than leaving them to accumulate indefinitely

### SE-5: Message Retention Misconfiguration Causing Unintended Loss or Indefinite Retention
- **Severity**: medium
- **Situation**: A workspace's message retention policy doesn't match what the organization actually needs — either messages are deleted before a legally required retention window, or sensitive content is retained indefinitely with no policy governing when it should be purged
- **Cause**: Retention settings are workspace/channel-type-level configuration that's easy to leave at default and not revisit as compliance requirements or organizational policy evolve
- **Symptoms**:
  - A legal/compliance request for historical messages finds they were already deleted per an unreviewed retention setting
  - Sensitive discussion content persists indefinitely with no retention policy ever having been deliberately set
- **Solution**: Set retention policy deliberately based on actual legal/compliance requirements (not left at default), document the rationale, and review it when compliance obligations change

## Recommended Tools

| Category | Tools |
|------|------|
| App development | Slack Bolt SDK, Block Kit Builder |
| Admin/governance | Slack Admin Analytics, App Directory review |
| Compliance | Slack Discovery API, enterprise compliance exports |
| Automation | Workflow Builder (no-code), Incoming/Outgoing Webhooks |

## Security & App Hygiene

**Full checklist**: → [extended/checklists.md#security-checklist]

## Related Resources

[Slack API Documentation](https://api.slack.com/) | [Slack OAuth Scopes Reference](https://api.slack.com/scopes) | [Slack Admin Guide](https://slack.com/help/categories/360000049043)

## Related Domains

[[pagerduty]] | [[confluence]] | [[jira]] | [[datadog]]
