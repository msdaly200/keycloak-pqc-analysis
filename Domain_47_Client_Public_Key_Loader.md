# Domain 47 — Client Public Key Loader (JWKS URL & Stored Cert)

[← Back to PQC Overview](pqc_overview.html)

## What is this?

Loads client public keys from JWKS URL, inline JWKS string, or stored X.509 certificate. Used for `private_key_jwt` authentication and JAR signature verification.

## Gap

**None** — already algorithm-agnostic.

**File:** `services/src/main/java/org/keycloak/keys/loader/ClientPublicKeyLoader.java`

## Current PQC State

**PENDING PROVIDERS**

**Two loading paths:**

1. **JWKS URL / inline JWKS** → calls `JWKSUtils.getKeyWrappersForUse()`
   - ✅ Already handles AKP (ML-DSA) key types
   - Will work automatically once providers exist

2. **Stored X.509 certificate** → uses `CertPathBuilder`
   - ✅ Algorithm-agnostic certificate parsing
   - Will work automatically once providers exist

**No code changes needed.** End-to-end verification depends on `SignatureProviderFactory` (GAP-15).

## Required Changes

**None.**

## Dependencies

**GAP-15** — ML-DSA SignatureProvider ([#48824](https://github.com/keycloak/keycloak/issues/48824))

## GitHub Issue Status

No dedicated issue needed. Covered by GAP-15.

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
