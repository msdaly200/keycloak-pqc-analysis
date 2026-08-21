# Domain 8 — JAR: Encrypted Request Object Decryption

[← Back to PQC Overview](../pqc_overview.html)

**ML-KEM implementation plan**  
**Gap:** GAP-20  
**Category:** PQC Readiness Gap Analysis

---

## What is this?

RFC 9101 (JAR) allows a client to **encrypt** its signed request object before sending it
to Keycloak. The client encrypts using Keycloak's public ENC key (found in its JWKS
endpoint), and Keycloak must decrypt it using the corresponding realm private key before
verifying the inner JWS.

The encrypted object is a JWE with:
- `alg` — the key-wrapping algorithm (e.g. `RSA-OAEP`, `ECDH-ES`) — determines how the
  Content Encryption Key (CEK) is unwrapped using Keycloak's private key.
- `enc` — the content encryption algorithm (e.g. `A256GCM`) — symmetric, not
  quantum-vulnerable.

For PQC readiness the concern is the `alg` value: RSA and ECDH-ES are quantum-vulnerable.
The PQC replacement is **ML-KEM** (FIPS 203), which defines a key-encapsulation mechanism
that replaces RSA and ECDH key transport in JWE.

This domain covers the **inbound** direction only. The outbound analogue — Keycloak
encrypting the ID token or JARM response to the client — is Domain 2.

## Gap

**BLOCKED (GAP-20).** Three independent gaps prevent ML-KEM decryption of JAR objects today.

### Gap 1 — No `JWEAlgorithmProvider` for ML-KEM in `CryptoIntegration`

[`JWE.verifyAndDecodeJwe()`](core/src/main/java/org/keycloak/jose/jwe/JWE.java#L223) resolves
the algorithm provider via:

```java
JWERegistry.getAlgProvider(header.getAlgorithm())
// which calls:
CryptoIntegration.getProvider().getAlgorithmProvider(JWEAlgorithmProvider.class, alg)
```

[`JWERegistry`](core/src/main/java/org/keycloak/jose/jwe/JWERegistry.java#L50) delegates
entirely to `CryptoIntegration` for any non-`dir` `alg`. No ML-KEM algorithm name
(`ML-KEM-512`, `ML-KEM-768`, `ML-KEM-1024`, or the JOSE draft identifier) is registered in
the BouncyCastle `CryptoProvider`. Without a `JWEAlgorithmProvider` implementation, calling
`verifyAndDecodeJwe()` on an ML-KEM JWE will throw `IllegalArgumentException: No provider
for alg`.

### Gap 2 — No ML-KEM `CekManagementProviderFactory`

[`OIDCWellKnownProvider`](services/src/main/java/org/keycloak/protocol/oidc/OIDCWellKnownProvider.java#L148)
builds `request_object_encryption_alg_values_supported` by enumerating all registered
`CekManagementProvider` factories:

```java
config.setRequestObjectEncryptionAlgValuesSupported(getSupportedEncryptionAlgorithms());
// which calls:
getSupportedAlgorithms(CekManagementProvider.class, false)
```

There is no `CekManagementProviderFactory` for any ML-KEM identifier. ML-KEM will not
appear in discovery, and client registration will not accept it as a valid
`request_object_encryption_alg` value.

### Gap 3 — No realm ML-KEM ENC key provider

[`DefaultTokenManager.decodeClientJWT()`](services/src/main/java/org/keycloak/jose/jws/DefaultTokenManager.java#L134)
selects the decryption key as follows:

```java
Stream<KeyWrapper> keys = session.keys().getKeysStream(session.getContext().getRealm());
if (kid == null) {
    activeKey = keys.filter(k -> KeyUse.ENC.equals(k.getUse()) && k.getPublicKey() != null)
                    .sorted(...).findFirst();
} else {
    activeKey = keys.filter(k -> KeyUse.ENC.equals(k.getUse()) && k.getKid().equals(kid))
                    .findAny();
}
Key privateKey = activeKey.map(KeyWrapper::getPrivateKey).orElseThrow(...);
jwe.getKeyStorage().setDecryptionKey(privateKey);
```

The filter is `KeyUse.ENC` only — no algorithm-type constraint. This means the key
selection logic itself is **already correct** and will work for any `KeyWrapper` with
`KeyUse.ENC`. However, there is no realm key provider that generates or stores an ML-KEM
key pair. Without an ML-KEM key in the realm's keystore, there is nothing to decrypt with.
This dependency is shared with Domain 29 (Generated RSA Encryption Key Provider for
ML-KEM).

## Required Changes

| # | Location | What to change |
|---|----------|---------------|
| 1 | `crypto/default/` and `crypto/fips1402/` | Implement `JWEAlgorithmProvider` for ML-KEM (key encapsulation/decapsulation using BouncyCastle's ML-KEM implementation) and register it in `CryptoProvider.getAlgorithmProvider()` |
| 2 | `services/src/main/java/org/keycloak/crypto/` | Add `MlKemCekManagementProvider` + `MlKemCekManagementProviderFactory` (one per key size: ML-KEM-512, ML-KEM-768, ML-KEM-1024, or a single factory covering all three) registered in `META-INF/services` for `CekManagementProviderFactory` |
| 3 | Realm key provider (Domain 29) | Add a Generated ML-KEM Encryption Key Provider so the realm holds an ML-KEM key pair with `KeyUse.ENC`; this also covers the JWKS endpoint advertising the public key |
| 4 | `OIDCConfigAttributes` / client registration | No change needed — per-client `request_object_encryption_alg` is a free-form string attribute; it will accept ML-KEM names once validation is relaxed or ML-KEM names are recognised by the `CekManagementProvider` SPI |

### Change 1 detail — `JWEAlgorithmProvider` for ML-KEM

New class (e.g. `MlKemJWEAlgorithmProvider`):

```java
public class MlKemJWEAlgorithmProvider implements JWEAlgorithmProvider {
    @Override
    public byte[] decodeCek(byte[] encodedCek, Key privateKey, JWEHeader header,
                            JWEEncryptionProvider encProvider) throws Exception {
        // ML-KEM decapsulation: privateKey is KEMPrivateKey, encodedCek is the ciphertext
        KemParameters params = ((MLKEMPrivateKeyParameters) ...).getParameters();
        MLKEMExtractor extractor = new MLKEMExtractor((MLKEMPrivateKeyParameters) ...);
        return extractor.extractSecret(encodedCek);  // returns shared secret = CEK
    }

    @Override
    public byte[] encodeCek(JWEEncryptionProvider encProvider, JWEKeyStorage keyStorage,
                            Key encKey, JWEHeaderBuilder headerBuilder) throws Exception {
        // ML-KEM encapsulation for the outbound direction (not used in this domain)
        ...
    }
}
```

Register in `BouncyCastleCryptoProvider.getAlgorithmProvider()` for each ML-KEM identifier.

### Change 2 detail — `CekManagementProviderFactory`

Pattern mirrors existing RSA factories — one factory per JWE algorithm identifier:

```java
public class MlKem768CekManagementProviderFactory implements CekManagementProviderFactory {
    public static final String ID = "ML-KEM-768"; // JOSE draft identifier TBD
    @Override public String getId() { return ID; }
    @Override public CekManagementProvider create(KeycloakSession session) {
        return new MlKemCekManagementProvider(session, ID);
    }
}
```

Register in `META-INF/services/org.keycloak.crypto.CekManagementProviderFactory`.

## What does NOT need changing

| Item | Why no change needed |
|------|---------------------|
| `DefaultTokenManager.decodeClientJWT()` key selection | Filters by `KeyUse.ENC` only — already algorithm-agnostic; will find ML-KEM key if present |
| `jwe.getKeyStorage().setDecryptionKey(privateKey)` | Accepts `java.security.Key`; ML-KEM private key implements this interface |
| `JWERegistry.getAlgProvider()` | Delegates to `CryptoIntegration` — pluggable once the provider is registered |
| `AuthzEndpointRequestObjectParser` validator | Checks `encryptionAlg` string equality only; no algorithm-type restriction |
| `OIDCWellKnownProvider` discovery | Enumerates `CekManagementProvider` factories — auto-includes ML-KEM once factory is registered |
| `JWERegistry` `ENC_PROVIDERS` (symmetric enc) | AES-GCM / AES-CBC content encryption is not quantum-vulnerable; no change needed |

## Dependencies

| Dependency | Domain | Blocking? |
|------------|--------|-----------|
| `JWEAlgorithmProvider` for ML-KEM in `CryptoIntegration` | Core crypto layer | Yes — required for `JWE.verifyAndDecodeJwe()` to function |
| ML-KEM `CekManagementProviderFactory` | New work | Yes — required for discovery and key negotiation |
| Generated ML-KEM Encryption Key Provider | Domain 29 | Yes — without a realm ML-KEM ENC key, there is no private key to decapsulate with |
| JOSE standard identifier for ML-KEM in JWE | IETF draft `draft-ietf-jose-pqc-kem` | Tracking — the `alg` header value is not yet finalised in an RFC |

## GitHub Issue

No existing issue covers this domain. A new sub-issue under
[#43690](https://github.com/keycloak/keycloak/issues/43690) (or #48821) should be created.

**Suggested title:** `PQC: Support ML-KEM for inbound JAR encrypted request object decryption (Domain 8)`

**Description:** Keycloak cannot currently decrypt JWE-encrypted JAR request objects
(`request=` or `request_uri=` parameters) that use ML-KEM key encapsulation. Three changes
are required: (1) a `JWEAlgorithmProvider` implementation for ML-KEM in the BouncyCastle
crypto backend; (2) a `CekManagementProviderFactory` for each ML-KEM key size so that the
algorithm appears in `request_object_encryption_alg_values_supported` in discovery; and
(3) a realm key provider for ML-KEM ENC keys (shared with Domain 29). The key-selection
and JWE decryption flow in `DefaultTokenManager.decodeClientJWT()` is already
algorithm-agnostic and requires no changes. Gated on finalisation of the JOSE ML-KEM
identifier in `draft-ietf-jose-pqc-kem`.

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690)
