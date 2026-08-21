# Domain 1 — Access Token / ID Token Signing

[← Back to PQC Overview](../pqc_overview.html)

**ML-DSA implementation plan**  
**GitHub Issues:** [#43692](https://github.com/keycloak/keycloak/issues/43692), [#50678](https://github.com/keycloak/keycloak/issues/50678) 

**Gaps:** GAP-15  
**Category:** PQC Readiness Gap Analysis

---

## Current PQC State

⚠️ **PARTIAL**

Early groundwork is in place but token signing cannot use ML-DSA end-to-end. `Algorithm.java` has `ML-DSA-44/65/87` string constants, `KeyType.AKP` exists, and JWK serialisation/parsing for AKP keys is fully implemented. What is missing is the signing infrastructure that connects these building blocks to the token signing pipeline.

### Existing Groundwork (no changes needed)

- `Algorithm.java` — `ML_DSA_44`, `ML_DSA_65`, `ML_DSA_87` constants (string values `"ML-DSA-44"`, `"ML-DSA-65"`, `"ML-DSA-87"`)
- `KeyType.AKP` — key type constant
- `AKPUtils.java` — DER prefix maps for all three security levels
- `JWKBuilder.akp()` — serialises ML-DSA public keys to JWK
- `JWKParser.toPublicKey()` — parses AKP JWKs back to `PublicKey`
- `AKPPublicJWK.java` — JWK representation class
- `DefaultTokenManager` — SPI-driven; picks up new providers automatically

---

## Required Changes — Summary

| # | File / Location | Change |
|---|-----------------|--------|
| 1 | `core/.../crypto/JavaAlgorithm.java` | Add ML-DSA constants + cases in `getJavaAlgorithm()` and `getKeyType()` |
| 2 | `services/.../crypto/MLDsa{44,65,87}SignatureProviderFactory.java` | New classes (×3) — one factory per security level |
| 3 | `services/.../crypto/MLDsaSignatureProvider.java` | New class — `SignatureProvider` using `KeyType.AKP` |
| 4 | `META-INF/services/...SignatureProviderFactory` | Register the three new factories |
| 5 | `services/.../keys/GeneratedMLDsaKeyProvider.java` | New class — loads stored ML-DSA keypair from component config |
| 6 | `services/.../keys/GeneratedMLDsaKeyProviderFactory.java` | New class — generates keypair on first use, implements `createFallbackKeys()` |
| 7 | `META-INF/services/...KeyProviderFactory` | Register `GeneratedMLDsaKeyProviderFactory` |

---

## 1. `core/src/main/java/org/keycloak/crypto/JavaAlgorithm.java`

### Add constants

```java
public static final String ML_DSA_44 = "ML-DSA-44";
public static final String ML_DSA_65 = "ML-DSA-65";
public static final String ML_DSA_87 = "ML-DSA-87";
```

### Add cases in `getJavaAlgorithm(String algorithm, String curve)`

Insert before the `default: throw` case:

```java
case Algorithm.ML_DSA_44:
    return ML_DSA_44;
case Algorithm.ML_DSA_65:
    return ML_DSA_65;
case Algorithm.ML_DSA_87:
    return ML_DSA_87;
```

### Add cases in `getKeyType(String keyAlgorithm)`

Insert before the `default: return KeyType.OCT` case:

```java
case Algorithm.ML_DSA_44:
case Algorithm.ML_DSA_65:
case Algorithm.ML_DSA_87:
    return KeyType.AKP;
```

**Note:** `getJavaAlgorithmForHash()` does NOT need a case for ML-DSA. ML-DSA (FIPS 204) is a stateless hash-based scheme that handles its own internal hashing — there is no separate pre-hash step exposed via JCA.

---

## 2. New `MLDsaSignatureProvider` + factories

### `services/.../crypto/MLDsaSignatureProvider.java`

Analogous to `EdDSASignatureProvider` but checks `KeyType.AKP` instead of `KeyType.OKP`:

```java
public class MLDsaSignatureProvider implements SignatureProvider {

    private final KeycloakSession session;
    private final String algorithm;

    public MLDsaSignatureProvider(KeycloakSession session, String algorithm) {
        this.session = session;
        this.algorithm = algorithm;
    }

    @Override
    public SignatureSignerContext signer() throws SignatureException {
        return new ServerAsymmetricSignatureSignerContext(session, algorithm);
    }

    @Override
    public SignatureSignerContext signer(KeyWrapper key) throws SignatureException {
        SignatureProvider.checkKeyForSignature(key, algorithm, KeyType.AKP);
        return new ServerAsymmetricSignatureSignerContext(key);
    }

    @Override
    public SignatureVerifierContext verifier(String kid) throws VerificationException {
        return new ServerAsymmetricSignatureVerifierContext(session, kid, algorithm);
    }

    @Override
    public SignatureVerifierContext verifier(KeyWrapper key) throws VerificationException {
        SignatureProvider.checkKeyForVerification(key, algorithm, KeyType.AKP);
        return new ServerAsymmetricSignatureVerifierContext(key);
    }

    @Override
    public boolean isAsymmetricAlgorithm() {
        return true;
    }
}
```

### `services/.../crypto/MLDsa44SignatureProviderFactory.java`

Following the pattern of `EdDSASignatureProviderFactory`. Repeat for `MLDsa65` and `MLDsa87` substituting `Algorithm.ML_DSA_65` / `Algorithm.ML_DSA_87`:

```java
public class MLDsa44SignatureProviderFactory implements SignatureProviderFactory {

    public static final String ID = Algorithm.ML_DSA_44;

    @Override
    public String getId() { return ID; }

    @Override
    public SignatureProvider create(KeycloakSession session) {
        return new MLDsaSignatureProvider(session, Algorithm.ML_DSA_44);
    }

    @Override
    public Set<String> getJwkPrivateKeyClaims() {
        return AKP_PRIVATE_JWK_CLAIMS;
    }
}
```

**Note:** `AKP_PRIVATE_JWK_CLAIMS` is a new constant (analogous to `OKP_PRIVATE_JWK_CLAIMS` in `SignatureProviderFactory`) that defines the private key claim names for the AKP JWK format.

---

## 4. Register `SignatureProviderFactory` services

Append to `services/src/main/resources/META-INF/services/org.keycloak.crypto.SignatureProviderFactory`:

```
org.keycloak.crypto.MLDsa44SignatureProviderFactory
org.keycloak.crypto.MLDsa65SignatureProviderFactory
org.keycloak.crypto.MLDsa87SignatureProviderFactory
```

---

## 5. New `GeneratedMLDsaKeyProvider` + factory

### `services/.../keys/GeneratedMLDsaKeyProvider.java`

Analogous to `GeneratedEddsaKeyProvider`. Loads the stored keypair from component config and reconstructs it using the ML-DSA `KeyFactory`:

```java
public class GeneratedMLDsaKeyProvider extends AbstractMLDsaKeyProvider {

    public GeneratedMLDsaKeyProvider(RealmModel realm, ComponentModel model) {
        super(realm, model);
    }

    @Override
    protected KeyWrapper loadKey(RealmModel realm, ComponentModel model) {
        String privateKeyEncoded = model.getConfig().getFirst(MLDSA_PRIVATE_KEY_CONFIG);
        String publicKeyEncoded  = model.getConfig().getFirst(MLDSA_PUBLIC_KEY_CONFIG);
        String algorithm         = model.getConfig().getFirst(MLDSA_ALGORITHM_CONFIG);

        try {
            KeyFactory kf = KeyFactory.getInstance(algorithm);

            PrivateKey privateKey = kf.generatePrivate(
                new PKCS8EncodedKeySpec(Base64.getMimeDecoder().decode(privateKeyEncoded)));
            PublicKey publicKey = kf.generatePublic(
                new X509EncodedKeySpec(Base64.getMimeDecoder().decode(publicKeyEncoded)));

            return createKeyWrapper(new KeyPair(publicKey, privateKey), algorithm);
        } catch (Exception e) {
            logger.warnf("Failed to load ML-DSA key: %s", e.toString());
            return null;
        }
    }
}
```

### `services/.../keys/GeneratedMLDsaKeyProviderFactory.java`

Analogous to `GeneratedEddsaKeyProviderFactory`. Generates a fresh keypair on first use and implements `createFallbackKeys()`:

```java
public class GeneratedMLDsaKeyProviderFactory extends AbstractMLDsaKeyProviderFactory {

    public static final String ID = "mldsa-generated";
    // ML-DSA-65 recommended as default: 128-bit post-quantum security, NIST security level 3
    public static final String DEFAULT_ALGORITHM = Algorithm.ML_DSA_65;

    @Override
    public String getId() { return ID; }

    @Override
    public KeyProvider create(KeycloakSession session, ComponentModel model) {
        return new GeneratedMLDsaKeyProvider(session.getContext().getRealm(), model);
    }

    @Override
    public boolean createFallbackKeys(KeycloakSession session, KeyUse keyUse, String algorithm) {
        if (keyUse.equals(KeyUse.SIG) && (
                algorithm.equals(Algorithm.ML_DSA_44) ||
                algorithm.equals(Algorithm.ML_DSA_65) ||
                algorithm.equals(Algorithm.ML_DSA_87))) {

            RealmModel realm = session.getContext().getRealm();
            ComponentModel generated = new ComponentModel();
            generated.setName("fallback-" + algorithm);
            generated.setParentId(realm.getId());
            generated.setProviderId(ID);
            generated.setProviderType(KeyProvider.class.getName());

            MultivaluedHashMap<String, String> config = new MultivaluedHashMap<>();
            config.putSingle(Attributes.PRIORITY_KEY, "-100");
            config.putSingle(MLDSA_ALGORITHM_CONFIG, algorithm);
            generated.setConfig(config);

            realm.addComponentModel(generated);
            return true;
        }
        return false;
    }

    @Override
    public void validateConfiguration(KeycloakSession session, RealmModel realm,
                                      ComponentModel model) throws ComponentValidationException {
        super.validateConfiguration(session, realm, model);

        String algorithm = model.get(MLDSA_ALGORITHM_CONFIG);
        if (algorithm == null) algorithm = DEFAULT_ALGORITHM;

        if (!(model.contains(MLDSA_PRIVATE_KEY_CONFIG) && model.contains(MLDSA_PUBLIC_KEY_CONFIG))) {
            generateKeys(model, algorithm);
        }
    }

    private void generateKeys(ComponentModel model, String algorithm) {
        try {
            KeyPair keyPair = KeyUtils.generateMLDsaKeyPair(algorithm);
            model.put(MLDSA_PRIVATE_KEY_CONFIG,
                Base64.getEncoder().encodeToString(keyPair.getPrivate().getEncoded()));
            model.put(MLDSA_PUBLIC_KEY_CONFIG,
                Base64.getEncoder().encodeToString(keyPair.getPublic().getEncoded()));
            model.put(MLDSA_ALGORITHM_CONFIG, algorithm);
        } catch (Throwable t) {
            throw new ComponentValidationException("Failed to generate ML-DSA keys", t);
        }
    }
}
```

**Note:** `KeyUtils.generateMLDsaKeyPair(algorithm)` requires a new helper method in `common/.../util/KeyUtils.java` analogous to `generateEddsaKeyPair()`, calling `KeyPairGenerator.getInstance(algorithm)` (e.g. `"ML-DSA-65"`) against the configured crypto provider.

---

## 7. Register `KeyProviderFactory` service

Append to `services/src/main/resources/META-INF/services/org.keycloak.keys.KeyProviderFactory`:

```
org.keycloak.keys.GeneratedMLDsaKeyProviderFactory
```

---

## What Does NOT Need Changing

✅ **No changes needed:**
- `DefaultTokenManager` — already SPI-driven; picks up ML-DSA automatically once `SignatureProvider` is registered
- `JWKBuilder` / `JWKParser` / `AKPUtils` / `AKPPublicJWK` — already fully implemented for AKP keys
- Token verification path (`AuthenticationManager`, `LoginActionsService`) — already SPI-driven
- `Algorithm.java` / `KeyType.java` — constants already present

---

## Runtime Dependency

⚠️ **Runtime dependency:** The underlying JCA provider must supply ML-DSA. BouncyCastle BCPQC exposes `"ML-DSA-44"`, `"ML-DSA-65"`, `"ML-DSA-87"` as `KeyFactory` and `KeyPairGenerator` algorithm names. JDK 24+ includes ML-DSA natively. Confirm availability in both `crypto/default` and `crypto/fips` Keycloak distribution modules before merging.

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness