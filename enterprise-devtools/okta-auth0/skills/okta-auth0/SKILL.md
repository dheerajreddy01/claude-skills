---
schema: "1.0"
name: okta-auth0
version: "1.0.0"
description: Okta and Auth0 identity/SSO — OIDC/SAML, token handling, and MFA policy practices
domain: technology
triggers:
  keywords:
    primary: [okta, auth0, SSO, identity provider, OIDC]
    secondary: [SAML, MFA, access token, refresh token, redirect uri]
  context_boost: [authentication, authorization, login flow]
  context_penalty: [vault, dynamodb, kubernetes rbac]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Okta / Auth0 (Identity & SSO)

> An ID token and an access token look similar and are not interchangeable — mixing them up is the most common integration bug

## Applicable Scenarios

- Integrating an application with Okta or Auth0 for authentication
- Choosing between OIDC and SAML for a given integration
- Designing MFA enforcement policy across applications and user segments
- Reviewing token lifetime and session configuration
- Mapping identity provider roles/groups to application-level authorization

## Core Knowledge

### OIDC vs. SAML

| Protocol | Format | Best For |
|------|------|------|
| **OIDC** (OpenID Connect, built on OAuth 2.0) | JSON/JWT | Modern web/mobile/SPA apps, API authorization |
| **SAML** | XML | Legacy enterprise SSO integrations, some enterprise IdP requirements |

OIDC is the default choice for new integrations; SAML remains relevant mainly because some enterprise customers' identity teams standardize on it.

### Token Types

| Token | Purpose | Common Mistake |
|------|------|------|
| **ID token** | Proves who the user is, to the client application | Used to call an API — it wasn't designed or validated for that purpose |
| **Access token** | Authorizes calls to a specific API/resource | Assumed to identify the user directly — it's an authorization credential, not necessarily user-descriptive |
| **Refresh token** | Used to obtain new access/ID tokens without re-authenticating | Stored insecurely (e.g., in browser localStorage) where it's exposed to XSS |

Each token has an intended audience (`aud` claim) and purpose — an API that accepts an ID token as if it were an access token is validating the wrong thing.

### MFA Policy

- MFA enrollment and enforcement policies can be scoped per application, per user group, or per risk signal (adaptive MFA) — a policy that only covers some login paths (e.g., the web app but not an API client credential flow, or SSO but not a legacy direct-login fallback) leaves the uncovered path as a bypass
- "MFA enabled" at the org level doesn't guarantee every actual authentication flow enforces it — each app/flow's policy binding needs to be checked individually

### Role/Group-Based Authorization

- Identity providers typically sync or assert group/role membership, which the application maps to its own authorization model
- This mapping introduces a **sync/propagation delay** — a role change in the IdP doesn't always take effect instantly in the application, depending on whether authorization is checked per-request (fresh) or cached in a long-lived session/token

## Best Practices

1. **Use access tokens for API authorization, ID tokens only for identifying the user to the client** — never let an API accept an ID token as if it were an access token
2. **Keep access/ID token lifetimes short**, relying on refresh tokens (stored securely, ideally httpOnly cookies or a secure backend session) for longevity
3. **Validate the `aud` (audience) and `iss` (issuer) claims** on every token server-side — don't just check the signature
4. **Enforce MFA consistently across every actual authentication path**, including API/service-to-service flows and any legacy fallback logins, not just the primary web SSO flow
5. **Strictly validate redirect URIs** — exact-match allowlisting, not prefix or wildcard matching, to prevent open-redirect/token-leak attacks
6. **Treat role/group sync lag as a real design constraint** — decide deliberately whether authorization checks are fresh-per-request or cached, and how stale a cached decision can safely be

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| API validates an ID token as if it were an access token | Issue and validate access tokens (with the correct `aud`) specifically for API authorization |
| Long-lived access tokens (hours/days) instead of short-lived + refresh | Keep access tokens short-lived; use refresh tokens for longevity, stored securely |
| Wildcard or prefix-matched redirect URI allowlists | Exact-match allowlist every valid redirect URI |
| MFA enforced on the main SSO login but not on a service-account/API flow | Audit and enforce MFA (or equivalent strong auth) across every actual auth path |
| Application trusts a cached role/group claim indefinitely | Set a deliberate freshness bound on cached authorization data, matched to how sensitive the access is |

## Sharp Edges

### SE-1: Confusing ID Tokens With Access Tokens
- **Severity**: critical
- **Situation**: An API accepts and validates an ID token as its bearer credential (because it "has the user's info in it and looks like a JWT"), when ID tokens were never designed to authorize API access — this can lead to accepting tokens that weren't scoped or intended for that API at all
- **Cause**: Both are JWTs and superficially similar, but they have different intended audiences and validation semantics — an ID token's `aud` claim is the *client application*, not the API
- **Symptoms**:
  - A security review finds an API validating tokens without checking the `aud` claim matches itself
  - Tokens minted for one client/purpose are accepted by an unrelated API
- **Solution**: Use access tokens (with the API as the intended `aud`) for API authorization, validate `aud` and `iss` explicitly server-side on every request, and treat ID tokens as client-side-only, never sent to or validated by a backend API
- **Details**: → [extended/checklists.md#token-handling-checklist]

### SE-2: Long-Lived Tokens Increasing Leak Blast Radius
- **Severity**: high
- **Situation**: Access tokens (or sessions) are configured with very long lifetimes for convenience, and a leaked token (via a logged request, a compromised client, or XSS) remains valid and exploitable for a correspondingly long window
- **Cause**: Longer token lifetimes reduce how often users need to re-authenticate, which is convenient, but directly increase the window during which a leaked token is usable by an attacker
- **Symptoms**:
  - A security incident's remediation requires manually revoking sessions because tokens can't simply expire soon on their own
  - Token lifetime configuration was set for UX convenience without an explicit risk tradeoff discussion
- **Solution**: Keep access/ID token lifetimes short (minutes to a couple hours, depending on sensitivity), use refresh tokens for longevity, store refresh tokens securely (httpOnly cookies, secure backend session storage — never client-side JS-accessible storage), and support token revocation for incident response

### SE-3: Redirect URI Validation Misconfiguration
- **Severity**: critical
- **Situation**: A redirect URI allowlist uses prefix or wildcard matching (e.g., `https://app.example.com/*`) instead of exact matching, and an attacker registers or finds an open redirect within that prefix, using it to capture the authorization code/token intended for the legitimate app
- **Cause**: OAuth/OIDC flows redirect the authorization response (code or token) to the registered redirect URI — if that matching is loose, an attacker-controlled path under the same prefix can receive it instead
- **Symptoms**:
  - A security audit or penetration test finds a redirect URI configuration accepting more than the exact intended callback path
- **Solution**: Configure exact-match redirect URI allowlisting (no wildcards, no prefix matching) for every registered application, and periodically audit the actual configured URIs against what's genuinely in use

### SE-4: MFA Policy Gaps Across Authentication Paths
- **Severity**: high
- **Situation**: MFA is enforced for the primary web SSO login flow, but a service-account/API client-credential flow, a legacy direct-login fallback, or a secondary application integration doesn't have the same policy applied — giving an attacker a path that bypasses MFA entirely
- **Cause**: MFA policies in most identity platforms are bound per-application or per-flow, not globally enforced by default — adding a new app/integration without explicitly binding the MFA policy leaves it uncovered
- **Symptoms**:
  - A security review finds one or more registered applications/flows without an MFA policy applied
  - An account compromise occurs through a path that bypassed MFA despite MFA being "enabled" at the org level
- **Solution**: Audit every registered application and authentication flow individually for MFA policy coverage, not just the primary login screen, and eliminate legacy no-MFA fallback flows where possible

### SE-5: Role/Group Sync Lag Causing Stale Authorization
- **Severity**: medium
- **Situation**: A user's access is revoked or downgraded in the identity provider (e.g., removed from an admin group), but the application continues honoring their old permissions because it cached the role/group claim in a long-lived session or token that hasn't refreshed yet
- **Cause**: Role/group information is often embedded in a token or session at authentication time; if the application doesn't re-check it per-request or on a short refresh cycle, a revocation doesn't take effect until the next full re-authentication
- **Symptoms**:
  - A user retains access to sensitive functionality for a noticeable window after being removed from the corresponding group
  - Offboarding/access-revocation incident response takes longer than expected to actually take effect
- **Solution**: For sensitive authorization decisions, check role/group membership fresh (or with a short cache TTL) rather than trusting a long-lived cached claim, and understand exactly how long a revocation takes to propagate in the current design so it can be communicated during incident response

## Recommended Tools

| Category | Tools |
|------|------|
| Token debugging | jwt.io, Okta/Auth0 debugger tools |
| Standards | OpenID Connect, OAuth 2.0, SAML 2.0 |
| SDKs | Okta SDKs, Auth0 SDKs (per-language/framework) |
| Testing | OWASP Authentication Testing Guide |

## Token Handling

**Full checklist**: → [extended/checklists.md#token-handling-checklist]

## Related Resources

[Okta Developer Docs](https://developer.okta.com/docs/) | [Auth0 Docs](https://auth0.com/docs) | [OpenID Connect Explained](https://openid.net/developers/how-connect-works/)

## Related Domains

[[vault]] | [[aws]] | [[javascript]] | [[python]]
