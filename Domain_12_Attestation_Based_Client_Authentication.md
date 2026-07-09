# Domain 12 — Attestation-Based Client Authentication

[← Back to PQC Overview](pqc_overview.html)

## What is this?

Attestation-Based Client Authentication is a new OAuth2/OIDC client authentication method defined in [draft-ietf-oauth-attestation-based-client-auth](https://datatracker.ietf.org/doc/draft-ietf-oauth-attestation-based-client-auth). It allows clients (such as mobile app instances) to authenticate using two JWTs sent in custom HTTP headers:

1. **Client Attestation JWT** — issued and signed by a trusted "Attester" (e.g., Google Play Integrity, Apple App Attest), containing the client instance's public key in the `cnf.jwk` claim
2. **Client Attestation PoP JWT** — signed by the client instance using the private key corresponding to the public key in the attestation JWT, proving possession

This is designed for high-assurance mobile client authentication where each app instance has its own keypair, and the attestation proves the app is genuine and running on a trusted platform.

Keycloak's implementation aligns with the [OpenID4VC High Assurance Interoperability Profile (HAIP) 1.0](https://openid.net/specs/openid4vc-high-assurance-interoperability-profile-1_0-final.html) and draft-07 of the spec.

## Gap

**Two gaps prevent end-to-end ML-DSA support:**

### Gap 1: Algorithm enforcement TODO (line 418)

**File:** `AttestationBasedClientAuthenticator.java` (line 418)

```java
// [TODO] The alg JOSE Header Parameter for both JWTs indicates a registered asymmetric digital signature algorithm
// [TODO] The key contained in the cnf claim of the Client Attestation JWT is not a private key
```

The draft spec requires that the `alg` header parameter in both JWTs must be a **registered asymmetric digital signature algorithm**. Currently, Keycloak does not enforce this requirement — it accepts the algorithm from the header and looks up a `SignatureProvider` without validating that:
1. The algorithm is asymmetric (not symmetric like HS256)
2. The algorithm is from a recognized signature algorithm family

**Impact:** If an ML-DSA `SignatureProvider` exists, the verification path at line 407-414 (Client Attestation JWT) and line 484-491 (Client Attestation PoP JWT) will work. However, without explicit algorithm-type enforcement, there's a risk of accepting inappropriate algorithms.

**What needs to change:**
- Add validation after line 406 and line 483 to check that `algorithm` is asymmetric (e.g., by checking if the provider is instance of `AsymmetricSignatureProvider`, or maintaining an allowlist similar to `JWTClientAuthenticator`)
- Reject symmetric algorithms (HS256, HS384, HS512) explicitly

### Gap 2: JWK thumbprint for AKP keys (GAP-1)

**File:** `JWKSUtils.java` (line 52-57)

```java
private static final Map<String, String[]> JWK_THUMBPRINT_REQUIRED_MEMBERS = new HashMap<>();
static {
    JWK_THUMBPRINT_REQUIRED_MEMBERS.put(KeyType.RSA, new String[] { RSAPublicJWK.MODULUS, RSAPublicJWK.PUBLIC_EXPONENT });
    JWK_THUMBPRINT_REQUIRED_MEMBERS.put(KeyType.EC, new String[] { ECPublicJWK.CRV, ECPublicJWK.X, ECPublicJWK.Y });
    JWK_THUMBPRINT_REQUIRED_MEMBERS.put(KeyType.OKP, new String[] { OKPPublicJWK.CRV, OKPPublicJWK.X });
    // AKP is missing
}
```

The `cnf.jwk` claim in the Client Attestation JWT contains the client instance's public key. If this is an ML-DSA (AKP) key, `JWKSUtils.computeThumbprint()` will throw `UnsupportedOperationException` at line 162 because `KeyType.AKP` is not present in `JWK_THUMBPRINT_REQUIRED_MEMBERS`.

**Why thumbprinting matters here:**

While the current code does not explicitly call `computeThumbprint()` on the `cnf.jwk` key in `AttestationBasedClientAuthenticator`, the spec contemplates **token binding** scenarios where the `cnf.jwk` thumbprint would be embedded in access tokens (similar to DPoP's `dpop_jkt` claim). Once ML-DSA is used for client instance keys, any attempt to compute a thumbprint for the `cnf.jwk` will fail.

**Impact:** 
- The current verification flow (lines 471-491) does **not** compute a thumbprint — it directly uses the JWK to build a `KeyWrapper` and verify the PoP signature. This path will work with ML-DSA keys once ML-DSA `SignatureProvider` exists.
- However, if future features (e.g., token binding with `cnf` thumbprints, or key rollover detection) call `JWKSUtils.computeThumbprint()` on the client instance key, it will fail for AKP keys.

**What needs to change:**
- Add `KeyType.AKP` to `JWK_THUMBPRINT_REQUIRED_MEMBERS` with the correct required members for ML-DSA JWKs (per draft JWK standards for PQC keys)
- This is the same fix required for **Domain 15 (DPoP)** — tracked as **GAP-1**

## Current PQC State

**PARTIAL**

Both verification paths are **SPI-driven** and will support ML-DSA automatically once ML-DSA `SignatureProvider` exists:

1. **Client Attestation JWT verification** (line 407): Uses `SignatureProvider` looked up by algorithm name from the JWS header → SPI-driven
2. **Client Attestation PoP JWT verification** (line 484): Uses `SignatureProvider` looked up by algorithm name from the JWS header → SPI-driven

**What already works:**
- Line 396: `findAttesterKey()` loads the attester's public key from a configured trust material provider (e.g., a Trust Broker IdP) — this already handles AKP keys via `JWKSUtils.getKeyWrappersForUse()`
- Line 332-339: `toPublicKeyWrapper()` converts the `cnf.jwk` into a `KeyWrapper` — this is algorithm-agnostic and will work with ML-DSA JWKs once `JWKParser` supports them

**What's missing:**
- Algorithm-type enforcement (TODO at line 418)
- JWK thumbprint support for AKP keys (GAP-1)

## Required Changes

### Change 1: Implement algorithm-type enforcement

**File:** `AttestationBasedClientAuthenticator.java`

**Location:** After line 406 (Client Attestation JWT) and after line 483 (Client Attestation PoP JWT)

**Current code (line 406-414):**
```java
Algorithm algorithm = jws.getHeader().getAlgorithm();
SignatureProvider signatureProvider = session.getProvider(SignatureProvider.class, algorithm.name());
if (signatureProvider == null) {
    throw new TokenVerificationException(attestationJwt, "Signature provider not found for algorithm: " + algorithm);
}
byte[] data = jws.getEncodedSignatureInput().getBytes(StandardCharsets.UTF_8);
if (!signatureProvider.verifier(attesterKey).verify(data, jws.getSignature())) {
    throw new TokenSignatureInvalidException(attestationJwt, "Invalid token signature");
}
```

**Proposed fix:**
```java
Algorithm algorithm = jws.getHeader().getAlgorithm();

// [CHANGE] Enforce asymmetric algorithm requirement
if (!isAsymmetricSignatureAlgorithm(algorithm)) {
    throw new TokenVerificationException(attestationJwt, 
        "Client Attestation JWT must use an asymmetric signature algorithm, not: " + algorithm);
}

SignatureProvider signatureProvider = session.getProvider(SignatureProvider.class, algorithm.name());
// ... rest unchanged
```

Add a helper method (similar to `JWTClientAuthenticator`):
```java
private boolean isAsymmetricSignatureAlgorithm(Algorithm algorithm) {
    // Reject symmetric algorithms explicitly
    if (algorithm == Algorithm.HS256 || algorithm == Algorithm.HS384 || algorithm == Algorithm.HS512) {
        return false;
    }
    // All RS*, PS*, ES*, EdDSA, and future ML-DSA* are asymmetric
    return true;
}
```

Apply the same pattern after line 483 for the Client Attestation PoP JWT verification.

### Change 2: Add AKP to JWK thumbprint support

**File:** `JWKSUtils.java`

**Location:** Line 52-57 static initializer

**Current code:**
```java
static {
    JWK_THUMBPRINT_REQUIRED_MEMBERS.put(KeyType.RSA, new String[] { RSAPublicJWK.MODULUS, RSAPublicJWK.PUBLIC_EXPONENT });
    JWK_THUMBPRINT_REQUIRED_MEMBERS.put(KeyType.EC, new String[] { ECPublicJWK.CRV, ECPublicJWK.X, ECPublicJWK.Y });
    JWK_THUMBPRINT_REQUIRED_MEMBERS.put(KeyType.OKP, new String[] { OKPPublicJWK.CRV, OKPPublicJWK.X });
}
```

**Proposed fix:**
```java
static {
    JWK_THUMBPRINT_REQUIRED_MEMBERS.put(KeyType.RSA, new String[] { RSAPublicJWK.MODULUS, RSAPublicJWK.PUBLIC_EXPONENT });
    JWK_THUMBPRINT_REQUIRED_MEMBERS.put(KeyType.EC, new String[] { ECPublicJWK.CRV, ECPublicJWK.X, ECPublicJWK.Y });
    JWK_THUMBPRINT_REQUIRED_MEMBERS.put(KeyType.OKP, new String[] { OKPPublicJWK.CRV, OKPPublicJWK.X });
    JWK_THUMBPRINT_REQUIRED_MEMBERS.put(KeyType.AKP, new String[] { AKPPublicJWK.CRV, AKPPublicJWK.X });
}
```

This requires defining `AKPPublicJWK` constants (similar to `OKPPublicJWK`) — likely `CRV` and `X` fields per draft PQC JWK specifications.

**Note:** This is the same fix required for **Domain 15 (DPoP)** and should be tracked under **GAP-1** as a shared dependency.

## What does NOT need changing

| Component | Why it's already PQC-ready |
|-----------|---------------------------|
| Attester key lookup (line 396) | `findAttesterKey()` → `TrustMaterialResolver.resolveKey()` → `JWKSUtils.getKeyWrappersForUse()` already supports AKP |
| `cnf.jwk` → `KeyWrapper` conversion (line 332) | `toPublicKeyWrapper()` is algorithm-agnostic; sets `kw.setType(jwk.getKeyType())` without hardcoding |
| Signature verification SPI path (lines 407, 484) | Fully SPI-driven via `SignatureProvider` — will support ML-DSA automatically once providers exist |
| Token claims validation (lines 375-392, 443-468) | Pure business logic checks — no cryptographic operations |

## Dependencies

1. **ML-DSA `SignatureProvider`** (tracked under #48821 / #48824) — required before attestation-based authentication can use ML-DSA end-to-end
2. **GAP-1 fix** (JWK thumbprint for AKP) — shared with Domain 15 (DPoP); needs a new issue under #43690
3. **Draft spec stabilization** — the attestation-based client auth spec is still in draft; algorithm requirements may evolve

## GitHub Issue Status

**Partially tracked:**
- The SPI-driven verification path is covered by the core ML-DSA provider work under **#48824**
- **GAP-1** (JWK thumbprint for AKP) is identified but not yet tracked by a GitHub issue — needs a new sub-issue under #43690
- **GAP-22** (algorithm enforcement TODO at line 418) is identified in the overview table but not yet tracked — should be created as a sub-issue under #43690 or #48824

**Recommended new issue:**
- Title: "Attestation-Based Client Auth: enforce asymmetric algorithm requirement and add AKP JWK thumbprint support"
- Description: "Complete PQC readiness for Domain 12 (Attestation-Based Client Authentication) by implementing the algorithm-type enforcement TODO at line 418 and adding AKP key support to JWKSUtils.computeThumbprint(). Both changes are required before ML-DSA client instance keys can be used in the `cnf.jwk` claim."
- Should reference both **GAP-1** and **GAP-22**

## Related Domains

- **Domain 10** — private_key_jwt Client Authentication (similar SPI-driven verification path)
- **Domain 15** — DPoP (same GAP-1 JWK thumbprint issue)
- **Domain 55** — Default Trust Identity Provider (trust material resolution for attester keys)

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness