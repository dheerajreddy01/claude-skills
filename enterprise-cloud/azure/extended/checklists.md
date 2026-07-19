# Azure — Extended Checklists

## Security Hardening Checklist

- [ ] No service identity holds `Owner`/`Contributor` at subscription scope without explicit justification
- [ ] RBAC role assignments reviewed at the narrowest scope (resource > resource group > subscription > management group)
- [ ] Managed Identities used for service-to-service auth instead of embedded credentials
- [ ] Storage Account "Allow Blob public access" disabled by default; exceptions documented
- [ ] Key Vault authorization model confirmed (RBAC preferred over legacy Access Policies) and consistently applied
- [ ] Resource locks (`CanNotDelete`) applied to production resource groups and critical resources
- [ ] Microsoft Defender for Cloud enabled across all subscriptions
- [ ] Entra ID Access Reviews scheduled periodically for privileged role assignments
- [ ] Conditional Access policies require MFA for privileged roles
- [ ] Diagnostic logging enabled and shipped to Log Analytics for all production resources

## Cost Control Checklist

- [ ] Cost Management budgets and alerts configured before deploying auto-scaling resources
- [ ] Cosmos DB uses autoscale or serverless throughput unless steady-state provisioned RU/s is justified by actual usage data
- [ ] Azure Advisor cost recommendations reviewed monthly
- [ ] Unused resources (unattached disks, idle App Service plans, orphaned public IPs) cleaned up regularly
- [ ] Reserved Instances/Savings Plans evaluated for steady-state VM/App Service workloads
- [ ] Resource tags applied consistently (environment, team, cost center) for cost attribution
- [ ] Autoscale rules on App Service/VMSS reviewed to ensure scale-in works, not just scale-out
