# Domain 2 — ID Token / JARM Encryption

[← Back to PQC Overview](pqc_overview.md)

**ML-KEM implementation plan**  
**New issue needed under:** [#48821](https://github.com/keycloak/keycloak/issues/48821)  
**Gaps:** GAP-4  
**Category:** PQC Readiness Gap Analysis

---

## What is this domain?

When a client registers with Keycloak and requests **encrypted ID tokens** (or encrypted JARM authorization response JWTs), Keycloak must encrypt the token before sending it. JWE (JSON Web Encryption) works in two layers:

- The token content is encrypted with a fast symmetric key called the **CEK** (Content Encryption Key)
- That CEK is then **wrapped/encrypted using the client's public key** — this is the asymmetric step

Domain 2 is entirely about that second step — the **asymmetric key-wrapping of the CEK**. Today Keycloak does this with RSA-OAEP or ECDH-ES, both of which are quantum-vulnerable. The PQC replacement is **ML-KEM** (FIPS 203, formerly Kyber), which performs key *encapsulation* rather than key wrapping — the client encapsulates a shared secret using its public key, and Keycloak decapsulates it using its private key to recover the CEK. Without ML-KEM support, any client requesting an encrypted token has no quantum-safe option.

---

## Current PQC State

⛔ **BLOCKED**

Zero ML-KEM code exists anywhere in the codebase. No constants, no `JWEAlgorithmProvider`, no `CekManagementProviderFactory`, no key provider. This is a ground-up implementation — unlike Domain 1 where JWK infrastructure was already partially in place, Domain 2 has no existing groundwork beyond BouncyCastle 1.84 providing the JCA implementation.

### No existing groundwork

Confirmed by codebase search: zero matches for `ML.KEM`, `ML_KEM`, or `mlkem` across all Java source files. `CryptoConstants.java`, `JWEConstants.java`, `Algorithm.java`, and `DefaultCryptoProvider.java` all have no ML-KEM entries. BouncyCastle 1.84 provides `org.bouncycastle.jcajce.provider.asymmetric.mlkem.*` — the crypto layer is available but nothing in Keycloak wires it.

---

## GitHub Issue

⚠️ **No existing issue covers this specifically.**

A new issue should be created as a sub-issue under [#48821](https://github.com/keycloak/keycloak/issues/48821) (PQC support for OAuth 2.0 and OpenID Connect).

**Proposed title:** Implement ML-KEM `CekManagementProviderFactory` for encrypted ID token / JARM support (FIPS 203)

**Proposed description:** Keycloak has no `CekManagementProviderFactory` implementation for ML-KEM (FIPS 203). All existing CEK management providers use RSA-OAEP or ECDH-ES. Clients requesting encrypted ID tokens or encrypted JARM authorization responses have no quantum-safe key-encapsulation option. This issue tracks implementing `MLKemJWEAlgorithmProvider`, `MLKemCekManagementProvider`, and three `MLKemCekManagementProviderFactory` classes (one per ML-KEM parameter set: 512, 768, 1024), registering them in `DefaultCryptoProvider` and the `CekManagementProviderFactory` service file. Depends on ML-KEM realm key generation. BouncyCastle 1.84 provides the JCA implementation.

---

## Required Changes — Summary

| # | File / Location | Change |
|---|-----------------|--------|
| 1 | `common/.../crypto/CryptoConstants.java` | Add ML-KEM algorithm name constants |
| 2 | `core/.../jose/jwe/JWEConstants.java` | Expose ML-KEM constants (derive from `CryptoConstants`) |
| 3 | `crypto/default/.../def/MLKemJWEAlgorithmProvider.java` | New class — implements `JWEAlgorithmProvider` using ML-KEM encapsulation / decapsulation |
| 4 | `crypto/default/.../def/DefaultCryptoProvider.java` | Register `MLKemJWEAlgorithmProvider` for each ML-KEM parameter set |
| 5 | `services/.../crypto/MLKemCekManagementProvider.java` | New class — implements `CekManagementProvider` |
| 6 | `services/.../crypto/MLKem{512,768,1024}CekManagementProviderFactory.java` | New classes (×3) — one factory per ML-KEM parameter set |
| 7 | `META-INF/services/...CekManagementProviderFactory` | Register the three new factories |
| 8 | ML-KEM realm `KeyProvider` | Covered by Domain 29 — realm must be able to hold an ML-KEM ENC key for decapsulation |

---

## 1. `common/src/main/java/org/keycloak/common/crypto/CryptoConstants.java`

Add ML-KEM algorithm name constants alongside the existing JWE algorithm strings:

```java
// ML-KEM Key Encapsulation (FIPS 203)
public static final String ML_KEM_512  = "ML-KEM-512";
public static final String ML_KEM_768  = "ML-KEM-768";
public static final String ML_KEM_1024 = "ML-KEM-1024";
```

---

## 2. `core/src/main/java/org/keycloak/jose/jwe/JWEConstants.java`

Expose the new constants from `JWEConstants`, following the same pattern as existing entries:

```java
public static final String ML_KEM_512  = CryptoConstants.ML_KEM_512;
public static final String ML_KEM_768  = CryptoConstants.ML_KEM_768;
public static final String ML_KEM_1024 = CryptoConstants.ML_KEM_1024;
```

---

## 3. New `MLKemJWEAlgorithmProvider`

### `crypto/default/.../def/MLKemJWEAlgorithmProvider.java`

ML-KEM uses *key encapsulation*, not key wrapping. `encodeCek()` calls ML-KEM *encapsulate* against the recipient's public key — the result is an encapsulated key (ciphertext) and a shared secret; the shared secret becomes (or derives) the CEK. `decodeCek()` calls ML-KEM *decapsulate* using the realm's private key to recover the shared secret / CEK.

```java
public class MLKemJWEAlgorithmProvider implements JWEAlgorithmProvider {

    @Override
    public byte[] encodeCek(JWEEncryptionProvider encryptionProvider,
                            JWEKeyStorage keyStorage,
                            Key recipientPublicKey,
                            JWEHeaderBuilder headerBuilder) throws Exception {
        // Use BouncyCastle ML-KEM KEM via JCA
        KeyGenerator kemGen = KeyGenerator.getInstance(recipientPublicKey.getAlgorithm());
        // BC ML-KEM: encapsulate returns {sharedSecret, encapsulation}
        KEMGenerateSpec spec = new KEMGenerateSpec((PublicKey) recipientPublicKey, "AES");
        kemGen.init(spec);
        SecretKeyWithEncapsulation result =
            (SecretKeyWithEncapsulation) kemGen.generateKey();

        // The shared secret becomes the CEK
        keyStorage.setCEKBytes(result.getEncoded());

        // Return the encapsulation (ciphertext) — stored in the JWE encrypted_key field
        return result.getEncapsulation();
    }

    @Override
    public byte[] decodeCek(byte[] encapsulation,
                            Key recipientPrivateKey,
                            JWEHeader header,
                            JWEEncryptionProvider encryptionProvider) throws Exception {
        // Decapsulate using realm private ML-KEM key
        KeyAgreement kem = KeyAgreement.getInstance("ML-KEM");
        kem.init(recipientPrivateKey);
        kem.doPhase(/* encapsulation key */ null, true);
        // BC API: use decapsulate to recover shared secret
        SecretKey sharedSecret = kem.generateSecret(
            new KEMExtractSpec((PrivateKey) recipientPrivateKey, encapsulation, "AES"));
        return sharedSecret.getEncoded();
    }
}
```

**Note:** The exact BouncyCastle JCA API for ML-KEM encapsulation/decapsulation uses `KEMGenerateSpec` and `KEMExtractSpec` from `org.bouncycastle.jcajce.spec`. The pseudocode above shows the intent; the precise call sequence should be validated against the BC 1.84 API and the IETF draft-irtf-cfrg-hpke / JOSE ML-KEM spec for the correct KDF step that derives the CEK from the ML-KEM shared secret.

---

## 4. Register in `DefaultCryptoProvider`

### `crypto/default/.../def/DefaultCryptoProvider.java`

Add three entries to the `providers` map in the constructor, after the existing ECDH-ES entries (lines 62–65):

```java
providers.put(CryptoConstants.ML_KEM_512,  new MLKemJWEAlgorithmProvider());
providers.put(CryptoConstants.ML_KEM_768,  new MLKemJWEAlgorithmProvider());
providers.put(CryptoConstants.ML_KEM_1024, new MLKemJWEAlgorithmProvider());
```

**Note:** The same pattern must be applied to `crypto/fips1402/.../FIPS1402Provider.java` once BC-FIPS includes ML-KEM support (currently blocked — see GAP-6). No change to the FIPS provider now.

---

## 5. New `MLKemCekManagementProvider` + factories

### `services/.../crypto/MLKemCekManagementProvider.java`

Analogous to `RsaCekManagementProvider` and `EcdhEsCekManagementProvider`. Returns the ML-KEM `JWEAlgorithmProvider` from the crypto provider:

```java
public class MLKemCekManagementProvider implements CekManagementProvider {

    private final KeycloakSession session;
    private final String jweAlgorithmName;

    public MLKemCekManagementProvider(KeycloakSession session, String jweAlgorithmName) {
        this.session = session;
        this.jweAlgorithmName = jweAlgorithmName;
    }

    @Override
    public JWEAlgorithmProvider jweAlgorithmProvider() {
        if (JWEConstants.ML_KEM_512.equals(jweAlgorithmName)
                || JWEConstants.ML_KEM_768.equals(jweAlgorithmName)
                || JWEConstants.ML_KEM_1024.equals(jweAlgorithmName)) {
            return CryptoIntegration.getProvider()
                .getAlgorithmProvider(JWEAlgorithmProvider.class, jweAlgorithmName);
        }
        return null;
    }
}
```

### `services/.../crypto/MLKem512CekManagementProviderFactory.java`

Following the pattern of `RsaesOaepCekManagementProviderFactory`. Repeat for `MLKem768` and `MLKem1024` substituting `JWEConstants.ML_KEM_768` / `JWEConstants.ML_KEM_1024`:

```java
public class MLKem512CekManagementProviderFactory implements CekManagementProviderFactory {

    public static final String ID = JWEConstants.ML_KEM_512;

    @Override
    public String getId() { return ID; }

    @Override
    public CekManagementProvider create(KeycloakSession session) {
        return new MLKemCekManagementProvider(session, ID);
    }
}
```

---

## 7. Register `CekManagementProviderFactory` services

Append to `services/src/main/resources/META-INF/services/org.keycloak.crypto.CekManagementProviderFactory`:

```
org.keycloak.crypto.MLKem512CekManagementProviderFactory
org.keycloak.crypto.MLKem768CekManagementProviderFactory
org.keycloak.crypto.MLKem1024CekManagementProviderFactory
```

---

## 8. ML-KEM Realm Key Provider (Domain 29)

For Keycloak to *decapsulate* an ML-KEM encrypted token (i.e. when Keycloak itself is the recipient), the realm must hold an ML-KEM private key registered for `KeyUse.ENC`. This is covered by **Domain 29** (Generated RSA Encryption Key Provider equivalent for ML-KEM) and is a prerequisite for the inbound decapsulation path. The outbound encapsulation path (Keycloak encrypting to a client's ML-KEM public key fetched from the client's JWKS) does not require a realm key — it uses the client's public key directly.

---

## What Does NOT Need Changing

✅ **No changes needed:**
- `DefaultTokenManager` — the JWE encryption path calls `CekManagementProvider` via SPI; no change needed once factories are registered
- `CekManagementProvider` / `CekManagementProviderFactory` SPI interfaces — already sufficient
- `JWEKeyStorage`, `JWE`, `JWEHeader` — content-agnostic; no change needed
- Content encryption (AES-GCM, AES-CBC) — unaffected; only the key-wrapping layer changes

---

## Dependencies

⚠️ **Dependencies:**
- **BouncyCastle 1.84** (non-FIPS) — confirmed to include `org.bouncycastle.jcajce.provider.asymmetric.mlkem.*`. Available now.
- **BC-FIPS** — ML-KEM not present in BC-FIPS 2.1.2. FIPS deployments remain blocked until a future BC-FIPS release. No change to the FIPS provider until then.
- **Domain 29** — ML-KEM realm `KeyProvider` is required before Keycloak can act as the *recipient* of an ML-KEM encrypted token. The outbound path (encrypting to a client) does not depend on Domain 29.
- **IETF JOSE ML-KEM spec** — the JWE algorithm identifier strings (`"ML-KEM-512"` etc.) and the KDF step between the ML-KEM shared secret and the CEK should be validated against the current IETF draft (draft-ietf-jose-json-web-algorithms-pqc or equivalent) before finalising.

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness