# Domain 14 — WebAuthn / Passkeys (FIDO2)

[← Back to PQC Overview](../pqc_overview.html)

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

**Blocked on ecosystem readiness — standards are in place, but no production authenticators or browsers support PQC yet.**

### ✅ UPDATE (2025-2026): COSE registry HAS ML-DSA assignments

The [IANA COSE Algorithms Registry](https://www.iana.org/assignments/cose/cose.xhtml#algorithms) **now has ML-DSA permanently registered** with "Recommended" status (registered Apr/Jul 2025):

- **ML-DSA-44**: COSE algorithm identifier `-48`
- **ML-DSA-65**: COSE algorithm identifier `-49`
- **ML-DSA-87**: COSE algorithm identifier `-50`

Based on [draft-ietf-cose-dilithium](https://datatracker.ietf.org/doc/draft-ietf-cose-dilithium/) ("ML-DSA for JOSE and COSE").

**FN-DSA (FIPS 206, FALCON)** is expected late 2026. **SLH-DSA (FIPS 205)** status TBD.

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

This method converts Keycloak's algorithm names (like "ES256") to COSE algorithm identifiers (like -7) using the **WebAuthn4J library's `COSEAlgorithmIdentifier` enum**.

**What needs to change:**
1. ✅ ~~IANA assigns COSE algorithm identifiers~~ — **DONE**: ML-DSA-44 (`-48`), ML-DSA-65 (`-49`), ML-DSA-87 (`-50`)
2. ⏳ **WebAuthn4J library adds support** — ML-DSA implementation exists in [PR #1337](https://github.com/webauthn4j/webauthn4j/pull/1337), blocked on FIDO conformance test vectors
3. ⏳ Keycloak adds new `case` statements to the switch block above
4. ⏳ Keycloak updates the admin UI to expose the new algorithms in the WebAuthn Policy configuration

### ✅ UPDATE: WebAuthn Level 3 is algorithm-agnostic

**[W3C WebAuthn Level 3](https://www.w3.org/TR/webauthn-3/)** (Candidate Recommendation, Jan 2026) is **algorithm-agnostic** — PQC algorithms can be added without spec changes. The WebAuthn spec simply references the COSE algorithm registry.

**FIDO Alliance** published a [PQC white paper](https://fidoalliance.org/white-paper-addressing-fido-alliances-technologies-in-post-quantum-world/) (Feb 2024) committing to seamless transition. They note the impact goes beyond signatures — CTAP PIN protocol uses ECDH which also needs PQC-safe key exchange.

**IETF [draft-vitap-ml-dsa-webauthn-04](https://datatracker.ietf.org/doc/draft-vitap-ml-dsa-webauthn/)** describes ML-DSA usage in WebAuthn (individual submission, not yet a working group document).

**Standards layer is largely complete** — the blocker is now ecosystem implementation (browsers, authenticators, conformance tests).

## Current PQC State

**EXTERNAL DEPENDENCY** — Standards ready, ecosystem not ready.

### Progress Update (as of investigation in #48826)

**Standards layer (✅ COMPLETE):**
1. ✅ **NIST** finalized ML-DSA (FIPS 204) and SLH-DSA (FIPS 205) in Aug 2024
2. ✅ **IANA** assigned COSE algorithm identifiers: ML-DSA-44 (`-48`), ML-DSA-65 (`-49`), ML-DSA-87 (`-50`)
3. ✅ **WebAuthn Level 3** (W3C CR, Jan 2026) is algorithm-agnostic
4. ✅ **FIDO Alliance** published PQC commitment white paper (Feb 2024)

**Library layer (⏳ READY, waiting on conformance tests):**
5. ⏳ **WebAuthn4J** — ML-DSA support implemented in [PR #1337](https://github.com/webauthn4j/webauthn4j/pull/1337) based on [draft-ietf-cose-dilithium](https://datatracker.ietf.org/doc/draft-ietf-cose-dilithium/). **Blocked on FIDO Alliance conformance test vectors** — without those, interoperability testing is not possible. Progress tracked at [webauthn4j#1200](https://github.com/webauthn4j/webauthn4j/issues/1200).
   - **Requires JDK 24+** ([JEP 497](https://openjdk.org/jeps/497)) OR BouncyCastle (already a Keycloak dependency)

**Platform crypto layer (⏳ READY, not wired to WebAuthn yet):**
6. ⏳ **Microsoft Windows CNG** — ML-DSA support GA (Nov 2025)
7. ⏳ **Apple CryptoKit** — ML-DSA support in iOS 26 / macOS Tahoe
   - **Note:** Crypto primitives are ready with hardware-backed key storage, but the WebAuthn/platform authenticator layer hasn't wired them in yet

**Browser layer (❌ NOT READY):**
8. ❌ **Browser vendors** (Chrome, Firefox, Safari, Edge) — No browser has been tested or confirmed to pass through ML-DSA to authenticators

**Hardware authenticator layer (❌ NOT READY):**
9. ❌ **Authenticator manufacturers** — **No production PQC-capable FIDO2 authenticator exists**. Only research prototypes:
   - **Google OpenSK** — Hybrid ECDSA + Dilithium (2023 research prototype)
   - **SandboxAQ + Nitrokey** — Dilithium3 for WebAuthn, Kyber768 for CTAP2 (2023 research prototype)
   - **Yubico** — Demonstrated ML-DSA on YubiKey-class hardware, but **new hardware required** (existing YubiKeys can't be firmware-updated)
   - **Cryptsoft + FEITIAN** — Hybrid ML-DSA-65 PoC (Apr 2026, not FIDO2 certified)
   - **"The Qey"** — Academic ML-DSA FIDO2 implementation
   - **RS-Key (RP2350)** — Experimental ML-DSA-44 firmware for RP2350 board ([GitHub](https://github.com/TheMaxMur/RS-Key))

**Realistic Timeline:** **2027-2028** for first production browser + authenticator support, given hardware refresh cycles.

## Required Changes

**Keycloak changes can be prepared now, but won't be usable until WebAuthn4J merges ML-DSA support.**

### When WebAuthn4J merges ML-DSA support

Once WebAuthn4J merges [PR #1337](https://github.com/webauthn4j/webauthn4j/pull/1337):

**Step 1: Update dependency**
- Upgrade to the WebAuthn4J version that includes ML-DSA `COSEAlgorithmIdentifier` enum values
- **Requires JDK 24+** OR ensure BouncyCastle ML-DSA support is used (discuss with webauthn4j maintainer)

**Step 2: Update algorithm conversion logic**

**File:** `WebAuthnRegister.java` (lines 372-408)

Add new cases to the `convertSignatureAlgorithms()` switch statement:
```java
case "ML-DSA-44":  // Keycloak algorithm name
    algs.add(COSEAlgorithmIdentifier.ML_DSA_44.getValue());  // COSE -48
    break;
case "ML-DSA-65":
    algs.add(COSEAlgorithmIdentifier.ML_DSA_65.getValue());  // COSE -49
    break;
case "ML-DSA-87":
    algs.add(COSEAlgorithmIdentifier.ML_DSA_87.getValue());  // COSE -50
    break;
// FN-DSA when COSE identifiers are assigned
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

### External (blocking production use)
1. ✅ ~~**FIDO Alliance**~~ — White paper published, CTAP PIN protocol PQC path identified
2. ✅ ~~**W3C WebAuthn Working Group**~~ — WebAuthn Level 3 is algorithm-agnostic
3. ✅ ~~**IANA COSE Registry**~~ — ML-DSA-44/65/87 assigned (`-48`/`-49`/`-50`)
4. ⏳ **FIDO Alliance conformance tests** — needed before WebAuthn4J can merge ML-DSA support
5. ❌ **Authenticator vendors** — must ship production PQC-capable hardware/firmware (Yubikey 6+, platform authenticators with PQC WebAuthn integration)
6. ❌ **Browser vendors** — must pass through ML-DSA COSE identifiers to authenticators (Chrome, Firefox, Safari, Edge)

### Internal (can be prepared now)
7. ⏳ **WebAuthn4J library** — ML-DSA PR #1337 ready, waiting on conformance tests
8. ⏳ **Keycloak JDK version** — Upgrade to JDK 24+ OR verify BouncyCastle ML-DSA integration in WebAuthn4J
9. ⏳ **Keycloak algorithm constants** — add ML-DSA-44/65/87 to `Algorithm.java`
10. ⏳ **Algorithm conversion logic** — add ML-DSA cases to `WebAuthnRegister.convertSignatureAlgorithms()`
11. ⏳ **Admin UI updates** — expose ML-DSA-44/65/87 in WebAuthn Policy configuration

## What this means for operators

**Today:**
- WebAuthn/Passkeys are quantum-vulnerable — attackers with future quantum computers could forge signatures from recorded authenticator assertions
- No mitigation available until FIDO Alliance and W3C define PQC support

**When PQC WebAuthn becomes available (estimated 2027-2028):**
1. Operators will need to **update authenticator hardware** — existing Yubikeys, TouchID sensors, and platform authenticators **cannot** support PQC via firmware update (requires new hardware, confirmed by Yubico)
2. Users will need to **re-register** their authenticators with new PQC credentials — cannot upgrade classical credentials in-place
3. **Hybrid mode** (classical + PQC dual signatures) has research prototypes (Google OpenSK, SandboxAQ/Nitrokey, Cryptsoft/FEITIAN) but is not yet standardized
4. Keycloak will require:
   - **JDK 24+ upgrade** (or BouncyCastle-based ML-DSA in WebAuthn4J)
   - **WebAuthn4J dependency upgrade** to version with ML-DSA support
   - **Minor code changes** to add ML-DSA COSE identifier mappings (3 new case statements)
   - **Admin UI update** to expose ML-DSA-44/65/87 in policy configuration

**Migration path:**
- Operators should monitor FIDO Alliance announcements for PQC roadmap
- Plan for a multi-year hardware refresh cycle once PQC authenticators are available
- Consider password-based or X.509 certificate-based authentication as PQC-ready alternatives in high-security environments today

## GitHub Issue Status

**Tracked under:**
- **[#48826](https://github.com/keycloak/keycloak/issues/48826)** - Investigate what is required for WebAuthn/passkeys to be PQC ready (CLOSED - investigation complete)
- **[#50084](https://github.com/keycloak/keycloak/issues/50084)** - PQC support for WebAuthn/Passkeys (OPEN - implementation milestone)
  - **[#50085](https://github.com/keycloak/keycloak/issues/50085)** - Upgrade webauthn4j to a version with ML-DSA support (OPEN)
  - **[#50086](https://github.com/keycloak/keycloak/issues/50086)** - Add ML-DSA COSE algorithm IDs to WebAuthn policies (OPEN)

**Status:** Standards and library layer ready, **blocked on ecosystem** (conformance tests, browsers, authenticators).

Operators should be informed in **#48823 (Operator migration guidance)** that:
- WebAuthn/Passkeys are quantum-vulnerable
- Standards are ready, but **no production authenticators or browsers support PQC yet** (realistic timeline: 2027-2028)
- COSE algorithm identifiers exist: ML-DSA-44 (`-48`), ML-DSA-65 (`-49`), ML-DSA-87 (`-50`)
- WebAuthn4J has ML-DSA implementation ready, waiting on FIDO conformance tests
- When ecosystem support arrives, Keycloak will require JDK 24+ (or BouncyCastle integration) and minor code updates
- **New authenticator hardware will be required** — existing devices cannot be firmware-updated for PQC

## Related Domains

- **All other authentication domains** use SPI-driven signature verification and will support PQC once providers exist
- **WebAuthn is unique** in being externally blocked by hardware standards

## Standards & Implementation to Monitor

- [FIDO Alliance PQC White Paper](https://fidoalliance.org/white-paper-addressing-fido-alliances-technologies-in-post-quantum-world/)
- [W3C WebAuthn Level 3](https://www.w3.org/TR/webauthn-3/) (algorithm-agnostic)
- [IANA COSE Algorithms Registry](https://www.iana.org/assignments/cose/cose.xhtml#algorithms) (ML-DSA-44/65/87 registered)
- [WebAuthn4J PR #1337](https://github.com/webauthn4j/webauthn4j/pull/1337) (ML-DSA implementation)
- [WebAuthn4J issue #1200](https://github.com/webauthn4j/webauthn4j/issues/1200) (PQC progress tracking)
- [draft-ietf-cose-dilithium](https://datatracker.ietf.org/doc/draft-ietf-cose-dilithium/) (ML-DSA for JOSE and COSE)
- [draft-vitap-ml-dsa-webauthn](https://datatracker.ietf.org/doc/draft-vitap-ml-dsa-webauthn/) (ML-DSA in WebAuthn)
- [JDK 24 JEP 497](https://openjdk.org/jeps/497) (Module-Lattice-Based Digital Signature Algorithm)

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness