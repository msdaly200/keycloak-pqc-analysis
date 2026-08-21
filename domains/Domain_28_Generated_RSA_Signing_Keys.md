# Domain 28 — Generated RSA Signing Key Provider

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

Realm key providers are responsible for generating and managing cryptographic keys used by Keycloak for signing tokens, credentials, and other artifacts. When a new realm is created, Keycloak automatically generates an RSA key pair for signing using the `GeneratedRsaKeyProviderFactory`.

**How it works:**

1. Realm is created or key provider is configured
2. `GeneratedRsaKeyProviderFactory` generates an RSA-2048 (or RSA-4096) key pair
3. The public key is exposed via JWKS endpoints for verifiers
4. The private key is used by `SignatureProvider` to sign tokens
5. Keys automatically rotate based on configured rotation period

**Key types in Keycloak:**
- **SIG (Signature)** — for signing JWTs, credentials, SAML assertions
- **ENC (Encryption)** — for encrypting JWE tokens, SAML assertions

Domain 28 covers **SIG** key generation (signing keys).

## Gap

**No ML-DSA key generation provider exists (GAP-15).**

### Missing: `GeneratedAKPKeyProviderFactory`

Currently, Keycloak has key provider factories for:
- `GeneratedRsaKeyProviderFactory` — RSA-2048/4096
- `GeneratedEcdsaKeyProviderFactory` — ECDSA with P-256/384/521
- `GeneratedEddsaKeyProviderFactory` — EdDSA with Ed25519

There is **no equivalent for ML-DSA** (AKP key type). Without this:
- Realms cannot auto-generate ML-DSA keys
- Operators must manually import ML-DSA keys from external sources
- The `DefaultKeyManager.createFallbackKeys()` mechanism won't create ML-DSA keys

### CryptoProvider.getKeyPairGen() gap

`CryptoProvider.getKeyPairGen(String algorithm)` must support ML-DSA algorithm names:
- `ML-DSA-44`
- `ML-DSA-65`
- `ML-DSA-87`

BouncyCastle already supports ML-DSA key generation, but Keycloak's `CryptoProvider` abstraction needs to be extended to invoke it.

## Current PQC State

**BLOCKED**

ML-DSA signing cannot work end-to-end until realms can generate ML-DSA keys. This is the **foundational gap** for all ML-DSA signing domains.

**GAP-15 is HIGH priority** because it blocks all other ML-DSA work from being complete.

## Required Changes

### Change 1: Implement `GeneratedAKPKeyProviderFactory`

**File:** Create new class `keys/GeneratedAKPKeyProviderFactory.java`

**Pattern:** Follow the structure of `GeneratedEcdsaKeyProviderFactory.java` or `GeneratedEddsaKeyProviderFactory.java`

**Key methods to implement:**
1. `getId()` — return `"ml-dsa"` or similar provider ID
2. `getHelpText()` — return "Generates ML-DSA key pairs for signing"
3. `create(KeycloakSession, ComponentModel)` — return an `AbstractAKPKeyProvider` instance
4. `getConfigProperties()` — expose configuration options:
   - ML-DSA variant (ML-DSA-44, ML-DSA-65, ML-DSA-87)
   - Key rotation period
   - Priority

### Change 2: Implement `AbstractAKPKeyProvider`

**File:** Create new class `keys/AbstractAKPKeyProvider.java`

**Pattern:** Follow the structure of `AbstractRsaKeyProvider.java`

**Key methods:**
- `generateKeyPair(String algorithm)` — call `CryptoProvider.getKeyPairGen()` with ML-DSA algorithm
- `createKeyWrapper()` — wrap the ML-DSA key pair in a `KeyWrapper`
- Support key rotation, passive keys, etc.

### Change 3: Update `CryptoProvider.getKeyPairGen()`

**File:** `common/crypto/CryptoProvider.java` (and BouncyCastle implementation)

**Add support for:**
```java
if (algorithm.startsWith("ML-DSA")) {
    // Return BouncyCastle ML-DSA KeyPairGenerator
    return KeyPairGenerator.getInstance(algorithm, "BC");
}
```

### Change 4: Update `DefaultKeyManager.createFallbackKeys()`

**File:** `DefaultKeyManager.java`

**Current behavior:** Auto-generates RSA keys if no keys exist

**Proposed behavior:** 
- Option A: Generate both RSA (for backward compat) and ML-DSA keys
- Option B: Make the fallback algorithm configurable
- Option C: Generate ML-DSA only if realm default algorithm is ML-DSA

### Change 5: Update Admin UI

Add "ML-DSA" as an option when configuring realm key providers in the admin console.

## What does NOT need changing

| Component | Why it's already PQC-ready (once provider exists) |
|-----------|---------------------------------------------------|
| Key storage | `ComponentModel` stores keys as opaque data - algorithm-agnostic |
| JWKS export | Exports any key type as JWK - will export AKP keys once they exist |
| Key rotation logic | Based on timestamps, not algorithms |
| Active/passive key selection | Algorithm-agnostic - works with any `KeyWrapper` |

## Dependencies

1. **BouncyCastle ML-DSA support** — BouncyCastle 1.78+ already has this (Keycloak uses BC 1.79)
2. **JDK ML-DSA support** — Optional; Keycloak can use BouncyCastle provider instead of JCA
3. **`SignatureProviderFactory` for ML-DSA** — Must exist to use the generated keys (tracked under [#48824](https://github.com/keycloak/keycloak/issues/48824))

## GitHub Issue Status

**GAP-15** is tracked under [#48824](https://github.com/keycloak/keycloak/issues/48824).

This is the **foundational issue** for ML-DSA signing support. Until ML-DSA key generation exists, all other ML-DSA signing domains (1, 3, 5, 6, 9, 10, etc.) cannot work end-to-end.

## What this means for operators

**Today:**

Realms can only auto-generate RSA, ECDSA, or EdDSA keys. ML-DSA keys must be manually imported from external sources (if supported at all).

**When GAP-15 is fixed:**

1. **Configure ML-DSA key provider** in realm settings
2. **Select ML-DSA variant** (ML-DSA-44, ML-DSA-65, or ML-DSA-87)
3. **Keycloak auto-generates** ML-DSA key pairs
4. **Keys automatically rotate** based on configured rotation period
5. **All signing domains** (tokens, credentials, etc.) can use ML-DSA automatically

**Migration path:**

- **New realms:** Configure ML-DSA key provider from the start
- **Existing realms:** Add ML-DSA key provider alongside existing RSA keys, then gradually migrate clients to ML-DSA (hybrid deployment)

## Related Domains

- **Domain 1** — Access Token / ID Token Signing (will use generated ML-DSA keys)
- **Domain 29** — Generated RSA Encryption Keys (similar gap for ML-KEM encryption keys)
- **Domain 33-34** — Keystore import (separate gaps for importing ML-DSA keys from BCFKS/PKCS12)
- **All PENDING PROVIDERS domains** — depend on ML-DSA keys existing before they can work

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
