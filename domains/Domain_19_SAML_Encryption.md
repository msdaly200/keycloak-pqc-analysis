# Domain 19 — SAML Assertion Encryption

[← Back to PQC Overview](pqc_overview.html)

## What is this?

SAML assertions can be encrypted using XML Encryption to protect sensitive user attributes during transmission. Keycloak supports encrypting outbound assertions (when acting as IdP) and decrypting inbound assertions (when acting as SP in a federation scenario).

**How it works:**

1. **Content encryption:** The assertion is encrypted with a symmetric key (AES-CBC or AES-GCM)
2. **Key transport:** The symmetric key is encrypted with the recipient's public key using RSA-OAEP or RSA1_5
3. The encrypted assertion and encrypted key are embedded in the SAML response as an `<EncryptedAssertion>` element

This is analogous to JWE (JSON Web Encryption) in OIDC, but uses XML Encryption standards instead.

## Gap

**No ML-KEM key transport support.**

### File: `XMLEncryptionUtil.java` (line 77, 186-203)

```java
private static final String RSA_ENCRYPTION_SCHEME = XMLCipher.RSA_OAEP_11;

// Later in encryptAssertion():
if ((XMLCipher.RSA_OAEP.equals(keyEncryptionAlgorithm) || XMLCipher.RSA_OAEP_11.equals(keyEncryptionAlgorithm))) {
    // ... RSA-OAEP key wrapping
}
```

The encryption utility only supports:
- **RSA-OAEP** (`http://www.w3.org/2001/04/xmlenc#rsa-oaep`)
- **RSA-OAEP-256** (`http://www.w3.org/2009/xmlenc11#rsa-oaep-256`) — same as RSA_OAEP_11
- **RSA1_5** (`http://www.w3.org/2001/04/xmlenc#rsa-1_5`) — deprecated

There is **no ML-KEM key encapsulation** path.

**Impact:**

Even if the JVM's XML Encryption library supports ML-KEM (via BouncyCastle), Keycloak's `XMLEncryptionUtil` does not have code to invoke it.

**When can ML-KEM be added?**

W3C and/or IETF must define XML Encryption algorithm URIs for ML-KEM (FIPS 203). The current XML Encryption specifications do not include post-quantum key encapsulation mechanisms.

## Current PQC State

**BLOCKED**

No ML-KEM key transport support exists. SAML assertion encryption is quantum-vulnerable (RSA-OAEP).

## Required Changes

### Change: Add ML-KEM key encapsulation support (when W3C/IETF defines URIs)

**File:** `XMLEncryptionUtil.java`

**Location:** Throughout the encryption/decryption methods

**Current code (simplified):**
```java
if (XMLCipher.RSA_OAEP.equals(keyEncryptionAlgorithm)) {
    // Wrap the AES key with RSA-OAEP
}
```

**Proposed fix (when ML-KEM URIs exist):**
```java
public static final String ML_KEM_768 = "http://www.w3.org/[future-spec]#ml-kem-768"; // hypothetical URI

// In encryptAssertion():
if (XMLCipher.RSA_OAEP.equals(keyEncryptionAlgorithm)) {
    // ... existing RSA-OAEP path
} else if (ML_KEM_768.equals(keyEncryptionAlgorithm)) {
    // Encapsulate the AES key with ML-KEM
    // 1. Call ML-KEM encapsulation: (ciphertext, sharedSecret) = Encaps(publicKey)
    // 2. Derive AES key from sharedSecret using KDF
    // 3. Embed ciphertext in <EncryptedKey>
}
```

**Similar changes needed for:**
- `decryptAssertion()` — ML-KEM decapsulation
- Key agreement parameter handling (ML-KEM doesn't use MGF1 or digest algorithms)

**Algorithm URI registration:**

Add ML-KEM URIs to `SAMLEncryptionAlgorithms.java` (if such a constants file exists) or directly in `XMLEncryptionUtil`.

## What does NOT need changing

| Component | Why it's already PQC-ready (once gap is fixed) |
|-----------|------------------------------------------------|
| Content encryption (AES-CBC / AES-GCM) | Symmetric encryption is quantum-safe — only the key transport needs ML-KEM |
| Encrypted assertion parsing | Algorithm-agnostic — reads the key encryption algorithm URI from `<EncryptedKey>` |
| Key storage (`KeyWrapper`) | Can hold ML-KEM keys once generated/imported |

## Dependencies

### External (blocking)
1. **W3C or IETF** must define XML Encryption algorithm URIs for ML-KEM-512, ML-KEM-768, ML-KEM-1024
2. **BouncyCastle XML Encryption library** must support ML-KEM key encapsulation (may already be available or in development)

### Internal (when standards exist)
3. **Implement ML-KEM encapsulation in `XMLEncryptionUtil`** (GAP-3 fix)
4. **Add realm/client configuration** for SAML encryption algorithm selection (currently hardcoded to RSA-OAEP)
5. **Update admin UI** to expose ML-KEM options
6. **ML-KEM key generation provider** for SAML ENC keys (Domain 29 — may share with OIDC JWE key provider)

## GitHub Issue Status

**Partially tracked:**

- [#50292](https://github.com/keycloak/keycloak/issues/50292) — SAML PQC (parent issue)
- [#50295](https://github.com/keycloak/keycloak/issues/50295) — SAML encryption (ML-KEM) (likely covers this domain)

**GAP-3** is identified in the overview table and should be tracked under [#50295](https://github.com/keycloak/keycloak/issues/50295).

## What this means for operators

**Today:**

SAML assertion encryption is quantum-vulnerable (RSA-OAEP key transport). An attacker with a future quantum computer could:
1. Record encrypted SAML assertions today
2. Decrypt them later using Shor's algorithm to break RSA

**When PQC XML Encryption becomes available (timeline unknown):**

1. **Keycloak update required:** Add ML-KEM key encapsulation to `XMLEncryptionUtil`
2. **Realm configuration:** Set SAML encryption algorithm to ML-KEM
3. **Key migration:** Generate or import ML-KEM ENC keys for SAML use
4. **Federation partners must support ML-KEM:**
   - External SPs receiving encrypted assertions from Keycloak must support ML-KEM decapsulation
   - External IdPs sending encrypted assertions to Keycloak must use ML-KEM

**Migration challenge:**

Unlike signing (where both parties can verify both classical and PQC), **encryption is one-way**:
- If Keycloak sends ML-KEM-encrypted assertions, the SP must support ML-KEM decryption **immediately**
- No graceful fallback to classical encryption if decryption fails

This requires **coordinated migration** with federation partners.

**Alternative:** Consider whether assertion encryption is necessary in modern TLS 1.3 environments (which provide transport-layer encryption). If assertions are only sent over HTTPS, the encryption may be redundant.

## Related Domains

- **Domain 2** — ID Token / JARM Encryption (OIDC equivalent; uses JWE with ML-KEM `CekManagementProvider`)
- **Domain 8** — JAR Encrypted Request Object Decryption (OIDC inbound JWE; similar to SAML inbound encryption)
- **Domain 18** — SAML Assertion Signing (separate gap for ML-DSA signatures)
- **Domain 29** — Generated RSA Encryption Key Provider (will need ML-KEM equivalent for SAML ENC keys)

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness