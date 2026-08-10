# Domain 32 — Generated ECDH Encryption Key Provider

[← Back to PQC Overview](pqc_overview.html)

## What is this?

Realm key providers generate ECDH (Elliptic Curve Diffie-Hellman) key pairs for **encryption** (KeyUse.ENC). ECDH uses elliptic curves (P-256, P-384, P-521) for key agreement in JWE encryption with algorithms like ECDH-ES and ECDH-ES+A256KW.

**How it works:**

1. Realm is created or ECDH key provider is configured
2. `GeneratedEcdhKeyProviderFactory` generates an ECDH key pair (e.g., P-256)
3. The public key is exposed via JWKS endpoints for clients to encrypt to
4. The private key is used for key agreement to derive the Content Encryption Key (CEK)
5. Keys automatically rotate based on configured rotation period

**ECDH vs RSA for encryption:**
- **RSA** (Domain 29) — Key transport: encrypt the CEK with RSA public key
- **ECDH** (Domain 32) — Key agreement: derive shared secret via Diffie-Hellman, then derive CEK
- **ML-KEM** (future) — Key encapsulation: generate CEK and encapsulate it with ML-KEM public key

## Gap

**Same as Domain 29 — no ML-KEM encryption key generation provider.**

Domains 29 (RSA ENC) and 32 (ECDH ENC) are both **classical** encryption key providers. The gap is the same: no **ML-KEM** (quantum-safe) key generation provider exists.

**This gap is separate from GAP-15:**
- **GAP-15** covers ML-DSA (signing) key generation (Domains 28, 30, 31)
- **Domains 29, 32** cover ML-KEM (encryption) key generation

## Current PQC State

**BLOCKED**

Same foundational blocker as Domain 29. Until ML-KEM key generation provider is implemented, realms cannot auto-generate ML-KEM keys for JWE encryption.

## Required Changes

**None specific to this domain.**

Covered by the same ML-KEM key generation implementation work as Domain 29. Once that gap is fixed, realms will be able to generate:
- RSA ENC keys (existing, Domain 29)
- ECDH ENC keys (existing, Domain 32)
- **ML-KEM keys** (new, Domains 29/32)

All three options will coexist for encryption key generation.

## Dependencies

Same as Domain 29:
1. **BouncyCastle ML-KEM support** — already available (BC 1.78+)
2. **ML-KEM key generation provider implementation** — needs explicit tracking under [#43690](https://github.com/keycloak/keycloak/issues/43690)
3. **`CekManagementProvider` for ML-KEM** — tracked under GAP-4, [#48821](https://github.com/keycloak/keycloak/issues/48821)

## GitHub Issue Status

**Identified as "Domains 29, 32" but not explicitly tracked.**

Same recommendation as Domain 29: create a sub-issue under [#43690](https://github.com/keycloak/keycloak/issues/43690) for ML-KEM key generation that covers both Domain 29 (RSA replacement) and Domain 32 (ECDH replacement).

No separate issue needed for Domain 32 - it shares the same gap as Domain 29.

## What this means for operators

**Today:**

Operators can choose between two classical encryption algorithms:
- **RSA** (Domain 29) — RSA-OAEP key transport, larger keys
- **ECDH** (Domain 32) — ECDH-ES key agreement, smaller keys, better performance

Both are quantum-vulnerable.

**When ML-KEM key generation is fixed:**

Operators will have a **third option**:
- **ML-KEM** — Quantum-safe key encapsulation, NIST FIPS 203 approved

**Migration consideration:**

ECDH is often chosen for smaller key sizes and better performance compared to RSA. ML-KEM key sizes:
- ECDH P-256 public key: ~65 bytes (uncompressed)
- ML-KEM-512 public key: 800 bytes
- ML-KEM-768 public key: 1,184 bytes
- ML-KEM-1024 public key: 1,568 bytes

ML-KEM public keys are **12-24x larger** than ECDH P-256 keys, which affects JWKS endpoint size.

## Related Domains

- **Domain 29** — Generated RSA Encryption Keys (same ML-KEM generation gap)
- **Domain 2** — ID Token / JARM Encryption (will use generated ML-KEM keys)
- **Domain 8** — JAR Encrypted Request Object (will use generated ML-KEM keys)
- **GAP-4** — ML-KEM `CekManagementProvider` (uses the keys this domain generates)

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
