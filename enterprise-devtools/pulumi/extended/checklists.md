# Pulumi — Extended Checklists

## State and Secrets Checklist

- [ ] State backend uses Pulumi Cloud's managed backend, or a self-managed backend with equivalent access control and encryption at rest
- [ ] `pulumi.secret()` used for every sensitive output; secret-ness verified to propagate through any transformation applied to it
- [ ] CLI/CI output audited for accidental secret exposure; any leak treated as requiring credential rotation
- [ ] `protect: true` set on critical stateful resources (databases, storage with irreplaceable data)
- [ ] Target stack double-checked before every `pulumi destroy`
- [ ] Provider-level credentials (for Pulumi to authenticate to the cloud) kept separate from and not conflated with infrastructure secrets Pulumi manages

## Program Reliability Checklist

- [ ] `pulumi preview` reviewed carefully before every `pulumi up`, same discipline as reviewing a Terraform plan
- [ ] Resource-defining logic kept deterministic — no unseeded randomness or wall-clock-dependent branching in program logic
- [ ] Stack references understood as live dependencies, not pinned versions; drift-sensitive infrastructure coordinates upstream/downstream deploys deliberately
- [ ] Unit tests (mocked resource providers) cover critical resource configuration logic
- [ ] Policy-as-code (CrossGuard) or equivalent used to enforce organizational guardrails automatically
