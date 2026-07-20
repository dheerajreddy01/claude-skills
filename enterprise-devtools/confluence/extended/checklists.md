# Confluence — Extended Checklists

## Permissions Checklist

- [ ] Every space has a named owner responsible for structure, permissions, and cleanup
- [ ] Sensitive child pages have explicit restrictions set — not assumed to inherit from a restricted parent
- [ ] Restriction inheritance verified directly (not assumed) for any page tree containing sensitive content
- [ ] Content requiring genuine isolation lives in a separate space with distinct membership, not just page-level restrictions
- [ ] Exports (PDF/Word) and copies of restricted pages reviewed for whether the new artifact needs its own access control
- [ ] Space-level default permissions reviewed periodically, not just set once at creation
- [ ] Global/space administrator access to restricted content is understood and accepted, not assumed to be zero

## Documentation Hygiene Checklist

- [ ] High-stakes docs (runbooks, architecture, onboarding) have a visible "last reviewed" date
- [ ] Documentation updates are part of the same change process as the system/process change itself where feasible
- [ ] Periodic review cycles scheduled for critical pages, not left to be caught reactively
- [ ] Templates used for recurring page types (meeting notes, RFCs, runbooks) to keep structure consistent
- [ ] Heavy macro nesting flagged and simplified where the dynamic content isn't essential
- [ ] Stale/abandoned spaces periodically audited and archived or consolidated
- [ ] A curated space directory maintained pointing to the authoritative location for each major topic
- [ ] Labels/tags applied consistently so search remains useful as content volume grows
