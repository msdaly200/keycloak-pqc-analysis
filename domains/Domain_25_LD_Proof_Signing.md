# Domain 25 — LD-Proof Credential Signing (Linked Data)

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

Linked Data Proofs (LD-Proofs) provide an alternative to JWT-based credential signing for verifiable credentials. Instead of wrapping the credential in a JWT, LD-Proofs embed a cryptographic signature directly in the JSON-LD credential as a `proof` object.

**How it works:**

1. Keycloak builds a JSON-LD verifiable credential
2. The credential is signed using a **cryptographic suite** (e.g., `Ed25519Signature2018`)
3. The signature is embedded as a `proof` object in the credential
4. Verifiers use the cryptographic suite to verify the proof

**Cryptographic suites** define the specific algorithms and procedures for signing/verifying LD-Proofs. Examples:
- `Ed25519Signature2018` — EdDSA with Ed25519 curve
- `EcdsaSecp256k1Signature2019` — ECDSA with secp256k1
- Future: ML-DSA suite (not yet standardized)

## Gap

**Hardcoded to Ed25519Signature2018 (GAP-7).**

### File: `LDCredentialSigner.java` (lines 76-80)

```java
if (Objects.equals(ldpProofType, Ed255192018Suite.PROOF_TYPE)) {
    return new Ed255192018Suite(signer);
}

throw new CredentialSignerException(String.format("Proof Type %s is not supported.", ldpProofType));
```

**Impact:**

- LD-Proof signing **only supports** `Ed25519Signature2018`
- Unlike JWT-VC signing (Domain 24), LD-Proofs do **not use** the `SignatureProvider` SPI
- ML-DSA support will **not be inherited automatically** when ML-DSA providers exist
- A new ML-DSA cryptographic suite must be implemented as a separate class

**Why LD-Proofs are different:**

LD-Proofs follow W3C standards for Linked Data cryptographic suites, which have different structure and canonicalization requirements than JWS/JWT. Each suite implements its own signature generation logic.

## Current PQC State

**BLOCKED**

LD-Proof signing is hardcoded to Ed25519. No ML-DSA cryptographic suite exists or is in development.

## Required Changes

### Change: Implement ML-DSA LD cryptographic suite (when spec exists)

**File:** Create new class `MlDsa2024Suite.java` (or similar)

**Steps:**

1. **Wait for W3C/IETF to define an ML-DSA LD cryptographic suite specification**
   - The suite name (e.g., `MlDsaSignature2024`)
   - The canonicalization algorithm
   - The signature format
   - The proof verification procedure

2. **Implement the suite class** following the pattern of `Ed255192018Suite.java`:
   ```java
   public class MlDsa2024Suite implements LinkedDataCryptographicSuite {
       public static final String PROOF_TYPE = "MlDsaSignature2024"; // hypothetical
       // ... implement signing/verification logic
   }
   ```

3. **Update `LDCredentialSigner.java`** to recognize the new suite:
   ```java
   if (Objects.equals(ldpProofType, Ed255192018Suite.PROOF_TYPE)) {
       return new Ed255192018Suite(signer);
   } else if (Objects.equals(ldpProofType, MlDsa2024Suite.PROOF_TYPE)) {
       return new MlDsa2024Suite(signer);
   }
   ```

4. **Update admin UI** to expose ML-DSA LD-Proof types in credential configuration

## What does NOT need changing

| Component | Why it's already PQC-ready (once suite exists) |
|-----------|------------------------------------------------|
| Proof embedding logic | Algorithm-agnostic — works with any `LinkedDataCryptographicSuite` |
| Credential serialization | JSON-LD format is independent of signature algorithm |
| Verification process | Delegated to the suite implementation |

## Dependencies

### External (blocking)
1. **W3C or IETF** must define a Linked Data cryptographic suite for ML-DSA
2. **JSON-LD canonicalization** for PQC signatures (if different from URDNA2015)
3. **Verifiable Credentials ecosystem** must support ML-DSA LD-Proofs

### Internal (when standards exist)
4. **Implement ML-DSA suite class** (GAP-7 fix)
5. **Update suite selection logic** in `LDCredentialSigner.java`
6. **ML-DSA `SignatureProvider`** (tracked under [#48821](https://github.com/keycloak/keycloak/issues/48821) / [#48824](https://github.com/keycloak/keycloak/issues/48824)) — the suite implementation may delegate to this for actual signing

## GitHub Issue Status

**GAP-7** is identified in the overview table but does not have a dedicated GitHub issue.

**Priority:** MEDIUM (spec-gated on W3C/IETF)

**Recommended approach:**

Create a **placeholder issue** under [#43690](https://github.com/keycloak/keycloak/issues/43690):
- **Title:** "OID4VC: implement ML-DSA Linked Data cryptographic suite (pending W3C spec)"
- **Description:**
  > LD-Proof credential signing is hardcoded to `Ed25519Signature2018`. Unlike JWT-VC signing (Domain 24), LD-Proofs do not use the `SignatureProvider` SPI and will not inherit ML-DSA support automatically.
  >
  > **Action required:** Once W3C/IETF defines an ML-DSA LD cryptographic suite specification:
  > 1. Implement the new suite class (e.g., `MlDsa2024Suite`)
  > 2. Update `LDCredentialSigner.java` to recognize the new proof type
  > 3. Update admin UI to expose ML-DSA LD-Proof options
  >
  > **Blocked on:** W3C/IETF Linked Data cryptographic suite specification for ML-DSA
  >
  > **Note:** This is **separate from** the general ML-DSA provider work (#48824). LD-Proofs require their own suite implementation.
- **Label:** `spec-gated`, `pqc-readiness`
- **Tracks:** GAP-7

## What this means for operators

**Today:**

LD-Proof credentials are quantum-vulnerable (Ed25519 only). No mitigation available until W3C/IETF define ML-DSA LD cryptographic suites.

**Alternative:** Use **JWT-VC or SD-JWT format** (Domain 24) instead of LD-Proofs for OID4VC credentials. JWT-based formats will support ML-DSA automatically once providers exist.

**When ML-DSA LD-Proofs become available (timeline unknown):**

1. **Keycloak update required** to implement the ML-DSA suite
2. **Credential type configuration:** Switch LD-Proof credentials to use the new ML-DSA suite
3. **Verifier support:** Wallets and verifiers must support the new ML-DSA LD cryptographic suite
4. **No automatic migration:** Unlike JWT-VC (where adding providers enables ML-DSA globally), LD-Proofs require explicit suite implementation

**Recommendation:** If PQC readiness is a priority, prefer JWT-VC or SD-JWT over LD-Proofs for OID4VC deployments.

## Related Domains

- **Domain 24** — JWT-VC / SD-JWT Credential Signing (SPI-driven, will support ML-DSA automatically)
- **Domain 56** — SD-JWT Issuer Signing (similar JWT-based signing)

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
