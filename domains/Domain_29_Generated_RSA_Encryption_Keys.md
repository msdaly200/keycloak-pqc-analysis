# Domain 29 — Generated RSA Encryption Key Provider

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

Realm key providers generate RSA key pairs for **encryption** (KeyUse.ENC). These keys are used for JWE (JSON Web Encryption) operations like encrypting ID tokens, JARM responses, and other encrypted JWTs.

**How it works:**

1. Realm is created or encryption key provider is configured
2. `GeneratedRsaEncKeyProviderFactory` generates an RSA-2048 (or RSA-4096) key pair
3. The public key is exposed via JWKS endpoints for clients to encrypt to
4. The private key is used to decrypt inbound JWE tokens
5. Keys automatically rotate based on configured rotation period

**Encryption vs Signing keys:**
- **SIG keys** (Domain 28) — ML-DSA for signing
- **ENC keys** (Domain 29) — ML-KEM for encryption (this domain)

## Gap

**No ML-KEM key generation provider exists.**

### Missing: ML-KEM Key Generation Provider

Currently, Keycloak has encryption key provider factories for:
- `GeneratedRsaEncKeyProviderFactory` — RSA-2048/4096 for RSA-OAEP
- `ImportedRsaEncKeyProviderFactory` — Import existing RSA ENC keys

There is **no equivalent for ML-KEM**. Without this:
- Realms cannot auto-generate ML-KEM keys for JWE encryption
- Operators must manually import ML-KEM keys from external sources (if supported at all)
- JWE encryption cannot use quantum-safe key encapsulation

**This gap is separate from GAP-15:**
- **GAP-15** covers ML-DSA (signing) key generation
- **Domains 29, 32** cover ML-KEM (encryption) key generation

Both are foundational gaps that must be fixed before PQC readiness is complete.

## Current PQC State

**BLOCKED**

ML-KEM encryption cannot work end-to-end until realms can generate ML-KEM keys. This blocks:
- **Domain 2** — ID Token / JARM Encryption (outbound JWE)
- **Domain 8** — JAR Encrypted Request Object (inbound JWE)
- **Domain 4** — UserInfo Endpoint (optional encryption)

## Required Changes

### Change 1: Implement ML-KEM Key Generation Provider

**File:** Create new class `keys/GeneratedMlKemKeyProviderFactory.java` (or similar naming)

**Pattern:** Follow the structure of `GeneratedRsaEncKeyProviderFactory.java`

**Key methods to implement:**
1. `getId()` — return `"ml-kem"` or similar provider ID
2. `getHelpText()` — return "Generates ML-KEM key pairs for encryption"
3. `create(KeycloakSession, ComponentModel)` — return an ML-KEM key provider instance
4. `getConfigProperties()` — expose configuration options:
   - ML-KEM variant (ML-KEM-512, ML-KEM-768, ML-KEM-1024)
   - Key rotation period
   - Priority

### Change 2: Update `CryptoProvider` for ML-KEM

**File:** `common/crypto/CryptoProvider.java` (and BouncyCastle implementation)

**Add support for:**
- ML-KEM key pair generation
- ML-KEM encapsulation (encrypt operation)
- ML-KEM decapsulation (decrypt operation)

**Note:** ML-KEM is different from traditional RSA key exchange:
- RSA uses public key encryption for key transport
- ML-KEM uses **key encapsulation** (KEM) — generates a shared secret + ciphertext

### Change 3: Implement `CekManagementProvider` for ML-KEM

**Related to GAP-4** (Domains 2, 8):

Once ML-KEM keys exist, Keycloak needs a `CekManagementProvider` that can:
- **Encapsulate**: Generate a Content Encryption Key (CEK) and encapsulate it with ML-KEM public key
- **Decapsulate**: Use ML-KEM private key to recover the CEK from the ciphertext

This is tracked under **GAP-4** separately.

### Change 4: Update `DefaultKeyManager.createFallbackKeys()`

**File:** `DefaultKeyManager.java`

Ensure fallback key generation creates both:
- ML-DSA keys for signing (GAP-15)
- ML-KEM keys for encryption (this gap)

### Change 5: Update Admin UI

Add "ML-KEM" as an option when configuring realm encryption key providers in the admin console.

## What does NOT need changing

| Component | Why it's already PQC-ready (once provider exists) |
|-----------|---------------------------------------------------|
| Key storage | `ComponentModel` stores keys as opaque data - algorithm-agnostic |
| JWKS export | Exports any key type as JWK - will export ML-KEM keys once they exist |
| Key rotation logic | Based on timestamps, not algorithms |
| Active/passive key selection | Algorithm-agnostic - works with any `KeyWrapper` |

## Dependencies

1. **BouncyCastle ML-KEM support** — BouncyCastle 1.78+ already has this (Keycloak uses BC 1.79)
2. **JDK ML-KEM support** — Optional; can use BouncyCastle provider instead of JCA
3. **`CekManagementProvider` for ML-KEM** — Must exist to use the generated keys (tracked under GAP-4, [#48821](https://github.com/keycloak/keycloak/issues/48821))

## GitHub Issue Status

**Identified but not explicitly tracked:**

The overview table lists **"Domains 29, 32"** as needing ML-KEM key generation, but there's no dedicated GitHub issue yet.

**Recommended new issue:**

Create a sub-issue under [#43690](https://github.com/keycloak/keycloak/issues/43690):
- **Title:** "Implement ML-KEM key generation provider for realm encryption keys"
- **Description:**
  > Similar to GAP-15 (ML-DSA key generation), realms need the ability to auto-generate ML-KEM keys for JWE encryption.
  >
  > **Missing:**
  > - No `GeneratedMlKemKeyProviderFactory` (equivalent of `GeneratedRsaEncKeyProviderFactory` for ML-KEM)
  > - `CryptoProvider` doesn't support ML-KEM key pair generation
  > - `DefaultKeyManager.createFallbackKeys()` doesn't create ML-KEM keys
  >
  > **Required:**
  > 1. Implement ML-KEM key generation provider factory
  > 2. Add ML-KEM support to `CryptoProvider` (key generation, encapsulation, decapsulation)
  > 3. Update fallback key creation
  > 4. Update admin UI to expose ML-KEM key provider
  >
  > **Blocks:** Domains 2, 4, 8 (JWE encryption with ML-KEM)
  >
  > **Related:** GAP-4 (`CekManagementProvider` for ML-KEM), GAP-15 (ML-DSA key generation)
- **Label:** `pqc-readiness`
- **Affects:** Domains 29, 32

## What this means for operators

**Today:**

Realms can only auto-generate RSA encryption keys. ML-KEM keys must be manually created externally (if at all).

**When this gap is fixed:**

1. **Configure ML-KEM key provider** in realm settings
2. **Select ML-KEM variant** (ML-KEM-512, ML-KEM-768, or ML-KEM-1024)
3. **Keycloak auto-generates** ML-KEM key pairs
4. **Keys automatically rotate** based on configured rotation period
5. **JWE encryption domains** (2, 4, 8) can use ML-KEM for quantum-safe encryption

**Security recommendation:** Use **ML-KEM-768** or **ML-KEM-1024** for production deployments. ML-KEM-512 is primarily for constrained environments.

**Migration path:**

- **New realms:** Configure both ML-DSA (signing) and ML-KEM (encryption) key providers from the start
- **Existing realms:** Add ML-KEM key provider alongside existing RSA ENC keys, configure JWE to prefer ML-KEM

## Related Domains

- **Domain 28** — Generated RSA Signing Keys (GAP-15, ML-DSA key generation - the signing equivalent of this domain)
- **Domain 2** — ID Token / JARM Encryption (will use generated ML-KEM keys)
- **Domain 8** — JAR Encrypted Request Object (will use generated ML-KEM keys for decryption)
- **Domain 32** — Imported RSA Encryption Keys (similar gap for importing ML-KEM keys)
- **GAP-4** — ML-KEM `CekManagementProvider` (uses the keys this domain generates)

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
