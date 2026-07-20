# Jenkins — Extended Checklists

## Operational Safety Checklist

- [ ] Plugin upgrades tested on a non-production instance with a representative plugin set before applying to production
- [ ] Plugin versions pinned deliberately (via JCasC or documented process), not left to drift on every restart
- [ ] Config/plugin backup taken before any upgrade, with a documented rollback path
- [ ] Every pipeline restricted to a labeled agent — no unrestricted builds running on the controller
- [ ] Executor count per agent sized to genuine resource capacity, not set arbitrarily high
- [ ] Build timeouts (`options { timeout(...) }`) set on every pipeline to prevent indefinite hangs
- [ ] Shared libraries referenced by pinned version/tag, not a mutable branch
- [ ] Script Security approval requests reviewed individually, not blanket-approved

## Credential & Access Checklist

- [ ] No credential value (raw or transformed) ever echoed, printed, or logged
- [ ] `withCredentials` used for every secret reference; no hardcoded tokens/passwords in Jenkinsfiles
- [ ] Credentials scoped to the narrowest folder/project level that needs them, not globally available
- [ ] Role-based access control configured so only appropriate users/teams can view or trigger sensitive pipelines
- [ ] Agent-to-controller communication secured (TLS, restricted network access)
- [ ] Audit trail (Audit Trail plugin or equivalent) enabled for configuration and credential changes
