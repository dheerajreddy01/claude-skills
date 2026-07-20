---
schema: "1.0"
name: oracle-cloud
version: "1.0.0"
description: Oracle Cloud Infrastructure compartments, IAM policy language, and Autonomous Database practices
domain: technology
triggers:
  keywords:
    primary: [oracle cloud, OCI, autonomous database, compartment]
    secondary: [VCN, always free tier, security list, OCI policy]
  context_boost: [enterprise oracle workloads, database migration]
  context_penalty: [aws, azure, gcp]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Oracle Cloud (OCI)

> Compartments and OCI's plain-language policy syntax read differently from AWS/Azure/GCP — assuming equivalence is how over-broad grants happen

## Applicable Scenarios

- Designing compartment hierarchy for resource isolation and cost tracking
- Writing or reviewing OCI IAM policies
- Provisioning Autonomous Database and understanding its auto-scaling cost model
- Using Always Free tier resources without unexpected reclamation
- Configuring VCN networking (security lists vs. network security groups)

## Core Knowledge

### Compartments

- **Compartments** are OCI's logical resource-isolation boundary — every resource belongs to exactly one compartment, and compartments can be nested to form a hierarchy
- Unlike AWS accounts or Azure subscriptions (which are billing boundaries too), a compartment is purely an organizational/access-control boundary within a single OCI tenancy — billing is tenancy-wide, tracked per-compartment via cost reports, not separated the way multi-account AWS billing works
- IAM policies attached at a parent compartment are inherited by nested child compartments by default

### IAM Policy Language

- OCI policies are written in a **plain-language-like syntax** (e.g., `Allow group Developers to manage instances in compartment Dev`), which reads more like an English sentence than AWS's JSON policy documents or Azure's RBAC role definitions
- This readability is a double-edged sword — the plain-language syntax can make a policy's actual scope less obvious than it appears, especially around resource-type wildcards (`manage all-resources`) and compartment inheritance

### Autonomous Database

- A self-managing, self-tuning database service (auto-patching, auto-scaling, auto-backup) available as Autonomous Transaction Processing (ATP) or Autonomous Data Warehouse (ADW)
- Auto-scaling adjusts OCPU/storage allocation based on load automatically (if enabled) — this is billed accordingly, meaning cost can scale with usage in ways that need monitoring, similar to any auto-scaling compute resource

### Networking

| Concept | Role |
|------|------|
| **VCN (Virtual Cloud Network)** | The private network boundary, analogous to an AWS VPC/Azure VNet |
| **Security Lists** | Stateless/stateful traffic rules applied at the **subnet** level |
| **Network Security Groups (NSGs)** | Traffic rules applied at the **individual resource (VNIC)** level, independent of subnet |

Security Lists and NSGs can both apply to the same resource simultaneously — traffic must be allowed by *both* (where both are in play) for a security list and an NSG covering the same VNIC, which is a common source of confusion when only one of the two is updated during a troubleshooting session.

### Always Free Tier

- OCI's Always Free tier includes genuinely perpetual (not time-limited) free resources (small Autonomous Database instances, small compute shapes) — but idle/underutilized Always Free resources can be subject to **reclamation** (the resource gets stopped/reclaimed) if it looks inactive

## Best Practices

1. **Read OCI policies for their actual scope, not just their apparent readability** — verify what `manage all-resources` or a broad verb actually grants before applying it
2. **Design compartment hierarchy deliberately** before provisioning resources — retrofitting compartment structure onto an existing flat resource layout is far more work later
3. **Scope policies to specific resource-types and compartments** rather than defaulting to broad verbs (`manage`) and `all-resources`/tenancy-wide scope
4. **Monitor Autonomous Database auto-scaling activity** against actual workload patterns, don't assume auto-scaling is cost-neutral
5. **Configure both Security Lists and NSGs consistently** — check both when troubleshooting connectivity, don't assume updating one is sufficient
6. **Keep Always Free tier resources genuinely active** (or understand the reclamation policy) if relying on them for anything persistent

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| `Allow group X to manage all-resources in tenancy` used for convenience | Scope policies to specific resource-types and the narrowest compartment that satisfies the need |
| Assuming compartments provide billing isolation like separate AWS accounts | Understand compartments as an access/organization boundary within one tenancy's billing, not a billing-separation mechanism |
| Updating only a Security List (or only an NSG) while troubleshooting connectivity | Check and reconcile both — traffic must be allowed by both when both apply to a resource |
| Enabling Autonomous Database auto-scaling without monitoring | Track actual OCPU/storage scaling activity against cost, treat it like any other auto-scaling billing driver |
| Leaving Always Free resources idle for extended periods | Understand and plan around the reclamation policy for inactive Always Free resources |

## Sharp Edges

### SE-1: IAM Policy Syntax Misjudged as Narrower Than It Is
- **Severity**: critical
- **Situation**: A policy written in OCI's plain-language syntax (e.g., `Allow group Developers to manage instances in tenancy`) grants far broader access than the author intended, because the "readable" sentence structure obscured how broad `manage` and `in tenancy` actually are compared to a narrower intended scope
- **Cause**: The plain-language policy syntax's readability can create false confidence about scope — verbs like `manage` (full CRUD + broader capability) versus `use`/`read` are easy to reach for without considering the difference, and `in tenancy` applies tenancy-wide rather than to a specific compartment
- **Symptoms**:
  - A group has access to resources far outside what a specific task required
  - A policy review finds broad `manage all-resources`-style grants applied for narrow, specific needs
- **Solution**: Default to the narrowest verb (`inspect`/`read`/`use` before `manage`) and the narrowest compartment scope that satisfies the actual need, and review existing policies periodically for scope creep — treat the readable syntax as something to scrutinize carefully, not evidence of safety
- **Details**: → [extended/checklists.md#iam-and-network-checklist]

### SE-2: Always Free Tier Resource Reclamation Causing Data Loss
- **Severity**: high
- **Situation**: An Always Free tier Autonomous Database or compute instance that hasn't been actively used gets reclaimed, and any data on it that wasn't backed up elsewhere is lost
- **Cause**: OCI's Always Free tier reclaims resources that appear idle/underutilized for an extended period as a cost-control and resource-management mechanism for the free tier — this isn't the same as a paid resource, which isn't subject to the same reclamation policy
- **Symptoms**:
  - A previously-available Always Free resource is no longer accessible, and its data isn't recoverable
- **Solution**: Keep Always Free resources genuinely active if relying on them for anything beyond disposable experimentation, maintain backups of anything important stored on them independent of the platform, and understand this risk profile is fundamentally different from a paid, non-free resource before using Always Free tier for anything with real data dependencies

### SE-3: Compartment Hierarchy Misunderstanding Leading to Policy Inheritance Surprises
- **Severity**: high
- **Situation**: A policy attached at a parent compartment is assumed to apply only to resources directly in that compartment, but it also grants access to every nested child compartment beneath it — resources in a child compartment intended to be more restricted inherit broader parent-level access
- **Cause**: OCI policies attached at a compartment are inherited downward through the compartment hierarchy by default — a broad grant at a high level in the hierarchy silently applies to everything nested beneath it
- **Symptoms**:
  - A user/group has access to a child compartment's resources that nobody explicitly granted access to at that level
  - A security review of a specific compartment's policies misses the actual source of an access grant, because it's inherited from a parent
- **Solution**: Design the compartment hierarchy with inheritance in mind from the start — put genuinely broad, org-wide grants only at levels where that inheritance is intended, and review the full inherited policy chain (not just policies directly attached to a compartment) when auditing access for a specific compartment

### SE-4: Autonomous Database Auto-Scaling Cost Surprises
- **Severity**: medium
- **Situation**: Auto-scaling on an Autonomous Database instance responds to a sustained load increase (legitimate traffic growth, or an inefficient query pattern) by scaling OCPU/storage allocation up, and the resulting bill is significantly higher than expected because nobody was monitoring the scaling activity
- **Cause**: Auto-scaling is billed based on actual allocated resources at any given time — it's designed to handle variable load automatically, but that also means cost varies with load rather than staying at a fixed, predictable baseline unless capped or monitored
- **Symptoms**:
  - A billing period shows a cost spike correlated with a sustained period of higher database load
- **Solution**: Monitor auto-scaling activity and correlate it with actual application load patterns, set alerts on cost/usage thresholds, and investigate whether load increases are legitimate traffic growth or an inefficient query pattern that should be fixed rather than scaled around

### SE-5: Security List / NSG Overlap Confusion
- **Severity**: medium
- **Situation**: Network connectivity to a resource is blocked (or unexpectedly allowed) because Security Lists and Network Security Groups, both applying to the same VNIC, have inconsistent rules, and troubleshooting only checked one of the two mechanisms
- **Cause**: OCI supports both subnet-level Security Lists and resource-level NSGs simultaneously on the same network interface — traffic must satisfy the rules of both where both are configured, and engineers used to a single-mechanism model (like a simpler security-group-only setup) often only check one during troubleshooting
- **Symptoms**:
  - A firewall rule is added to fix connectivity, but the issue persists because the other mechanism (Security List or NSG) still blocks it
  - Rules that "look correct" in one place don't reflect actual observed traffic behavior
- **Solution**: When troubleshooting connectivity, check and reconcile both Security Lists (subnet-level) and NSGs (resource-level) rather than assuming only one is in play, and prefer standardizing on one mechanism (typically NSGs, for more granular per-resource control) where the added complexity of both isn't specifically needed

## Recommended Tools

| Category | Tools |
|------|------|
| Infrastructure as Code | Terraform OCI provider, OCI Resource Manager |
| CLI/SDKs | OCI CLI, language-specific OCI SDKs |
| Monitoring | OCI Monitoring, Cost Analysis |
| Policy management | IAM Policy Language reference, Policy Simulator |

## IAM & Network Safety

**Full checklist**: → [extended/checklists.md#iam-and-network-checklist]

## Related Resources

[OCI Documentation](https://docs.oracle.com/en-us/iaas/Content/home.htm) | [OCI IAM Policy Reference](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/policyreference.htm) | [Always Free Resources](https://docs.oracle.com/en-us/iaas/Content/FreeTier/freetier_topic-Always_Free_Resources.htm)

## Related Domains

[[aws]] | [[azure]] | [[gcp]] | [[terraform]]
