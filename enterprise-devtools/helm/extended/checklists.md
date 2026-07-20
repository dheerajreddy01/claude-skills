# Helm — Extended Checklists

## Chart Safety Checklist

- [ ] `helm lint` and `helm template` run in CI against every real environment's values file, not just chart defaults
- [ ] Edge-case values (empty lists, zero replicas, missing optional keys) explicitly tested for template rendering correctness
- [ ] Immutable fields on underlying resource types identified and handled deliberately (avoided in-place, or via explicit delete/recreate hooks)
- [ ] CRD installation/upgrade managed as an explicit separate step, not assumed to be handled by `helm upgrade`
- [ ] Resource requests/limits set with sane defaults in the chart itself, not left unset or only documented
- [ ] Subchart dependency versions pinned explicitly in `Chart.yaml`
- [ ] `Chart.yaml`'s `version` and `appVersion` both updated deliberately and independently on release
- [ ] `helm get values -a`/`helm template` used to verify actual merged/rendered output before trusting values file assumptions

## Release Management Checklist

- [ ] Environment-specific values kept in separate, version-controlled files layered onto shared defaults
- [ ] `--set` avoided for production deploys; reserved for local debugging only
- [ ] `helm history` reviewed as part of incident investigation when a release behaves unexpectedly
- [ ] Rollback tested (not just assumed to work) for releases with stateful resources or CRDs
- [ ] Release naming/namespace conventions consistent enough that `helm list` output is unambiguous across environments
- [ ] Chart repository (ChartMuseum, OCI registry) access controlled appropriately for who can publish new chart versions
- [ ] Post-upgrade smoke tests or hooks configured to verify a release actually came up healthy, not just that `helm upgrade` exited successfully
