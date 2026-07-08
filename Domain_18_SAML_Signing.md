# Domain 18 — SAML Assertion & Document Signing

## What is this?

SAML (Security Assertion Markup Language) uses XML Digital Signatures to sign assertions and protocol messages. Keycloak acts as both a SAML Identity Provider (IdP) and Service Provider (SP) in federation scenarios.

**Signing operations:**

1. **Outbound:** Keycloak signs SAML responses, assertions, and logout requests when acting as an IdP
2. **Inbound:** Keycloak verifies signatures on SAML requests and responses when acting as an SP

SAML uses **XML Signature** (XMLDSig) with algorithm URIs like:
- `http://www.w3.org/2001/04/xmldsig-more#rsa-sha256` (RSA-SHA256)
- `http://www.w3.org/2007/05/xmldsig-more#sha256-rsa-MGF1` (RSA-PSS)

## Gap

**Two-level gap blocks ML-DSA support:**

### Gap 1: No ML-DSA algorithm URIs registered (GAP-2)

**File:** `SignatureAlgorithm.java` (lines 28-35)

```java
public enum SignatureAlgorithm {
    RSA_SHA1("http://www.w3.org/2000/09/xmldsig#rsa-sha1", ...),
    RSA_SHA256("http://www.w3.org/2001/04/xmldsig-more#rsa-sha256", ...),
    RSA_SHA256_MGF1("http://www.w3.org/2007/05/xmldsig-more#sha256-rsa-MGF1", ...),
    RSA_SHA512("http://www.w3.org/2001/04/xmldsig-more#rsa-sha512", ...),
    RSA_SHA512_MGF1("http://www.w3.org/2007/05/xmldsig-more#sha512-rsa-MGF1", ...),
    DSA_SHA1("http://www.w3.org/2000/09/xmldsig#dsa-sha1", ...)
    // No ML-DSA, FN-DSA, or SLH-DSA entries
}
```

The enum defines the mapping between:
- XML Signature algorithm URIs (e.g., `http://...#rsa-sha256`)
- Java signature algorithm names (e.g., `SHA256withRSA`)

**Impact:** Even if a PQC-capable JVM supports `SHA256withML-DSA`, Keycloak cannot use it for SAML because there's no XML Signature URI registered.

**When can ML-DSA URIs be added?**

W3C and/or IETF must define XML Signature algorithm URIs for ML-DSA, FN-DSA, and SLH-DSA. The current XMLDSig specifications do not include post-quantum algorithms.

Tracking:
- [W3C XML Security Working Group](https://www.w3.org/2008/xmlsec/)
- IETF may also publish PQC XMLDSig URIs

### Gap 2: Hardcoded RS256 key selection (GAP-9)

Even if Gap 1 is resolved, there are **4 call sites** that hardcode `Algorithm.RS256` when selecting keys for SAML signing:

**File:** `SamlProtocol.java` (line 544)

```java
KeyWrapper keyPair = keyManager.getActiveKey(realm, KeyUse.SIG, Algorithm.RS256);
```

**File:** `SamlService.java` (line 965)

```java
List<KeyWrapper> keys = session.keys().getKeysStream(realm, KeyUse.SIG, Algorithm.RS256)
```

**File:** `SAMLIdentityProvider.java` (line 506) — mentioned in overview, not read in this analysis

**File:** (4th call site mentioned in overview, not read in this analysis)

**Impact:** These hardcoded `Algorithm.RS256` lookups mean:
- SAML signing will always use an RSA key, never an ML-DSA key
- Even if ML-DSA algorithm URIs exist, the key selection code bypasses them

This is a **two-level gap**:
1. Algorithm layer: no ML-DSA URIs in `SignatureAlgorithm.java`
2. Key selection layer: hardcoded RS256 lookups that never reach ML-DSA keys

Fixing only Gap 1 without Gap 2 means ML-DSA support would never be used in practice.

## Current PQC State

**BLOCKED**

SAML is entirely classical at **two independent levels**. Both must be fixed before SAML can support ML-DSA.

## Required Changes

### Change 1: Add ML-DSA algorithm URIs (when W3C/IETF defines them)

**File:** `SignatureAlgorithm.java`

**Location:** Lines 28-35 enum definition

**Current code:**
```java
public enum SignatureAlgorithm {
    RSA_SHA1("http://www.w3.org/2000/09/xmldsig#rsa-sha1", "http://www.w3.org/2000/09/xmldsig#sha1", "SHA1withRSA"),
    // ... existing RSA/DSA entries
}
```

**Proposed fix (when standard URIs exist):**
```java
public enum SignatureAlgorithm {
    // ... existing entries
    ML_DSA_44("http://www.w3.org/[future-spec]#ml-dsa-44", "http://www.w3.org/2001/04/xmlenc#sha256", "SHA256withML-DSA-44"),
    ML_DSA_65("http://www.w3.org/[future-spec]#ml-dsa-65", "http://www.w3.org/2001/04/xmlenc#sha256", "SHA256withML-DSA-65"),
    ML_DSA_87("http://www.w3.org/[future-spec]#ml-dsa-87", "http://www.w3.org/2001/04/xmlenc#sha256", "SHA256withML-DSA-87"),
    // Similar for FN-DSA, SLH-DSA
}
```

**Also update the static maps (lines 44-56):**
```java
static {
    signatureMethodMap.put(ML_DSA_44.getXmlSignatureMethod(), ML_DSA_44);
    signatureMethodMap.put(ML_DSA_65.getXmlSignatureMethod(), ML_DSA_65);
    // ... etc
}
```

### Change 2: Fix hardcoded RS256 key selection

**File:** `SamlProtocol.java` (line 544)

**Current code:**
```java
KeyWrapper keyPair = keyManager.getActiveKey(realm, KeyUse.SIG, Algorithm.RS256);
```

**Proposed fix:**
```java
// Read the realm's configured SAML signature algorithm
String samlSignatureAlgorithm = realm.getAttribute("saml.signature.algorithm"); // or similar config
if (samlSignatureAlgorithm == null) {
    samlSignatureAlgorithm = Algorithm.RS256; // backward-compatible default
}
KeyWrapper keyPair = keyManager.getActiveKey(realm, KeyUse.SIG, samlSignatureAlgorithm);
```

**Similar fixes needed in:**
- `SamlService.java` line 965
- `SAMLIdentityProvider.java` line 506
- (4th call site)

**Alternative approach:** Instead of realm-level configuration, read the algorithm from the SAML client's configuration (similar to how OIDC clients have per-client algorithm attributes).

## What does NOT need changing

| Component | Why it's already PQC-ready (once gaps are fixed) |
|-----------|---------------------------------------------------|
| XML Signature creation (`XMLSignatureUtil`) | Takes a `SignatureAlgorithm` enum value and uses it — algorithm-agnostic once URIs exist |
| XML Signature verification | Uses `Signature.getInstance()` with the algorithm name from the enum — will work with PQC once JVM supports it |
| SAML protocol message parsing | Algorithm-agnostic — reads the signature method URI from the XML |
| Key storage (`KeyWrapper`) | Algorithm-agnostic — can hold any key type |

## Dependencies

### External (blocking)
1. **W3C or IETF** must define XML Signature algorithm URIs for ML-DSA, FN-DSA, SLH-DSA
2. **JVM (BouncyCastle)** must support the Java algorithm names (e.g., `SHA256withML-DSA-44`) — likely already available

### Internal (when standards exist)
3. **Add URIs to `SignatureAlgorithm.java`** (Gap 1 fix)
4. **Fix 4 hardcoded RS256 call sites** (Gap 2 fix)
5. **Add realm/client configuration** for SAML signature algorithm selection
6. **Update admin UI** to expose ML-DSA options

## GitHub Issue Status

**Partially tracked:**

- **#50292** — SAML PQC (parent issue)
- **#50294** — SAML signing algorithm URIs (likely covers Gap 1)

**GAP-2** and **GAP-9** are identified in the overview table. Check if #50294 covers both or if Gap 2 (URIs) and Gap 9 (hardcoded key selection) need separate issues.

**Recommended:**

If #50294 only covers the URI registration (Gap 1), create a second sub-issue under #50292:
- **Title:** "SAML: replace hardcoded RS256 key selection with configurable algorithm"
- **Description:** "4 call sites hardcode `Algorithm.RS256` when selecting SAML signing keys. Even after ML-DSA URIs are added, these lookups bypass ML-DSA keys. Make algorithm selection configurable."
- **Tracks:** GAP-9

## What this means for operators

**Today:**

SAML signing and verification are quantum-vulnerable (RSA/RSA-PSS only). No mitigation available until W3C/IETF define PQC XMLDSig URIs.

**When PQC XMLDSig becomes available (timeline unknown):**

1. **Keycloak update required:** Add ML-DSA URIs to `SignatureAlgorithm.java` and fix hardcoded RS256 lookups
2. **Realm configuration:** Set SAML signature algorithm to ML-DSA
3. **Key migration:** Generate or import ML-DSA keys for SAML use
4. **Federation partners must support PQC SAML:**
   - External IdPs sending assertions to Keycloak (SP role) must use ML-DSA signatures
   - External SPs receiving assertions from Keycloak (IdP role) must verify ML-DSA signatures

**Migration challenge:**

Unlike OIDC (where each client can use a different algorithm), SAML typically uses a **single realm-wide signature algorithm**. Migrating all federation partners simultaneously may be difficult. Consider:
- Maintaining separate realms for classical vs PQC SAML during transition
- Dual-signature approach (if SAML spec allows — unlikely)

## Related Domains

- **Domain 19** — SAML Assertion Encryption (ML-KEM for key transport; separate gap)
- **Domain 20** — SAML IdP Broker (same GAP-9 hardcoded RS256 issue at line 506)
- **Domain 59** — SAML Metadata Public Key Loader (depends on GAP-2 and GAP-9 being resolved)
- **Domain 60** — SAML Artifact Resolution (depends on GAP-2 and GAP-9)
- **Domain 61** — SAML2Signature hardcoded RSA-SHA1 default (additional hardcoding layer)

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness