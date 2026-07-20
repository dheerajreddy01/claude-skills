---
schema: "1.0"
name: digitalocean
version: "1.0.0"
description: DigitalOcean Droplets, App Platform, managed services, and simplified cloud infrastructure practices
domain: technology
triggers:
  keywords:
    primary: [digitalocean, droplet, DO, app platform]
    secondary: [spaces, managed database, DOKS, floating IP]
  context_boost: [simple hosting, small team infrastructure, VPS]
  context_penalty: [aws, azure, gcp]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# DigitalOcean

> Simpler and flatter-priced than the hyperscalers — which also means fewer guardrails are on by default

## Applicable Scenarios

- Choosing between Droplets, App Platform, and managed Kubernetes (DOKS)
- Setting up backups, firewalls, and networking for Droplet-based infrastructure
- Designing for high availability on a platform with fewer managed-HA primitives than AWS/Azure/GCP
- Using Spaces (object storage) and managed databases
- Reviewing cost structure for predictable, flat-rate billing

## Core Knowledge

### Compute Options

| Option | Model |
|------|------|
| **Droplets** | VMs — full OS control, you manage everything above the hypervisor |
| **App Platform** | PaaS — git-push deployment, managed runtime, less control but less ops overhead |
| **DOKS** | Managed Kubernetes — control plane managed, worker nodes are Droplets you still pay for directly |

DigitalOcean's value proposition is simplicity and flat, predictable pricing relative to AWS/Azure/GCP's more granular (and more complex) usage-based billing — but that simplicity also means fewer built-in guardrails and managed HA primitives are available out of the box.

### Networking & Security

- **Firewalls** (cloud firewalls, not host-level `iptables`) are opt-in per Droplet/tag — a newly created Droplet has no cloud firewall applied unless one is explicitly attached
- **VPCs** isolate private networking per region; Droplets in the same VPC can communicate over private IPs, reducing exposure and often avoiding bandwidth charges for internal traffic
- **Floating IPs** provide a reassignable public IP, useful for manual failover between Droplets (not automatic — reassignment is a deliberate action or scripted response to a health check)

### Storage

- **Spaces** is S3-API-compatible object storage — existing S3 tooling/SDKs generally work with a changed endpoint
- **Volumes** are block storage attachable to Droplets, independent of the Droplet's lifecycle (data persists if the Droplet is destroyed, as long as the volume isn't also deleted)

### Backups & Snapshots

- Droplet **backups** (automated, weekly/daily depending on plan) are opt-in and cost extra — not enabled by default
- **Snapshots** are manual, point-in-time images — useful before risky changes, but not a substitute for a scheduled backup policy

## Best Practices

1. **Explicitly attach a cloud firewall to every Droplet** — none is applied by default
2. **Enable automated backups for anything that matters** — it's opt-in, not a default safety net
3. **Use a VPC for private inter-Droplet communication** rather than routing internal traffic over public IPs
4. **Test snapshot/backup restore procedures**, not just their existence — an untested backup is an assumption, not a guarantee
5. **Design for HA deliberately** — DigitalOcean doesn't provide the same breadth of managed multi-AZ/auto-failover primitives as the hyperscalers, so redundancy (load balancers across multiple Droplets, managed database with standby) needs explicit design
6. **Use App Platform or DOKS instead of raw Droplets** when the operational simplicity is worth the reduced low-level control, especially for teams without dedicated infrastructure staff

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Assuming a new Droplet is firewalled by default | Explicitly create and attach a cloud firewall (or apply one via a tag-based rule) to every Droplet |
| No automated backups enabled | Enable Droplet backups (or a custom snapshot schedule) for anything with data that matters |
| Single Droplet architecture with no redundancy for a production service | Use a Load Balancer across multiple Droplets (or App Platform/DOKS) for anything requiring uptime guarantees |
| Spaces bucket left with public access broader than intended | Review and scope Spaces access permissions explicitly, similar to S3 bucket policy review |
| Assuming a snapshot/backup restores correctly without ever testing it | Periodically test the actual restore process, not just confirm backups are "enabled" |

## Sharp Edges

### SE-1: No Default Automated Backups
- **Severity**: critical
- **Situation**: A Droplet hosting production data is destroyed, corrupted, or compromised, and there's no recent backup to restore from — because Droplet backups were never explicitly enabled
- **Cause**: Automated Droplet backups are an opt-in, paid add-on — a newly created Droplet has none configured unless explicitly turned on during or after creation
- **Symptoms**:
  - A restore is needed and none exists; only an old or nonexistent manual snapshot is available
- **Solution**: Enable automated backups for any Droplet holding data that matters at creation time, and additionally take manual snapshots before risky changes (upgrades, major config changes) as a supplement, not a replacement
- **Details**: → [extended/checklists.md#infrastructure-safety-checklist]

### SE-2: Firewall Not Applied by Default
- **Severity**: critical
- **Situation**: A newly created Droplet is reachable on all its listening ports from the public internet, because no cloud firewall was attached — a service intended to be internal-only (a database, an admin panel) is accidentally exposed
- **Cause**: DigitalOcean cloud firewalls are a separate resource that must be explicitly created and attached to a Droplet or Droplet tag — nothing is applied automatically at Droplet creation
- **Symptoms**:
  - A port scan or security audit finds a service reachable from the public internet that was assumed to be firewalled
- **Solution**: Create and attach a cloud firewall (scoped to only the ports/sources genuinely needed) as a standard part of Droplet provisioning, ideally via a tag-based rule applied automatically to any Droplet with a matching tag, and audit exposed ports periodically

### SE-3: Spaces Bucket Public Access Misconfiguration
- **Severity**: high
- **Situation**: A Spaces bucket (or specific objects within it) intended for private/internal use is discovered publicly readable, exposing data
- **Cause**: Similar to S3, Spaces access control is configurable per-bucket and per-object, and it's easy to leave a bucket or an uploaded object's ACL more permissive than intended, especially when copying configuration patterns from tutorials that default to public read for simplicity
- **Symptoms**:
  - A security scan or manual check finds bucket contents accessible via a direct URL with no authentication
- **Solution**: Default to private bucket/object access, explicitly grant public read only for content genuinely meant to be public (e.g., static website assets), and audit Spaces ACLs periodically similar to S3 bucket policy reviews

### SE-4: Single-Droplet SPOF Architectures
- **Severity**: high
- **Situation**: A production service runs entirely on a single Droplet, and a hardware failure, OS-level crash, or resource exhaustion on that Droplet takes the entire service down with no automatic failover
- **Cause**: Unlike some hyperscaler managed services with built-in multi-AZ redundancy, a Droplet is a single VM — high availability requires explicitly architecting for it (multiple Droplets behind a Load Balancer, a managed database with a standby node), not something that comes free with the base compute product
- **Symptoms**:
  - A single Droplet failure or reboot causes a full service outage rather than a graceful failover
- **Solution**: Run production services across multiple Droplets behind a Load Balancer (or migrate to App Platform/DOKS, which provide more of this redundancy managed for you), and use managed databases with standby nodes rather than a self-hosted single-instance database on a Droplet

### SE-5: Untested Snapshot/Backup Restore Process
- **Severity**: medium
- **Situation**: Backups or snapshots have been "enabled" for a long time, but when an actual restore is needed, the process fails, takes far longer than expected, or restores to an unusable state — because the restore path was never actually exercised
- **Cause**: A backup existing is not the same guarantee as a backup being restorable — configuration drift, undocumented manual changes, or an untested restore procedure can all mean the backup doesn't actually get you back to a working state
- **Symptoms**:
  - A restore during an actual incident reveals missing configuration, wrong versions, or an incomplete environment compared to what was running
- **Solution**: Periodically perform an actual test restore (to a separate, non-production Droplet) and verify the restored service genuinely works, treating "backup exists" and "backup is restorable" as two different, both-necessary claims

## Recommended Tools

| Category | Tools |
|------|------|
| Infrastructure as Code | Terraform DigitalOcean provider, `doctl` CLI |
| Monitoring | DigitalOcean Monitoring, Uptime alerts |
| Container platform | DOKS (managed Kubernetes) |
| Object storage tooling | Any S3-compatible SDK/CLI (pointed at the Spaces endpoint) |

## Infrastructure Safety

**Full checklist**: → [extended/checklists.md#infrastructure-safety-checklist]

## Related Resources

[DigitalOcean Docs](https://docs.digitalocean.com/) | [DigitalOcean Community Tutorials](https://www.digitalocean.com/community/tutorials) | [App Platform Docs](https://docs.digitalocean.com/products/app-platform/)

## Related Domains

[[aws]] | [[terraform]] | [[docker]] | [[kubernetes]]
