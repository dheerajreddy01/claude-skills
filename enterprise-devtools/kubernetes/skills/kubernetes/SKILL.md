---
schema: "1.0"
name: kubernetes
version: "1.0.0"
description: Kubernetes workload design, resource requests/limits, RBAC, networking, and cluster reliability practices
domain: technology
triggers:
  keywords:
    primary: [kubernetes, k8s, pod, deployment, helm]
    secondary: [service, ingress, configmap, secret, rbac, statefulset, hpa]
  context_boost: [cluster, orchestration, container platform]
  context_penalty: [docker-compose, single container]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Kubernetes

> Declarative workloads that stay scheduled, bounded, and least-privileged

## Applicable Scenarios

- Designing or reviewing Kubernetes manifests (Deployments, Services, Ingress)
- Setting resource requests/limits and autoscaling policy
- Diagnosing crash loops, scheduling failures, or networking issues
- Writing or auditing RBAC roles and bindings
- Structuring Helm charts for reusable deployments

## Core Knowledge

### Architecture

| Component | Role |
|------|------|
| **Control plane** | API server, scheduler, controller manager, etcd — decides *what should be running where* |
| **Node** | Runs workloads via the kubelet + container runtime |
| **kubelet** | Per-node agent that ensures containers described in Pod specs are actually running |
| **etcd** | Cluster's source of truth (key-value store) — backing this up is backing up the entire cluster state |

### Core Objects

| Object | Purpose |
|------|------|
| **Pod** | Smallest deployable unit — one or more tightly-coupled containers sharing network/storage |
| **Deployment** | Manages a ReplicaSet of Pods, handles rolling updates/rollbacks |
| **Service** | Stable network identity/load balancing for a set of Pods (ClusterIP, NodePort, LoadBalancer) |
| **Ingress** | L7 HTTP(S) routing into the cluster, typically backed by an Ingress controller |
| **ConfigMap / Secret** | External configuration/sensitive data injected into Pods — Secrets are base64-encoded, **not encrypted**, at rest by default |
| **StatefulSet** | Like a Deployment, but for workloads needing stable identity/storage (databases, queues) |

### Scheduling & Resources

- The scheduler places Pods on nodes based on **requests** (guaranteed minimum) — **limits** cap actual usage
- `requests` drive scheduling decisions; `limits` drive runtime enforcement (CPU throttling, OOM kill on memory)
- A Pod with no requests/limits set is treated as "BestEffort" QoS — first to be evicted under node pressure, and can also starve neighbors before eviction kicks in

### Networking

- Every Pod gets its own IP; Pods communicate across nodes via the cluster's CNI (Calico, Cilium, etc.) without NAT
- **Services** provide a stable virtual IP/DNS name that load-balances across the Pods matching its label selector — Pods themselves are ephemeral and shouldn't be addressed directly
- **Ingress** routes external HTTP(S) traffic to Services based on host/path rules, needs an Ingress controller (nginx, ALB, etc.) actually running in the cluster to do anything

### RBAC

- **Roles/ClusterRoles** define permissions (verbs on resources); **RoleBindings/ClusterRoleBindings** grant them to users/groups/service accounts
- Namespaced `Role`+`RoleBinding` scopes access to one namespace; `ClusterRole`+`ClusterRoleBinding` can grant cluster-wide access — using cluster-wide grants for namespace-scoped needs is a common over-permissioning mistake

## Best Practices

1. **Always set resource requests and limits** — an unbounded Pod can starve its node or be evicted unpredictably
2. **Configure both liveness and readiness probes**, and make sure they check different things — liveness for "should this be restarted," readiness for "should this receive traffic"
3. **Use least-privilege RBAC** — namespaced Roles over ClusterRoles wherever the access is genuinely namespace-scoped
4. **Never assume Secrets are encrypted at rest by default** — enable encryption at rest (KMS-backed) or use an external secrets manager for anything sensitive
5. **Use Deployments (not bare Pods)** for anything that should self-heal, and StatefulSets for anything needing stable identity/storage
6. **Version and template Helm charts** rather than hand-editing manifests per environment — drift between environments is a common source of "works in staging, not in prod"

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| No resource requests/limits set | Set both, based on observed usage, not guesses |
| Liveness probe checking the same thing as readiness | Liveness = "is the process alive and not deadlocked," readiness = "is it ready to serve traffic" — make them meaningfully different |
| Storing sensitive data in a ConfigMap | Use a Secret (and encryption at rest / an external secrets manager) — ConfigMaps are plaintext |
| Granting `cluster-admin` to a service account "to make it work" | Scope a namespaced Role to only the verbs/resources actually needed |
| Rolling update with no readiness probe | Add a readiness probe so the rollout waits for new Pods to be genuinely ready before shifting traffic |

## Sharp Edges

### SE-1: Missing Resource Requests/Limits Causing Node Instability
- **Severity**: critical
- **Situation**: A Pod with no memory limit leaks memory and consumes all available memory on its node, triggering the kernel OOM killer to kill unrelated Pods on the same node
- **Cause**: Without a memory limit, a Pod can consume unbounded memory; without a request, the scheduler has no accurate picture of the Pod's actual footprint, leading to over-scheduling on a node
- **Symptoms**:
  - Unrelated Pods on the same node get OOM-killed or evicted
  - `kubectl describe node` shows memory pressure conditions
- **Solution**: Set both requests and limits based on observed usage (via `kubectl top` or a metrics pipeline over time), and use `LimitRange`/`ResourceQuota` at the namespace level to enforce sane defaults and caps
- **Details**: → [extended/checklists.md#reliability-checklist]

### SE-2: Liveness Probe Misconfiguration Causing Crash Loops
- **Severity**: high
- **Situation**: A liveness probe with too short a timeout or too aggressive a failure threshold kills a Pod that's just slow to start or briefly under load, and Kubernetes keeps restarting it — never letting it stabilize (`CrashLoopBackOff`)
- **Cause**: Liveness probes trigger a container restart on failure; if the probe's timing doesn't account for real startup/load variance, a healthy-but-slow Pod looks "dead" to the probe
- **Symptoms**:
  - Pod status shows `CrashLoopBackOff` despite the application logs showing no actual crash, just slow startup
  - Increasing replica count makes the problem worse (more Pods competing for the same node resources during startup)
- **Solution**: Use a `startupProbe` for slow-starting applications (delays liveness checks until startup completes), and tune liveness probe thresholds/timeouts against real observed startup and response times, not defaults copied from an unrelated service

### SE-3: Secrets Are Not Encrypted at Rest by Default
- **Severity**: critical
- **Situation**: Sensitive data stored in a Kubernetes Secret is assumed to be securely encrypted, but etcd stores it as base64-encoded (trivially decodable) plaintext unless encryption at rest is explicitly configured
- **Cause**: Base64 is an encoding, not encryption; Kubernetes only encrypts Secret data at rest if the cluster has `EncryptionConfiguration` set up with a KMS provider
- **Symptoms**:
  - A security audit finds Secret data recoverable directly from an etcd backup or snapshot
  - Anyone with `get`/`list` RBAC access on Secrets (often broader than intended) can trivially decode them
- **Solution**: Enable encryption at rest via a KMS provider, restrict RBAC access to Secrets tightly (few roles should have `get`/`list` on Secrets), and consider an external secrets manager (Vault, cloud provider secret store) with Kubernetes as a thin integration layer rather than the source of truth

### SE-4: Overly Permissive RBAC via ClusterRoleBindings
- **Severity**: critical
- **Situation**: A service account is bound to `cluster-admin` (or another broad ClusterRole) to unblock a deployment, and it's never revisited — that workload can now read/modify anything in the cluster, including other teams' namespaces
- **Cause**: Broad bindings are the fastest way to make an RBAC error disappear, so they accumulate under delivery pressure and are rarely tightened afterward
- **Symptoms**:
  - `kubectl auth can-i --list --as=system:serviceaccount:<ns>:<sa>` shows far more permissions than the workload's actual function requires
  - A compromised workload/container can affect resources in unrelated namespaces
- **Solution**: Scope every ServiceAccount to a namespaced Role with only the verbs/resources actually used, audit ClusterRoleBindings periodically, and treat any `cluster-admin` binding to a workload identity as something requiring explicit justification

### SE-5: Rolling Updates Without Readiness Probes Causing Downtime
- **Severity**: high
- **Situation**: A Deployment rollout shifts traffic to new Pods before they're actually able to serve requests (still initializing DB connections, warming caches), causing a spike in errors during every deploy
- **Cause**: Without a readiness probe, Kubernetes considers a Pod "ready" as soon as its containers start, not when the application inside is actually able to handle traffic
- **Symptoms**:
  - Error rate/latency spikes correlate precisely with deploy events
  - Rollouts "complete successfully" per Kubernetes, but user-facing errors briefly increase each time
- **Solution**: Add a readiness probe that genuinely reflects "able to serve traffic" (e.g., a dependency-aware health endpoint, not just "process is running"), and tune `maxUnavailable`/`maxSurge` in the rollout strategy to control how aggressively traffic shifts

## Recommended Tools

| Category | Tools |
|------|------|
| Package/templating | Helm, Kustomize |
| Observability | kubectl top, Metrics Server, Prometheus + kube-state-metrics |
| Security | kube-bench, Kyverno/OPA Gatekeeter (policy enforcement) |
| GitOps | ArgoCD, Flux |

## Reliability & Security

**Full checklist**: → [extended/checklists.md#reliability-checklist]

## Related Resources

[Kubernetes Docs](https://kubernetes.io/docs/) | [Kubernetes Patterns](https://k8spatterns.io/) | [CNCF Landscape](https://landscape.cncf.io/)

## Related Domains

[[docker]] | [[aws]] | [[gcp]] | [[azure]] | [[grafana-prometheus]]
