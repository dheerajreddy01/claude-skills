# Kubernetes — Extended Checklists

## Reliability Checklist

- [ ] Every container has explicit CPU/memory `requests` and `limits`, based on observed usage
- [ ] Namespace-level `LimitRange`/`ResourceQuota` set to enforce sane defaults and caps
- [ ] Readiness probe reflects genuine "able to serve traffic" state, not just "process started"
- [ ] Liveness probe timing/thresholds tuned against real observed startup and response behavior
- [ ] `startupProbe` used for slow-starting applications instead of a lenient liveness probe
- [ ] Rollout strategy (`maxUnavailable`/`maxSurge`) reviewed so deploys don't cause traffic-serving gaps
- [ ] PodDisruptionBudgets set for workloads that shouldn't lose all replicas simultaneously during node maintenance
- [ ] StatefulSets/PVCs use a storage class with the durability guarantees the workload actually needs

## Security Checklist

- [ ] Cluster has `EncryptionConfiguration`/KMS provider enabled for Secrets at rest
- [ ] RBAC access to Secrets restricted — few roles have `get`/`list` on Secret resources
- [ ] No ServiceAccount bound to `cluster-admin` without explicit, documented justification
- [ ] `kubectl auth can-i --list` run for critical ServiceAccounts to verify actual granted permissions match intended scope
- [ ] Namespaced Roles used instead of ClusterRoles wherever access is genuinely namespace-scoped
- [ ] Network policies applied to restrict Pod-to-Pod traffic to what's actually needed, not fully open by default
- [ ] Images scanned for vulnerabilities before deployment (ties into [[docker]] security practices)
- [ ] Admission control/policy enforcement (Kyverno, OPA Gatekeeper) used to block non-compliant manifests (e.g., missing resource limits, running as root)
