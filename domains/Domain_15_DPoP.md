# Domain 15 — DPoP (Demonstrating Proof of Possession)

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

DPoP (Demonstrating Proof of Possession) is an OAuth 2.0 extension ([RFC 9449](https://www.rfc-editor.org/rfc/rfc9449.html)) that binds access tokens to the client's cryptographic key, preventing token theft and replay attacks.

**How it works:**

1. The client generates a key pair (typically ECDSA or RSA)
2. With each token request, the client sends a **DPoP proof** — a signed JWT containing:
   - The client's public key (in the `jwk` header)
   - The HTTP method and URL (`htm`, `htu` claims)
   - A unique identifier (`jti` for replay prevention)
   - The access token hash (`ath` claim when using the token)
3. Keycloak verifies the DPoP proof signature and binds the access token to the key by embedding a **JWK thumbprint** (`jkt`) in the token's `cnf` (confirmation) claim
4. When the client uses the access token, it must present a fresh DPoP proof signed with the same key
5. Resource servers verify that the `jkt` in the token matches the thumbprint of the key in the new DPoP proof

This makes stolen tokens useless — an attacker cannot use them without the private key.

## Gap

**One critical gap prevents ML-DSA support:**

### JWK Thumbprint for AKP Keys (GAP-1)

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

**Impact on DPoP:**

**File:** `DPoPUtil.java` (line 242)

```java
DPoP dPoP = verifier.verify().getToken();
dPoP.setThumbprint(JWKSUtils.computeThumbprint(jwk));  // <-- THROWS if jwk is AKP
return dPoP;
```

If the DPoP proof's `jwk` header contains an ML-DSA (AKP) public key, `JWKSUtils.computeThumbprint()` will throw `UnsupportedOperationException` at line 162 of `JWKSUtils.java` because `KeyType.AKP` is not in the `JWK_THUMBPRINT_REQUIRED_MEMBERS` map.

**This is a hard blocker:** DPoP cannot work with ML-DSA keys until this is fixed, even if ML-DSA `SignatureProvider` exists.

**Why thumbprinting is essential for DPoP:**

The JWK thumbprint is the **core binding mechanism** in DPoP:
1. Thumbprint is computed from the DPoP proof's public key (line 242)
2. Thumbprint is embedded in the access token's `cnf.jkt` claim (line 630)
3. Resource servers verify that future DPoP proofs use the same key by comparing thumbprints

Without the ability to compute thumbprints for AKP keys, the entire DPoP flow breaks.

## Current PQC State

**PARTIAL**

DPoP proof **signature verification** is fully SPI-driven and will support ML-DSA automatically once ML-DSA `SignatureProvider` exists:

**File:** `DPoPUtil.java` (lines 196-228)

```java
String algorithm = header.getAlgorithm().name();
if (!getDPoPSupportedAlgorithms(session).contains(algorithm)) {
    throw new VerificationException("Unsupported DPoP algorithm: " + header.getAlgorithm());
}
// ... later ...
SignatureVerifierContext signatureVerifier = CryptoUtils.getSignatureProvider(session, algorithm).verifier(key);
verifier.verifierContext(signatureVerifier);
```

The `getDPoPSupportedAlgorithms()` method (lines 358-365) dynamically lists all registered **asymmetric** `SignatureProvider` implementations. Once ML-DSA providers are registered, ML-DSA will automatically appear in the supported algorithms list.

**What already works:**
- Algorithm allowlist is dynamic (line 359) — will include ML-DSA automatically
- Asymmetric algorithm enforcement (line 362) — filters out symmetric algorithms
- Signature verification is SPI-driven (line 228) — will use ML-DSA `SignatureProvider` when available
- JWK → `KeyWrapper` conversion (line 207) — `JWKSUtils.getKeyWrapper()` already handles AKP keys

**What's broken:**
- **JWK thumbprint computation** (line 242) — throws for AKP keys

## Required Changes

### Change: Add AKP to JWK thumbprint support

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

This requires defining `AKPPublicJWK` constants (similar to `OKPPublicJWK`). Based on draft PQC JWK specifications, ML-DSA public keys likely use:
- `crv` — curve identifier (e.g., "ML-DSA-44", "ML-DSA-65", "ML-DSA-87")
- `x` — public key bytes (base64url-encoded)

**Note:** The exact JWK structure for ML-DSA keys depends on the IETF COSE/JOSE working group's draft specifications. Keycloak should align with whatever standard emerges for PQC JWK representations.

**RFC 7638 (JWK Thumbprint) compliance:**

[RFC 7638](https://www.rfc-editor.org/rfc/rfc7638.html) defines how to compute JWK thumbprints by hashing a canonical JSON representation of specific required members. For each key type:
- RSA: `e`, `kty`, `n`
- EC: `crv`, `kty`, `x`, `y`
- OKP: `crv`, `kty`, `x`

For AKP (ML-DSA), the required members should be:
- `kty`: "AKP"
- `crv`: the curve/parameter set identifier
- `x`: the public key value

This matches the OKP pattern (EdDSA uses the same structure).

## What does NOT need changing

| Component | Why it's already PQC-ready |
|-----------|---------------------------|
| Algorithm allowlist (line 358-365) | `getDPoPSupportedAlgorithms()` dynamically queries registered `SignatureProvider` factories — will include ML-DSA automatically |
| Asymmetric algorithm enforcement (line 362) | Uses `isAsymmetricAlgorithm()` — will correctly identify ML-DSA as asymmetric |
| Signature verification (line 228) | Fully SPI-driven via `SignatureProvider` — will use ML-DSA provider when available |
| JWK → `KeyWrapper` conversion (line 207) | `JWKSUtils.getKeyWrapper()` already handles AKP keys (returns a `KeyWrapper` with the public key set) |
| Replay prevention (line 405-414) | Uses `jti` hash — algorithm-agnostic |
| Timestamp validation (line 428-455) | Pure time-based logic — no cryptographic operations |
| HTTP binding check (line 378-390) | Compares `htm`/`htu` claims — no cryptographic operations |
| Access token hash check (line 462-476) | Uses SHA-256 hash of the access token — quantum-safe |
| Confirmation claim embedding (line 626-631) | Stores the thumbprint string — algorithm-agnostic storage |

## Dependencies

1. **ML-DSA `SignatureProvider`** (tracked under [#48821](https://github.com/keycloak/keycloak/issues/48821) / [#48824](https://github.com/keycloak/keycloak/issues/48824)) — required before DPoP can use ML-DSA for signing proofs
2. **GAP-1 fix** (JWK thumbprint for AKP) — **critical blocker** for DPoP with ML-DSA; needs a new issue under [#43690](https://github.com/keycloak/keycloak/issues/43690)
3. **PQC JWK specification** — IETF COSE/JOSE working group must finalize the structure for ML-DSA public keys in JWK format (may already exist in draft form)

## GitHub Issue Status

**Not yet tracked:**

**GAP-1** (JWK thumbprint for AKP) is identified in the overview table but does not have a dedicated GitHub issue. This gap affects:
- **Domain 12** — Attestation-Based Client Authentication (same thumbprint issue for `cnf.jwk` client instance keys)
- **Domain 15** — DPoP (this domain — critical blocker)

**Recommended new issue:**

- **Title:** "Add AKP (ML-DSA) support to JWK thumbprint computation (RFC 7638)"
- **Description:** 
  > `JWKSUtils.computeThumbprint()` does not support `KeyType.AKP` (ML-DSA/FN-DSA/SLH-DSA keys). This blocks:
  > - DPoP proof validation when clients use ML-DSA keys (Domain 15)
  > - Attestation-based client authentication with ML-DSA client instance keys (Domain 12)
  > 
  > **Fix:** Add `KeyType.AKP` to `JWK_THUMBPRINT_REQUIRED_MEMBERS` map in `JWKSUtils.java` with appropriate required members (`crv`, `x`) per draft PQC JWK specifications.
  > 
  > **Depends on:** IETF finalizing PQC JWK structure (may already exist in COSE/JOSE drafts).
- **Sub-issue under:** [#43690](https://github.com/keycloak/keycloak/issues/43690)
- **Related domains:** Domain 12, Domain 15

## What this means for operators

**When migrating to PQC:**

1. **DPoP-bound tokens will support ML-DSA keys** once:
   - The JWK thumbprint fix (GAP-1) is deployed
   - ML-DSA `SignatureProvider` exists

2. **No configuration changes needed** — the algorithm allowlist is dynamic

3. **Client migration:**
   - Clients currently using RSA or ECDSA for DPoP can switch to ML-DSA by:
     - Generating a new ML-DSA key pair
     - Sending DPoP proofs with `alg: ML-DSA-44` (or -65, -87) header
   - Keycloak will automatically accept and verify ML-DSA DPoP proofs

4. **Token binding is quantum-safe:**
   - Even if the DPoP signature uses classical algorithms today, the binding mechanism (SHA-256 thumbprint in `cnf.jkt`) is quantum-safe
   - Migrating to ML-DSA signatures eliminates the quantum vulnerability in the proof verification step

5. **Hybrid deployments supported:**
   - Some clients can use ML-DSA DPoP while others continue with RSA/ECDSA
   - The token format (`cnf.jkt`) is algorithm-agnostic

## Related Domains

- **Domain 10** — private_key_jwt Client Authentication (similar SPI-driven signature verification)
- **Domain 12** — Attestation-Based Client Authentication (same GAP-1 JWK thumbprint issue for `cnf.jwk`)
- **Domain 57** — Token Exchange Grant (mentions DPoP binding dependency in the overview)

## Standards References

- [RFC 9449: OAuth 2.0 Demonstrating Proof of Possession (DPoP)](https://www.rfc-editor.org/rfc/rfc9449.html)
- [RFC 7638: JSON Web Key (JWK) Thumbprint](https://www.rfc-editor.org/rfc/rfc7638.html)
- [Draft IETF COSE PQC specifications](https://datatracker.ietf.org/wg/cose/documents/) — for ML-DSA JWK structure

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness