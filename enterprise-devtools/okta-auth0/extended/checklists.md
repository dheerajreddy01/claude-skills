# Okta / Auth0 — Extended Checklists

## Token Handling Checklist

- [ ] APIs validate access tokens (not ID tokens) for authorization, checking `aud` and `iss` claims explicitly
- [ ] Access/ID token lifetimes kept short; refresh tokens used for longevity
- [ ] Refresh tokens stored securely (httpOnly cookies or secure backend session), never in client-side-accessible storage
- [ ] Redirect URIs configured with exact-match allowlisting, no wildcards or prefix matching
- [ ] Token revocation supported and tested for incident response scenarios
- [ ] Every registered application/flow reviewed to confirm token validation is implemented correctly, not copy-pasted incorrectly from another integration
- [ ] Client secrets (for confidential clients) stored in a secrets manager, not embedded in source or client-side code

## Access Policy Checklist

- [ ] MFA policy coverage audited across every application and authentication flow individually, not assumed from an org-level toggle
- [ ] Legacy no-MFA fallback login flows eliminated or brought under the same MFA policy
- [ ] Role/group-to-authorization mapping documented, with sync/cache freshness explicitly decided (fresh-per-request vs. cached with a bounded TTL)
- [ ] Offboarding/access-revocation propagation time measured and understood for incident response planning
- [ ] Adaptive/risk-based MFA policies reviewed periodically against actual login risk signals
- [ ] Session timeout and concurrent-session policy reviewed against the sensitivity of what's being protected
