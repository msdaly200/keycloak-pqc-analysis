# Domain 20 — SAML IdP Broker — SP Metadata & Federation Signing

## What is this?

When Keycloak acts as a SAML Service Provider (SP) federating with an external SAML Identity Provider (IdP), it must:

1. **Publish SP metadata:** An XML document describing Keycloak's SAML endpoints, certificates, and capabilities
2. **Sign metadata:** The SP metadata is signed to prove authenticity
3. **Sign protocol messages:** SAML authentication requests, logout requests, and artifact resolve requests sent to the external IdP are signed
4. **Verify inbound responses:** SAML responses and assertions from the external IdP are verified

This domain covers Keycloak's **SP role** in SAML federation (not IdP role, which is Domain 18).

## Gap

**Same two-level gap as Domain 18 (SAML Signing).**

### Gap 1: No ML-DSA algorithm URIs (GAP-2)

**Shared with Domain 18:** The `SignatureAlgorithm.java` enum does not include ML-DSA, FN-DSA, or SLH-DSA algorithm URIs.

This affects:
- SP metadata signing
- Outbound SAML request signing (AuthnRequest, LogoutRequest, ArtifactResolve)
- Inbound SAML response verification

See **Domain 18** for full analysis of GAP-2.

### Gap 2: Hardcoded RS256 key selection for SP metadata (GAP-9)

**File:** `SAMLIdentityProvider.java` (line 506) — mentioned in overview

The code that generates and signs SP metadata hardcodes `Algorithm.RS256` when selecting the signing key, similar to the 4 call sites in Domain 18.

**Impact:**

Even after ML-DSA algorithm URIs are added, the SP metadata signing will always use an RSA key, never an ML-DSA key.

**Additional issue:** The overview mentions that **ML-DSA keys are excluded from exported IdP metadata descriptor entirely**. This suggests that even if the realm has ML-DSA keys, they are filtered out when building the metadata's `<KeyDescriptor>` elements.

## Current PQC State

**BLOCKED**

Same as Domain 18 — SAML is classical at two independent levels:
1. **Algorithm layer:** No ML-DSA URIs in `SignatureAlgorithm.java` (GAP-2)
2. **Key selection layer:** Hardcoded RS256 lookup and ML-DSA key exclusion from metadata (GAP-9)

Both must be fixed before SAML federation can use ML-DSA.

## Required Changes

### Change 1: Add ML-DSA algorithm URIs

**Same as Domain 18, Change 1.**

Add ML-DSA, FN-DSA, SLH-DSA entries to `SignatureAlgorithm.java` once W3C/IETF defines XML Signature URIs for them.

### Change 2: Fix hardcoded RS256 in SP metadata signing

**File:** `SAMLIdentityProvider.java`

**Location:** Line 506 (mentioned in overview, not read in this analysis)

**Expected current code:**
```java
KeyWrapper keyPair = keyManager.getActiveKey(realm, KeyUse.SIG, Algorithm.RS256);
// Use keyPair to sign SP metadata
```

**Proposed fix:**
```java
String samlSignatureAlgorithm = getSamlSignatureAlgorithm(realm); // read from config
KeyWrapper keyPair = keyManager.getActiveKey(realm, KeyUse.SIG, samlSignatureAlgorithm);
```

### Change 3: Include ML-DSA keys in IdP metadata descriptor export

**File:** Unknown (likely `SAMLIdentityProviderFactory.java` or similar)

**Current behavior:** When exporting the IdP metadata descriptor (the XML document that describes the external IdP to SPs), ML-DSA keys are excluded.

**Required fix:**

When building the `<IDPSSODescriptor>` element's `<KeyDescriptor>` entries, include ML-DSA keys:

```xml
<KeyDescriptor use="signing">
  <KeyInfo xmlns="http://www.w3.org/2000/09/xmldsig#">
    <X509Data>
      <X509Certificate>... ML-DSA certificate ...</X509Certificate>
    </X509Data>
  </KeyInfo>
</KeyDescriptor>
```

**Challenge:** SAML metadata uses X.509 certificates to represent public keys. The certificate itself must be signed with ML-DSA by the issuing CA. This is the same requirement as Domain 11/13 (X.509 authentication).

## What does NOT need changing

| Component | Why it's already PQC-ready (once gaps are fixed) |
|-----------|--------------------------------------------------|
| SAML protocol message signing/verification | Uses `SignatureAlgorithm` enum — same as Domain 18 |
| Metadata XML generation | Algorithm-agnostic — will work once keys are included |
| Inbound response verification from external IdP | Uses `XMLSignatureUtil` — same as Domain 18 |

## Dependencies

### External (blocking)
1. **W3C or IETF** must define XML Signature algorithm URIs for ML-DSA (same as Domain 18)
2. **External IdPs** must support ML-DSA signatures if Keycloak sends ML-DSA-signed requests

### Internal (when standards exist)
3. **Add URIs to `SignatureAlgorithm.java`** (same as Domain 18)
4. **Fix hardcoded RS256 at line 506** (GAP-9 fix specific to this domain)
5. **Include ML-DSA keys in metadata export** (GAP-9 fix specific to this domain)
6. **Update admin UI** to expose ML-DSA options for SAML broker configuration

## GitHub Issue Status

**Partially tracked:**

- **#50292** — SAML PQC (parent issue)
- **#50295** — SAML encryption and signing (may cover this domain)

**GAP-9** is identified in the overview table. Check if #50295 covers both signing (Domain 18) and federation/brokering (Domain 20), or if they need separate sub-issues.

## What this means for operators

**Today:**

SAML federation is quantum-vulnerable (RSA/RSA-PSS signatures). No mitigation available until W3C/IETF define PQC XMLDSig URIs.

**When PQC SAML becomes available:**

1. **Keycloak update required** (same as Domain 18)
2. **External IdP coordination:**
   - If Keycloak sends ML-DSA-signed AuthnRequests, the external IdP must verify ML-DSA signatures
   - If the external IdP sends ML-DSA-signed responses, Keycloak will verify them automatically (once providers exist)
3. **Certificate management:**
   - SP metadata includes X.509 certificates
   - Certificates must be signed by a PQC-capable CA

**Migration path:**

SAML federation requires **bilateral coordination**:
- Keycloak (SP) and the external IdP must migrate to ML-DSA simultaneously
- Unlike OIDC (where each client can use a different algorithm), SAML typically uses a single algorithm per federation relationship

Consider maintaining **separate SAML identity broker configurations** during migration:
- One broker using classical algorithms (RSA/ECDSA)
- One broker using ML-DSA
- Gradually migrate users from the classical broker to the ML-DSA broker

## Related Domains

- **Domain 18** — SAML Assertion & Document Signing (shared GAP-2 and similar GAP-9)
- **Domain 19** — SAML Assertion Encryption (separate ML-KEM gap)
- **Domain 59** — SAML Metadata Public Key Loader (depends on GAP-2 and GAP-9 being resolved)
- **Domain 60** — SAML Artifact Resolution (depends on GAP-2 and GAP-9)

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness