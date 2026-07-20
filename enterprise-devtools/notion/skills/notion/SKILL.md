---
schema: "1.0"
name: notion
version: "1.0.0"
description: Notion workspace architecture, database design, and permission inheritance practices
domain: technology
triggers:
  keywords:
    primary: [notion, notion database, workspace, block]
    secondary: [notion api, relation, rollup, template]
  context_boost: [knowledge base, documentation, internal tool]
  context_penalty: [confluence, jira, linear]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Notion

> A block-based canvas flexible enough to become either a knowledge base or an unnavigable mess, depending entirely on structure discipline

## Applicable Scenarios

- Designing workspace information architecture (pages, databases, hierarchy)
- Building databases with relations/rollups for structured, queryable content
- Setting sharing/permission levels for pages and databases
- Diagnosing performance issues in large databases or automations hitting API limits
- Establishing template consistency across a growing workspace

## Core Knowledge

### Block-Based Content Model

- Every piece of content in Notion — a paragraph, a heading, an embedded database — is a **block**, nestable inside other blocks; this flexibility is what makes Notion good as both a wiki and a lightweight app-builder, and also what makes structure entirely a matter of user discipline rather than enforced schema
- A **page** is itself a block and can contain any mix of content and sub-pages — there's no structural distinction forcing "this is a doc" vs. "this is a database" the way other tools separate those concerns

### Databases

- A **database** is a structured collection of pages, each with defined **properties** (text, select, date, relation, etc.) — views (table, board, calendar, gallery) are just different renderings of the same underlying data, filtered/sorted independently
- **Relations** link entries between databases; **rollups** pull/aggregate data from a related database's properties — powerful for building interconnected systems (projects ↔ tasks ↔ people), but each relation/rollup adds computation overhead
- Large databases (many thousands of entries) with multiple relations/rollups and complex filtered views can become noticeably slow to load and edit

### Permissions & Sharing

- Sharing/permission settings are set per-page, and **child pages inherit the parent's sharing by default** unless explicitly overridden — similar in spirit to Confluence's model, with the same category of surprise risk
- Workspace-level roles (owner, member, guest) set a baseline; individual page-sharing can grant broader or narrower access on top of that baseline

### API & Automation

- The Notion API is **rate-limited** (a modest number of requests per second per integration) — automations/integrations built without backoff/retry logic will start failing under load or with high-volume syncs
- Integrations must be explicitly granted access to specific pages/databases (they don't automatically see the whole workspace) — but a broadly-shared integration effectively has broad access, same principle as OAuth scope minimization elsewhere

## Best Practices

1. **Design information architecture deliberately before content accumulates** — retrofitting structure onto a large, organically-grown workspace is expensive
2. **Verify sharing/permission inheritance explicitly** for any page containing sensitive content — don't assume a child page's access matches what you intend
3. **Keep relations/rollups purposeful** — each one adds real computation cost, especially in large databases
4. **Establish and enforce template consistency** for recurring content types (meeting notes, project pages, specs)
5. **Build API integrations with rate-limit backoff/retry logic** from the start, not as an afterthought once they start failing
6. **Document workspace ownership explicitly** — critical structure shouldn't depend on one person's institutional knowledge

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Assuming a child page's sharing is independent of its parent | Verify inheritance explicitly; child pages default to inheriting parent sharing unless overridden |
| Unbounded relations/rollups added to a very large database | Add relations/rollups purposefully, and monitor database performance as entry count grows |
| No workspace-wide information architecture, pages created ad hoc | Establish a top-level structure and naming convention deliberately, before organic growth makes retrofitting expensive |
| API integrations with no retry/backoff logic | Build in rate-limit handling from the start |
| Critical workspace structure known only to one person | Document ownership and structure explicitly, so it survives personnel changes |

## Sharp Edges

### SE-1: Permission Inheritance Surprises Exposing Sensitive Content
- **Severity**: critical
- **Situation**: A page is shared broadly (e.g., with the whole workspace) at a high level in the hierarchy for convenience, and a child page created later — containing sensitive information — inherits that broad sharing without anyone realizing it, until the exposure is discovered
- **Cause**: Child pages default to inheriting their parent's sharing settings; nothing prompts a review of that inheritance when new, more sensitive content is added beneath an already-broadly-shared parent
- **Symptoms**:
  - A permissions review finds sensitive content accessible to a far broader audience than intended
  - Someone reports being able to see content they shouldn't have access to
- **Solution**: Explicitly review and, where needed, override sharing settings on any page containing sensitive content rather than assuming inheritance is fine, and periodically audit sharing settings on sensitive page trees
- **Details**: → [extended/checklists.md#permissions-checklist]

### SE-2: Database Relation/Rollup Performance Degradation at Scale
- **Severity**: high
- **Situation**: A database that started small and fast, with several relations and rollups to other databases, becomes noticeably slow to load, filter, and edit once it grows to many thousands of entries
- **Cause**: Each relation/rollup requires Notion to compute cross-database lookups/aggregations, and this cost compounds with both entry count and the number/complexity of relations — a small database with rollups feels instant, a large one with the same structure does not
- **Symptoms**:
  - Specific database views take noticeably longer to load as entry count grows
  - Editing a page with several rollup-dependent properties feels laggy
- **Solution**: Periodically review whether every relation/rollup on a growing database is still earning its cost, consider splitting an overly large database or simplifying rollup chains, and design new databases with expected scale in mind rather than assuming small-database performance will hold

### SE-3: Uncontrolled Page/Database Sprawl With No Information Architecture
- **Severity**: medium
- **Situation**: Pages and databases are created organically over time with no top-level structure, and the workspace becomes a maze of similarly-named, undiscoverable pages that new members can't navigate without asking someone directly
- **Cause**: Notion's flexibility means nothing forces an information architecture decision — a workspace can grow purely bottom-up with no imposed hierarchy
- **Symptoms**:
  - Search returns many stale or duplicate-seeming pages for the same topic
  - Onboarding new members requires a guided tour because the structure isn't self-evident
- **Solution**: Establish a deliberate top-level structure (a small number of clearly-scoped top-level pages/databases) early, and periodically consolidate or archive content that's drifted outside it

### SE-4: API Rate Limiting Breaking Unprepared Automations
- **Severity**: medium
- **Situation**: An integration or automation built against the Notion API works fine in testing but starts failing intermittently once it's handling real production volume, because it hits the API's rate limit with no retry/backoff logic to handle the resulting errors gracefully
- **Cause**: The Notion API enforces a modest per-integration rate limit; code that fires requests without checking for rate-limit responses and backing off will simply fail once that limit is exceeded
- **Symptoms**:
  - An automation that "worked" in testing starts dropping or failing operations under real load
  - API error logs show `429` rate-limit responses with no corresponding retry
- **Solution**: Build rate-limit-aware retry/backoff logic into any Notion API integration from the start, and batch/throttle bulk operations (e.g., large syncs) deliberately rather than firing requests as fast as possible

### SE-5: Single Point of Failure From Undocumented Workspace Ownership
- **Severity**: medium
- **Situation**: One person who originally set up the workspace's structure, templates, and key databases leaves the team, and nobody else understands how the pieces fit together well enough to maintain or extend it safely
- **Cause**: Notion doesn't require or prompt for documented ownership/structure — a workspace can grow to depend entirely on one person's institutional knowledge with nothing written down
- **Symptoms**:
  - Changes to core databases/templates are made cautiously or avoided entirely because nobody remaining fully understands the dependencies
  - Onboarding a new workspace admin takes disproportionately long
- **Solution**: Document critical workspace structure, database schemas, and automation dependencies explicitly (ironically, in Notion itself, in a clearly-owned admin/meta page), and ensure ownership of key structural decisions isn't concentrated in one person with no handoff plan

## Recommended Tools

| Category | Tools |
|------|------|
| API/automation | Notion API, Notion SDK for JS |
| Templates | Notion Templates gallery, workspace-level template pages |
| Sync/integration | Zapier/Make, native Slack/GitHub integrations |
| Migration | Notion's importers (Confluence, Trello, Asana, etc.) |

## Permissions & Structure

**Full checklist**: → [extended/checklists.md#permissions-checklist]

## Related Resources

[Notion Help Center](https://www.notion.so/help) | [Notion API Reference](https://developers.notion.com/) | [Notion Template Gallery](https://www.notion.so/templates)

## Related Domains

[[confluence]] | [[linear]] | [[slack]] | [[javascript]]
