# Terraform — Extended Checklists

## Apply Safety Checklist

- [ ] Every `plan` output reviewed line-by-line before apply, with special attention to `-/+` (destroy-and-recreate) markers
- [ ] `lifecycle { prevent_destroy = true }` set on critical stateful resources (primary databases, etc.)
- [ ] Production applies gated by human approval in CI, not auto-applied on merge
- [ ] Remote backend configured with locking (S3+DynamoDB, Terraform Cloud, GCS with locking, etc.)
- [ ] State file never committed to version control; `.gitignore` covers `*.tfstate*`
- [ ] Separate state per environment (workspaces or separate backend configs) — no shared dev/staging/prod state
- [ ] `.terraform.lock.hcl` committed for reproducible provider versions
- [ ] Module and provider version constraints bounded (e.g., `~> 5.2`), not open-ended

## Security & Drift Checklist

- [ ] State backend encrypted at rest with tightly scoped access control (state can contain plaintext secrets)
- [ ] Long-lived secrets sourced from a secrets manager reference rather than generated/stored directly in Terraform where avoidable
- [ ] Scheduled `terraform plan` (drift detection) run in CI to catch out-of-band manual changes early
- [ ] tflint/Checkov/tfsec run in CI to catch misconfigurations before apply
- [ ] Any emergency manual out-of-band change reconciled back into `.tf` config promptly, not left to silently drift
