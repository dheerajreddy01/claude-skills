# AWS CloudFormation — Extended Checklists

## Stack Safety Checklist

- [ ] Every change set reviewed before execution, with specific attention to `Replacement: True` entries on stateful resources
- [ ] `DeletionPolicy: Retain`/`Snapshot` set explicitly on every stateful resource where accidental deletion would be a real incident
- [ ] `CAPABILITY_IAM`/`CAPABILITY_NAMED_IAM` acknowledgment treated as a genuine review trigger — IAM resources/permissions inspected before approval
- [ ] Drift detection run periodically, not only reactively after a failed update
- [ ] Console-level write access to CloudFormation-managed resources restricted via IAM/SCP where feasible
- [ ] `ContinueUpdateRollback` procedure documented so a stuck `UPDATE_ROLLBACK_FAILED` stack can be recovered without improvising under pressure
- [ ] Circular dependencies avoided by restructuring mutual references rather than relying on `DependsOn` to force an impossible order

## Template Design Checklist

- [ ] Large infrastructure modularized via nested stacks or separate stacks per logical unit, not one monolithic template
- [ ] StackSets used for organization-wide baseline resources instead of manually replicating a stack per account/region
- [ ] `cfn-lint`/`cfn-nag`/`cfn-guard` run in CI to catch template issues and policy violations before deploy
- [ ] Parameters validated with `AllowedValues`/`AllowedPattern` where the input space should be constrained
- [ ] Outputs exported deliberately, with cross-stack references (`Fn::ImportValue`) reviewed for the coupling they create between stacks
