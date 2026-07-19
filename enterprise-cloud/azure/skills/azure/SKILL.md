---
schema: "1.0"
name: azure
version: "1.0.0"
description: Azure core services, Entra ID/RBAC security, cost management, and Well-Architected Framework practices
domain: technology
triggers:
  keywords:
    primary: [azure, azure functions, app service, entra id, azure devops]
    secondary: [resource group, cosmos db, azure sql, aks, key vault, arm template, bicep]
  context_boost: [infrastructure, deployment, cloud architecture]
  context_penalty: [aws, gcp, on-premise]
  priority: high
dependencies:
  software-skills: [git-workflows, infrastructure-as-code]
author: claude-domain-skills
---

# Azure

> Resource hierarchy, Entra ID RBAC, and cost-aware architecture on Microsoft's cloud

## Applicable Scenarios

- Designing or reviewing Azure infrastructure and service selection
- Writing or auditing Entra ID (formerly Azure AD) role assignments
- Diagnosing unexpected Azure cost spikes
- Choosing compute (App Service vs Functions vs AKS vs VMs) or data services
- Applying the Azure Well-Architected Framework to a design review

## Core Knowledge

### Core Service Categories

| Category | Services | Notes |
|------|------|------|
| **Compute** | VMs, App Service, Azure Functions, AKS, Container Apps | App Service for web apps; Functions for event-driven; AKS for full Kubernetes control |
| **Storage** | Blob Storage, Azure Files, Disks | Blob = object storage; Files = SMB-shared file system; Disks = VM-attached block storage |
| **Database** | Azure SQL, Cosmos DB, PostgreSQL/MySQL Flexible Server | Cosmos DB = globally distributed multi-model NoSQL, billed by provisioned/consumed RU |
| **Networking** | VNet, Azure Front Door, Application Gateway, Load Balancer | VNet is the network boundary; Front Door/App Gateway front it at L7 |
| **Identity** | Entra ID (Azure AD), RBAC, Managed Identities | The security perimeter for almost everything else |

### Resource Hierarchy

```
Management Group → Subscription → Resource Group → Resource
```

- **Management Groups** apply policy/RBAC across multiple subscriptions (org-wide governance)
- **Subscriptions** are billing + isolation boundaries
- **Resource Groups** are logical containers, typically scoped per application/environment — deleting a resource group deletes everything inside it (a common accidental-deletion risk)

### Identity & RBAC

- **Entra ID** is the identity provider for both users and workloads (via Service Principals/Managed Identities)
- **Managed Identities** let a resource (e.g., a VM or Function App) authenticate to other Azure services without embedding credentials — always prefer this over storing secrets
- **RBAC role assignments** are scoped at Management Group, Subscription, Resource Group, or Resource level — assign at the narrowest scope that works
- Built-in roles like `Owner`/`Contributor` at subscription scope are broad — prefer custom roles or narrower built-ins (e.g., `Storage Blob Data Contributor`) for service identities

### Compute Selection

| Need | Choice |
|------|------|
| Web app, minimal ops overhead | **App Service** |
| Event-driven, short execution | **Azure Functions** |
| Full Kubernetes control | **AKS** |
| Containerized without cluster management | **Container Apps** |
| Full OS control, specialized workloads | **Virtual Machines** |

### Well-Architected Framework Pillars

1. **Reliability** — design for failure, define recovery targets (RTO/RPO)
2. **Security** — Zero Trust, least privilege, encrypt everywhere
3. **Cost Optimization** — right-size, monitor spend continuously
4. **Operational Excellence** — automate, monitor, standardize deployment
5. **Performance Efficiency** — scale to meet demand without over-provisioning

## Best Practices

1. **Use Managed Identities** instead of storing service credentials in Key Vault-adjacent app config
2. **Assign RBAC at the narrowest scope** that satisfies the need — resource, not subscription, when possible
3. **Use Infrastructure as Code** (Bicep or Terraform) — avoid manual portal changes to production
4. **Set up Cost Management budgets and alerts** before enabling anything that auto-scales
5. **Separate environments by subscription or resource group**, governed by Management Group policy
6. **Enable Microsoft Defender for Cloud** for security posture monitoring across subscriptions

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Assigning `Owner`/`Contributor` at subscription scope to a service identity | Use a narrowly-scoped custom role at the resource/resource-group level |
| Storing connection strings/secrets in App Settings in plaintext | Use Key Vault references or Managed Identity-based auth |
| Deleting a resource group without checking what's inside | Review resource group contents and use resource locks on critical resources |
| Leaving Storage Account public blob access enabled | Disable public access at the account level unless explicitly required |
| Provisioning Cosmos DB RU/s far above actual usage "just in case" | Use autoscale RU/s or serverless tier, monitor actual consumption |

## Sharp Edges

### SE-1: Overly Broad RBAC Role Assignments
- **Severity**: critical
- **Situation**: A Managed Identity or service principal is assigned `Contributor` at the subscription level "to make it work," giving it write access to every resource in the subscription
- **Cause**: Broad roles avoid the friction of scoping precisely, and Azure doesn't warn when a role assignment is broader than needed
- **Symptoms**:
  - Access reviews (Entra ID) flag identities with far more permission than their usage shows
  - A compromised app identity can modify unrelated resources
- **Solution**: Scope role assignments to the specific resource or resource group, use built-in data-plane roles (e.g. `Storage Blob Data Reader`) instead of control-plane roles like `Contributor` where only data access is needed
- **Details**: → [extended/checklists.md#security-hardening-checklist]

### SE-2: Public Storage Account Access
- **Severity**: critical
- **Situation**: A Storage Account's blob containers are publicly readable, exposing data that was meant to be private
- **Cause**: Public access can be enabled per-container and is sometimes turned on to simplify sharing during development, then never revisited
- **Symptoms**:
  - Defender for Cloud flags public blob access
  - Data appears accessible via a direct blob URL with no authentication
- **Solution**: Disable "Allow Blob public access" at the Storage Account level by default; use SAS tokens with short expiry or Managed Identity-based access for legitimate external access needs

### SE-3: Resource Group Deletion Cascades
- **Severity**: high
- **Situation**: Deleting a resource group to "clean up" removes a resource still in active use, because resource groups don't warn about cross-group dependencies
- **Cause**: A resource group is just a container; Azure has no built-in dependency graph across resource groups that blocks deletion when something depends on a resource inside it
- **Symptoms**:
  - An unrelated application breaks immediately after an "unused" resource group is deleted
- **Solution**: Apply resource locks (`CanNotDelete`) on production resource groups and critical resources, audit cross-resource-group references before deleting anything

### SE-4: Cosmos DB Cost Surprises From Over-Provisioned RU/s
- **Severity**: high
- **Situation**: Cosmos DB is provisioned with fixed RU/s sized for peak load "to be safe," and the bill reflects that peak 24/7 even during idle periods
- **Cause**: Cosmos DB in provisioned-throughput mode bills for allocated RU/s regardless of actual consumption, unlike consumption-based services
- **Symptoms**:
  - Cost Management shows a large, flat Cosmos DB line item disproportionate to actual traffic
  - Metrics show RU consumption far below provisioned RU/s most of the day
- **Solution**: Use autoscale throughput or the serverless tier for spiky/unpredictable workloads, monitor RU consumption and right-size provisioned throughput based on actual p95 usage

### SE-5: Key Vault Access Policy Misconfiguration
- **Severity**: high
- **Situation**: An application can't retrieve a secret it needs (or, worse, an application can retrieve secrets it shouldn't), due to Key Vault access policy or RBAC misconfiguration
- **Cause**: Key Vault supports two authorization models — legacy Access Policies and Azure RBAC — and mixing assumptions about which one is active on a given vault leads to either broken access or unintended over-grants
- **Symptoms**:
  - "Access denied" errors despite an apparently correct role assignment (wrong authorization model checked)
  - An identity has broader secret access than intended after a policy was copied from another vault
- **Solution**: Standardize on Azure RBAC authorization for Key Vault (the recommended model) across the org, verify which model a given vault uses before troubleshooting access, and grant `Key Vault Secrets User` (not broader) to consuming identities

## Recommended Tools

| Category | Tools |
|------|------|
| Infrastructure as Code | Bicep, Terraform, ARM templates |
| Security auditing | Microsoft Defender for Cloud, Entra ID Access Reviews |
| Cost management | Cost Management + Billing, Azure Advisor |
| Observability | Azure Monitor, Application Insights, Log Analytics |

## Security & Cost Hardening

**Full checklist**: → [extended/checklists.md#security-hardening-checklist]

## Related Resources

[Azure Well-Architected Framework](https://learn.microsoft.com/azure/well-architected/) | [Azure Docs](https://learn.microsoft.com/azure/) | [Azure Architecture Center](https://learn.microsoft.com/azure/architecture/)

## Related Domains

[[aws]] | [[gcp]] | [[java]] | [[python]]
