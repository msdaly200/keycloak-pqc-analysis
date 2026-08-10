# Domain 34 — Java Keystore Key Provider (PKCS12 / BCFKS)

[← Back to PQC Overview](pqc_overview.html)

## What is this?

Allows operators to import keys from Java keystores (JKS, PKCS12, or BCFKS format) into Keycloak realms. BCFKS (BouncyCastle FIPS KeyStore) is the recommended format for storing PQC keys.

**How it works:**

1. Operator generates keys externally (e.g., using `keytool` with BouncyCastle provider)
2. Keys are stored in a keystore file (PKCS12 or BCFKS)
3. Operator configures Java Keystore key provider in Keycloak admin console
4. Keycloak loads keys from the keystore file
5. Keys are used for signing/encryption operations

## Gap

**Two gaps prevent ML-DSA keystore import (GAP-5, GAP-16):**

### GAP-5: Import logic missing AKP support

`JavaKeystoreKeyProviderFactory` currently supports:
- **RSA** keys
- **EC** (ECDSA/ECDH) keys
- **OKP** (EdDSA) keys

The import logic has no **AKP** (ML-DSA) branch. Even though BCFKS keystores can physically store ML-DSA keys, Keycloak won't load them.

### GAP-16: Admin UI missing ML-DSA algorithms

`mergedAlgorithmProperties()` builds the algorithm dropdown shown in the admin UI. ML-DSA algorithms are absent from this list, so operators cannot configure ML-DSA keystore providers even if GAP-5 were fixed.

## Current PQC State

**BLOCKED**

Both the backend import logic (GAP-5) and frontend configuration (GAP-16) must be fixed before operators can import ML-DSA keys from keystores.

## Required Changes

### Change 1: Add AKP import logic (GAP-5)

**File:** `JavaKeystoreKeyProvider.java`

Add support for loading AKP (ML-DSA) keys from keystores alongside existing RSA/EC/OKP support.

### Change 2: Add ML-DSA to admin UI (GAP-16)

**File:** `JavaKeystoreKeyProviderFactory.java`

Update `mergedAlgorithmProperties()` to include ML-DSA algorithm options (ML-DSA-44, ML-DSA-65, ML-DSA-87) in the admin UI dropdown.

## Dependencies

1. **BouncyCastle ML-DSA keystore support** — already available (BCFKS can store ML-DSA keys)
2. **`keytool` with BouncyCastle provider** — operators need this to generate ML-DSA keys externally
3. **GAP-5 + GAP-16 fixes** — tracked under [#50679](https://github.com/keycloak/keycloak/issues/50679)

## GitHub Issue Status

**Tracked under [#50679](https://github.com/keycloak/keycloak/issues/50679)** — "Support loading ML-DSA keys from Java keystores"

This issue covers both GAP-5 (import logic) and GAP-16 (admin UI) for Domains 33 and 34.

## What this means for operators

**Today:**

Cannot import ML-DSA keys from keystores, even though BCFKS format supports storing them.

**When GAP-5 + GAP-16 are fixed:**

```bash
# Example workflow (once fixed):
# 1. Generate ML-DSA key externally
keytool -genkeypair -alias ml-dsa-key \
  -keyalg ML-DSA-65 \
  -keystore mykeystore.bcfks \
  -storetype BCFKS \
  -provider org.bouncycastle.jcajce.provider.BouncyCastleFipsProvider

# 2. Configure Java Keystore provider in Keycloak admin console:
#    - Upload mykeystore.bcfks
#    - Select algorithm: ML-DSA-65
#    - Enter keystore password
#    - Enter key alias

# 3. Keycloak loads and uses the ML-DSA key
```

## Related Domains

- **Domain 33** — Imported RSA Signing Keys (same GAP-5 + GAP-16)
- **Domain 28** — Generated RSA Signing Keys (alternative: generate instead of import)
- **[#50680](https://github.com/keycloak/keycloak/issues/50680)** — Add test coverage for truststore loading with PQC certificates

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
