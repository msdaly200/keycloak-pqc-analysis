# Domain 30 — Generated ECDSA Signing Key Provider

[← Back to PQC Overview](pqc_overview.html)

## What is this?

Realm key providers generate ECDSA key pairs for signing tokens. ECDSA is an alternative to RSA for signing, using elliptic curve cryptography with curves P-256, P-384, or P-521.

**How it works:**

1. Realm is created or ECDSA key provider is configured
2. `GeneratedEcdsaKeyProviderFactory` generates an ECDSA key pair (e.g., P-256)
3. The public key is exposed via JWKS endpoints for verifiers
4. The private key is used by `SignatureProvider` to sign tokens
5. Keys automatically rotate based on configured rotation period

## Gap

**Same as Domain 28 — no ML-DSA key generation provider (GAP-15).**

Both Domain 28 (RSA signing keys) and Domain 30 (ECDSA signing keys) are **classical** signing key providers. The gap is the same: no **ML-DSA** (quantum-safe) key generation provider exists.

## Current PQC State

**BLOCKED**

Same foundational blocker as Domain 28. Until `GeneratedAKPKeyProviderFactory` is implemented (GAP-15), realms cannot auto-generate ML-DSA keys.

## Required Changes

**None specific to this domain.**

Covered by the same `GeneratedAKPKeyProviderFactory` implementation work as Domain 28. Once GAP-15 is fixed, realms will be able to generate:
- RSA keys (existing)
- ECDSA keys (existing)
- EdDSA keys (existing)
- **ML-DSA keys** (new)

## Dependencies

Same as Domain 28:
1. **BouncyCastle ML-DSA support** — already available
2. **GAP-15 implementation** — tracked under [#48824](https://github.com/keycloak/keycloak/issues/48824)

## GitHub Issue Status

**GAP-15** tracked under [#48824](https://github.com/keycloak/keycloak/issues/48824).

No separate issue needed for Domain 30 - it shares the same gap as Domain 28.

## What this means for operators

**Today:**

Operators can choose between RSA, ECDSA, or EdDSA for signing keys, all of which are quantum-vulnerable.

**When GAP-15 is fixed:**

Operators will be able to choose **ML-DSA** as a fourth option alongside RSA/ECDSA/EdDSA.

**Note:** Domains 28, 30, and 31 (EdDSA) are all different **classical** key generation providers that will be joined by one **quantum-safe** provider (ML-DSA) once GAP-15 is fixed.

## Related Domains

- **Domain 28** — Generated RSA Signing Keys (same GAP-15)
- **Domain 31** — Generated EdDSA Signing Keys (same GAP-15)
- **All PENDING PROVIDERS domains** — depend on GAP-15 being fixed

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
