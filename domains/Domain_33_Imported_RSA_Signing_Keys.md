# Domain 33 — Imported RSA Signing Key Provider

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

Allows operators to import externally-generated RSA signing keys into Keycloak realms. This is used when keys are generated outside Keycloak (e.g., in hardware security modules, external key management systems, or manually created keystores).

## Gap

**No ML-DSA key import path (GAP-5, GAP-16).**

### GAP-5: Import logic missing

`ImportedRsaKeyProviderFactory` and related keystore providers support:
- RSA keys
- ECDSA keys  
- EdDSA (OKP) keys

But **not ML-DSA (AKP) keys**. Even though BCFKS and PKCS12 keystores can physically store ML-DSA keys, Keycloak's import logic has no AKP branch.

### GAP-16: Admin UI missing

Even if GAP-5 is fixed, the admin UI doesn't expose ML-DSA algorithm options for keystore-based providers. Both the backend import logic and frontend configuration must be fixed together.

## Current PQC State

**BLOCKED**

Operators cannot import ML-DSA keys from keystores.

## Required Changes

See **Domain 34** for detailed implementation plan. Domains 33 and 34 share the same gaps (GAP-5, GAP-16) and will be fixed together.

## Dependencies

1. **BouncyCastle ML-DSA keystore support** — already available
2. **GAP-5 fix** — add AKP import logic ([#50679](https://github.com/keycloak/keycloak/issues/50679))
3. **GAP-16 fix** — add ML-DSA to admin UI algorithm list ([#50679](https://github.com/keycloak/keycloak/issues/50679))

## GitHub Issue Status

**Tracked under [#50679](https://github.com/keycloak/keycloak/issues/50679)** — "Support loading ML-DSA keys from Java keystores"

This issue covers both GAP-5 (import logic) and GAP-16 (admin UI) for Domains 33 and 34.

## What this means for operators

**Today:**

Operators who generate ML-DSA keys externally (e.g., via `keytool` or HSM) cannot import them into Keycloak.

**Workaround:** None. Must use generated keys (Domain 28) once GAP-15 is fixed.

**When GAP-5 + GAP-16 are fixed:**

1. Generate ML-DSA keys externally and store in BCFKS or PKCS12 keystore
2. Configure Java Keystore key provider in Keycloak
3. Select ML-DSA algorithm from admin UI dropdown
4. Import the keystore file
5. Keycloak loads and uses the ML-DSA key for signing

## Related Domains

- **Domain 34** — Java Keystore Key Provider (same GAP-5 + GAP-16, detailed implementation plan)
- **Domain 28** — Generated RSA Signing Keys (alternative: generate instead of import)

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
