# Istio / Linkerd — Extended Checklists

## Mesh Rollout Checklist

- [ ] mTLS rolled out `PERMISSIVE` first, with full workload enrollment verified via mesh telemetry before switching to `STRICT`
- [ ] Retry ownership decided explicitly per call path (application vs. mesh), with the non-owning layer's retries disabled for that path
- [ ] Retry budgets/circuit breakers configured at the mesh level to cap amplification during real outages
- [ ] Sidecar resource requests/limits budgeted explicitly in cluster capacity planning
- [ ] Added latency from sidecar hops measured in a representative environment before full production rollout
- [ ] `AuthorizationPolicy`/mTLS configuration changes tracked and correlated with incident timing during on-call investigations
- [ ] Canary/traffic-split configuration tested in non-production before routing real traffic

## Control Plane Operations Checklist

- [ ] Control plane run with HA configuration appropriate to its blast radius (affects every mesh-enrolled service)
- [ ] Mesh version upgrades tested in a non-production cluster before production rollout
- [ ] Explicit rollback plan documented for control plane/mesh-wide policy changes
- [ ] Mesh visualization/observability tooling (Kiali, Linkerd Viz) used as a standard part of on-call tooling, not just for one-off debugging
- [ ] Sidecar and control plane logs included in the standard incident-investigation runbook for inter-service connectivity issues
- [ ] Decision to adopt the mesh re-validated periodically against actual service count/complexity — not assumed permanently justified
