---
schema: "1.0"
name: confluence
version: "1.0.0"
description: Confluence space hierarchy, page permissions, templates, and documentation hygiene practices
domain: technology
triggers:
  keywords:
    primary: [confluence, wiki, documentation space, page tree]
    secondary: [atlassian, macro, space permissions, page restriction]
  context_boost: [documentation, knowledge base, internal wiki]
  context_penalty: [jira, notion, github wiki]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Confluence

> Documentation that's easy to write and easy to lose track of — permissions and drift are the real maintenance burden

## Applicable Scenarios

- Designing space and page hierarchy for a team or project's documentation
- Setting page/space permissions and restrictions for sensitive content
- Building reusable page templates and macros for consistent documentation
- Auditing stale or drifted documentation against the systems it describes
- Integrating Confluence with Jira for linked requirements/specs

## Core Knowledge

### Spaces & Page Hierarchy

- A **space** is the top-level container (typically per team, product, or department); **pages** nest under it in a tree, with child pages inheriting the parent's position in navigation but not automatically its content
- Space **permissions** control who can view/edit/admin within a space; **page restrictions** layer on top, narrowing (never widening) access to a specific page or its children below the space-level default
- Deeply nested page trees without a clear owner become hard to navigate — a flatter structure with strong search/labels often ages better than a deep hierarchy that made sense to whoever built it

### Page Restrictions

- Restrictions are **additive narrowing**: a page restricted to a specific group is invisible/inaccessible to anyone outside that group, regardless of their space-level permission
- Restrictions do **not** automatically cascade the way people expect in every direction — a child page created under a restricted parent does not automatically inherit that restriction unless explicitly set, and copying/moving a restricted page can change its effective visibility
- Space administrators and Confluence global administrators can typically still see restricted content structure even when individual page restrictions apply — restriction is not a substitute for a separate space with genuinely different membership when true isolation is required

### Templates & Macros

- **Templates** standardize recurring page types (meeting notes, RFCs, runbooks) so structure stays consistent across authors and over time
- **Macros** embed dynamic content (Jira issue lists, table of contents, status indicators, other pages' content via excerpt-include) — powerful, but each macro is a piece of logic that can break silently on migration, permission changes, or app deprecation
- Jira/Confluence integration via macros (e.g., embedding a live Jira issue list) keeps documentation and work tracking loosely synced, but only for what's explicitly wired together — most prose content has no such live link and drifts silently

## Best Practices

1. **Assign explicit space ownership** — every space should have a named owner responsible for its structure and cleanup, not an implicit "whoever created it"
2. **Set page restrictions deliberately, not reflexively** — verify what a restriction actually covers (space-level default vs. page-level narrowing) before relying on it for sensitive content
3. **Use templates for recurring page types** to keep structure consistent and reduce the blank-page problem
4. **Label and tag pages consistently** so search and space-level views stay useful as content grows
5. **Review and archive stale pages periodically** — treat documentation like code that needs deprecation, not a permanent artifact
6. **Minimize deep macro dependency chains** — a page built from many nested macros/includes is fragile to migrate and slow to render

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Assuming a child page inherits its parent's restriction automatically | Explicitly set restrictions on each page that needs them; verify inheritance behavior rather than assuming it |
| No named space owner, so cleanup never happens | Assign explicit space ownership as part of space creation |
| Copying a restricted page into a broader space "to reuse the content" | Recreate or reference content deliberately, checking the destination's own permissions rather than assuming restriction travels with a copy |
| Pages never revisited after the system/process they document changes | Schedule periodic documentation reviews, especially for runbooks and onboarding docs |
| Deep macro-in-macro nesting for a "smart" page | Prefer simpler, more maintainable structures — flag heavy macro use as a migration/fragility risk |

## Sharp Edges

### SE-1: Page Restriction Inheritance Confusion Exposing Sensitive Content
- **Severity**: critical
- **Situation**: A team assumes that restricting a top-level page also restricts every page created beneath it, and a child page containing sensitive information (credentials references, incident postmortems, compensation data) is left accessible to the entire space's normal membership
- **Cause**: Confluence page restrictions do not automatically cascade to child pages in the way many users expect — each page's effective visibility depends on its own restriction settings combined with space-level permissions, not simply "inside a restricted section"
- **Symptoms**:
  - A permissions audit finds a sensitive child page with no restriction, sitting under a restricted parent
  - Someone outside the intended audience reports being able to view content the team believed was locked down
- **Solution**: Explicitly set restrictions on every page that needs them rather than relying on a parent page's restriction, and periodically audit sensitive content trees to confirm restrictions actually apply where assumed; for content requiring genuine isolation, use a separate space with distinct membership instead of relying solely on page-level restrictions
- **Details**: → [extended/checklists.md#permissions-checklist]

### SE-2: Documentation Drift Becoming Actively Misleading
- **Severity**: high
- **Situation**: A runbook, architecture doc, or onboarding page describes a system that has since changed significantly, and someone follows it during an incident or onboarding, taking wrong or dangerous action based on stale information
- **Cause**: Nothing in Confluence enforces that documentation updates alongside the systems/processes it describes — pages persist indefinitely with no automatic staleness signal
- **Symptoms**:
  - A runbook's steps reference infrastructure/tools that no longer exist or work differently
  - New hires report following documentation that turned out to be wrong, causing confusion or incidents
- **Solution**: Treat documentation updates as part of the same change process as the system change itself (linked in the same PR/ticket where feasible), schedule periodic review cycles for high-stakes docs (runbooks, architecture, onboarding), and add visible "last reviewed" dates so staleness is at least detectable

### SE-3: Macro and Plugin Sprawl Slowing Pages and Complicating Migration
- **Severity**: medium
- **Situation**: Pages accumulate many nested macros (live Jira lists, embedded excerpts from other pages, third-party app macros) over time, and page load becomes slow while any future migration or export becomes far harder because macro behavior doesn't always translate
- **Cause**: Each macro is effectively a small piece of dynamic logic evaluated at render time; nesting many of them compounds render cost, and third-party/app-provided macros can be deprecated or behave differently after a Confluence version/plugin update
- **Symptoms**:
  - Specific pages are noticeably slower to load than plain-text pages
  - A Confluence upgrade or space export breaks specific pages that relied on deprecated or app-specific macros
- **Solution**: Prefer simpler page structures over heavy macro nesting where the dynamic content isn't essential, periodically audit which macros/apps are actually load-bearing versus historical clutter, and treat heavy macro dependency as a flag when planning any migration

### SE-4: Uncontrolled Space Proliferation With No Ownership
- **Severity**: medium
- **Situation**: Teams create new spaces freely for one-off projects or initiatives, nobody is assigned ownership, and over time the instance accumulates dozens of abandoned, undiscoverable spaces that fragment where information actually lives
- **Cause**: Space creation has low friction and no default ownership/lifecycle requirement, so spaces outlive the initiatives that created them with no process to archive or consolidate
- **Symptoms**:
  - Search results return multiple stale, near-duplicate spaces for the same topic
  - New team members can't tell which of several similarly-named spaces is the current source of truth
- **Solution**: Require a named owner at space creation, periodically audit space activity and archive/consolidate inactive ones, and maintain a small, curated space directory pointing to the current, authoritative location for each major topic

### SE-5: Exported or Copied Content Leaking Restricted Information
- **Severity**: high
- **Situation**: A restricted page's content is copied into a new page (or exported as PDF/Word and shared externally), and the restriction that applied to the original doesn't travel with the copy — the sensitive content is now available to a much broader audience than intended
- **Cause**: Page restrictions are a property of the specific page object, not of the content itself — copying, exporting, or including content via macro creates a new artifact that must have its own access controls set deliberately
- **Symptoms**:
  - Sensitive content surfaces in an exported document or a duplicated page with no restriction applied
- **Solution**: Treat any copy, export, or macro-included reuse of restricted content as requiring its own explicit permission review, and avoid routine exporting of sensitive pages to formats (PDF, Word, shared links) that bypass Confluence's access control entirely

## Recommended Tools

| Category | Tools |
|------|------|
| Templates | Confluence Templates, Blueprints |
| Integration | Jira macros, Confluence REST API |
| Governance | Space permissions audit, Content Reporting |
| Migration | Confluence Cloud Migration Assistant |

## Permissions & Documentation Hygiene

**Full checklist**: → [extended/checklists.md#permissions-checklist]

## Related Resources

[Confluence Documentation](https://support.atlassian.com/confluence-cloud/) | [Confluence Permissions Guide](https://support.atlassian.com/confluence-cloud/docs/what-are-confluence-permissions/) | [Atlassian Community](https://community.atlassian.com/)

## Related Domains

[[jira]] | [[notion]] | [[slack]] | [[project-management]]
