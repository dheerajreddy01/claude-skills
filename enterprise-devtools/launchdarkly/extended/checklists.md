# LaunchDarkly — Extended Checklists

## Flag Lifecycle Checklist

- [ ] Every flag has a clear owner and an intended cleanup date/trigger condition at creation time
- [ ] Flags at 100%/0% for an extended period are flagged for review and removal
- [ ] Dead code paths removed promptly after a flag's rollout is complete and stable
- [ ] Flag evaluation data reviewed periodically to identify settled (single-value) flags as cleanup candidates
- [ ] Rollout flags clearly distinguished from intentional long-term operational toggles
- [ ] Number of concurrently active flags per code path kept low to bound combinatorial complexity
- [ ] Targeting rule changes tested/previewed against representative contexts before applying to production
- [ ] Actual flag evaluation distribution monitored against intended rollout percentage as a sanity check

## Risk & Security Checklist

- [ ] Risky changes (new integrations, rewritten critical paths) get a kill-switch flag wired in before shipping, not after an incident
- [ ] Permissions on modifying targeting rules restricted for high-risk/high-blast-radius flags
- [ ] Client-side flag SDK payloads reviewed for accidental exposure of unreleased feature names/targeting logic
- [ ] Sensitive/embargoed flags evaluated server-side rather than delivered to client-side SDKs
- [ ] Flag-related incidents (misconfiguration, unintended rollout) included in postmortem review with process fixes, not just the immediate flag correction
