---
schema: "1.0"
name: aws
version: "1.0.0"
description: AWS core services, IAM security, cost management, and Well-Architected Framework practices
domain: technology
triggers:
  keywords:
    primary: [aws, ec2, s3, lambda, iam, cloudformation]
    secondary: [vpc, rds, dynamodb, ecs, eks, cloudwatch, terraform]
  context_boost: [infrastructure, deployment, cloud architecture]
  context_penalty: [azure, gcp, on-premise]
  priority: high
dependencies:
  software-skills: [git-workflows, infrastructure-as-code]
author: claude-domain-skills
---

# AWS

> Least-privilege IAM, cost-aware architecture, and the Well-Architected Framework

## Applicable Scenarios

- Designing or reviewing AWS infrastructure and service selection
- Writing or auditing IAM policies for least privilege
- Diagnosing unexpected AWS cost spikes
- Choosing compute (EC2 vs Lambda vs ECS vs EKS) or storage/database services
- Applying the Well-Architected Framework to a design review

## Core Knowledge

### Core Service Categories

| Category | Services | Notes |
|------|------|------|
| **Compute** | EC2, Lambda, ECS, EKS, Fargate | Lambda for event-driven/short-lived; EC2/ECS/EKS for long-running or stateful workloads |
| **Storage** | S3, EBS, EFS | S3 = object storage; EBS = block storage attached to one EC2 instance; EFS = shared network file system |
| **Database** | RDS, DynamoDB, Aurora | RDS/Aurora = relational; DynamoDB = serverless key-value/document, scales horizontally |
| **Networking** | VPC, Route 53, CloudFront, ALB/NLB | VPC is the network boundary; everything else sits inside or fronts it |
| **IAM** | Users, Roles, Policies | The security perimeter for almost everything else in this list |

### IAM Model

- **Principle of least privilege**: grant only the actions/resources a role actually needs, scoped with resource ARNs — never `"Action": "*", "Resource": "*"` outside of break-glass admin roles
- **Roles over long-lived users**: EC2/Lambda/ECS should assume IAM roles, not embed access keys
- **Policy evaluation**: an explicit `Deny` always wins over an `Allow`, regardless of where it's attached (identity policy, resource policy, SCP, permissions boundary)
- **MFA** required on root and human IAM users at minimum; root account should not be used for day-to-day operations

### Compute Selection

| Need | Choice |
|------|------|
| Event-driven, short execution, no server management | **Lambda** |
| Containerized, need orchestration control | **ECS** (AWS-native) or **EKS** (Kubernetes) |
| Long-running, full OS control, specialized instance types (GPU, etc.) | **EC2** |
| Containerized, don't want to manage EC2 instances | **Fargate** (serverless ECS/EKS) |

### Well-Architected Framework Pillars

1. **Operational Excellence** — run and monitor systems, improve processes continuously
2. **Security** — protect data, systems, and assets (IAM, encryption, detective controls)
3. **Reliability** — recover from failures, scale to meet demand
4. **Performance Efficiency** — use resources efficiently as demand changes
5. **Cost Optimization** — avoid unnecessary spend, right-size resources
6. **Sustainability** — minimize environmental impact of workloads

## Best Practices

1. **Tag everything** — cost allocation tags on every resource make cost attribution and cleanup possible
2. **Use Infrastructure as Code** (CloudFormation, CDK, or Terraform) — no manual console changes to production
3. **Enable CloudTrail and GuardDuty** in every account for audit logging and threat detection
4. **Separate accounts by environment/team** (AWS Organizations + SCPs) rather than one shared account with tags
5. **Set budget alerts** before deploying anything that scales automatically (Lambda concurrency, Auto Scaling Groups)
6. **Encrypt by default** — S3 buckets, EBS volumes, RDS instances should default to encryption at rest

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Wildcard IAM policies (`"Action": "*"`) | Scope to specific actions and resource ARNs |
| Public S3 buckets by accident | Enable S3 Block Public Access at the account level by default |
| Hardcoding access keys in code/config | Use IAM roles (EC2 instance profiles, Lambda execution roles) |
| No budget alerts before enabling auto-scaling | Set AWS Budgets alerts before any auto-scaling deploy |
| Using the root account for daily operations | Create IAM users/roles with MFA; lock away root credentials |

## Sharp Edges

### SE-1: Public S3 Buckets Exposing Data
- **Severity**: critical
- **Situation**: An S3 bucket intended for internal use is discovered publicly readable/writable, exposing sensitive data or allowing tampering
- **Cause**: A bucket policy or ACL grants public access, often from a copy-pasted policy or a misunderstanding of "public" defaults in older bucket configurations
- **Symptoms**:
  - Security scanner or AWS Trusted Advisor flags a public bucket
  - Unexpected data appears in search engines or third-party scanners
- **Solution**: Enable "S3 Block Public Access" at the account level as a default; use bucket policies with explicit principal ARNs, never `Principal: "*"` unless the bucket is genuinely meant to be public (e.g., static website assets)
- **Details**: → [extended/checklists.md#security-hardening-checklist]

### SE-2: Overly Permissive IAM Policies
- **Severity**: critical
- **Situation**: A service role has `"Action": "*"` or `"Resource": "*"`, so a compromised credential or a bug in that service can affect the entire account
- **Cause**: Broad policies are faster to write and "just work," so they accumulate under time pressure and rarely get tightened afterward
- **Symptoms**:
  - IAM Access Analyzer flags unused permissions
  - A single compromised Lambda/EC2 role can access unrelated resources
- **Solution**: Scope every policy to the specific actions and resource ARNs actually used; use IAM Access Analyzer to identify and trim unused permissions; use permissions boundaries for delegated role creation

### SE-3: Runaway Costs From Unbounded Lambda/Auto Scaling
- **Severity**: high
- **Situation**: A recursive Lambda trigger, a misconfigured Auto Scaling Group, or an unthrottled API generates a massive, unexpected bill
- **Cause**: Serverless and auto-scaling services are designed to scale automatically with demand — without concurrency limits or budget alerts, a bug (e.g., an S3 event triggering the same Lambda that writes back to the same bucket) can scale exponentially
- **Symptoms**:
  - Sudden, sharp cost spike with no corresponding legitimate traffic increase
  - CloudWatch shows Lambda invocation count far exceeding expected load
- **Solution**: Set Lambda reserved/max concurrency limits, set AWS Budgets alerts with automated actions, review event-source-to-target loops for accidental recursion before deploying

### SE-4: Data Transfer Cost Surprises
- **Severity**: medium
- **Situation**: A bill spikes because of cross-AZ, cross-region, or internet egress data transfer that wasn't accounted for in the architecture
- **Cause**: AWS charges for data leaving a region/AZ and for internet egress, but not for ingress — architectures that fan data across AZs/regions or serve large payloads directly from origin accumulate this invisibly
- **Symptoms**:
  - Cost Explorer shows a large, unexplained "Data Transfer" line item
- **Solution**: Use a CDN (CloudFront) for egress-heavy content, co-locate chatty services in the same AZ where latency/cost matters, review Cost Explorer by usage type regularly

### SE-5: Root Account and Missing MFA
- **Severity**: critical
- **Situation**: The AWS root account (or privileged IAM users) has no MFA enabled, and a leaked password gives an attacker full account control
- **Cause**: Root account setup doesn't force MFA by default, and it's easy to skip during initial account creation
- **Symptoms**:
  - Security Hub / IAM credential report flags MFA-less privileged accounts
- **Solution**: Enable MFA on the root account immediately, lock away root credentials after initial setup, require MFA for all human IAM users via an SCP or IAM policy condition

## Recommended Tools

| Category | Tools |
|------|------|
| Infrastructure as Code | CloudFormation, AWS CDK, Terraform |
| Security auditing | IAM Access Analyzer, Security Hub, GuardDuty, Trusted Advisor |
| Cost management | Cost Explorer, AWS Budgets, Compute Optimizer |
| Observability | CloudWatch, X-Ray |

## Security & Cost Hardening

**Full checklist**: → [extended/checklists.md#security-hardening-checklist]

## Related Resources

[AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/) | [AWS Docs](https://docs.aws.amazon.com/) | [AWS Security Best Practices](https://aws.amazon.com/architecture/security-identity-compliance/)

## Related Domains

[[azure]] | [[gcp]] | [[python]] | [[go]]
