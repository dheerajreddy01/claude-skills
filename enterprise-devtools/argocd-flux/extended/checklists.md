# ArgoCD / Flux — Extended Checklists

## GitOps Safety Checklist

- [ ] No plaintext secrets committed to the GitOps repo — Sealed Secrets, SOPS, or an External Secrets Operator used instead
- [ ] Secret scanning runs in CI/pre-commit on the GitOps repo
- [ ] Prune behavior tested in a non-production environment before enabling on production Applications
- [ ] File renames/path restructuring reviewed with extra scrutiny in PR review for prune-affecting impact
- [ ] Deletion protection/finalizers applied to critical stateful resources
- [ ] Sync waves / `dependsOn` configured for resources with genuine startup order dependencies
- [ ] Health checks configured so the reconciler waits for prerequisites to be healthy, not just applied
- [ ] Production uses manual sync or automated-sync-with-approval, not fully automated sync with no review gate
- [ ] Emergency-change procedure documented and rehearsed (pause reconciliation, or fix via git) — not left for on-call to improvise mid-incident
- [ ] GitOps-managed namespaces/resources clearly labeled or documented so accidental out-of-band changes are less likely

## Operational Checklist

- [ ] Sync/health status monitored as a first-class signal, not just deployment success
- [ ] RBAC restricts direct write access to GitOps-managed namespaces for anyone not working through the git-based process
- [ ] Multi-cluster/multi-tenant structure (ApplicationSets, Flux multi-tenancy) reviewed for appropriate isolation between teams/environments
- [ ] Repo structure documented so new team members understand which paths map to which clusters/environments
- [ ] Reconciliation interval tuned appropriately — too frequent adds unnecessary API load, too infrequent delays drift correction
