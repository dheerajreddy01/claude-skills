# Oracle Cloud (OCI) — Extended Checklists

## IAM and Network Checklist

- [ ] Policies scoped to the narrowest verb (`inspect`/`read`/`use` before `manage`) and narrowest compartment, not defaulting to `manage all-resources in tenancy`
- [ ] Full inherited policy chain reviewed when auditing a specific compartment's access, not just policies directly attached to it
- [ ] Compartment hierarchy designed deliberately before provisioning, with inheritance behavior accounted for
- [ ] Security Lists and Network Security Groups reconciled together when troubleshooting connectivity — both checked, not just one
- [ ] NSGs preferred over Security Lists for granular per-resource control where the added complexity of running both isn't specifically needed
- [ ] Periodic policy review conducted to catch scope creep in existing grants
- [ ] Policy Simulator (or equivalent) used to verify actual effective access before applying a new policy broadly

## Cost & Data Safety Checklist

- [ ] Always Free tier resources kept genuinely active, or their reclamation risk explicitly accepted, for anything beyond disposable experimentation
- [ ] Backups of Always Free tier data maintained independently of the platform
- [ ] Autonomous Database auto-scaling activity monitored and correlated with actual application load
- [ ] Cost/usage alerts configured on Autonomous Database and other auto-scaling resources
- [ ] Query patterns reviewed for inefficiency before assuming auto-scaling cost growth reflects legitimate traffic growth
