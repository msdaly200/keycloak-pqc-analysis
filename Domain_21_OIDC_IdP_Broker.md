# Domain 21 — OIDC IdP — Token Signature Verification & JWE Decryption

[← Back to PQC Overview](pqc_overview.html)

## What is this?

When Keycloak acts as an OIDC/OAuth2 client federating with an external Identity Provider (IdP), it:
- **Inbound:** Verifies signatures on ID tokens and access tokens from the external IdP
- **Inbound:** Decrypts JWE-encrypted tokens using the realm's private ENC key
- **Outbound:** Signs client assertions for `private_key_jwt` authentication to the external IdP

## Gap

**One gap: hardcoded RS256 fallback for outbound client assertions (GAP-10).**

### Hardcoded RS256 fallback (GAP-10)

**File:** `AbstractOAuth2IdentityProvider.java` (line 697)

```java
String alg = getConfig().getClientAssertionSigningAlg() != null ? getConfig().getClientAssertionSigningAlg() : Algorithm.RS256;
```

**Impact:**

The fallback algorithm for `private_key_jwt` broker assertions is hardcoded to `Algorithm.RS256`. When an OIDC IdP broker is configured to use `private_key_jwt` authentication **without explicitly setting `clientAssertionSigningAlg`**, it will default to RS256 even after ML-DSA providers exist.

**What works:**
- If `clientAssertionSigningAlg` is explicitly configured to `ML-DSA-65`, the broker will use ML-DSA
- The code reads the configuration first, then falls back to RS256

**The gap:**
- No way to change the **default** fallback from RS256 to ML-DSA globally
- Every OIDC IdP broker must be manually configured to use ML-DSA
- This differs from regular OIDC clients (Domain 10), where the realm default can influence algorithm selection

**Note on GAP-10 scope:**

GAP-10 technically includes **two** hardcoded fallbacks in this file:
- **Line 689:** `Algorithm.HS256` for `client_secret_jwt` (symmetric, not PQC-relevant)
- **Line 697:** `Algorithm.RS256` for `private_key_jwt` (asymmetric, PQC-relevant)

This domain focuses on line 697 (asymmetric signing) since HS256 is quantum-safe (symmetric algorithms are not vulnerable to Shor's algorithm).

## Current PQC State

**PARTIAL**

### What already works

1. **Inbound token signature verification** (ID tokens, access tokens from external IdP)
   - Fully SPI-driven via `SignatureProvider`
   - Will automatically support ML-DSA once providers exist
   - Algorithm read from JWT header (`alg` claim)
   - No code changes needed

2. **Inbound JWE decryption** (encrypted tokens from external IdP)
   - Uses realm's private ENC key
   - Algorithm-agnostic at the application layer
   - Currently supports RSA-OAEP and ECDH-ES
   - Will support ML-KEM once Domain 2/8 infrastructure exists

3. **Outbound client assertion signing** (when configured explicitly)
   - If `clientAssertionSigningAlg` is set to `ML-DSA-65` in broker config, it will work
   - Uses `SignatureProvider` SPI (line 698)

### What's blocked

4. **Outbound client assertion signing** (default behavior)
   - Hardcoded RS256 fallback (line 697)
   - No way to change the global default
   - Requires manual configuration per broker to use ML-DSA

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

1. **ML-DSA `SignatureProvider`** (tracked under [#48821](https://github.com/keycloak/keycloak/issues/48821) / [#48824](https://github.com/keycloak/keycloak/issues/48824)) — required before brokers can use ML-DSA for outbound assertions
2. **Configuration mechanism** — need to decide whether to:
   - Make RS256 fallback configurable at the realm level (global default)
   - Or require explicit per-broker configuration (current behavior)

## GitHub Issue Status

**GAP-10** is identified in the overview table but does not have a dedicated GitHub issue.

**Recommended new issue:**

Create a sub-issue under [#43690](https://github.com/keycloak/keycloak/issues/43690):
- **Title:** "OIDC IdP Broker: make RS256 fallback configurable for private_key_jwt assertions"
- **Description:**
  > `AbstractOAuth2IdentityProvider.java` line 697 hardcodes `Algorithm.RS256` as the fallback when `clientAssertionSigningAlg` is not configured. This means OIDC IdP brokers default to RS256 even after ML-DSA providers exist.
  >
  > **Current behavior:**
  > - If `clientAssertionSigningAlg` is explicitly set, that algorithm is used (ML-DSA will work)
  > - If not set, fallback to RS256 (no way to default to ML-DSA)
  >
  > **Proposed fix:**
  > - Option A: Add realm-level configuration for default broker assertion algorithm
  > - Option B: Make the fallback read from a global config instead of hardcoding
  > - Option C: Use the realm's default signature algorithm as the fallback
  >
  > **Impact:** Affects all OIDC IdP brokers using `private_key_jwt` authentication (Domain 21)
- **Label:** `pqc-readiness`
- **Tracks:** GAP-10

## What this means for operators

**Today:**

OIDC IdP broker client assertions use RS256 by default, which is quantum-vulnerable.

**When migrating to PQC:**

1. **Inbound verification:** Automatic — external IdPs can send ML-DSA-signed tokens once Keycloak has ML-DSA providers
2. **Outbound assertions:** Manual configuration required per broker:
   - Edit each OIDC IdP broker configuration
   - Set `Client Assertion Signing Algorithm` to `ML-DSA-65` (or desired variant)
   - Without this, the broker will continue using RS256

**Migration challenge:**

Unlike regular OIDC clients (Domain 10), there's no realm-level default that applies to brokers. Every broker must be individually configured to use ML-DSA.

## Related Domains

- **Domain 10** — private_key_jwt Client Authentication (same authentication method, but for regular clients instead of broker clients; fully SPI-driven)
- **Domain 2** — ID Token / JARM Encryption (JWE decryption path shared with broker inbound token decryption)
- **Domain 17** — JWT Authorization Grant (similar inbound token verification pattern)

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
