# OpenShift / OKD — Extended Checklists

## Platform Compatibility Checklist

- [ ] Container images don't hardcode a fixed UID; writable paths use group ownership (GID 0) for arbitrary-UID compatibility under the `restricted` SCC
- [ ] SCC rejections diagnosed at the root cause before requesting a broader SCC (`anyuid`, `privileged`)
- [ ] Ingress annotations from other controllers (nginx-ingress, etc.) audited for OpenShift Route-native equivalents during migration
- [ ] BuildConfig/S2I builder image conventions confirmed to match the application's actual project structure, or a Dockerfile strategy used instead
- [ ] Project ResourceQuota/LimitRange checked before deploying, especially in shared multi-tenant clusters
- [ ] `oc get events`/Pod description reviewed for admission denials before assuming a deployment failure is application-level
- [ ] Multi-tenancy boundaries (Projects, RBAC, network policies) reviewed for workloads shared across teams

## Operator & Build Governance Checklist

- [ ] OLM subscription channels set deliberately (manual approval or a stable channel) for stateful/upgrade-sensitive Operators
- [ ] Operator upgrade release notes reviewed before approving, not auto-applied
- [ ] BuildConfig triggers (webhook, image change) reviewed so builds fire intentionally, not on unexpected triggers
- [ ] Custom S2I builder images version-pinned and maintained where the standard builder conventions don't fit
- [ ] Route TLS termination strategy (edge/passthrough/re-encrypt) chosen deliberately based on where encryption needs to terminate
- [ ] Cluster-wide vs Project-scoped Operator installation reviewed for whether cluster-wide access is actually warranted
