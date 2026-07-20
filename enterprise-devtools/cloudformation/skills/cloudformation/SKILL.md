---
schema: "1.0"
name: cloudformation
version: "1.0.0"
description: AWS CloudFormation template design, stack update safety, deletion policy, and drift management practices
domain: technology
triggers:
  keywords:
    primary: [cloudformation, cfn, cloudformation stack, cloudformation template]
    secondary: [change set, nested stack, stackset, drift detection]
  context_boost: [aws infrastructure, iac, aws-native provisioning]
  context_penalty: [terraform, pulumi, ansible]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# AWS CloudFormation

> The AWS-native IaC tool where a stuck rollback and a default deletion policy are the two things that bite everyone eventually

## Applicable Scenarios

- Writing or reviewing CloudFormation templates (Resources/Parameters/Outputs/Conditions)
- Reviewing change sets before applying a stack update
- Recovering a stack stuck in a failed update/rollback state
- Structuring nested stacks or StackSets for multi-account/multi-region deployment
- Reconciling drift between a stack's template and actual resource state

## Core Knowledge

### Template Structure

- A template's core sections: **Parameters** (inputs), **Resources** (what gets created — the only required section), **Outputs** (values exported for use elsewhere), **Conditions** (conditional resource creation/configuration), **Mappings** (static lookup tables)
- CloudFormation determines resource creation/update/deletion order automatically based on `Ref`/`Fn::GetAtt` dependencies between resources — implicit dependency inference, not something you declare directly (unless you need `DependsOn` for a dependency CloudFormation can't infer)

### Stack Updates & Change Sets

- A **change set** previews exactly what a template update will do (create/modify/replace/delete) before it's actually executed — the CloudFormation equivalent of `terraform plan`
- Some attribute changes force **replacement** (the resource is deleted and recreated) rather than an in-place update — shown in a change set as `Replacement: True`, easy to miss in a large change set if not reviewed carefully
- If a stack update fails partway through, CloudFormation attempts an automatic **rollback** to the last known-good state — but the rollback itself can fail, leaving the stack in `UPDATE_ROLLBACK_FAILED`, which blocks further updates until manually resolved

### Deletion Policy

- By default, deleting a resource from a template (or deleting the whole stack) **deletes the underlying AWS resource** — including stateful resources like RDS databases or S3 buckets, unless explicitly told otherwise
- `DeletionPolicy: Retain` keeps the resource even if it's removed from the stack; `DeletionPolicy: Snapshot` (for supported resource types like RDS/EBS) takes a final snapshot before deletion — neither is the default

### Nested Stacks & StackSets

| Mechanism | Purpose |
|------|------|
| **Nested stacks** | A stack's resource can itself be another CloudFormation stack — for modularizing large templates |
| **StackSets** | Deploy the same stack across multiple accounts/regions from one operation — for organization-wide baseline resources |

### Drift Detection

- CloudFormation can detect **drift** — when a resource's actual configuration no longer matches what the template declares (typically from a manual console change) — but this is an on-demand check, not continuous monitoring, and drifted resources aren't automatically corrected

## Best Practices

1. **Always review a change set before executing it**, paying specific attention to any `Replacement: True` entries on stateful resources
2. **Set `DeletionPolicy: Retain` (or `Snapshot` where supported) explicitly** on any resource where accidental deletion would be a real incident
3. **Understand `CAPABILITY_IAM`/`CAPABILITY_NAMED_IAM` acknowledgments** before approving them — review exactly what IAM resources/permissions a template creates
4. **Run drift detection periodically** on stacks where manual console access is possible, not just when something already seems wrong
5. **Use nested stacks or modular templates** for large infrastructure, rather than one enormous monolithic template that's hard to review and slow to update
6. **Use StackSets for organization-wide baseline resources** (logging, security guardrails) rather than manually replicating a stack per account

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Applying a stack update without reviewing the change set | Always review the change set, especially for `Replacement: True` on stateful resources |
| Leaving `DeletionPolicy` at its default for critical stateful resources | Set `Retain` or `Snapshot` explicitly on anything where deletion would be a real incident |
| Blindly acknowledging `CAPABILITY_IAM` without reviewing what permissions the template grants | Review the actual IAM resources/policies in the template before acknowledging |
| Manually changing a CloudFormation-managed resource in the console | Make changes through the template and a stack update; run drift detection if a manual change is suspected |
| One enormous monolithic template for all infrastructure | Modularize with nested stacks or separate stacks per logical unit |

## Sharp Edges

### SE-1: Stack Stuck in `UPDATE_ROLLBACK_FAILED`
- **Severity**: critical
- **Situation**: A stack update fails, CloudFormation's automatic rollback also fails (often because the resource being rolled back to is itself now in an inconsistent state), and the stack becomes stuck in `UPDATE_ROLLBACK_FAILED` — blocking any further updates until manually resolved
- **Cause**: Rollback isn't guaranteed to succeed — it's itself a set of AWS API calls that can fail for the same reasons the original update could (permissions, resource conflicts, service limits), and CloudFormation doesn't have a fully automated recovery path for a failed rollback
- **Symptoms**:
  - Stack status shows `UPDATE_ROLLBACK_FAILED`, and any new update attempt is rejected
  - Deploys for that stack are completely blocked until manual intervention
- **Solution**: Use `ContinueUpdateRollback` (optionally skipping specific resources that are the actual cause of the rollback failure) to manually push the rollback through, then investigate and fix the root cause before attempting the original update again — and design templates/change processes to minimize the chance of a rollback-failure scenario (e.g., avoiding manual console changes that leave resources CloudFormation doesn't expect)
- **Details**: → [extended/checklists.md#stack-safety-checklist]

### SE-2: Default Deletion Policy Causing Accidental Data Loss
- **Severity**: critical
- **Situation**: A resource is removed from a template (intentionally or by mistake) or the whole stack is deleted, and a stateful resource like an RDS database or S3 bucket is deleted along with it — including its data — because `DeletionPolicy` was left at its default
- **Cause**: CloudFormation's default behavior treats infrastructure as fully declarative and disposable — a resource no longer in the template (or a deleted stack) is deleted from AWS too, with no built-in extra caution for resources holding irreplaceable data
- **Symptoms**:
  - A database or bucket disappears (with its data) as a side effect of a template edit or stack deletion that wasn't meant to touch it
- **Solution**: Set `DeletionPolicy: Retain` or `Snapshot` explicitly on every stateful resource where this matters, as a standing template-review requirement, not something decided ad hoc per resource

### SE-3: Circular Dependency Errors From Implicit Resource Ordering
- **Severity**: medium
- **Situation**: Two resources reference each other (directly or through a chain), and CloudFormation can't determine a valid creation order, failing the deployment with a circular dependency error
- **Cause**: CloudFormation infers dependency order automatically from `Ref`/`Fn::GetAtt` usage between resources — if resource A needs an attribute of resource B, and B (directly or transitively) needs an attribute of A, there's no valid order to create them in
- **Symptoms**:
  - Template validation or stack creation fails with a circular dependency error, sometimes not obviously pointing at the actual resource pair involved
- **Solution**: Restructure the template to break the cycle — often by decoupling the mutual reference (e.g., creating one resource first without the reference, then using a separate resource like a policy attachment to establish the connection afterward), and use `DependsOn` sparingly and only for genuine ordering needs CloudFormation can't infer

### SE-4: Drift From Manual Console Changes Undetected Until the Next Update Fails
- **Severity**: high
- **Situation**: Someone makes a manual console change to a CloudFormation-managed resource (an emergency fix, or simply not realizing it's managed), and the drift goes unnoticed until the next legitimate stack update fails or behaves unexpectedly because CloudFormation's assumed prior state doesn't match reality
- **Cause**: CloudFormation doesn't continuously monitor for drift — it only checks when drift detection is explicitly run, so manual changes can persist invisibly for a long time
- **Symptoms**:
  - A routine stack update fails with an error referencing a resource property that doesn't match what the template expected
  - Running drift detection after the fact reveals a manual change made weeks or months earlier
- **Solution**: Run drift detection periodically (not just reactively), restrict console-level write access to CloudFormation-managed resources via IAM/SCP where feasible, and treat any necessary emergency manual change as something that must be reconciled back into the template promptly

### SE-5: Blind `CAPABILITY_IAM` Acknowledgment Granting Unreviewed Permissions
- **Severity**: high
- **Situation**: A stack deployment requires acknowledging `CAPABILITY_IAM`/`CAPABILITY_NAMED_IAM` because the template creates IAM resources, and the acknowledgment is given reflexively (it's often just a required checkbox/flag to get the deploy to proceed) without actually reviewing what permissions the template is granting
- **Cause**: CloudFormation requires this explicit acknowledgment specifically because IAM resource creation is security-sensitive — but in practice it's frequently treated as pipeline boilerplate rather than a genuine review gate
- **Symptoms**:
  - A template creates an overly permissive IAM role/policy that goes unnoticed because the required acknowledgment step didn't prompt an actual review
- **Solution**: Treat `CAPABILITY_IAM` acknowledgment as a real review trigger — inspect exactly what IAM resources and permissions a template creates or modifies before approving, especially in automated CI/CD pipelines where this step can otherwise become a rubber stamp

## Recommended Tools

| Category | Tools |
|------|------|
| Linting/validation | cfn-lint, cfn-nag, cfn-guard |
| Higher-level authoring | AWS CDK (compiles to CloudFormation) |
| Drift/compliance | AWS Config, drift detection API |
| Multi-account | AWS Organizations + StackSets |

## Stack Safety

**Full checklist**: → [extended/checklists.md#stack-safety-checklist]

## Related Resources

[CloudFormation Docs](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html) | [CloudFormation Best Practices](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/best-practices.html) | [AWS CDK Docs](https://docs.aws.amazon.com/cdk/)

## Related Domains

[[aws]] | [[terraform]] | [[pulumi]]
