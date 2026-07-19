# GCP — Extended Checklists

## Security Hardening Checklist

- [ ] No service account keys downloaded/distributed where Workload Identity Federation or attached identities are viable
- [ ] Organization policy `constraints/iam.disableServiceAccountKeyCreation` considered for environments that don't need keys
- [ ] IAM roles are predefined/custom and scoped to the specific service, not primitive `Owner`/`Editor`/`Viewer`, for service identities
- [ ] `constraints/storage.publicAccessPrevention` org policy enforced; no bucket bound to `allUsers` without explicit justification
- [ ] Default VPC network not used for production workloads; custom VPCs with minimal firewall rules in place
- [ ] SSH/RDP access routed through Identity-Aware Proxy (IAP) tunneling, not exposed to `0.0.0.0/0`
- [ ] Security Command Center enabled at the organization level
- [ ] IAM bindings reviewed with Policy Analyzer/IAM Recommender for unused or overly broad inherited access
- [ ] Audit logs (Admin Activity + Data Access) enabled and exported to a centralized, access-restricted project

## Cost Control Checklist

- [ ] Large/frequently-queried BigQuery tables are partitioned and clustered
- [ ] `maximum_bytes_billed` set on interactive/exploratory queries
- [ ] BigQuery query cost estimated via dry run before running on large tables
- [ ] Billing budgets and alerts configured per project
- [ ] Committed Use Discounts / Sustained Use Discounts evaluated for steady-state Compute Engine workloads
- [ ] Unused resources (idle VMs, unattached disks, orphaned static IPs) reviewed and cleaned up regularly
- [ ] Cloud Run/Cloud Functions concurrency and max-instance limits set to bound worst-case cost
