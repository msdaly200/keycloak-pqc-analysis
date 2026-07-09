# Domain 27 — OID4VC c_nonce JWT Signing

[← Back to PQC Overview](pqc_overview.html)

## What is this?

The `c_nonce` (challenge nonce) is a cryptographic nonce used in OID4VC (OpenID for Verifiable Credentials) to prevent replay attacks during credential issuance. Keycloak signs the `c_nonce` as a JWT and returns it to the wallet, which must include it in the key-binding proof (Domain 26).

**How it works:**

1. A wallet requests a verifiable credential from Keycloak
2. Keycloak generates a random nonce value
3. Keycloak signs the nonce as a JWT (the **c_nonce JWT**)
4. Keycloak returns the signed c_nonce to the wallet
5. The wallet includes this c_nonce in its key-binding proof JWT
6. Keycloak validates that the c_nonce in the proof matches and hasn't expired
7. This prevents an attacker from replaying old proofs

## Gap

**Hardcoded algorithm selection bypasses realm default (GAP-18).**

### File: `JwtCNonceHandler.java` (lines 189, 195)

```java
protected KeyWrapper selectSigningKey(RealmModel realm) {
    KeyWrapper signingKey;
    try {
        signingKey = keycloakSession.keys().getActiveKey(realm, KeyUse.SIG, Algorithm.ES256);
    } catch (RuntimeException ex) {
        logger.debugf("Failed to find active ES256 signing key for realm %s. Falling back to RSA...",
                     realm.getName());
        // use RSA only as fallback since the preferred algorithm by OpenID4VC is elliptic curve
        signingKey = keycloakSession.keys().getActiveKey(realm, KeyUse.SIG, Algorithm.RS256);
    }
    return signingKey;
}
```

**Impact:**

- **Hardcoded to ES256 first, RS256 fallback** - never considers ML-DSA
- **Bypasses realm default algorithm** - does not check `Constants.DEFAULT_SIGNATURE_ALGORITHM`
- Even after ML-DSA providers exist and an ML-DSA key is active, c_nonce JWTs will **never use it**
- The comment suggests ES256 is preferred by OpenID4VC spec, but this is no longer accurate once PQC is standardized

**Why this is different from other domains:**

Most signing domains use the `SignatureProvider` SPI or read from configurable algorithm settings. This domain has a **hardcoded preference** that cannot be overridden.

## Current PQC State

**BLOCKED**

c_nonce JWT signing will always use ES256 or RS256, never ML-DSA, even after ML-DSA providers exist.

## Required Changes

### Change: Replace hardcoded algorithm selection with configurable lookup

**File:** `JwtCNonceHandler.java`

**Location:** Lines 186-198 (`selectSigningKey()` method)

**Option A: Use realm default algorithm**
```java
protected KeyWrapper selectSigningKey(RealmModel realm) {
    String algorithm = realm.getDefaultSignatureAlgorithm(); // or Constants.DEFAULT_SIGNATURE_ALGORITHM
    if (algorithm == null) {
        algorithm = Algorithm.ES256; // backward-compatible default
    }
    
    KeyWrapper signingKey;
    try {
        signingKey = keycloakSession.keys().getActiveKey(realm, KeyUse.SIG, algorithm);
    } catch (RuntimeException ex) {
        // Fallback to RS256 if configured algorithm not available
        logger.debugf("Failed to find active %s signing key for realm %s. Falling back to RS256...",
                     algorithm, realm.getName());
        signingKey = keycloakSession.keys().getActiveKey(realm, KeyUse.SIG, Algorithm.RS256);
    }
    return signingKey;
}
```

**Option B: Add OID4VC-specific configuration**
```java
// Add a new realm attribute: oid4vc.cnonce.signing.algorithm
String algorithm = realm.getAttribute("oid4vc.cnonce.signing.algorithm");
if (algorithm == null) {
    algorithm = Algorithm.ES256; // backward-compatible default
}
// ... rest same as Option A
```

**Option C: Dynamic SPI-based selection**
```java
// Select the first available asymmetric algorithm from the SPI
List<String> availableAlgs = CryptoUtils.getSupportedAsymmetricSignatureAlgorithms(keycloakSession);
String algorithm = availableAlgs.stream()
    .filter(alg -> keycloakSession.keys().getActiveKey(realm, KeyUse.SIG, alg) != null)
    .findFirst()
    .orElse(Algorithm.RS256);
// ... use algorithm
```

**Recommendation:** Option A (use realm default) is simplest and most consistent with other domains.

## What does NOT need changing

| Component | Why it's already PQC-ready (once gap is fixed) |
|-----------|------------------------------------------------|
| JWT generation | Uses standard JWS signing - algorithm-agnostic once key is selected |
| Nonce value generation | Uses random bytes - no cryptographic algorithm involved |
| Expiration validation | Time-based logic - no cryptographic operations |

## Dependencies

1. **ML-DSA `SignatureProvider`** (tracked under [#48821](https://github.com/keycloak/keycloak/issues/48821) / [#48824](https://github.com/keycloak/keycloak/issues/48824))
2. **GAP-18 fix** — update hardcoded algorithm selection logic
3. **Configuration decision** — choose whether to use realm default or add OID4VC-specific config

## GitHub Issue Status

**GAP-18** is identified in the overview table but does not have a dedicated GitHub issue.

**Priority:** MEDIUM (blocks OID4VC PQC readiness)

**Recommended new issue:**

Create a sub-issue under [#43690](https://github.com/keycloak/keycloak/issues/43690):
- **Title:** "OID4VC: replace hardcoded ES256/RS256 c_nonce signing with configurable algorithm"
- **Description:**
  > `JwtCNonceHandler.selectSigningKey()` hardcodes ES256 as primary and RS256 as fallback for c_nonce JWT signing. This bypasses the realm's default signature algorithm and will never use ML-DSA keys even after ML-DSA providers exist.
  >
  > **Current behavior:**
  > ```java
  > signingKey = keycloakSession.keys().getActiveKey(realm, KeyUse.SIG, Algorithm.ES256);
  > // Falls back to RS256 if ES256 not found
  > ```
  >
  > **Proposed fix:**
  > - Option A: Use realm default signature algorithm (consistent with OIDC token signing)
  > - Option B: Add OID4VC-specific configuration attribute
  > - Option C: Dynamically select first available asymmetric algorithm
  >
  > **Impact:** Affects OID4VC credential issuance replay protection (Domain 27)
- **Label:** `pqc-readiness`
- **Tracks:** GAP-18

## What this means for operators

**Today:**

c_nonce JWTs are signed with ES256 (or RS256 fallback), which is quantum-vulnerable.

**When migrating to PQC:**

1. **GAP-18 must be fixed first** — otherwise c_nonce will continue using classical algorithms
2. After the fix, operators can either:
   - Set realm default algorithm to ML-DSA (if Option A is implemented)
   - Configure OID4VC c_nonce algorithm explicitly (if Option B is implemented)
3. **No wallet changes needed** — wallets just include the c_nonce in their proof; they don't care which algorithm signed it

**Note:** The c_nonce signature only needs to be verified by Keycloak itself (during proof validation in Domain 26), not by wallets. This is an **internal JWT** for replay protection, not a credential JWT that gets distributed.

## Related Domains

- **Domain 26** — OID4VC Key Binding (validates the proof that contains the c_nonce)
- **Domain 24** — JWT-VC / SD-JWT Credential Signing (the actual credentials issued)
- **Domain 1** — Access Token / ID Token Signing (uses realm default algorithm correctly)

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
