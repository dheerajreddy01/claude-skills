---
schema: "1.0"
name: argocd-flux
version: "1.0.0"
description: GitOps continuous delivery practices with ArgoCD and Flux — reconciliation, sync strategy, and secret handling
domain: technology
triggers:
  keywords:
    primary: [argocd, flux, gitops, continuous delivery]
    secondary: [application sync, kustomization, sealed secrets, sync wave]
  context_boost: [kubernetes deployment, declarative delivery]
  context_penalty: [jenkins, github actions, manual kubectl]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# ArgoCD / Flux (GitOps)

> Git is the source of truth, which means the cluster will always be pulled back to match it — including your emergency fix

## Applicable Scenarios

- Setting up ArgoCD Applications or Flux Kustomizations/HelmReleases
- Designing a GitOps repo structure and sync strategy
- Handling secrets in a model where the git repo shouldn't contain them in plaintext
- Diagnosing drift, sync failures, or unexpected resource deletion
- Sequencing dependent resource rollout with sync waves/health checks

## Core Knowledge

### The Reconciliation Loop

- Both tools continuously compare the **desired state** (what's declared in a git repo) against the **live state** (what's actually running in the cluster), and reconcile differences — this is the core GitOps principle: git is the single source of truth, not `kubectl apply` run by a human or a CI job
- Reconciliation is continuous, not one-time — any drift between git and the cluster gets corrected on the next sync cycle, whether that drift was intentional or not

### Core Objects

| Tool | Object | Purpose |
|------|------|------|
| **ArgoCD** | `Application` | Maps a git source (repo/path/revision) to a cluster destination (namespace/cluster) |
| **Flux** | `Kustomization` / `HelmRelease` | Similar mapping — a source (Git/Helm repo) reconciled into a target |

### Sync Behavior

- **Manual sync**: changes are staged and require explicit approval before applying — safer, but reintroduces a manual step GitOps is partly meant to remove
- **Automated sync**: changes apply automatically once detected in git — faster feedback loop, but means a bad commit to the tracked path deploys immediately
- **Prune**: when enabled, resources removed from the git-declared manifest set get deleted from the cluster automatically — this is correct GitOps behavior but a common source of "why did my resource just disappear" incidents when a manifest is accidentally removed or renamed
- **Sync waves** (ArgoCD) / **dependency ordering** (Flux) let you sequence resource application (e.g., a CRD before the resources that use it, or a database before the app that connects to it) — without this, resources apply in an order Kubernetes/the tool considers convenient, not necessarily what the application needs

### Secrets in a GitOps Model

- Git repos, even private ones, aren't an appropriate place for plaintext secrets — but application manifests often need to reference secret values
- Standard patterns: **Sealed Secrets** (encrypt secrets client-side, only the cluster's controller can decrypt), **SOPS** (encrypt specific values in a manifest, decrypted at apply time), or **External Secrets Operator** (manifest references an external secret store like Vault/AWS Secrets Manager, actual value never touches git)

## Best Practices

1. **Never commit plaintext secrets to a GitOps repo**, even a private one — use Sealed Secrets, SOPS, or an external secrets operator
2. **Understand prune behavior before enabling it** — know exactly what happens when a manifest is removed or renamed
3. **Use sync waves/dependency ordering** for resources with genuine startup/creation order requirements
4. **Make all changes through git**, never `kubectl edit`/`kubectl apply` directly against a GitOps-managed resource — it will be reverted on the next reconciliation
5. **Use manual sync (or automated sync with a required approval step) for production**, reserving fully automated sync for lower environments where fast iteration matters more than a review gate
6. **Monitor sync/health status as a first-class signal**, not just deployment success — a resource can be "synced" but unhealthy, or stuck in a perpetual out-of-sync state

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Committing a secret value directly into a manifest in the GitOps repo | Use Sealed Secrets, SOPS, or an External Secrets Operator reference instead |
| Making an emergency `kubectl` fix directly against a GitOps-managed resource | Fix it in git and let reconciliation apply it — a direct cluster edit gets silently reverted |
| Enabling prune without understanding its blast radius | Test prune behavior in a non-production environment and understand exactly what "removed from git" triggers |
| No sync wave/ordering for resources with real dependencies (CRD before CR, DB before app) | Configure explicit ordering for anything with a genuine startup dependency |
| Fully automated sync straight to production with no review gate | Use manual sync or an automated-sync-with-approval pattern for production environments |

## Sharp Edges

### SE-1: Plaintext Secrets Committed "Because It's Just Config"
- **Severity**: critical
- **Situation**: A Kubernetes `Secret` manifest (or a value later revealed to be sensitive) gets committed directly to the GitOps repository in plaintext, because it "looked like" ordinary configuration at the time
- **Cause**: GitOps repos hold all application configuration together, and it's easy to treat a Secret manifest the same as a ConfigMap without recognizing the git history implication — anyone with repo read access (and anyone with access to any historical clone) can now read the secret indefinitely
- **Symptoms**:
  - A secret scanner flags a credential in the GitOps repo's history
  - The "fix" of deleting the file in a later commit doesn't actually remove it from history
- **Solution**: Adopt Sealed Secrets, SOPS, or an External Secrets Operator from day one for any GitOps repo, treat any committed plaintext secret as compromised requiring immediate rotation, and use secret-scanning pre-commit hooks/CI checks to catch this before merge
- **Details**: → [extended/checklists.md#gitops-safety-checklist]

### SE-2: Automatic Prune Deleting Resources Unexpectedly
- **Severity**: critical
- **Situation**: A manifest gets renamed, refactored into a different file, or accidentally removed during a restructuring commit, and the GitOps controller — seeing it's no longer declared — deletes the corresponding live resource, even though the intent was never to remove it
- **Cause**: Prune-enabled reconciliation treats "no longer present in the declared state" as "should be deleted," with no distinction between "intentionally removed" and "accidentally missing due to a refactor mistake"
- **Symptoms**:
  - A resource (sometimes something stateful, like a PVC-backed workload) disappears from the cluster immediately after a manifest restructuring commit, with no explicit deletion intent
- **Solution**: Review prune-affecting changes (file renames, path restructuring) with extra scrutiny in PR review, use resource-level deletion protection/finalizers for genuinely critical stateful resources, and consider a non-pruning sync policy for the highest-risk resource types

### SE-3: Sync Wave/Health Check Misconfiguration Causing Out-of-Order Deployment
- **Severity**: high
- **Situation**: A dependent resource (an application Deployment) gets created before a genuine prerequisite (a CRD it relies on, or a database migration Job) has finished, causing the dependent resource to crash-loop or fail until the prerequisite catches up on its own
- **Cause**: Without explicit sync waves or dependency ordering, resources apply in whatever order the reconciler/Kubernetes API considers convenient, which doesn't account for application-level startup dependencies the tool has no visibility into
- **Symptoms**:
  - Deployments intermittently start in a broken state right after a sync, self-correcting only after a subsequent reconciliation
- **Solution**: Use ArgoCD sync waves (or Flux's `dependsOn`) to explicitly sequence resources with genuine ordering requirements, and pair with health checks so the reconciler actually waits for a prerequisite to become healthy before proceeding to dependents, not just "applied"

### SE-4: GitOps Reconciliation Fighting a Manual Emergency Fix
- **Severity**: high
- **Situation**: During an incident, an engineer makes a direct `kubectl` change to quickly mitigate an issue (e.g., scaling up a Deployment, patching a ConfigMap), and the GitOps controller reverts it on the next reconciliation cycle — sometimes within minutes — undoing the mitigation without warning
- **Cause**: The reconciliation loop has no awareness of "emergency" context — it simply sees the live state diverging from git and corrects it, exactly as designed, which is precisely the wrong behavior in this specific moment
- **Symptoms**:
  - An incident mitigation that appeared to work "un-fixes itself" a short time later
  - On-call engineers are surprised by GitOps behavior they didn't fully internalize under incident pressure
- **Solution**: Establish and rehearse an explicit emergency procedure — either pause reconciliation for the affected resource/Application before making a manual change, or (better) make the fix directly in git and let the controller apply it, accepting the small latency cost for consistency

### SE-5: Out-of-Band Helm/kubectl Changes Silently Reverted
- **Severity**: medium
- **Situation**: A team member unfamiliar with the GitOps setup runs `helm upgrade` or `kubectl apply` directly against a resource that's actually managed by ArgoCD/Flux, the change appears to work, and then silently disappears on the next reconciliation with no explanation to whoever made it
- **Cause**: Nothing about a raw `kubectl`/`helm` command warns the operator that the target resource is GitOps-managed and will be reverted — the tooling boundary is invisible unless the team has established clear conventions
- **Symptoms**:
  - Confused reports of "I made this change and it just disappeared"
  - Wasted debugging time investigating what looks like a Kubernetes bug rather than expected GitOps reconciliation
- **Solution**: Label/annotate GitOps-managed namespaces or resources clearly, document the GitOps boundary in onboarding, and use RBAC to restrict direct write access to GitOps-managed namespaces for anyone who isn't explicitly working through the git-based process

## Recommended Tools

| Category | Tools |
|------|------|
| Secret management | Sealed Secrets, SOPS, External Secrets Operator |
| Multi-cluster/app-of-apps | ArgoCD ApplicationSets, Flux multi-tenancy |
| Progressive delivery | Argo Rollouts, Flagger |
| Visualization | ArgoCD UI, Flux CLI (`flux get`) |

## GitOps Safety

**Full checklist**: → [extended/checklists.md#gitops-safety-checklist]

## Related Resources

[ArgoCD Docs](https://argo-cd.readthedocs.io/) | [Flux Docs](https://fluxcd.io/flux/) | [OpenGitOps Principles](https://opengitops.dev/)

## Related Domains

[[kubernetes]] | [[helm]] | [[terraform]] | [[vault]]
