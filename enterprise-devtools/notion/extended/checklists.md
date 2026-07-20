# Notion — Extended Checklists

## Permissions Checklist

- [ ] Sharing/permission inheritance explicitly reviewed for any page tree containing sensitive content
- [ ] Child pages under a broadly-shared parent audited for whether they should override that inheritance
- [ ] Workspace-level roles (owner/member/guest) reviewed periodically against actual current membership
- [ ] Integrations granted access only to the specific pages/databases they need, not broad workspace access
- [ ] Sensitive databases/pages spot-checked periodically for unintended broad sharing

## Structure & Performance Checklist

- [ ] Top-level workspace information architecture documented and deliberately maintained
- [ ] Relations/rollups on large, growing databases reviewed periodically for whether they're still earning their performance cost
- [ ] Templates used consistently for recurring content types
- [ ] API integrations/automations include rate-limit-aware retry and backoff logic
- [ ] Bulk sync operations batched/throttled rather than firing requests at maximum rate
- [ ] Critical workspace structure and database schemas documented, not dependent on one person's institutional knowledge
- [ ] Stale or duplicate pages/databases periodically consolidated or archived
