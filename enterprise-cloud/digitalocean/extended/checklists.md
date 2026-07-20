# DigitalOcean — Extended Checklists

## Infrastructure Safety Checklist

- [ ] Cloud firewall explicitly created and attached to every Droplet (or applied via tag-based rule) — none applied by default
- [ ] Automated backups enabled for any Droplet holding data that matters
- [ ] Manual snapshots taken before risky changes (upgrades, major config changes), supplementing (not replacing) automated backups
- [ ] Restore process actually tested periodically against a non-production Droplet
- [ ] Spaces buckets/objects default to private access; public read scoped explicitly only where genuinely needed
- [ ] VPC used for private inter-Droplet communication instead of routing internal traffic over public IPs
- [ ] Production services run across multiple Droplets behind a Load Balancer, not a single-Droplet SPOF
- [ ] Managed databases used with a standby node for anything requiring high availability, rather than a self-hosted single-instance database

## Cost & Operations Checklist

- [ ] Resource tagging used consistently for cost attribution and firewall/rule targeting
- [ ] Unused Droplets, volumes, and snapshots reviewed and cleaned up periodically
- [ ] Monitoring/alerting configured for Droplet resource usage (CPU, memory, disk) before it becomes an incident
- [ ] App Platform or DOKS evaluated as an alternative to raw Droplets where reduced ops overhead is worth the tradeoff
- [ ] `doctl`/Terraform used for reproducible provisioning instead of manual dashboard-only setup for production infrastructure
