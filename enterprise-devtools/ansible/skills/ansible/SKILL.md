---
schema: "1.0"
name: ansible
version: "1.0.0"
description: Ansible playbook design, idempotency discipline, inventory management, and Vault secrets practices
domain: technology
triggers:
  keywords:
    primary: [ansible, playbook, ansible-playbook, inventory]
    secondary: [ansible vault, idempotent, role, become]
  context_boost: [configuration management, server provisioning, agentless]
  context_penalty: [terraform, pulumi, kubernetes]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Ansible

> Idempotency is a design discipline you owe every task, not a property Ansible gives you automatically

## Applicable Scenarios

- Writing or reviewing playbooks, roles, and inventory structure
- Designing idempotent automation for server configuration/provisioning
- Managing secrets via Ansible Vault
- Diagnosing inconsistent results across repeated playbook runs
- Structuring privilege escalation (`become`) correctly across tasks

## Core Knowledge

### Agentless, Push-Based Model

- Ansible connects over SSH (or WinRM for Windows) and pushes changes — no persistent agent needs to be installed on managed hosts, unlike Puppet/Chef's typical pull-agent model
- This makes onboarding a new host trivial (just needs SSH access + Python), but also means Ansible has no continuous drift-detection loop — it only knows about a host's state during the moment a playbook actually runs against it

### Idempotency Is a Design Goal, Not a Guarantee

- Most built-in Ansible **modules** (e.g., `apt`, `template`, `file`, `user`) are written to be idempotent — running them repeatedly converges to the same end state and reports "unchanged" if nothing needed to change
- The `command` and `shell` modules are explicitly **not** idempotent by default — they just run the given command every time, with no built-in concept of "was this already done." Using them for anything beyond one-off, side-effect-free operations breaks the idempotency guarantee the rest of Ansible relies on

### Inventory

- **Static inventory**: a file (INI or YAML) listing hosts and groups — simple, but requires manual updates as infrastructure changes
- **Dynamic inventory**: a script/plugin that queries a live source (cloud provider API, CMDB) at runtime — stays accurate as infrastructure changes, at the cost of needing that integration maintained
- Inventory drift (static inventory not matching actual live infrastructure) is a common failure mode in environments where infrastructure changes frequently but the inventory file doesn't get updated in lockstep

### Ansible Vault

- Encrypts sensitive data (variable files, or specific values) so secrets can be committed to version control safely
- Requires a vault password (or password file/script) to decrypt at runtime — password management itself becomes the new secret-handling problem (who has it, how is it rotated, how is it supplied in CI)

### Privilege Escalation (`become`)

- `become: true` (with `become_user`, typically `root`) escalates privileges for a task, analogous to `sudo`
- Partial privilege failures — where escalation works for some tasks in a play but not others due to inconsistent `become` scoping — can leave a host in an inconsistent, half-applied state

## Best Practices

1. **Prefer purpose-built modules over `command`/`shell`** — they carry real idempotency guarantees; reserve `command`/`shell` for genuinely one-off actions, guarded with `creates`/`changed_when` if repeated
2. **Keep inventory as close to a live source of truth as possible** — use dynamic inventory where infrastructure changes frequently
3. **Encrypt all sensitive variables with Ansible Vault**, and manage the vault password through a proper secrets pipeline, not a shared plaintext file
4. **Scope `become` deliberately and consistently** across a play, not ad hoc per task
5. **Disable unnecessary fact-gathering** (`gather_facts: false`) for playbooks that don't use host facts, to reduce unnecessary per-run overhead at scale
6. **Test playbooks for idempotency explicitly** — run twice and confirm the second run reports no changes, as a standard check before considering a playbook done

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Using `shell`/`command` for something a purpose-built module already handles | Use the dedicated module (`apt`, `copy`, `template`, `service`, etc.) for real idempotency |
| Running `shell`/`command` repeatedly with no guard against re-running an already-done action | Add `creates`/`changed_when`/`when` conditions, or switch to an idempotent module |
| Committing an Ansible Vault password to the same repo as the encrypted secrets | Manage the vault password through a separate, properly access-controlled secrets pipeline |
| Static inventory left stale as infrastructure changes | Use dynamic inventory sourced from the actual infrastructure provider/CMDB |
| Gathering facts on every play regardless of whether they're used | Disable fact-gathering (`gather_facts: false`) when facts aren't needed, especially at scale |

## Sharp Edges

### SE-1: Non-Idempotent Shell/Command Tasks Causing Different Results on Re-Run
- **Severity**: high
- **Situation**: A playbook using `shell`/`command` to perform a setup action (e.g., appending a line to a config file, creating a resource) produces a different or broken result the second time it runs, because the action wasn't actually idempotent
- **Cause**: `shell`/`command` execute exactly what's specified every single run with no built-in "already done" check — an append operation run twice appends twice, a "create" operation run twice may error or duplicate
- **Symptoms**:
  - A playbook that worked on first run fails or produces duplicated/corrupted state on a subsequent run
  - Re-running a playbook for a routine, no-op-expected reason (e.g., after adding an unrelated task) causes unexpected side effects on already-configured hosts
- **Solution**: Replace `shell`/`command` with a purpose-built idempotent module wherever one exists (`lineinfile`, `blockinfile`, `template`, etc.), and for genuinely custom shell logic, add `creates`, `changed_when`, or an explicit `when` guard so re-runs behave correctly
- **Details**: → [extended/checklists.md#idempotency-checklist]

### SE-2: Ansible Vault Password Mismanagement Exposing Secrets
- **Severity**: critical
- **Situation**: The Ansible Vault password itself — the key that decrypts every encrypted secret in the repo — ends up committed to version control, shared insecurely over chat, or hardcoded in a CI script, undermining the entire point of using Vault
- **Cause**: Vault protects the *encrypted content*, but the vault password is a separate secret that has to be managed with equal or greater care — it's easy to treat it as "just another config value" rather than the master key it actually is
- **Symptoms**:
  - A security review finds the vault password in a place with broader access than the secrets it protects
- **Solution**: Store the vault password in a proper secrets manager (not a file in the repo), use `--vault-password-file` pointing to a securely-provisioned file at runtime (never committed), and rotate the vault password (re-encrypting all secrets) if it's ever suspected of exposure

### SE-3: Partial Privilege Escalation Failures Leaving Hosts Half-Configured
- **Severity**: high
- **Situation**: A play with inconsistent `become` scoping succeeds for some tasks (which had appropriate privileges) and fails partway through on a task that needed escalation it didn't have, leaving the host in a state that's neither the old configuration nor the fully new one
- **Cause**: `become` can be set at different levels (play, block, task) and inconsistently applying it means a play can partially succeed with real side effects already applied before hitting a permission failure
- **Symptoms**:
  - A host shows some but not all of the intended configuration changes after a failed playbook run
  - Re-running the playbook after fixing the privilege issue is the only way to reach a fully consistent state (which is fine if idempotent, risky if not)
- **Solution**: Set `become` consistently at the play or role level rather than scattering it across individual tasks, test playbooks against a representative non-privileged connecting user to catch escalation gaps before they hit production, and combine with the idempotency practices above so a resumed/re-run playbook can safely complete the job

### SE-4: Inventory Drift From Static Inventory Not Matching Live Infrastructure
- **Severity**: high
- **Situation**: A playbook run targets a static inventory group that no longer accurately reflects live infrastructure — it's missing newly-provisioned hosts (which never get configured) or still includes decommissioned hosts (which fail to connect, or worse, if reused for something else, get unexpectedly reconfigured)
- **Cause**: Static inventory is just a file — nothing keeps it automatically synchronized with actual infrastructure changes made through other tooling (Terraform, cloud console, autoscaling)
- **Symptoms**:
  - Newly provisioned hosts don't receive expected configuration until someone remembers to add them to inventory
  - A playbook run against a stale inventory entry fails with a connection error, or worse, succeeds against a host that was repurposed
- **Solution**: Use dynamic inventory sourced from the actual infrastructure provider (cloud API, Terraform state, CMDB) wherever infrastructure changes with any regularity, and treat static inventory as appropriate only for genuinely stable, manually-managed environments

### SE-5: Unnecessary Fact-Gathering Overhead at Scale
- **Severity**: medium
- **Situation**: A playbook run against hundreds of hosts takes noticeably longer than expected, and the bottleneck turns out to be the default fact-gathering step (`setup` module) running against every host even though the playbook never actually references any gathered facts
- **Cause**: `gather_facts` defaults to `true`, and the fact-gathering step queries a substantial amount of information from each host over SSH before any actual task runs — at scale, this per-host overhead adds up regardless of whether the playbook uses the results
- **Symptoms**:
  - Playbook run time scales up disproportionately with host count relative to the actual task work being done
- **Solution**: Set `gather_facts: false` for playbooks/roles that don't reference any host facts, and where facts genuinely are needed, consider narrowing gathered facts (`gather_subset`) instead of collecting everything by default

## Recommended Tools

| Category | Tools |
|------|------|
| Testing | Molecule (role testing), `ansible-lint` |
| Dynamic inventory | Cloud provider inventory plugins (AWS, Azure, GCP), Terraform inventory |
| Secrets | Ansible Vault, integration with HashiCorp Vault |
| Execution at scale | Ansible Automation Platform (AAP), `mitogen` for performance |

## Idempotency & Safety

**Full checklist**: → [extended/checklists.md#idempotency-checklist]

## Related Resources

[Ansible Docs](https://docs.ansible.com/) | [Ansible Best Practices](https://docs.ansible.com/ansible/latest/tips_tricks/ansible_tips_tricks.html) | [Molecule Testing](https://ansible.readthedocs.io/projects/molecule/)

## Related Domains

[[terraform]] | [[vault]] | [[aws]] | [[python]]
