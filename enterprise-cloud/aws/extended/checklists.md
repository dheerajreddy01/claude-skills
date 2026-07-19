# AWS — Extended Checklists

## Security Hardening Checklist

- [ ] S3 Block Public Access enabled at the account level; per-bucket exceptions explicitly justified
- [ ] No IAM policy uses `"Action": "*"` or `"Resource": "*"` outside a documented break-glass admin role
- [ ] Root account has MFA enabled and is not used for day-to-day operations
- [ ] All human IAM users have MFA enforced via policy condition
- [ ] IAM Access Analyzer reviewed for unused permissions on service roles
- [ ] CloudTrail enabled in all regions, logs shipped to a dedicated, access-restricted account
- [ ] GuardDuty enabled for threat detection
- [ ] Encryption at rest enabled by default for S3, EBS, RDS
- [ ] Security groups reviewed for `0.0.0.0/0` ingress on anything other than public load balancers (ports 80/443)
- [ ] Secrets stored in Secrets Manager/Parameter Store, never in environment variables checked into IaC

## Cost Control Checklist

- [ ] AWS Budgets alerts configured before deploying anything with auto-scaling or serverless concurrency
- [ ] Lambda reserved/max concurrency set explicitly for functions with external triggers (S3, SNS, API Gateway)
- [ ] Cost allocation tags applied consistently (environment, team, service) for attribution
- [ ] Cost Explorer reviewed by usage type monthly to catch data-transfer or unused-resource surprises
- [ ] Unused EBS volumes, unattached Elastic IPs, and idle load balancers cleaned up (Trusted Advisor / Compute Optimizer)
- [ ] Reserved Instances/Savings Plans evaluated for steady-state workloads
- [ ] Event-source-to-target chains reviewed for accidental recursive triggers (e.g., S3 → Lambda → same S3 bucket)
