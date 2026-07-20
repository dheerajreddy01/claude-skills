---
schema: "1.0"
name: openshift
version: "1.0.0"
description: OpenShift/OKD Security Context Constraints, Routes, source-to-image builds, and Operator-based platform practices
domain: technology
triggers:
  keywords:
    primary: [openshift, okd, red hat kubernetes, security context constraint]
    secondary: [route, buildconfig, imagestream, operator lifecycle manager, project]
  context_boost: [enterprise kubernetes, s2i build, multi-tenant cluster]
  context_penalty: [vanilla kubernetes, minikube, kind]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# OpenShift / OKD (Enterprise Kubernetes)

> Kubernetes-compatible, but opinionated defaults mean "it works on vanilla k8s" isn't a guarantee here

## Applicable Scenarios

- Migrating or adapting Kubernetes workloads to run on OpenShift/OKD
- Diagnosing Security Context Constraint (SCC) rejections
- Choosing between Routes and Ingress for external traffic
- Setting up source-to-image (S2I) BuildConfigs
- Managing platform capabilities via Operators (OLM)

## Core Knowledge

### How OpenShift Differs From Vanilla Kubernetes

| Kubernetes Concept | OpenShift Equivalent/Addition |
|------|------|
| Ingress | **Route** — OpenShift's native L7 routing object, predates and coexists with Ingress support |
| Namespace | **Project** — a Namespace with additional OpenShift-specific defaults, quotas, and RBAC scaffolding |
| Pod SecurityContext (permissive by default in vanilla k8s) | **Security Context Constraints (SCCs)** — restrictive by default, control what UID/capabilities/volumes a Pod can use |
| `docker build` + push, done externally | **BuildConfig + Source-to-Image (S2I)** — in-cluster build pipeline from source to running image |

OpenShift is Kubernetes-conformant underneath — standard `kubectl`/manifests mostly work — but its defaults are meaningfully more restrictive and its platform layer (Routes, Builds, Operators) adds concepts with no direct vanilla-Kubernetes equivalent.

### Security Context Constraints (SCCs)

- SCCs are OpenShift's admission-time policy layer controlling what a Pod is allowed to do — which UID it can run as, whether it can run privileged, which volume types it can mount, which capabilities it can hold
- The default `restricted` SCC (or `restricted-v2` in newer versions) forbids running as a specific fixed UID (assigns an arbitrary one from the Project's allowed range instead) and drops most capabilities — a container image that hardcodes `USER 1000` or expects to run as root will fail to schedule or fail at runtime
- SCCs are assigned per ServiceAccount, evaluated at admission — a workload needing broader permissions needs an explicit SCC grant, not just a Pod spec change

### Routes vs. Ingress

- A **Route** exposes a Service externally with OpenShift-native features (edge/passthrough/re-encrypt TLS termination, built into the Router) — this predates Kubernetes Ingress and remains the more OpenShift-idiomatic option
- OpenShift also supports standard Kubernetes `Ingress` objects (translated internally to Routes by the cluster), but some Ingress annotations from other Ingress controllers (nginx-ingress, etc.) don't translate directly

### Builds & Operators

- **BuildConfig** + **Source-to-Image (S2I)**: OpenShift can build a container image directly from source code in-cluster, using a builder image appropriate to the language/framework — an alternative to building images externally and pushing to a registry
- **Operator Lifecycle Manager (OLM)**: OpenShift's model for installing and managing platform capabilities (databases, monitoring stacks, service meshes) as Operators, with subscription channels controlling automatic upgrade behavior

## Best Practices

1. **Don't hardcode a specific UID in container images** meant to run on OpenShift — design for an arbitrary assigned UID (group-writable permissions, `GID`-based access) under the default `restricted` SCC
2. **Prefer Routes for OpenShift-native TLS termination features**, but understand which Ingress annotations (if migrating from another platform) won't translate
3. **Request the narrowest SCC that actually satisfies the workload's needs** — don't reach for `anyuid`/`privileged` as the default fix for an SCC rejection
4. **Set OLM subscription channels deliberately** (not always "latest"/automatic) for Operators managing anything stateful or sensitive to unplanned upgrades
5. **Review Project-level ResourceQuotas and LimitRanges** before deploying — OpenShift Projects often ship with tighter default quotas than an unconstrained vanilla-Kubernetes namespace
6. **Use S2I builds where they fit the workflow**, but understand they're an OpenShift-specific build model — don't assume portability to plain Kubernetes without adapting the build pipeline

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Container image hardcodes `USER 1000` (or root) | Design the image to run under any arbitrarily assigned UID, using group permissions (GID 0) for writable paths |
| Assuming an nginx-ingress-style annotation works on an OpenShift Route/Ingress | Check which annotations actually translate, or use Route-native fields for OpenShift-specific TLS/routing behavior |
| Requesting `anyuid` or `privileged` SCC to unblock a deployment | Diagnose the actual SCC rejection reason first — most workloads fit under `restricted` with minor image adjustments |
| Leaving OLM subscriptions on automatic/latest channel for stateful Operators | Pin to a specific channel and review changelogs before allowing an upgrade |
| Assuming a Project has the same default resource availability as an empty vanilla-Kubernetes namespace | Check Project-level ResourceQuota/LimitRange before deploying, especially in shared multi-tenant clusters |

## Sharp Edges

### SE-1: Default SCC Rejecting Workloads That Ran Fine on Vanilla Kubernetes
- **Severity**: critical
- **Situation**: A workload/Helm chart that deploys cleanly on vanilla Kubernetes (e.g., a container image hardcoding `USER 1000` for a specific data directory's ownership) fails to schedule on OpenShift with a permission-denied or SCC-related admission error
- **Cause**: The default `restricted` SCC doesn't allow a Pod to run as a specific fixed UID — it assigns an arbitrary UID from the Project's allowed range instead, which breaks images that assume a fixed UID owns specific files/directories
- **Symptoms**:
  - Pod fails to start with a permission error on a mounted volume or application data directory
  - `oc get events` or Pod description shows an SCC-related admission denial
- **Solution**: Build images to work under an arbitrary UID (set group ownership to `root`/GID 0 and make relevant paths group-writable, per OpenShift's guidelines for arbitrary UID support) rather than requesting a broader SCC (`anyuid`) as the default fix
- **Details**: → [extended/checklists.md#platform-compatibility-checklist]

### SE-2: Route/Ingress Feature Mismatch During Migration
- **Severity**: high
- **Situation**: A set of Kubernetes manifests using nginx-ingress-specific annotations (rewrite rules, custom headers, rate limiting annotations) is deployed to OpenShift, and the expected routing behavior silently doesn't happen because those annotations aren't recognized by OpenShift's router
- **Cause**: Ingress annotations are controller-specific — OpenShift's Ingress-to-Route translation and its native Router don't implement the same annotation vocabulary as nginx-ingress or other third-party controllers
- **Symptoms**:
  - Traffic reaches the Service, but expected routing behavior (path rewrites, custom headers, rate limits) doesn't happen
  - No error is raised — the annotation is just silently ignored
- **Solution**: When migrating from another Ingress controller, audit every annotation in use and find its OpenShift Route-native equivalent (or confirm there isn't one and the behavior needs to move elsewhere, e.g., into the application or a service mesh)

### SE-3: BuildConfig/S2I Failures From Assumptions That Don't Hold
- **Severity**: medium
- **Situation**: A source-to-image build fails or produces an unexpected image because the application's build assumptions (a specific build tool version, a non-standard build script location) don't match what the S2I builder image expects
- **Cause**: S2I builder images encode opinionated conventions (expected file locations, supported build commands) for their target language/framework — an application that deviates from those conventions needs explicit configuration or a custom builder image, not just "it'll figure it out"
- **Symptoms**:
  - BuildConfig runs fail with an error indicating a missing expected file or an unrecognized project structure
  - A build that works fine with a plain `docker build` using a custom Dockerfile fails under S2I's conventions
- **Solution**: Either adapt the application to fit the S2I builder's conventions, use a Dockerfile-based BuildConfig strategy instead of S2I when the project doesn't fit those conventions, or build a custom S2I builder image for non-standard cases

### SE-4: Operator Lifecycle Manager Auto-Upgrading Unexpectedly
- **Severity**: high
- **Situation**: An Operator managing a stateful platform component (a database, a message broker) auto-upgrades to a new version because its OLM subscription channel was left on an automatic/latest-tracking setting, and the upgrade introduces a breaking change or brief instability with no advance warning
- **Cause**: OLM subscriptions can be configured for automatic approval of upgrades within a channel — convenient for keeping current, but it means a channel's new release ships to production the moment it's published, without a deliberate review step
- **Symptoms**:
  - A stateful service becomes briefly unavailable or behaves differently, correlated with an Operator's `ClusterServiceVersion` changing with no corresponding change from the team
- **Solution**: Set manual approval (or a more conservative channel) for OLM subscriptions on anything stateful or upgrade-sensitive, and review release notes before approving an Operator upgrade rather than letting it apply automatically

### SE-5: Project ResourceQuota Defaults Blocking Deployments
- **Severity**: medium
- **Situation**: A workload that would deploy fine on an unconstrained vanilla-Kubernetes namespace fails to schedule (or gets throttled unexpectedly) on OpenShift because the Project it's deployed into has tighter default ResourceQuota/LimitRange settings than the team assumed
- **Cause**: OpenShift Projects, especially in shared multi-tenant clusters, are commonly provisioned with default quotas as a governance mechanism — this is an intentional platform behavior, but teams coming from unconstrained Kubernetes namespaces don't always expect it
- **Symptoms**:
  - Pod creation fails with a quota-exceeded admission error despite the cluster overall having ample capacity
  - A deployment that specifies resource requests works in one Project but fails in another with a lower quota
- **Solution**: Check `oc describe quota`/`oc describe limitrange` for the target Project before deploying, and request a quota increase through the platform team's actual process rather than assuming resource availability

## Recommended Tools

| Category | Tools |
|------|------|
| CLI | `oc` (OpenShift CLI, superset of `kubectl`) |
| Operator management | Operator Lifecycle Manager (OLM), OperatorHub |
| Build | Source-to-Image (S2I), BuildConfig, Tekton Pipelines |
| Security | `oc adm policy`, SCC review tooling |

## Platform Compatibility

**Full checklist**: → [extended/checklists.md#platform-compatibility-checklist]

## Related Resources

[OpenShift Docs](https://docs.openshift.com/) | [OKD Docs](https://docs.okd.io/) | [Container Image Guidelines for OpenShift](https://docs.openshift.com/container-platform/latest/openshift_images/create-images.html)

## Related Domains

[[kubernetes]] | [[docker]] | [[helm]] | [[aws]]
