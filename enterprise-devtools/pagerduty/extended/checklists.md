# PagerDuty — Extended Checklists

## On-Call Reliability Checklist

- [ ] Every escalation policy has at least two independent levels (no level resolves to the same unavailable person as the one before it)
- [ ] Every on-call user has a real urgent notification rule configured (push/SMS/phone), not only email
- [ ] Schedules reviewed in each affected member's local time zone, not just the admin's view
- [ ] Schedule gaps/overlaps report checked before relying on a new or modified rotation
- [ ] Secondary/backup layer configured on schedules for critical services, not a lone primary rotation
- [ ] Overrides used for one-off swaps instead of editing the underlying rotation
- [ ] Event orchestration deduplication (`dedup_key`) configured for known-noisy upstream integrations
- [ ] Service/alert urgency reviewed against actual incident severity data, not left at a static default
- [ ] Every service has a runbook link or clear next-steps description attached
- [ ] Postmortems completed for every high-severity incident, with action items tracked to closure (not just written and forgotten)
