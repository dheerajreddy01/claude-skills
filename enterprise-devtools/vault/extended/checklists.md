# HashiCorp Vault — Extended Checklists

## Operational Checklist

- [ ] Unseal key shares distributed to separate trusted individuals with secure independent storage, or auto-unseal via cloud KMS configured for production
- [ ] Root token revoked after initial setup; scoped tokens used for ongoing operations
- [ ] Vault runs in HA mode (integrated storage/Raft or supported HA backend) for production
- [ ] Disaster recovery process (backup, restore, re-unseal) tested periodically, not just configured
- [ ] Audit logging enabled and shipped to a centralized, access-restricted log store
- [ ] TLS enabled for all Vault API traffic, not left on plaintext HTTP even internally
- [ ] Vault version kept current with security patches tracked

## Secrets & Access Checklist

- [ ] Dynamic secrets engines (database, cloud, PKI) used wherever the target system supports them, in preference to static KV storage
- [ ] Policies scoped to specific paths — no wildcard grants across an entire secret tree
- [ ] Machine identities authenticate via AppRole or platform-native auth (Kubernetes auth), not long-lived static tokens
- [ ] Lease renewal handled explicitly (Vault Agent or application-level renewal logic) before expiry
- [ ] Lease renewal failures monitored and alerted on as an operational signal
- [ ] Policy grants audited periodically for drift toward over-permissioning
- [ ] Secret rotation cadence reviewed for any remaining static secrets that can't use a dynamic engine
