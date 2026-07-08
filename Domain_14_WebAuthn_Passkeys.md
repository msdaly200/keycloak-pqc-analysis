# Domain 14 — WebAuthn / Passkeys (FIDO2)

## What is this?

WebAuthn (Web Authentication) is a W3C standard that allows users to authenticate using FIDO2 authenticators — hardware security keys, platform authenticators (TouchID, Windows Hello), or passkeys. Instead of passwords, users prove possession of a cryptographic key stored in the authenticator.

During registration:
1. Keycloak sends a challenge to the browser
2. The browser requests the authenticator to create a new key pair
3. The authenticator generates a key pair and returns the public key + attestation
4. Keycloak verifies the attestation and stores the public key

During authentication:
1. Keycloak sends a challenge
2. The browser requests the authenticator to sign the challenge with the private key
3. The authenticator returns a signed assertion
4. Keycloak verifies the signature using the stored public key

**Key algorithms used:**
- **ECDSA** with P-256, P-384, P-521 curves (COSE algorithm identifiers -7, -35, -36)
- **RSA-PKCS1v15** with SHA-256, SHA-384, SHA-512 (COSE identifiers -257, -258, -259)
- **EdDSA** with Ed25519 (COSE identifier -8)

These algorithms are **quantum-vulnerable**. Post-quantum algorithms would need:
- New COSE algorithm identifiers assigned by IANA
- Updates to the FIDO2/WebAuthn specifications
- Support in authenticator hardware/firmware
- Support in browser WebAuthn APIs

## Gap

**Blocked on external standards — no Keycloak action possible today.**

### The COSE registry has no PQC algorithm assignments

The [IANA COSE Algorithms Registry](https://www.iana.org/assignments/cose/cose.xhtml#algorithms) does not yet define COSE algorithm identifiers for ML-DSA, FN-DSA, or SLH-DSA.

**File:** `WebAuthnRegister.java` (lines 372-408)

```java
private static List<Long> convertSignatureAlgorithms(List<String> signatureAlgorithmsList) {
    List<Long> algs = new ArrayList<>();
    for (String s : signatureAlgorithmsList) {
        switch (s) {
        case Algorithm.ES256 :
            algs.add(COSEAlgorithmIdentifier.ES256.getValue());  // COSE -7
            break;
        case Algorithm.RS256 :
            algs.add(COSEAlgorithmIdentifier.RS256.getValue());  // COSE -257
            break;
        // ... ES384, RS384, ES512, RS512, EdDSA
        default:
            // NOP — unrecognized algorithms are silently skipped
        }
    }
    return algs;
}
```

This method converts Keycloak's algorithm names (like "ES256") to COSE algorithm identifiers (like -7) using the **WebAuthn4J library's `COSEAlgorithmIdentifier` enum**. WebAuthn4J tracks the COSE registry, which does not yet include ML-DSA.

**What would need to change (when standards exist):**
1. IANA assigns COSE algorithm identifiers for ML-DSA, FN-DSA, SLH-DSA (e.g., hypothetical `-50` for ML-DSA-44)
2. WebAuthn4J library adds the new `COSEAlgorithmIdentifier` enum values
3. Keycloak adds new `case` statements to the switch block above
4. Keycloak updates the admin UI to expose the new algorithms in the WebAuthn Policy configuration

### The FIDO2/WebAuthn spec does not define PQC support

The [W3C WebAuthn Level 2 specification](https://www.w3.org/TR/webauthn-2/) and [FIDO2 specifications](https://fidoalliance.org/specifications/) do not yet define how authenticators should generate, store, or use post-quantum keys.

Open questions that the FIDO Alliance and W3C must answer:
- How do authenticators signal PQC capability to the relying party?
- What attestation formats support PQC certificates?
- How are PQC public keys encoded in the `attestedCredentialData`?
- Are hybrid classical+PQC modes supported for transition?

## Current PQC State

**EXTERNAL DEPENDENCY**

Keycloak's WebAuthn implementation is **entirely dependent** on:
1. **FIDO Alliance** defining PQC support in the FIDO2 specification
2. **W3C** updating the WebAuthn specification to include PQC algorithms
3. **IANA** assigning COSE algorithm identifiers for ML-DSA, FN-DSA, SLH-DSA
4. **WebAuthn4J library** (Keycloak's dependency) adding support for the new COSE identifiers
5. **Browser vendors** (Chrome, Firefox, Safari, Edge) implementing PQC support in their WebAuthn APIs
6. **Authenticator manufacturers** (Yubikey, Google Titan, Apple Secure Enclave, Windows Hello) shipping firmware/hardware that supports PQC key generation and signing

**Timeline:** The FIDO Alliance has acknowledged post-quantum cryptography as a future concern but has not published a roadmap. This is likely a **multi-year effort** (2027-2030+) given the hardware refresh cycles required.

## Required Changes

**No Keycloak changes actionable today.**

### When standards become available

Once the FIDO Alliance, W3C, and IANA have defined PQC support:

**Step 1: Update dependency**
- Upgrade to a version of **WebAuthn4J** that includes the new COSE algorithm identifiers

**Step 2: Update algorithm conversion logic**

**File:** `WebAuthnRegister.java` (lines 372-408)

Add new cases to the `convertSignatureAlgorithms()` switch statement:
```java
case "ML-DSA-44":  // hypothetical Keycloak algorithm name
    algs.add(COSEAlgorithmIdentifier.ML_DSA_44.getValue());  // hypothetical COSE -50
    break;
case "ML-DSA-65":
    algs.add(COSEAlgorithmIdentifier.ML_DSA_65.getValue());  // hypothetical COSE -51
    break;
case "ML-DSA-87":
    algs.add(COSEAlgorithmIdentifier.ML_DSA_87.getValue());  // hypothetical COSE -52
    break;
// Similar for FN-DSA, SLH-DSA
```

**Step 3: Update admin UI**

The WebAuthn Policy configuration UI (realm settings) would need to expose the new algorithm options in the "Signature Algorithms" multi-select dropdown.

**Step 4: Update defaults**

Consider whether `DEFAULT_WEBAUTHN_POLICY_SIGNATURE_ALGORITHMS` (currently `ES256,RS256`) should include a PQC algorithm once they're available.

**Step 5: Documentation**

Update operator documentation to explain:
- Which authenticators support PQC (hardware compatibility matrix)
- Browser compatibility requirements
- Migration path from classical to PQC WebAuthn credentials

## What does NOT need changing

| Component | Why it's already PQC-ready (once standards exist) |
|-----------|---------------------------------------------------|
| WebAuthn4J library integration | Keycloak delegates all WebAuthn verification to WebAuthn4J via `WebAuthnRegistrationManager` — once WebAuthn4J supports new COSE identifiers, the verification path is automatic |
| Credential storage (`WebAuthnCredentialModel`) | Stores the public key as a byte array — algorithm-agnostic storage format |
| Challenge generation | Uses random bytes — no algorithm-specific logic |
| Attestation verification | Delegated to WebAuthn4J's `AttestationStatementVerifier` — will support PQC attestation formats once WebAuthn4J implements them |
| User verification checks | Business logic unrelated to signature algorithms |
| AAGUID filtering (`acceptableAaguids`) | GUID-based allowlist — independent of algorithms |

## Dependencies

### External (blocking)
1. **FIDO Alliance** — must publish PQC extensions to FIDO2 CTAP specification
2. **W3C WebAuthn Working Group** — must update WebAuthn spec to define PQC algorithm support
3. **IANA COSE Registry** — must assign algorithm identifiers for ML-DSA, FN-DSA, SLH-DSA
4. **Authenticator vendors** — must ship PQC-capable hardware/firmware (Yubikey 6+, platform authenticators)
5. **Browser vendors** — must implement PQC support in WebAuthn API (Chrome, Firefox, Safari, Edge)

### Internal (when standards exist)
6. **WebAuthn4J library** — must add `COSEAlgorithmIdentifier` enum values for PQC algorithms
7. **Keycloak algorithm constants** — add ML-DSA/FN-DSA/SLH-DSA to `Algorithm.java`
8. **Admin UI updates** — expose new algorithms in WebAuthn Policy configuration

## What this means for operators

**Today:**
- WebAuthn/Passkeys are quantum-vulnerable — attackers with future quantum computers could forge signatures from recorded authenticator assertions
- No mitigation available until FIDO Alliance and W3C define PQC support

**When PQC WebAuthn becomes available (2027+):**
1. Operators will need to **update authenticator hardware** — existing Yubikeys, TouchID sensors, and platform authenticators may not support PQC via firmware update (likely requires new hardware)
2. Users will need to **re-register** their authenticators with new PQC credentials — cannot upgrade classical credentials in-place
3. **Hybrid mode** may be supported (classical + PQC dual signatures) during the transition period — depends on FIDO Alliance design
4. Keycloak will require a **minor version update** to add the new COSE identifier mappings (one-line code changes per algorithm)

**Migration path:**
- Operators should monitor FIDO Alliance announcements for PQC roadmap
- Plan for a multi-year hardware refresh cycle once PQC authenticators are available
- Consider password-based or X.509 certificate-based authentication as PQC-ready alternatives in high-security environments today

## GitHub Issue Status

No Keycloak issue needed.

This domain is **entirely spec-blocked** — there is no Keycloak work to track until external standards are published.

Operators should be informed in **#48823 (Operator migration guidance)** that:
- WebAuthn/Passkeys are quantum-vulnerable
- No mitigation available today
- FIDO Alliance and W3C are responsible for defining PQC support (multi-year timeline)
- When available, Keycloak will require a minor update and operators will need new authenticator hardware

## Related Domains

- **All other authentication domains** use SPI-driven signature verification and will support PQC once providers exist
- **WebAuthn is unique** in being externally blocked by hardware standards

## Standards to monitor

- [FIDO Alliance specifications](https://fidoalliance.org/specifications/)
- [W3C WebAuthn Working Group](https://www.w3.org/groups/wg/webauthn/)
- [IANA COSE Algorithms Registry](https://www.iana.org/assignments/cose/cose.xhtml#algorithms)
- [WebAuthn4J library releases](https://github.com/webauthn4j/webauthn4j)

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness