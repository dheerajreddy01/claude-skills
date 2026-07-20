---
schema: "1.0"
name: terraform
version: "1.0.0"
description: Terraform state management, module design, and safe plan/apply practices
domain: technology
triggers:
  keywords:
    primary: [terraform, IaC, hcl, terraform plan, terraform apply]
    secondary: [state file, provider, module, workspace, remote backend]
  context_boost: [infrastructure as code, provisioning, cloud resources]
  context_penalty: [ansible, cloudformation, pulumi]
  priority: high
dependencies:
  software-skills: [git-workflows]
author: claude-domain-skills
---

# Terraform

> Declarative infrastructure where the state file is the single most dangerous file in the repo

## Applicable Scenarios

- Writing or reviewing Terraform configuration and modules
- Designing remote state storage, locking, and workspace strategy
- Reviewing a `terraform plan` before apply, especially for destructive changes
- Diagnosing state drift or corruption
- Structuring reusable modules across environments

## Core Knowledge

### The State File

- Terraform tracks real-world resource IDs and attributes in a **state file** (`terraform.tfstate`) — this is how it knows what exists and what a `plan` needs to change
- State can contain **sensitive values in plaintext** (e.g., a generated database password) even if the source `.tf` files don't
- Local state files are a liability for team use — no locking, no shared source of truth, and easy to lose. **Remote backends** (S3+DynamoDB, Terraform Cloud, GCS, Azure Storage) provide shared, locked state

### Plan / Apply Workflow

```
terraform init → terraform plan → (review) → terraform apply
```

- `plan` shows exactly what will change — added (`+`), changed (`~`), or destroyed (`-`) resources — **always read it**, especially any `-/+` (destroy and recreate) line
- Some attribute changes force resource **replacement** (destroy + recreate) rather than an in-place update — this is often surprising and can cause real downtime/data loss if the resource is stateful (e.g., a database)
- `terraform apply` without a saved plan file re-computes the plan at apply time — if the underlying infrastructure changed between `plan` and `apply`, the executed plan can differ from what was reviewed

### Modules & Providers

- **Modules** package reusable resource groups (e.g., "a standard VPC") — pin module *and* provider versions explicitly (`version = "~> 5.0"`), since an unpinned upstream module change applies silently on the next `init`
- **Providers** are the plugins that translate HCL into API calls to a specific platform (AWS, Azure, GCP, etc.) — provider version bumps can change default behaviors, not just add features

### State Locking & Concurrency

- Remote backends support **locking** (e.g., DynamoDB for S3 backend) so two people/pipelines can't run `apply` concurrently against the same state
- Without locking, concurrent applies can corrupt the state file or produce conflicting infrastructure changes that neither run's plan accounted for

## Best Practices

1. **Always use a remote backend with locking** for anything beyond solo experimentation
2. **Read every `plan` output before applying**, especially `-/+` (destroy-and-recreate) lines on stateful resources
3. **Pin provider and module versions explicitly** — don't let `init` silently pull a newer major version
4. **Never commit the state file** to version control — it can contain secrets and belongs in the remote backend only
5. **Use workspaces or separate state files per environment** (dev/staging/prod) — never share one state file across environments
6. **Run `plan` in CI on every PR**, apply only after review, ideally gated by a human approval for production

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Committing `terraform.tfstate` to git | Use a remote backend; add state files to `.gitignore` |
| Applying without reading the plan output | Always review `plan`, especially destroy/replace lines |
| Unpinned module/provider versions (`version = ">= 1.0"` with no upper bound) | Pin with a bounded constraint (`~> 5.2`) and upgrade deliberately |
| One state file shared across dev/staging/prod | Separate state per environment (workspaces or separate backend configs) |
| Manually editing resources in the cloud console after provisioning with Terraform | Make all changes through Terraform, or explicitly `terraform import`/reconcile drift |

## Sharp Edges

### SE-1: Destructive Replacement Applied Without Review
- **Severity**: critical
- **Situation**: A seemingly minor configuration change (e.g., changing an immutable attribute like an AWS RDS engine version in a way that forces replacement) causes Terraform to destroy and recreate a stateful resource, losing data
- **Cause**: Some resource attributes are immutable in the underlying provider API — changing them doesn't yield an in-place update, it forces a destroy-then-create, and `plan` output shows this as `-/+` but it's easy to miss in a large diff
- **Symptoms**:
  - A resource that "should have" been updated in place is destroyed and recreated, causing downtime or data loss
  - Post-incident review shows the `-/+` was present in the plan output but not caught before apply
- **Solution**: Read every `plan` line-by-line for `-/+` markers before applying, use `lifecycle { prevent_destroy = true }` on critical stateful resources, and for genuinely required replacements, plan an explicit migration (e.g., create-new-then-cutover) rather than letting Terraform destroy-then-create blindly
- **Details**: → [extended/checklists.md#apply-safety-checklist]

### SE-2: Secrets Stored in Plaintext in State
- **Severity**: critical
- **Situation**: A resource that generates or accepts a secret (e.g., a randomly generated DB password, an API key passed as a resource argument) ends up stored in plaintext in the state file, and anyone with state file access can read it
- **Cause**: Terraform state records the full resource attributes it's tracking, including sensitive ones, regardless of whether they're marked `sensitive` in the config (which only affects CLI output display, not state storage)
- **Symptoms**:
  - A state file, if exposed (misconfigured bucket permissions, leaked backup), reveals plaintext credentials
- **Solution**: Use a remote backend with encryption at rest and tightly scoped access control, avoid generating long-lived secrets via Terraform where possible (prefer a secrets manager reference), and treat state file access as equivalent to production credential access

### SE-3: Unlocked Concurrent Applies Corrupting State
- **Severity**: high
- **Situation**: Two engineers (or a human and a CI pipeline) run `terraform apply` against the same state at the same time, without locking, and the state file ends up corrupted or reflecting only one of the two changes
- **Cause**: Without a locking mechanism (DynamoDB table for S3 backend, native locking in Terraform Cloud, etc.), nothing prevents concurrent writes to the same state file
- **Symptoms**:
  - State file shows resources that don't match actual infrastructure
  - `terraform plan` shows unexpected changes that don't correspond to any recent `.tf` edit
- **Solution**: Always configure a backend with locking support, and never allow both manual local applies and CI-driven applies against the same state without clear ownership/coordination

### SE-4: Unpinned Module/Provider Versions Applying Silent Upstream Changes
- **Severity**: high
- **Situation**: `terraform init` pulls a newer version of a third-party module or provider with a loosely-bounded version constraint, and the next `plan`/`apply` reflects behavior changes nobody in the team actually reviewed or decided on
- **Cause**: An unbounded or overly permissive version constraint (`>= 1.0`, or no constraint at all) lets `init` resolve to whatever the latest matching version is at the time it runs
- **Symptoms**:
  - A `plan` shows unexpected changes with no corresponding `.tf` diff in the PR
  - Infrastructure behavior changes after a routine `init`/`apply` with no intentional config change
- **Solution**: Pin modules and providers to a bounded version constraint (e.g., `~> 5.2`, allowing patch/minor but not major bumps), review the changelog before deliberately bumping a major version, and commit a lockfile (`.terraform.lock.hcl`) for provider version reproducibility

### SE-5: Configuration Drift From Manual Out-of-Band Changes
- **Severity**: medium
- **Situation**: Someone modifies a Terraform-managed resource directly in the cloud console "just this once," and the next `plan`/`apply` either reverts the manual change unexpectedly or, worse, the drift causes a confusing partial-update plan
- **Cause**: Terraform assumes it's the sole source of truth for resources in its state; any change made outside Terraform isn't reflected in state until the next `refresh`, and then it's treated as drift to reconcile (usually by reverting to match config)
- **Symptoms**:
  - A manual "quick fix" made in the console mysteriously reverts on the next deploy
  - `plan` shows unexpected changes to a resource nobody edited in `.tf` recently
- **Solution**: Make all changes to Terraform-managed resources through Terraform; if an out-of-band change is unavoidable (emergency fix), update the `.tf` config to match afterward and reconcile deliberately, rather than letting the next `plan` silently revert it

## Recommended Tools

| Category | Tools |
|------|------|
| Linting/security | tflint, Checkov, tfsec |
| State management | Terraform Cloud, S3+DynamoDB backend |
| Drift detection | `terraform plan` in scheduled CI, driftctl |
| Testing | Terratest |

## Apply Safety

**Full checklist**: → [extended/checklists.md#apply-safety-checklist]

## Related Resources

[Terraform Docs](https://developer.hashicorp.com/terraform/docs) | [Terraform Best Practices](https://developer.hashicorp.com/terraform/language) | [Registry](https://registry.terraform.io/)

## Related Domains

[[aws]] | [[azure]] | [[gcp]] | [[github-actions]]
