# Domain 31 — Generated EdDSA Signing Key Provider

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

Realm key providers generate EdDSA (Edwards-curve Digital Signature Algorithm) key pairs for signing tokens. EdDSA uses the Ed25519 curve, which is known for fast signing/verification and small signatures.

**How it works:**

1. Realm is created or EdDSA key provider is configured
2. `GeneratedEddsaKeyProviderFactory` generates an Ed25519 key pair
3. The public key is exposed via JWKS endpoints for verifiers
4. The private key is used by `SignatureProvider` to sign tokens
5. Keys automatically rotate based on configured rotation period

**EdDSA vs ECDSA vs RSA:**
- **RSA** (Domain 28) — Large keys/signatures, widely supported
- **ECDSA** (Domain 30) — NIST curves (P-256/384/521), moderate performance
- **EdDSA** (Domain 31) — Ed25519, fast performance, small signatures
- **ML-DSA** (GAP-15, future) — Quantum-safe, larger signatures than classical

## Gap

**Same as Domain 28 — no ML-DSA key generation provider (GAP-15).**

Domains 28, 30, and 31 are all **classical** signing key providers (RSA, ECDSA, EdDSA). The gap is the same: no **ML-DSA** (quantum-safe) key generation provider exists.

## Current PQC State

**BLOCKED**

Same foundational blocker as Domains 28 and 30. Until `GeneratedAKPKeyProviderFactory` is implemented (GAP-15), realms cannot auto-generate ML-DSA keys.

**Important operational consideration:** ML-DSA signatures are **40-70x larger** than EdDSA signatures. This will significantly increase token sizes and network overhead.

## Required Changes

**None specific to this domain.**

Covered by the same `GeneratedAKPKeyProviderFactory` implementation work as Domain 28. Once GAP-15 is fixed, realms will be able to generate:
- RSA keys (existing, Domain 28)
- ECDSA keys (existing, Domain 30)
- EdDSA keys (existing, Domain 31)
- **ML-DSA keys** (new, GAP-15)

All four options will coexist, allowing operators to choose based on their requirements.

## Dependencies

Same as Domains 28 and 30:
1. **BouncyCastle ML-DSA support** — already available
2. **GAP-15 implementation** — tracked under [#48824](https://github.com/keycloak/keycloak/issues/48824)

## GitHub Issue Status

**GAP-15** tracked under [#48824](https://github.com/keycloak/keycloak/issues/48824).

No separate issue needed for Domain 31 - it shares the same gap as Domains 28 and 30.

## What this means for operators

**Today:**

Operators can choose between three classical signing algorithms:
- **RSA** — Maximum compatibility, large keys
- **ECDSA** — NIST-approved curves, moderate size
- **EdDSA** — Modern, fast, small signatures

All three are quantum-vulnerable.

**When GAP-15 is fixed:**

Operators will have a **fourth option**:
- **ML-DSA** — Quantum-safe, NIST FIPS 204 approved

**Migration consideration:**

EdDSA (Ed25519) is often chosen for performance and small signature size. ML-DSA signatures are **significantly larger**:
- Ed25519 signature: 64 bytes
- ML-DSA-44 signature: ~2,420 bytes
- ML-DSA-65 signature: ~3,309 bytes
- ML-DSA-87 signature: ~4,627 bytes

Operators should plan for increased token sizes when migrating from EdDSA to ML-DSA.

## Related Domains

- **Domain 28** — Generated RSA Signing Keys (same GAP-15)
- **Domain 30** — Generated ECDSA Signing Keys (same GAP-15)
- **All PENDING PROVIDERS domains** — depend on GAP-15 being fixed

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
