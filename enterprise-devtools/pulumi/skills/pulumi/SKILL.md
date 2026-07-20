---
schema: "1.0"
name: pulumi
version: "1.0.0"
description: Pulumi infrastructure-as-code using general-purpose languages, state management, and stack reference practices
domain: technology
triggers:
  keywords:
    primary: [pulumi, infrastructure as code, pulumi stack]
    secondary: [stack reference, pulumi state, resource provider, pulumi secret]
  context_boost: [cloud provisioning, IaC, typescript infrastructure]
  context_penalty: [terraform, cloudformation, ansible]
  priority: high
dependencies:
  software-skills: [git-workflows]
author: claude-domain-skills
---

# Pulumi

> Real programming languages for infrastructure means real language footguns apply to infrastructure too

## Applicable Scenarios

- Writing or reviewing Pulumi programs (TypeScript, Python, Go, etc.) for cloud provisioning
- Designing state backend and stack organization
- Using stack references to compose infrastructure across multiple stacks
- Handling secret outputs correctly
- Diagnosing non-deterministic preview/update behavior

## Core Knowledge

### General-Purpose Language IaC

- Unlike Terraform's HCL or CloudFormation's YAML/JSON, Pulumi programs are written in an actual general-purpose language (TypeScript, Python, Go, C#, Java) — real loops, conditionals, functions, and package ecosystems are available for expressing infrastructure
- This is a genuine capability advantage for complex, parameterized infrastructure, but it also means the language's full expressiveness — including things that don't map cleanly to a declarative model — is available to be misused

### State Management

- Like Terraform, Pulumi tracks resource state to compute diffs between desired and actual infrastructure — stored in a **backend**: Pulumi Cloud (managed, default), or a self-managed backend (S3, Azure Blob, GCS, local file)
- State access control and encryption matter exactly as much as with Terraform — state can contain sensitive resource attributes and is the definitive record of what infrastructure exists

### Stack References

- A **stack** is an isolated instance of a Pulumi program (e.g., per environment: dev/staging/prod) — each has its own state and configuration
- **Stack references** let one stack read outputs from another (e.g., a networking stack's VPC ID consumed by an application stack) — this creates a dependency where the consuming stack picks up whatever the referenced stack's *current* output is at the time it runs, which may not be the version the consuming stack's author was expecting

### Secret Handling

- Values marked via `pulumi.secret()` are encrypted in state and masked in CLI output/logs — but a value derived from a secret (e.g., interpolated into a plain string, or a resource output that doesn't propagate the secret-ness) can lose that protection unless handled carefully
- Provider-level secrets (cloud credentials for Pulumi itself to authenticate) are a separate concern from secrets managed *by* Pulumi *for* the infrastructure it provisions — don't conflate the two

## Best Practices

1. **Use `pulumi.secret()` explicitly for any sensitive output**, and verify secret-ness propagates through any transformation applied to it
2. **Use a state backend with proper access control and encryption**, understanding it holds the same class of sensitive data a Terraform state file would
3. **Treat stack references as a real dependency** — understand what happens when the upstream stack changes, and pin/version where that matters
4. **Avoid nondeterministic constructs** (real randomness, wall-clock-dependent branching, unseeded IDs) in resource definition logic — infrastructure code needs deterministic preview/diff behavior
5. **Enable resource protection** (`protect: true`) on critical stateful resources to prevent accidental deletion via `pulumi destroy` or a removed resource definition
6. **Review `pulumi preview` output carefully before `pulumi up`**, exactly as you would a Terraform plan — the language flexibility doesn't reduce the need for this discipline

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Interpolating a secret output into a plain string without preserving its secret-ness | Use Pulumi's secret-aware output composition so the result stays marked as a secret |
| Using real language randomness/branching that changes between preview and up | Keep resource definition logic deterministic; push genuine runtime randomness into resource properties Pulumi/the provider controls, not your own program logic |
| No resource protection on stateful resources (databases, storage) | Set `protect: true` on anything where accidental deletion would be a real incident |
| Treating a stack reference as a one-time value rather than a live dependency | Understand that a stack reference reads the referenced stack's current output at run time, and version/pin accordingly if drift matters |
| Skipping `pulumi preview` review before `pulumi up` | Review preview output for unexpected replacements/deletions before every apply, same discipline as reviewing a Terraform plan |

## Sharp Edges

### SE-1: State Backend Misconfiguration Exposing Sensitive Infrastructure Data
- **Severity**: critical
- **Situation**: A self-managed state backend (e.g., an S3 bucket) is misconfigured with overly broad access, and anyone with access can read the full state — including any resource attributes that weren't explicitly marked as secrets but are still sensitive (internal IPs, resource IDs, configuration details)
- **Cause**: Pulumi state, like Terraform state, is a comprehensive record of provisioned infrastructure — teams less familiar with this risk (since Pulumi's "just code" framing can obscure the state file's importance) may not apply the same access rigor they'd apply to a Terraform backend
- **Symptoms**:
  - A security review finds a state backend with broader read access than intended
- **Solution**: Apply the same access control and encryption-at-rest discipline to Pulumi's state backend as you would to any other infrastructure state store, and default to Pulumi Cloud's managed backend (which handles this) unless there's a specific reason to self-manage
- **Details**: → [extended/checklists.md#state-and-secrets-checklist]

### SE-2: Secrets Losing Their Secret-Ness Through Transformation
- **Severity**: high
- **Situation**: A value correctly wrapped with `pulumi.secret()` gets interpolated into a plain string or combined with non-secret data in a way that doesn't preserve the secret marking, and the resulting value shows up unmasked in CLI output, logs, or state
- **Cause**: Secret-ness in Pulumi propagates through its output composition APIs (`.apply()`, secret-aware interpolation) — but a careless plain-JavaScript/Python string operation on the underlying value can produce a new value that Pulumi no longer recognizes as sensitive
- **Symptoms**:
  - A value that should be masked appears in plaintext in `pulumi up` output or CI logs
- **Solution**: Use Pulumi's secret-aware composition functions consistently when building any value derived from a secret, and audit CLI/CI output for accidental secret exposure, treating any leak as requiring credential rotation

### SE-3: Nondeterministic Program Logic Producing Unreliable Previews
- **Severity**: medium
- **Situation**: A Pulumi program uses real language constructs (unseeded random values, branching on wall-clock time, non-deterministic iteration order over an unordered collection) in a way that makes `pulumi preview` show different results from what actually happens on `pulumi up`, or shows different diffs on repeated runs with no underlying infrastructure change
- **Cause**: Because Pulumi programs are genuine imperative code, nothing prevents writing infrastructure logic that isn't deterministic — but deterministic preview/diff behavior is exactly the property that makes reviewing a plan meaningful
- **Symptoms**:
  - `pulumi preview` output differs between consecutive runs with no actual config or infrastructure change
  - A reviewed preview doesn't match what `pulumi up` actually does
- **Solution**: Keep resource-defining logic deterministic — push genuine randomness into resource properties designed for it (e.g., a provider-generated unique suffix) rather than program-level `Math.random()`, and treat any preview/apply mismatch as a bug in the program's determinism, not a Pulumi quirk to work around

### SE-4: Stack Reference Version Drift Applying Unexpected Upstream Changes
- **Severity**: medium
- **Situation**: A dependent stack (e.g., an application stack referencing a networking stack's outputs) picks up a changed value from the upstream stack the next time it runs — even though nobody touched the dependent stack's own code — because stack references always resolve to the *current* state of the referenced stack
- **Cause**: A stack reference isn't a pinned dependency in the way a package version constraint is — it's a live read of whatever the upstream stack's output currently is at the moment the dependent stack runs
- **Symptoms**:
  - A stack's `pulumi preview` shows unexpected changes with no corresponding edit to that stack's own program
  - Investigation traces the change back to an unrelated upstream stack's most recent deploy
- **Solution**: Understand that stack references are live, not pinned, and for infrastructure where drift-on-upstream-change is undesirable, either version the upstream output explicitly or coordinate deploys of dependent stacks deliberately rather than assuming isolation

### SE-5: Missing Resource Protection Allowing Accidental Deletion
- **Severity**: high
- **Situation**: A critical stateful resource (a production database, a storage bucket with irreplaceable data) gets deleted because a `pulumi destroy` was run against the wrong stack, or a refactor accidentally removed the resource's definition from the program, and Pulumi dutifully deletes what's no longer declared
- **Cause**: Without `protect: true` set explicitly, Pulumi treats resource deletion (whether from `destroy` or a removed definition) the same as any other diff to reconcile — there's no default extra caution for stateful resources
- **Symptoms**:
  - A resource is deleted as an unintended side effect of an unrelated change or a `destroy` run against the wrong target
- **Solution**: Set `protect: true` on any resource where accidental deletion would be a real incident, requiring an explicit unprotect step before deletion is even possible, and double-check the target stack before ever running `pulumi destroy`

## Recommended Tools

| Category | Tools |
|------|------|
| State backend | Pulumi Cloud, self-managed S3/Azure Blob/GCS backend |
| Testing | Pulumi unit testing framework (mocked resource providers), policy-as-code (CrossGuard) |
| CI integration | Pulumi GitHub Actions / GitLab CI templates |
| Secrets | Pulumi ESC (Environments, Secrets, and Configuration) |

## State & Secrets Safety

**Full checklist**: → [extended/checklists.md#state-and-secrets-checklist]

## Related Resources

[Pulumi Docs](https://www.pulumi.com/docs/) | [Pulumi Best Practices](https://www.pulumi.com/docs/using-pulumi/best-practices/) | [Pulumi Registry](https://www.pulumi.com/registry/)

## Related Domains

[[terraform]] | [[aws]] | [[azure]] | [[gcp]]
