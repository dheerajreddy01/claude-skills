# Slack — Extended Checklists

## Security Checklist

- [ ] Every installed app's OAuth scopes reviewed against what its functionality actually uses
- [ ] Bot tokens used instead of user tokens unless acting-as-the-installing-user is genuinely required
- [ ] No webhook URLs committed to version control or pasted into broadly-visible documents
- [ ] Webhook URLs stored in a secret manager/environment variable, rotated immediately if exposure is suspected
- [ ] Installed-app audit performed periodically at the workspace admin level, not just at install time
- [ ] Message retention policy set deliberately based on actual compliance requirements, not left at default
- [ ] External/shared channels (Slack Connect) reviewed for what internal information is exposed to external parties
- [ ] Sensitive discussions kept out of broadly-accessible public channels

## Workspace & Bot Hygiene Checklist

- [ ] Channel naming and archival convention established and actually enforced
- [ ] Inactive channels periodically flagged and archived
- [ ] Notification/alert bots use batching, thresholds, or severity-based routing instead of posting on every event
- [ ] Threads used for sub-conversations to keep channels scannable
- [ ] Bot-generated noise reviewed periodically against whether the channel is still being read/acted on
- [ ] Critical automations (Workflow Builder flows, webhook integrations) documented with an owner, not left as an undocumented dependency
