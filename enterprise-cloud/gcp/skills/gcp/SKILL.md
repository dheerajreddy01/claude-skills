---
schema: "1.0"
name: gcp
version: "1.0.0"
description: Google Cloud core services, IAM security, cost management, and Well-Architected Framework practices
domain: technology
triggers:
  keywords:
    primary: [gcp, google cloud, cloud run, bigquery, gke]
    secondary: [cloud functions, firestore, pubsub, service account, cloud storage, vpc]
  context_boost: [infrastructure, deployment, cloud architecture]
  context_penalty: [aws, azure, on-premise]
  priority: high
dependencies:
  software-skills: [git-workflows, infrastructure-as-code]
author: claude-domain-skills
---

# Google Cloud (GCP)

> Project-scoped IAM, serverless-first compute, and BigQuery cost discipline

## Applicable Scenarios

- Designing or reviewing GCP infrastructure and service selection
- Writing or auditing IAM bindings and service account usage
- Diagnosing unexpected GCP cost spikes (especially BigQuery)
- Choosing compute (Cloud Run vs Cloud Functions vs GKE vs Compute Engine)
- Applying the Google Cloud Architecture Framework to a design review

## Core Knowledge

### Core Service Categories

| Category | Services | Notes |
|------|------|------|
| **Compute** | Compute Engine, Cloud Run, Cloud Functions, GKE | Cloud Run for containerized serverless; Cloud Functions for event-driven; GKE for full Kubernetes control |
| **Storage** | Cloud Storage, Persistent Disk, Filestore | Cloud Storage = object storage (multi-regional/regional classes); Filestore = managed NFS |
| **Database** | Cloud SQL, Firestore, BigQuery, Spanner | BigQuery = serverless data warehouse, billed by data scanned; Spanner = globally consistent relational at scale |
| **Networking** | VPC, Cloud Load Balancing, Cloud CDN | VPCs are global (not regional) in GCP, unlike AWS/Azure |
| **Identity** | IAM, Service Accounts, Workload Identity | Resource hierarchy: Organization → Folder → Project → Resource |

### Resource Hierarchy

```
Organization → Folder → Project → Resource
```

- **Projects** are the primary billing and isolation boundary (equivalent to an AWS account or Azure subscription)
- IAM policies can be set at any level and are inherited downward — a broad grant at the Organization/Folder level applies to everything below it
- Every resource's effective permissions are the union of bindings at every level above it — there's no way to "deny" a permission granted higher up except via an IAM Deny policy or org policy constraint

### IAM Model

- **Principle of least privilege**: use predefined roles scoped to a service (e.g., `roles/storage.objectViewer`) rather than broad primitive roles (`Owner`, `Editor`, `Viewer`)
- **Service Accounts** are identities for workloads; avoid downloading and distributing service account **keys** — prefer **Workload Identity** (GKE) or attached service accounts (Compute Engine, Cloud Run) so credentials never leave Google's infrastructure
- **IAM Conditions** allow time-bound or attribute-based conditional grants for finer control than role bindings alone

### Compute Selection

| Need | Choice |
|------|------|
| Containerized, request-driven, scale-to-zero | **Cloud Run** |
| Event-driven, single-purpose functions | **Cloud Functions** |
| Full Kubernetes control | **GKE** |
| Full VM control, specialized workloads | **Compute Engine** |

### Google Cloud Architecture Framework Pillars

1. **Operational Excellence** — automate operations, plan for incidents
2. **Security, Privacy, Compliance** — least privilege, defense in depth
3. **Reliability** — design for failure, define SLOs
4. **Cost Optimization** — align spend to business value
5. **Performance Optimization** — scale efficiently with demand

## Best Practices

1. **Avoid service account keys** — use Workload Identity Federation or attached identities wherever possible
2. **Use predefined or custom IAM roles**, never primitive `Owner`/`Editor` for service identities
3. **Use Infrastructure as Code** (Terraform is the de facto standard for GCP) — avoid manual console changes to production
4. **Set BigQuery cost controls** — query cost estimates, custom quotas, and `maximum_bytes_billed` before running exploratory queries on large tables
5. **Structure projects by environment/team**, governed by Folder-level IAM and org policies
6. **Enable Security Command Center** for org-wide security posture visibility

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Downloading and distributing service account key files | Use Workload Identity Federation or attached service accounts |
| Granting `roles/editor` to a service account "to make it work" | Use narrowly scoped predefined roles for the specific service |
| Running `SELECT *` on a large BigQuery table without checking scanned bytes | Use the query validator/dry-run to estimate cost before running |
| Leaving Cloud Storage buckets with public/allUsers access | Use uniform bucket-level access and grant specific principals only |
| Setting IAM at the Organization level for a single-project need | Scope grants to the Project or Folder that actually needs them |

## Sharp Edges

### SE-1: Service Account Key Sprawl
- **Severity**: critical
- **Situation**: A downloaded service account JSON key is committed to a repo, embedded in a container image, or shared over chat, and later used by an attacker to access GCP resources
- **Cause**: Service account keys are long-lived, static credentials that work from anywhere until explicitly revoked — unlike Workload Identity, which never exposes a portable secret
- **Symptoms**:
  - Security scanners (e.g., in CI or GitHub secret scanning) flag a committed key file
  - Unexpected API activity from a service account with no corresponding legitimate workload
- **Solution**: Prefer Workload Identity Federation (external workloads) or attached service accounts (GCE/Cloud Run/GKE) so no portable key ever exists; if a key is unavoidable, set an organization policy restricting key creation and rotate/audit aggressively
- **Details**: → [extended/checklists.md#security-hardening-checklist]

### SE-2: BigQuery Cost From Full Table Scans
- **Severity**: high
- **Situation**: An exploratory `SELECT *` (or a query without a partition/cluster filter) scans an entire multi-terabyte table, and the on-demand billing model charges per byte scanned
- **Cause**: BigQuery on-demand pricing bills by bytes scanned, not query runtime — a "quick check" query on an unpartitioned or unfiltered table can scan the whole table
- **Symptoms**:
  - A single ad-hoc query shows up as a disproportionately large cost line
  - Billing spikes correlate with analyst/dashboard query activity, not data ingestion
- **Solution**: Partition and cluster large tables, select only needed columns, use the query validator (dry run) to check bytes-scanned before running, set `maximum_bytes_billed` on interactive queries, and consider flat-rate/capacity pricing for predictable heavy usage

### SE-3: Public Cloud Storage Buckets
- **Severity**: critical
- **Situation**: A Cloud Storage bucket intended for internal data is discovered publicly accessible via `allUsers` or `allAuthenticatedUsers` IAM bindings
- **Cause**: `allUsers`/`allAuthenticatedUsers` are valid IAM principals in GCP (not a separate "public access" toggle), so a bucket can be made public simply by an IAM binding — easy to add without realizing the scope
- **Symptoms**:
  - Security Command Center flags a bucket with `allUsers` bound to a reader/writer role
  - Data appears accessible via a direct public URL
- **Solution**: Enforce the `constraints/storage.publicAccessPrevention` org policy, use uniform bucket-level access, and grant specific principals (not `allUsers`) for any legitimate external access need

### SE-4: Default VPC Firewall Rules Too Permissive
- **Severity**: high
- **Situation**: The auto-created "default" VPC network ships with firewall rules allowing broad internal traffic and, depending on configuration, SSH/RDP from any IP
- **Cause**: The default network's implied firewall rules are convenience defaults meant for quick starts, not production security postures
- **Symptoms**:
  - Security scanners flag `0.0.0.0/0` ingress on SSH (22) or RDP (3389)
  - Lateral movement possible between VMs that shouldn't be able to reach each other
- **Solution**: Avoid using the default network for production; create custom VPCs with explicit, minimal firewall rules; restrict SSH/RDP to IAP tunneling or a bastion, never open to the internet

### SE-5: Project/Org IAM Inheritance Surprises
- **Severity**: medium
- **Situation**: A role granted at the Organization or Folder level to "make onboarding easier" turns out to apply to every project underneath it, including ones the grantee shouldn't access
- **Cause**: IAM bindings are inherited downward through the resource hierarchy with no way to override or deny at a lower level except via an explicit IAM Deny policy
- **Symptoms**:
  - A user or service account has access to a project nobody remembers granting them access to
  - IAM Recommender flags unused inherited permissions
- **Solution**: Grant roles at the lowest level that satisfies the need (Project, not Folder or Org, unless the access genuinely spans every project underneath), review inherited bindings with the Policy Analyzer/IAM Recommender periodically

## Recommended Tools

| Category | Tools |
|------|------|
| Infrastructure as Code | Terraform, Deployment Manager |
| Security auditing | Security Command Center, IAM Recommender, Policy Analyzer |
| Cost management | Cloud Billing Reports, BigQuery query validator |
| Observability | Cloud Monitoring, Cloud Logging, Cloud Trace |

## Security & Cost Hardening

**Full checklist**: → [extended/checklists.md#security-hardening-checklist]

## Related Resources

[Google Cloud Architecture Framework](https://cloud.google.com/architecture/framework) | [GCP Docs](https://cloud.google.com/docs) | [BigQuery Best Practices](https://cloud.google.com/bigquery/docs/best-practices-costs)

## Related Domains

[[aws]] | [[azure]] | [[python]] | [[go]]
