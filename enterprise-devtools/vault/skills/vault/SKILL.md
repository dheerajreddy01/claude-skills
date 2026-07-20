---
schema: "1.0"
name: vault
version: "1.0.0"
description: HashiCorp Vault secrets engines, dynamic secrets, unsealing, and HA practices
domain: technology
triggers:
  keywords:
    primary: [vault, hashicorp vault, secrets management, dynamic secrets]
    secondary: [unseal, approle, lease, audit log, seal wrap]
  context_boost: [secret rotation, credential management, PKI]
  context_penalty: [aws secrets manager, azure key vault]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# HashiCorp Vault

> Static secrets in Vault are just a fancier place to store the same long-lived risk

## Applicable Scenarios

- Designing secrets engine strategy (KV vs. dynamic database/cloud credentials)
- Choosing auth methods for services and humans (AppRole, Kubernetes auth, LDAP)
- Planning unsealing, root token handling, and disaster recovery
- Reviewing policies for least-privilege access to secret paths
- Diagnosing lease expiration or renewal failures

## Core Knowledge

### Secrets Engines

| Engine | Behavior |
|------|------|
| **KV (Key-Value)** | Static secret storage — Vault stores whatever you put in, versioned in KV v2 |
| **Database** | Generates **dynamic**, short-lived database credentials per request, revoked automatically on lease expiry |
| **Cloud (AWS/Azure/GCP)** | Generates dynamic, short-lived cloud IAM credentials |
| **PKI** | Issues short-lived TLS certificates on demand |

The dynamic engines are what differentiate Vault from "just an encrypted key-value store" — a credential that's generated on demand and expires automatically limits the blast radius of a leak far more than a static, long-lived one ever can.

### Auth Methods

- **AppRole**: designed for machine-to-machine auth — a `role_id` (like a username) plus a `secret_id` (like a password, often short-lived and delivered via a secure bootstrap process)
- **Kubernetes auth**: lets a pod authenticate using its projected service account token, avoiding the need to distribute a Vault credential to the pod at all
- Human auth typically goes through an identity provider (LDAP, OIDC) rather than long-lived tokens

### Leases & Renewal

- Every dynamic secret is issued with a **lease** — a TTL after which Vault revokes it
- Leases can be **renewed** (extending the TTL) up to a maximum TTL, after which the secret must be reissued
- An application that doesn't handle lease renewal or reissuance will simply lose access when the lease expires — this needs to be built into the application/client, not assumed to be automatic

### Unsealing & the Root of Trust

- Vault's storage is encrypted; the **unseal process** reconstructs the master key from a threshold of key shares (Shamir's Secret Sharing) to decrypt it and bring Vault online after a restart
- The **root token** has unrestricted access to everything in Vault — it should be generated once, used only for initial setup/emergency break-glass, and revoked (not kept lying around for routine use)
- **Auto-unseal** (using a cloud KMS to unseal automatically) removes the manual unseal-key-holder ceremony but shifts trust to that KMS

## Best Practices

1. **Prefer dynamic secrets over static KV storage** wherever the target system supports it — expiring credentials fundamentally limit leak impact
2. **Scope policies to specific paths**, never wildcard grants across the whole secret tree
3. **Use AppRole or platform-native auth (Kubernetes auth)** for machine identities — avoid distributing long-lived tokens
4. **Handle lease renewal explicitly in application code**, or use a sidecar (Vault Agent) that manages it transparently
5. **Revoke the root token after initial setup**, and use scoped, audited tokens for ongoing operations
6. **Run Vault in HA mode with auto-unseal** for production — a single-instance, manually-unsealed Vault is a standing operational risk

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Storing a static database password in KV instead of using the database secrets engine | Use the database secrets engine for dynamic, auto-expiring credentials |
| Wildcard policy paths (`secret/*`) granted broadly | Scope policies to the specific paths a role actually needs |
| Root token kept active and used for routine operations | Revoke after setup; use scoped tokens with appropriate policies for daily use |
| No lease renewal logic in the application | Use Vault Agent or explicit renewal calls before lease expiry |
| Single Vault instance with manual unseal, no HA | Run Vault in HA mode with auto-unseal via a cloud KMS for production |

## Sharp Edges

### SE-1: Unseal Key / Root Token Mismanagement
- **Severity**: critical
- **Situation**: Too few people hold unseal key shares (or they're stored insecurely, e.g., in a shared document), and either Vault can't be unsealed after an outage because key holders are unavailable, or the keys are compromised, defeating Vault's entire security model
- **Cause**: Shamir's Secret Sharing requires a threshold of key shares to reconstruct the master key — this security property only holds if the shares are actually distributed to separate, trusted parties and stored securely; concentrating them defeats the purpose
- **Symptoms**:
  - Vault outage recovery is delayed or blocked because key holders aren't reachable
  - A security audit finds unseal keys stored together in one insecure location
- **Solution**: Distribute unseal key shares to separate trusted individuals with secure, independent storage, or use auto-unseal via a cloud KMS to remove the manual ceremony entirely for production — and treat the root token as a break-glass credential, generated once, used for setup, then revoked
- **Details**: → [extended/checklists.md#operational-checklist]

### SE-2: Static Secrets Undermining Vault's Core Value
- **Severity**: high
- **Situation**: Vault is deployed, but every team just stores static, long-lived credentials in the KV engine — functionally identical to any other encrypted key-value store, none of Vault's dynamic-secret leak-mitigation benefit is realized
- **Cause**: KV storage is the easiest engine to adopt (just put a value, get a value) and requires no integration work, so it's the path of least resistance even when a dynamic engine (database, cloud) would be a better fit and is available
- **Symptoms**:
  - A leaked credential from Vault's KV store remains valid indefinitely, just like any static secret would
  - Security review finds Vault used purely as "a nicer secrets store" with no dynamic engines in use
- **Solution**: Migrate to dynamic secrets engines (database, cloud IAM, PKI) wherever the target system supports credential generation on demand, reserving KV for genuinely static values (e.g., a third-party API key with no dynamic issuance option)

### SE-3: Lease Expiration Not Handled by Applications
- **Severity**: high
- **Situation**: A dynamic database credential's lease expires, the application wasn't renewing or reissuing it, and every subsequent database connection attempt starts failing with an authentication error — often well after deployment, when the issue is least expected
- **Cause**: Lease renewal is not automatic from the application's perspective unless explicitly built in (via client-side renewal logic or a sidecar like Vault Agent) — Vault revokes the underlying credential precisely on schedule regardless of whether the application is ready for that
- **Symptoms**:
  - Authentication failures that start suddenly, hours or days after a deployment, with no corresponding code change
  - Failure timing correlates with the configured lease TTL
- **Solution**: Use Vault Agent (which handles renewal/reissuance transparently and writes fresh credentials to a file the app reads) or implement explicit renewal logic in the application well before lease expiry, and monitor for lease renewal failures as an operational signal

### SE-4: Overly Broad Policy Grants
- **Severity**: high
- **Situation**: A policy grants access to a wildcard path (`secret/data/*` or similar) "to avoid friction," and any service or user with that policy can read every secret in the tree, not just the ones it actually needs
- **Cause**: Wildcard policies are faster to write and rarely need revisiting when new secrets are added under the same path prefix, so they accumulate as the path of least resistance
- **Symptoms**:
  - A policy audit shows grants far broader than the actual number of secrets a role reads in practice
  - A single compromised service identity can read secrets belonging to unrelated services
- **Solution**: Scope policies to the specific paths a role actually needs, using Vault's templated policies (parameterized by identity) where many similar roles need access to their own namespaced secrets, and audit policies periodically for drift toward over-permissioning

### SE-5: Single Point of Failure Without HA/Auto-Unseal
- **Severity**: high
- **Situation**: A single Vault instance goes down (host failure, restart for maintenance) and every service depending on it for secrets/credentials loses access simultaneously — and bringing it back up requires manually gathering unseal key holders, extending the outage
- **Cause**: A non-HA Vault deployment has no failover, and manual unsealing adds operational latency to recovery exactly when speed matters most
- **Symptoms**:
  - A Vault outage cascades into a broad multi-service outage
  - Recovery time is dominated by "waiting for unseal key holders to be available," not the underlying infrastructure fix
- **Solution**: Run Vault in HA mode (integrated storage/Raft or a supported HA backend) with auto-unseal via a cloud KMS, so a node failure fails over automatically without a manual unsealing ceremony

## Recommended Tools

| Category | Tools |
|------|------|
| Client integration | Vault Agent, Vault CSI provider (Kubernetes) |
| Dynamic secrets engines | Database, AWS/Azure/GCP, PKI engines |
| Monitoring | Vault audit log, Prometheus telemetry |
| Alternatives/complements | Cloud-native secret managers (AWS Secrets Manager, Azure Key Vault) for simpler single-cloud needs |

## Operational Hardening

**Full checklist**: → [extended/checklists.md#operational-checklist]

## Related Resources

[Vault Docs](https://developer.hashicorp.com/vault/docs) | [Vault Production Hardening Guide](https://developer.hashicorp.com/vault/tutorials/operations/production-hardening) | [Vault Secrets Engines](https://developer.hashicorp.com/vault/docs/secrets)

## Related Domains

[[kubernetes]] | [[terraform]] | [[aws]] | [[okta-auth0]]
