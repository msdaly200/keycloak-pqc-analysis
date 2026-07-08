# Domain 21 — OIDC IdP — Token Signature Verification & JWE Decryption

## What is this?

When Keycloak acts as an OIDC/OAuth2 client federating with an external Identity Provider (IdP), it:
- **Inbound:** Verifies signatures on ID tokens and access tokens from the external IdP
- **Inbound:** Decrypts JWE-encrypted tokens using the realm's private ENC key
- **Outbound:** Signs client assertions for `private_key_jwt` authentication to the external IdP

## Gap

**One gap: hardcoded RS256 for outbound client assertions (GAP-10).**

**File:** `AbstractOAuth2IdentityProvider.java` (line 697)
The fallback algorithm for `private_key_jwt` broker assertions is hardcoded to `Algorithm.RS256`. No path exists to select ML-DSA as the outbound default.

## Current PQC State

**PARTIAL**

- **Inbound verification:** SPI-driven, will inherit ML-DSA automatically
- **Outbound signing:** Hardcoded RS256 fallback (GAP-10)

## Required Changes

### Change: Make outbound broker assertion algorithm configurable

**File:** `AbstractOAuth2IdentityProvider.java` (line 697)

**Proposed fix:**
```java
// Read configured algorithm from broker config
String algorithm = config.getClientAuthenticationSigningAlg();
if (algorithm == null) {
    algorithm = Algorithm.RS256; // backward-compatible default
}
```

Add configuration attribute to OIDC IdP broker settings for `client_assertion_signing_alg`.

## Dependencies

1. ML-DSA `SignatureProvider` (#48824)

## GitHub Issue Status

**GAP-10** needs a new issue.

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
