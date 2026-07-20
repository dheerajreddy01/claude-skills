# Ansible — Extended Checklists

## Idempotency Checklist

- [ ] Purpose-built modules used instead of `shell`/`command` wherever an idempotent module exists
- [ ] Any remaining `shell`/`command` tasks guarded with `creates`, `changed_when`, or `when` to avoid re-run side effects
- [ ] Playbooks tested by running twice — second run reports no unexpected changes
- [ ] `become` scoped consistently at the play/role level, not scattered inconsistently across individual tasks
- [ ] Playbooks tested against a representative non-privileged connecting user to catch escalation gaps early
- [ ] `gather_facts: false` set for playbooks/roles that don't reference host facts
- [ ] Handlers used for restart/reload actions (triggered only on actual change) instead of unconditional restart tasks

## Secrets & Inventory Checklist

- [ ] All sensitive variables encrypted with Ansible Vault before being committed
- [ ] Vault password managed through a proper secrets pipeline, never committed to the same repo as encrypted secrets
- [ ] Vault password rotation process documented and exercised, not just assumed possible
- [ ] Dynamic inventory used for any environment where infrastructure changes with regularity
- [ ] Static inventory (where used) reviewed/updated as part of the same change process that provisions/decommissions hosts
- [ ] `ansible-lint` run in CI to catch common playbook issues before merge
- [ ] Role testing (Molecule or equivalent) covers idempotency, not just successful first-run application
