# core-authn Team: New PQC GitHub Issues

**Generated from:** `pqc_overview.html`, `gh_issues/Github_PQC_issues_overview.md`, `gh_issues/core-authn_gap_analysis.md`  
**Analysis date:** 2026-09-15  
**Parent feature issue:** [#43690 — Post-Quantum Cryptography (PQC) readiness](https://github.com/keycloak/keycloak/issues/43690)

---

## Gap-to-Issue Mapping Summary

The table below shows the cross-reference between gaps identified in `core-authn_gap_analysis.md`
and the existing issues in `Github_PQC_issues_overview.md`. Only gaps **not already covered** by
an existing filed issue are carried forward as new issues.

| Gap / Domain | Description | Existing Issue? | New Issue? |
|---|---|---|---|
| GAP-1 | JWK Thumbprint — AKP (ML-DSA) keys | Proposed in overview; **no filed issue** | ✅ Issue 1 |
| GAP-22 | Attestation-based client auth algorithm enforcement | **No filed issue** | ✅ Issue 2 |
| GAP-30 | Client Asymmetric Signature Verifier — RSA guard blocks AKP | **No filed issue** | ✅ Issue 3 |
| GAP-6 | FIPS 140-2 / X.509 browser + mTLS client auth — ML-DSA blocked | **No filed issue** | ✅ Issue 4 |
| GAP-10 | IdP Broker — hardcoded RS256/HS256 fallback in `AbstractOAuth2IdentityProvider` | **No filed issue** | ✅ Issue 5 |
| GAP-8 / Domain 41 | FAPI client policy allowlist — ML-DSA actively rejected at client auth layer | **No filed issue** specific to core-authn's client authenticator policy surface | ✅ Issue 6 |
| Domain 14 / WebAuthn | `WebAuthnRegister` / `WebAuthnPolicy` PQC COSE algorithm readiness (Keycloak-side wiring) | #50085/#50086 cover `webauthn4j` upgrade and COSE ID addition only; **no issue** for `WebAuthnRegister` conversion logic and `WebAuthnPolicy` admin surface | ✅ Issue 7 |
| Domain 38 / GAP-13 | Admin CLI `kcadm.sh`/`kcreg.sh` — hardcoded RS256 for `private_key_jwt` client assertions | **No filed issue** | ✅ Issue 8 |
| GAP-28 | Client SDK `JWTClientCredentialsProvider` — missing AKP switch case | **No filed issue** | ✅ Issue 9 |
| Password hashing (#48828) | Password hashing PQC review | #48828 **CLOSED** ✔ | Already covered |
| Cookies (#49865) | Cookie hashing/encryption PQC | #49865 **CLOSED** ✔ | Already covered |
| WebAuthn webauthn4j (#50085) | Upgrade webauthn4j for ML-DSA | #50085 **OPEN** ✔ | Already covered |
| WebAuthn COSE IDs (#50086) | Add ML-DSA COSE algorithm IDs to `WebAuthnPolicy` admin model | #50086 **OPEN** ✔ | Already covered |

**9 new issues required.** Issues 1–6 are CRITICAL/HIGH priority. Issues 7–9 are MEDIUM/LOW priority.

---

## New Issues

---

### Issue 1 — [CRITICAL] JWK Thumbprint: Add AKP (ML-DSA) Support to `JWKSUtils.computeThumbprint()`

**Labels:** `area/authentication`, `area/oidc`, `kind/enhancement`, `team/core-authn`, `pqc`  
**Priority:** CRITICAL  
**Suggested Parent Issue:** [#43692 — Support ML-DSA for OAuth and OpenID Connect](https://github.com/keycloak/keycloak/issues/43692)  
**Suggested Assignee Team:** core-authn (primary); core-protocols (review)

---

#### Summary

`JWKSUtils.computeThumbprint()` throws `UnsupportedOperationException` for AKP (ML-DSA) key types because `JWK_THUMBPRINT_REQUIRED_MEMBERS` does not include an AKP entry. This directly blocks two core-authn-owned authentication flows:

1. **DPoP** (`dpop_jkt` binding) — the `JWKSUtils.computeThumbprint()` path in `DPoPUtil` will fail at runtime for any ML-DSA DPoP proof.
2. **Attestation-Based Client Authentication** — `cnf.jwk`-to-`KeyWrapper` thumbprint computation in `AttestationBasedClientAuthenticator` will throw before verification can proceed.

#### Background / Context

NIST FIPS 204 (ML-DSA, formerly CRYSTALS-Dilithium) defines the AKP key type used throughout Keycloak's PQC groundwork. The codebase already has `AKPPublicJWK.java`, `JWKBuilder.akp()`, and `JWKParser` AKP parsing — the serialisation layer is ready. However, `JWKSUtils` was not updated to include AKP in the thumbprint required-members map.

Per RFC 7638, a JWK thumbprint is computed over the lexicographically ordered, required members of the JWK. For AKP keys this is analogous to the OKP case (`crv`, `kty`, `x`), except that the AKP public key byte array is significantly larger (1312 bytes for ML-DSA-44, 1952 for ML-DSA-65, 2592 for ML-DSA-87). The required members for AKP JWK thumbprints must be specified and implemented before DPoP can function with ML-DSA keys.

This is a **CRITICAL** blocker because DPoP is a widely deployed client authentication enhancement and is a core-authn-owned code path. Without this fix, any realm that enables ML-DSA and has clients using DPoP will receive runtime exceptions.

#### Scope

**In scope:**
- Add `KeyType.AKP` entry to `JWK_THUMBPRINT_REQUIRED_MEMBERS` in `core/…/util/JWKSUtils.java` with the correct required member list for AKP JWKs.
- Implement `computeThumbprint()` for AKP JWKs, following the same SHA-256 hash-over-canonical-JSON pattern used for RSA/EC/OKP.
- Unit tests covering thumbprint computation for ML-DSA-44, ML-DSA-65, and ML-DSA-87 key sizes.
- Integration test verifying DPoP `dpop_jkt` binding with an ML-DSA key pair does not throw.

**Out of scope:**
- ML-DSA `SignatureProviderFactory` implementation (tracked separately under #43692).
- JWK serialisation/parsing changes (already implemented in `AKPPublicJWK.java`).
- SHA-3 or alternative thumbprint hash algorithms.

#### Acceptance Criteria

- [ ] `JWKSUtils.computeThumbprint()` returns a non-null, deterministic thumbprint for an AKP (`KeyType.AKP`) `KeyWrapper` with a valid ML-DSA public key.
- [ ] The AKP thumbprint is computed using the canonicalized, lexicographically ordered required members as defined for the AKP JWK type (minimum: `kty`, `pub` or equivalent AKP-specific required members per the draft AKP JWK specification).
- [ ] `computeThumbprint()` does **not** throw `UnsupportedOperationException` for any AKP key size (ML-DSA-44, -65, -87).
- [ ] Existing unit tests for RSA, EC, and OKP thumbprints continue to pass without modification.
- [ ] New unit tests assert thumbprint byte-for-byte reproducibility (same key → same thumbprint) for all three ML-DSA parameter sets.
- [ ] An integration test confirms that a DPoP proof JWT containing a `jwk` claim with an ML-DSA public key successfully produces a `dpop_jkt` binding value at the token endpoint.

#### Dependencies

- `JWKBuilder.akp()` and `JWKParser` AKP support — already implemented; no code change needed.
- `KeyType.AKP` constant — already exists.
- Specification reference for AKP JWK required thumbprint members: draft-ietf-jose-fully-specified-algorithms or the AKP JWK specification once finalised. Use `pub` key as the sole required member for AKP pending spec stabilisation.
- **Blocks:** Issue 2 (attestation-based client auth, which calls the same thumbprint function for `cnf.jwk` verification).
- **Blocks:** Full DPoP support with ML-DSA keys (Domain 15 in `pqc_overview.html`).

---

### Issue 2 — [HIGH] Attestation-Based Client Auth: Implement Algorithm Enforcement and Fix AKP `cnf.jwk` Thumbprint Path

**Labels:** `area/authentication`, `area/oidc`, `kind/enhancement`, `team/core-authn`, `pqc`  
**Priority:** HIGH  
**Suggested Parent Issue:** [#48824 — Investigate and plan what is needed for OpenID Connect and OAuth 2.0 to be PQC ready](https://github.com/keycloak/keycloak/issues/48824)  
**Suggested Assignee Team:** core-authn

---

#### Summary

`AttestationBasedClientAuthenticator.java` has two independent PQC gaps:

1. **Unimplemented `[TODO]` at line 418** — algorithm-type enforcement for client assertion JWTs is not implemented. The comment marks the enforcement as intended but deferred.
2. **AKP `cnf.jwk` thumbprint failure** — when the `cnf.jwk` claim contains an ML-DSA (AKP) key, the call to `JWKSUtils.computeThumbprint()` throws `UnsupportedOperationException` (GAP-1), halting the attestation flow entirely before verification can proceed.

Together these gaps mean attestation-based client authentication cannot be used with ML-DSA keys.

#### Background / Context

Attestation-based client authentication (`AttestationBasedClientAuthenticator.java`) verifies client identity by examining the `cnf.jwk` claim in an attestation JWT. The flow:
1. Extracts the public key from `cnf.jwk`.
2. Computes a JWK thumbprint to bind the key to the token.
3. Verifies the outer JWT signature using that key.
4. (Intended, TODO) Enforces that the key algorithm meets policy requirements.

Both the SPI-driven signature verification (step 3) and the algorithm enforcement (step 4) will eventually support ML-DSA once providers exist — but step 2 fails immediately for AKP keys today, blocking the whole flow. The `[TODO]` at step 4 is a security concern: without algorithm enforcement, a client could present a weak-algorithm key and bypass ML-DSA requirements even in a PQC-enforcing realm.

#### Scope

**In scope:**
- Fix the `cnf.jwk` → `KeyWrapper` conversion and thumbprint computation path to handle `KeyType.AKP` keys (depends on Issue 1).
- Implement the deferred algorithm enforcement at line 418: validate that the key algorithm in `cnf.jwk` matches or is at least as strong as the realm's configured minimum algorithm requirement.
- Unit/integration tests covering:
  - Attestation auth with an ML-DSA AKP `cnf.jwk` key.
  - Rejection of a classical (RSA) `cnf.jwk` key when realm policy mandates ML-DSA or stronger.
  - Acceptance of a classical key when realm policy does not enforce PQC.

**Out of scope:**
- ML-DSA `SignatureProviderFactory` (prerequisite; tracked under #43692/#43684).
- JWK thumbprint implementation (Issue 1).
- Changes to the attestation JWT format or protocol.

#### Acceptance Criteria

- [ ] `AttestationBasedClientAuthenticator` successfully processes an attestation JWT whose `cnf.jwk` contains an AKP (ML-DSA) public key without throwing `UnsupportedOperationException`.
- [ ] The algorithm enforcement at line 418 is implemented: if the realm has a PQC-minimum algorithm requirement configured, assertion keys that do not meet it are rejected with a descriptive error.
- [ ] When no algorithm policy is set, the flow behaves identically to today (backward compatible).
- [ ] New integration test: attestation-based client auth succeeds end-to-end with an ML-DSA key pair (once ML-DSA `SignatureProviderFactory` is available; test may be gated with an `@RequireProvider` annotation until then).
- [ ] No regression in existing attestation-based client auth tests.

#### Dependencies

- **Depends on:** Issue 1 (JWK thumbprint for AKP keys must be implemented first).
- **Depends on:** #43684 (ML-DSA `SignatureProviderFactory`) for full end-to-end test execution.
- **Related:** #48824 (OIDC PQC readiness spike, parent for this work).
- GAP-22 in `pqc_overview.html`.

---

### Issue 3 — [HIGH] Client Asymmetric Signature Verifier: Remove RSA-Only Guard and Add AKP Client Verifier Factory

**Labels:** `area/authentication`, `area/oidc`, `kind/enhancement`, `team/core-authn`, `pqc`  
**Priority:** HIGH  
**Suggested Parent Issue:** [#43692 — Support ML-DSA for OAuth and OpenID Connect](https://github.com/keycloak/keycloak/issues/43692)  
**Suggested Assignee Team:** core-authn

---

#### Summary

`ClientAsymmetricSignatureVerifierContext.getKey()` (line 36) contains an explicit guard:

```java
throw new VerificationException("Key Type is not RSA");
```

This unconditionally rejects every non-RSA key — including ECDSA, EdDSA, and ML-DSA (AKP) client keys — when they are routed through the RSA verifier context. ML-DSA client assertions currently have no routed verifier path at all: the RSA context rejects them, and there is no `AKPClientSignatureVerifierContext` or `AKPClientSignatureVerifierProviderFactory`. Any realm that registers an ML-DSA key for `private_key_jwt` client authentication will fail with a `VerificationException` at runtime.

#### Background / Context

Keycloak's client-side signature verifier stack has three parallel paths:
- **RSA path:** `ClientAsymmetricSignatureVerifierContext` + `AsymmetricClientSignatureVerifierProvider`
- **ECDSA path:** `ClientECDSASignatureVerifierContext` + `ECDSAClientSignatureVerifierProvider` + ES* factories
- **EdDSA path:** `ClientEdDSASignatureVerifierContext` + `EdDSAClientSignatureVerifierProvider` + factory

The ECDSA and EdDSA paths are SPI-driven and will support ML-DSA automatically once ML-DSA providers exist. The RSA path has a hard `KeyType.RSA` guard that is not SPI-driven. To enable ML-DSA `private_key_jwt` client authentication, either:
1. The RSA guard must be generalised to accept any asymmetric key type and delegate to the SPI, **or**
2. A dedicated `AKPClientSignatureVerifierContext`, `AKPClientSignatureVerifierProvider`, and `AKPClientSignatureVerifierProviderFactory` must be added, following the same pattern as the EdDSA path.

Option 2 is preferred for consistency with the existing architecture.

NIST FIPS 204 (ML-DSA) is a standardised post-quantum digital signature algorithm. Supporting it in `private_key_jwt` client authentication is a prerequisite for PQC-compliant OAuth 2.0 confidential client authentication.

#### Scope

**In scope:**
- Add `AKPClientSignatureVerifierContext.java` (analogous to `ClientEdDSASignatureVerifierContext`).
- Add `AKPClientSignatureVerifierProvider.java` (analogous to `EdDSAClientSignatureVerifierProvider`).
- Add `ML_DSA_44_ClientSignatureVerifierProviderFactory`, `ML_DSA_65_ClientSignatureVerifierProviderFactory`, `ML_DSA_87_ClientSignatureVerifierProviderFactory` (analogous to `ES256ClientSignatureVerifierProviderFactory` etc.), registered via `ServiceLoader`.
- Ensure `PublicKeyStorageManager.getClientPublicKeyWrapper()` routes AKP keys to the new factories.
- Unit tests for each ML-DSA variant.
- Integration test: `private_key_jwt` client authentication with an ML-DSA key pair.

**Out of scope:**
- Changing the existing RSA/ECDSA/EdDSA verifier paths (no modifications to existing classes).
- Algorithm negotiation or auto-detection logic.
- Key generation for clients (tracked under Issue 8 / GAP-27).

#### Acceptance Criteria

- [ ] `ClientAsymmetricSignatureVerifierContext.getKey()` does **not** throw for an AKP key type (either the guard is removed or AKP keys are routed to the new AKP context before reaching this class).
- [ ] A client configured with an ML-DSA public key can successfully authenticate via `private_key_jwt` using an ML-DSA-signed client assertion JWT.
- [ ] `ML_DSA_44`, `ML_DSA_65`, and `ML_DSA_87` are all supported through their respective provider factories.
- [ ] The new factories are listed in the appropriate `ServiceLoader` registration files.
- [ ] Existing `private_key_jwt` tests for RSA, ECDSA, and EdDSA continue to pass without modification.
- [ ] New integration test in `ClientAuthPostMethodTest` (or equivalent) covers ML-DSA `private_key_jwt` authentication end-to-end.

#### Dependencies

- **Depends on:** #43684 (ML-DSA `SignatureProviderFactory` — must exist for signature verification to work).
- **Depends on:** #44142 (Generated keys provider for ML-DSA, to create test key pairs).
- **Related:** #43692 (ML-DSA for OAuth and OpenID Connect parent).
- **Related:** Issue 9 (Client SDK `JWTClientCredentialsProvider` — client-side signing counterpart).
- GAP-30 in `pqc_overview.html`.

---

### Issue 4 — [HIGH] X.509 Client Auth and Browser Auth: Track FIPS 140-3 BC-FIPS Dependency for ML-DSA Certificate Support

**Labels:** `area/authentication`, `kind/enhancement`, `team/core-authn`, `pqc`, `area/dist/quarkus`  
**Priority:** HIGH (EXTERNAL DEPENDENCY — tracking / migration planning)  
**Suggested Parent Issue:** [#45168 — Review what is needed for PQC readiness in Keycloak](https://github.com/keycloak/keycloak/issues/45168)  
**Suggested Assignee Team:** core-authn (tracking); upstream: BC-FIPS project

---

#### Summary

Both X.509-based authentication flows owned by core-authn — **X.509 Browser Authentication** (`CertificateValidator`, Domain 13) and **X.509 mTLS Client Authentication** (`AbstractX509ClientCertificateAuthenticator`, Domain 11) — are fully PQC-ready in non-FIPS mode, because `CertPathBuilder` delegation through `CryptoProvider` is algorithm-agnostic. However, both are **hard-blocked in FIPS-regulated deployments**: BC-FIPS 2.1.2 (currently pinned in `pom.xml`) does not contain ML-DSA or ML-KEM classes. FIPS 140-3 validation for these algorithms requires a forthcoming BC-FIPS 2.x release.

This issue tracks the dependency, defines the migration plan, and ensures core-authn is prepared to test and validate these flows as soon as BC-FIPS support becomes available.

#### Background / Context

Keycloak's FIPS 140-2/3 deployment mode routes all cryptographic operations through `FIPS1402Provider`. Inspection of the BC-FIPS 2.1.2 JAR confirms it contains only LMS under `org.bouncycastle.crypto.internal.pqc` — no ML-DSA (`FIPS 204`) and no ML-KEM (`FIPS 203`) classes are present. This means:

- **X.509 Browser Authentication:** In FIPS mode, `CertificateValidator` cannot validate a certificate chain where any certificate uses an ML-DSA key, because the underlying JVM security provider does not support ML-DSA key parsing or signature verification. This affects regulated deployments where a CA has issued ML-DSA end-entity certificates.
- **X.509 mTLS Client Authentication:** Same constraint — mTLS handshakes presenting ML-DSA client certificates will fail in FIPS mode.
- **OCSP response verification** in `CertificateValidator` uses BouncyCastle for OCSP parsing; this is also blocked if the OCSP responder signs responses with ML-DSA.

Non-FIPS deployments are unaffected (BouncyCastle 1.84, already in use, includes full ML-DSA support via `bcprov-jdk18on`).

#### Scope

**In scope:**
- Document the FIPS blocker and its impact on X.509 browser and mTLS client auth in operator-facing migration documentation (tracked under #48830 and #48823).
- Add a CI/test marker (e.g., `@RequireNonFips` or `@Ignore("FIPS-blocked: GAP-6")`) to any new PQC X.509 tests so they are excluded from FIPS CI profiles until BC-FIPS support is available.
- Define acceptance criteria for the follow-on validation work so it can be executed immediately once BC-FIPS with ML-DSA is available.
- Monitor the [BC-FIPS release page](https://www.bouncycastle.org/fips-java/) and the `pom.xml` `bc-fips` version pin; update the pin and remove the CI markers in a follow-on PR when a qualifying version ships.

**Out of scope:**
- Implementing ML-DSA support in BC-FIPS (upstream responsibility).
- X.509 changes for non-FIPS mode (no code changes needed).
- TLS layer PQC (tracked under #43691 / #50674).

#### Acceptance Criteria

- [ ] A documentation comment or issue note in this tracker records exactly which BC-FIPS version (major.minor.patch) will unblock this gap, with a link to the BC-FIPS release notes once known.
- [ ] Any new PQC integration tests for X.509 browser auth or mTLS client auth that exercise ML-DSA certificate chains carry a `@RequireNonFips` (or equivalent) guard and are excluded from FIPS CI profiles.
- [ ] Operator migration documentation (under #48823 / #48830) includes a clear statement: "In FIPS mode, X.509 certificate chain validation with ML-DSA certificates is not supported until BC-FIPS ≥ [version]. Non-FIPS mode is unaffected."
- [ ] When BC-FIPS with ML-DSA support is released: the `bc-fips` version pin in `pom.xml` is updated, CI markers are removed, and a passing integration test confirms X.509 browser auth and mTLS client auth work end-to-end with ML-DSA certificates in FIPS mode.

#### Dependencies

- **Upstream blocker:** BC-FIPS 2.x release with FIPS 140-3 validated ML-DSA/ML-KEM support.
- **Related:** GAP-6 in `pqc_overview.html` (also affects Domain 43 — CryptoProvider FIPS backend).
- **Related:** #48823 (PQC production readiness planning).
- **Related:** #48830 (PQC documentation plan).
- **Related:** Domain 43 — the `FIPS1402Provider` upgrade when BC-FIPS ships.

---

### Issue 5 — [MEDIUM] OIDC IdP Broker: Make Outbound Client Assertion Algorithm Configurable (Remove Hardcoded RS256/HS256 Fallback)

**Labels:** `area/authentication`, `area/oidc`, `kind/enhancement`, `team/core-authn`, `pqc`  
**Priority:** MEDIUM  
**Suggested Parent Issue:** [#48824 — Investigate and plan what is needed for OpenID Connect and OAuth 2.0 to be PQC ready](https://github.com/keycloak/keycloak/issues/48824)  
**Suggested Assignee Team:** core-authn (primary); cross-dependency with core-protocols (IdP broker protocol layer)

---

#### Summary

`AbstractOAuth2IdentityProvider.java` hardcodes the signing algorithm for outbound `private_key_jwt` client assertions sent to a federated OIDC IdP:

- Line 745: `Algorithm.RS256` as the fallback for `private_key_jwt` broker client assertions.
- Line 737: `Algorithm.HS256` as the fallback for `client_secret_jwt` broker assertions.

These hardcoded fallbacks mean that even after ML-DSA `SignatureProviderFactory` providers exist, a realm brokering to an ML-DSA-only upstream IdP will be unable to authenticate using `private_key_jwt` with an ML-DSA key — the outbound assertion will always be signed with RS256 unless the administrator explicitly overrides (and no UI option exists for this today).

#### Background / Context

Keycloak's OIDC identity provider broker (`AbstractOAuth2IdentityProvider`) acts as an OAuth 2.0 client toward upstream identity providers. When configured to use `private_key_jwt` client authentication, it signs the assertion with the realm's key. The algorithm selection is currently:

```java
// AbstractOAuth2IdentityProvider.java, line ~745
String alg = config.getClientAssertionSigningAlg();
if (alg == null) alg = Algorithm.RS256; // <-- hardcoded fallback
```

There is no admin UI field for `clientAssertionSigningAlg` in the IdP configuration panel, meaning operators cannot select ML-DSA even when the upstream IdP requires it. This becomes a migration blocker for operators federating against PQC-enabled IdPs (e.g., another Keycloak instance migrated to ML-DSA, or a government IdP mandating NIST FIPS 204 signatures).

#### Scope

**In scope:**
- Add a `clientAssertionSigningAlg` configuration attribute to the OIDC IdP configuration model if not already present.
- Expose the attribute in the IdP admin UI (algorithm selection dropdown) for `private_key_jwt` client authentication method, populated with the realm's available `SignatureProvider` algorithms (dynamic, same as token signing dropdowns).
- Change the hardcoded `Algorithm.RS256` fallback to use `Constants.DEFAULT_SIGNATURE_ALGORITHM` or the realm's configured default, falling back to RS256 only when no realm default exists (backward compatible).
- Unit test: broker client assertion signed with ML-DSA is produced when ML-DSA is configured as the assertion algorithm.
- Migration note: document that existing OIDC IdP configurations that relied on the implicit RS256 default are unaffected (fallback preserved).

**Out of scope:**
- Changes to inbound assertion verification (already SPI-driven and PQC-ready once providers exist).
- `client_secret_jwt` HS256 fallback (symmetric algorithm — quantum-safe by configuration, not a PQC concern).
- SAMLIdentityProvider (separate SAML broker — tracked under #50292).

#### Acceptance Criteria

- [ ] A new `clientAssertionSigningAlg` field (or equivalent configuration attribute) is available in the OIDC IdP configuration admin UI, with algorithm options populated dynamically from registered `SignatureProvider` instances.
- [ ] When `clientAssertionSigningAlg` is set to an ML-DSA variant (e.g., `ML_DSA_65`), `AbstractOAuth2IdentityProvider` produces a `private_key_jwt` assertion signed with that algorithm.
- [ ] When `clientAssertionSigningAlg` is not set, behavior is backward compatible: RS256 is used (or realm default if one is configured).
- [ ] No regression in existing OIDC IdP broker integration tests.
- [ ] Admin UI test confirms the algorithm dropdown displays ML-DSA options when an ML-DSA `SignatureProviderFactory` is registered.

#### Dependencies

- **Depends on:** #43684 (ML-DSA `SignatureProviderFactory`) — required for ML-DSA options to appear in the dropdown.
- **Depends on:** #43692 (ML-DSA for OAuth and OIDC parent).
- **Related:** #48824 (OIDC PQC readiness spike).
- GAP-10 in `pqc_overview.html` (Domain 21 — OIDC IdP Broker).

---

### Issue 6 — [MEDIUM] FAPI Client Policy Executor: Add ML-DSA to `FapiConstant.ALLOWED_ALGORITHMS` (Spec-Gated)

**Labels:** `area/authentication`, `area/oidc`, `kind/enhancement`, `team/core-authn`, `pqc`  
**Priority:** MEDIUM (EXTERNAL DEPENDENCY — spec-gated on FAPI 2.0 PQC profile)  
**Suggested Parent Issue:** [#48824 — Investigate and plan what is needed for OpenID Connect and OAuth 2.0 to be PQC ready](https://github.com/keycloak/keycloak/issues/48824)  
**Suggested Assignee Team:** core-authn (client policy executors); cross-dependency with core-protocols (FAPI enforcement)

---

#### Summary

`FapiConstant.ALLOWED_ALGORITHMS` contains only classical algorithm identifiers (PS256, PS384, PS512, ES256, ES384, ES512). `SecureSigningAlgorithmExecutor.isSecureAlgorithm()` calls `ALLOWED_ALGORITHMS.contains(sigAlg)` and will **actively reject** ML-DSA as an insecure algorithm even after ML-DSA `SignatureProviderFactory` providers exist. This means:

1. Any realm with a FAPI client policy applied cannot use ML-DSA for `private_key_jwt` client authentication.
2. CIBA backchannel auth requests signed with ML-DSA are rejected at the policy layer.
3. The admin UI algorithm option lists in `SecureSigningAlgorithmExecutorFactory` and `SecureCibaAuthenticationRequestSigningAlgorithmExecutorFactory` do not surface ML-DSA.

This issue tracks the required code change and is **gated on FAPI 2.0 / FAPI Grant Management / Financial-grade API profiles officially including PQC algorithms**.

#### Background / Context

The FAPI (Financial-grade API) security profile (currently FAPI 2.0 Security Profile) mandates specific approved signing algorithms to prevent weak-algorithm downgrade attacks. This is an appropriate security control for high-assurance deployments. However, as NIST FIPS 204 (ML-DSA) is now standardised and FAPI working groups are expected to include PQC algorithms in updated profile versions, Keycloak must be prepared to surface ML-DSA in the FAPI allowlist once specs permit.

The `ALLOWED_ALGORITHMS` constant is a static set in `FapiConstant.java`. Extending it to include ML-DSA identifiers is a one- or two-line code change once the spec gates lift, but the admin UI dropdowns (`SecureSigningAlgorithmExecutorFactory`, `SecureCibaAuthenticationRequestSigningAlgorithmExecutorFactory`) also need updating to include ML-DSA in their selectable lists.

#### Scope

**In scope:**
- Add ML-DSA-44, ML-DSA-65, and ML-DSA-87 algorithm identifiers to `FapiConstant.ALLOWED_ALGORITHMS` once the FAPI 2.0 Security Profile (or a successor) formally includes PQC algorithms.
- Update the algorithm option lists in `SecureSigningAlgorithmExecutorFactory` and `SecureCibaAuthenticationRequestSigningAlgorithmExecutorFactory` to include ML-DSA variants.
- Add a test asserting that ML-DSA is accepted (not rejected) by `isSecureAlgorithm()` after the allowlist is updated.
- Until the spec gates lift: add a code comment referencing this issue number and the spec blockers, so that the one-line fix is easy to locate and apply.

**Out of scope:**
- FAPI 2.0 spec compliance itself (external, spec-body work).
- Adding new FAPI compliance tests (out of scope for this gap fix).
- CIBA protocol changes beyond the algorithm allowlist.

#### Acceptance Criteria

- [ ] `FapiConstant.ALLOWED_ALGORITHMS` includes ML-DSA-44, ML-DSA-65, and ML-DSA-87 identifiers once FAPI 2.0 officially lists them.
- [ ] `SecureSigningAlgorithmExecutor.isSecureAlgorithm("ML_DSA_65")` (and equivalent for -44 and -87) returns `true`.
- [ ] The admin UI algorithm dropdowns for FAPI secure signing and CIBA secure signing executors display ML-DSA options.
- [ ] A realm with a FAPI client policy applied can configure ML-DSA as the required signing algorithm without policy enforcement errors.
- [ ] Existing FAPI compliance test suite passes unchanged.
- [ ] A code comment is present in `FapiConstant.java` referencing this issue number and the external spec dependency, until the spec gates lift.

#### Dependencies

- **External blocker:** FAPI 2.0 Security Profile (or Financial-grade API equivalent) must formally include ML-DSA in its approved algorithm set. Monitor [openid.net/fapi](https://openid.net/fapi/) and the OpenID Foundation FAPI WG mailing list.
- **Depends on:** #43684 (ML-DSA `SignatureProviderFactory` — needed before FAPI enforcement tests can verify ML-DSA).
- **Related:** GAP-8, GAP-17 in `pqc_overview.html`.
- **Related:** Domain 41 (FAPI Allowlist), Domain 16 (CIBA).

---

### Issue 7 — [MEDIUM] WebAuthn: Update `WebAuthnRegister` and `WebAuthnPolicy` for ML-DSA COSE Algorithm Wiring (Keycloak-Side)

**Labels:** `area/authentication/webauthn`, `area/authentication`, `kind/enhancement`, `team/core-authn`, `pqc`  
**Priority:** MEDIUM (EXTERNAL DEPENDENCY — spec-gated on FIDO Alliance / W3C WebAuthn PQC COSE assignment)  
**Suggested Parent Issue:** [#50084 — PQC support for WebAuthn/Passkeys](https://github.com/keycloak/keycloak/issues/50084)  
**Suggested Assignee Team:** core-authn

---

#### Summary

Existing issues #50085 (upgrade webauthn4j) and #50086 (add ML-DSA COSE algorithm IDs to `WebAuthnPolicy`) cover the upstream library upgrade and the static data model. This issue addresses the **Keycloak-side wiring** that is not covered by those issues: the `WebAuthnRegister` required action and `WebAuthnAuthenticator` conversion logic that translates between Keycloak's credential model and webauthn4j's algorithm representation.

When webauthn4j gains ML-DSA COSE support (#50085) and ML-DSA COSE IDs are added to the `WebAuthnPolicy` model (#50086), `WebAuthnRegister.java` will need to correctly map those COSE algorithm identifiers through the credential creation ceremony, and `WebAuthnAuthenticator` will need to verify ML-DSA assertions. Without this wiring, the COSE IDs registered in the policy model will have no effect at runtime.

#### Background / Context

The FIDO2/WebAuthn specification does not yet standardise PQC algorithms (as of mid-2026). The FIDO Alliance and W3C WebAuthn working group are actively working on COSE algorithm assignments for ML-DSA. This issue is therefore spec-gated. However, the Keycloak-side preparation work can and should be done now so that when COSE algorithm identifiers are finalised and webauthn4j gains support, the integration layer is ready.

The core-authn team owns:
- `WebAuthnRegister.java` — required action that handles the credential creation ceremony and stores the created credential.
- `WebAuthnAuthenticator.java` / `WebAuthnConditionalUIAuthenticator.java` — authenticators that verify WebAuthn assertions.
- `WebAuthnCredentialProvider.java` — credential provider that stores and retrieves WebAuthn credentials.
- `WebAuthnPolicy.java` model — defines accepted algorithm lists.

`WebAuthnRegister` currently maps COSE algorithm IDs to a fixed set of internal representations. Once ML-DSA COSE IDs are defined, the conversion code in `WebAuthnRegister.buildRegistrationOption()` and `WebAuthnAuthenticator.buildAssertionOption()` must be updated to pass through ML-DSA COSE IDs to webauthn4j.

#### Scope

**In scope:**
- Audit `WebAuthnRegister.java` and `WebAuthnAuthenticator.java` for hardcoded COSE algorithm lists and add ML-DSA COSE IDs (values TBD; use named constants from #50086 once available).
- Ensure `WebAuthnCredentialProvider` correctly stores and retrieves credentials created with ML-DSA authenticators.
- Ensure `WebAuthnPolicy.getSignatureAlgorithms()` passes ML-DSA COSE IDs through to webauthn4j's `ServerProperty` configuration.
- Add `@RequireProvider` or `@IgnoreUntil` guards on new tests until FIDO Alliance finalises ML-DSA COSE assignments.
- Integration test: registration + authentication with a simulated ML-DSA WebAuthn authenticator (may require a mock or updated virtual authenticator when browser support exists).

**Out of scope:**
- webauthn4j ML-DSA implementation (tracked under #50085).
- COSE algorithm ID assignment (external — FIDO Alliance / IANA).
- `WebAuthnPolicy` admin UI algorithm picker (tracked under #50086).
- U2F migration path or AppId extension (separate; see GitHub issue #17636).

#### Acceptance Criteria

- [ ] `WebAuthnRegister` passes ML-DSA COSE algorithm IDs to webauthn4j's `PublicKeyCredentialParameters` in the credential creation options once those COSE IDs are available in webauthn4j.
- [ ] `WebAuthnAuthenticator` passes ML-DSA COSE algorithm IDs to webauthn4j's assertion verification once those COSE IDs are available.
- [ ] `WebAuthnCredentialProvider` stores the COSE algorithm ID from an ML-DSA registration response without data truncation.
- [ ] Existing WebAuthn registration and authentication integration tests (including `WebAuthnOtherSettingsTest`) pass without regression.
- [ ] New tests (gated by FIDO Alliance spec status) confirm ML-DSA WebAuthn credential registration and assertion work end-to-end with an updated virtual authenticator.

#### Dependencies

- **Depends on:** #50085 (webauthn4j ML-DSA support — upstream library prerequisite).
- **Depends on:** #50086 (ML-DSA COSE IDs in `WebAuthnPolicy` model).
- **External blocker:** FIDO Alliance / W3C WebAuthn PQC COSE algorithm assignment.
- **Related:** Domain 14 in `pqc_overview.html`.
- **Related:** #50084 (PQC WebAuthn/Passkeys milestone parent).

---

### Issue 8 — [MEDIUM] Admin CLI (`kcadm.sh` / `kcreg.sh`): Add `--sigalg` Parameter for `private_key_jwt` Client Assertions

**Labels:** `area/authentication`, `area/dist/quarkus`, `kind/enhancement`, `team/core-authn`, `pqc`  
**Priority:** MEDIUM  
**Suggested Parent Issue:** [#48824 — Investigate and plan what is needed for OpenID Connect and OAuth 2.0 to be PQC ready](https://github.com/keycloak/keycloak/issues/48824)  
**Suggested Assignee Team:** core-authn (CLI authentication layer); cross-dependency with dist/quarkus (CLI packaging)

---

#### Summary

`AuthUtil.getSignedRequestToken()` (used by both `kcadm.sh` and `kcreg.sh`) hardcodes RS256 via `new JWSBuilder().rsa256(keypair.getPrivate())`, regardless of the algorithm of the key loaded from the configured keystore. No `--sigalg` parameter is available on CLI credential configuration commands.

If a realm mandates ML-DSA for `private_key_jwt` client authentication (e.g., via a FAPI-like client policy), both CLI tools will be unable to authenticate against that realm using the keystore-based authentication flow. This is a tooling gap that blocks operators from managing PQC-enabled realms via the CLI.

#### Background / Context

Keycloak's admin CLI tools (`kcadm.sh` / `kcreg.sh`) support `private_key_jwt` client authentication using a local keystore. The signing path in `AuthUtil`:

```java
// AuthUtil.java — hardcoded RS256
String token = new JWSBuilder()
    .rsa256(keypair.getPrivate())
    .build(...);
```

This bypasses the SPI entirely and uses a hard-coded RSA SHA-256 signer. To support ML-DSA:
1. `getSignedRequestToken()` must accept an algorithm parameter.
2. A `--sigalg` option must be plumbed through the CLI credential configuration command chain (e.g., `config credentials --sigalg ML_DSA_65`).
3. The JWSBuilder call must be replaced with an algorithm-aware signer that supports AKP key types.

#### Scope

**In scope:**
- Add an `algorithm` parameter to `AuthUtil.getSignedRequestToken()`.
- Add a `--sigalg` CLI option to the keystore-based credential configuration command in `kcadm.sh` and `kcreg.sh`.
- Replace the hardcoded `rsa256()` JWSBuilder call with an algorithm-aware path that delegates to the appropriate signer for RSA, EC, EdDSA, and AKP key types.
- Update CLI documentation (man pages / `--help` output) to document the new option.
- Integration test: `kcadm.sh config credentials` with a BCFKS keystore containing an ML-DSA key pair successfully authenticates against a realm requiring ML-DSA `private_key_jwt`.

**Out of scope:**
- GUI / admin console changes.
- Changes to non-keystore CLI authentication methods (client secret, etc.).
- `kcreg.sh` endpoints unrelated to authentication.

#### Acceptance Criteria

- [ ] `AuthUtil.getSignedRequestToken()` accepts an `algorithm` string parameter; when set to `ML_DSA_65` (or another ML-DSA variant), it produces a JWS signed with the corresponding ML-DSA algorithm.
- [ ] `kcadm.sh config credentials` and `kcreg.sh config credentials` accept a `--sigalg <algorithm>` option.
- [ ] When `--sigalg` is omitted, the default behavior is RS256 (backward compatible).
- [ ] An integration test confirms that `kcadm.sh` can authenticate against a realm configured for ML-DSA `private_key_jwt` when invoked with `--sigalg ML_DSA_65` and a BCFKS keystore containing an ML-DSA key.
- [ ] `--help` output for the CLI credential commands documents the new `--sigalg` option.

#### Dependencies

- **Depends on:** #43684 (ML-DSA `SignatureProviderFactory`) — needed for the server side to verify ML-DSA client assertions.
- **Depends on:** #50679 (Support loading ML-DSA keys from Java keystores) — needed to load AKP keys from BCFKS/PKCS12 in the CLI.
- **Related:** #48824 (OIDC PQC readiness spike, parent for this work).
- **Related:** Issue 9 (Client SDK `JWTClientCredentialsProvider` — shares the same `JWSBuilder` / assertion signing gap).
- GAP-13 in `pqc_overview.html` (Domain 38 — Admin CLI).

---

### Issue 9 — [LOW] Client SDK `JWTClientCredentialsProvider`: Add AKP Case to `setupKeyPair()` for ML-DSA `private_key_jwt`

**Labels:** `area/authentication`, `area/oidc`, `kind/enhancement`, `team/core-authn`, `pqc`  
**Priority:** LOW  
**Suggested Parent Issue:** [#48824 — Investigate and plan what is needed for OpenID Connect and OAuth 2.0 to be PQC ready](https://github.com/keycloak/keycloak/issues/48824)  
**Suggested Assignee Team:** core-authn

---

#### Summary

`JWTClientCredentialsProvider.setupKeyPair()` has a `switch` on `JavaAlgorithm.getKeyType()` that handles RSA, EC, and OKP (EdDSA) key types but has no `AKP` case. Passing an ML-DSA key pair throws `RuntimeException("Invalid KeyPair algorithm")`. This prevents Keycloak adapter / SDK users from authenticating as a client with `private_key_jwt` using ML-DSA keys from the client side.

#### Background / Context

`JWTClientCredentialsProvider` is used by Keycloak's Java SDK / adapter for client-side `private_key_jwt` client authentication. The algorithm-aware key setup path routes:
- RSA keys → `AsymmetricSignatureSignerContext(RS256/PS*)`
- EC keys → `AsymmetricSignatureSignerContext(ES*)`
- OKP keys → `AsymmetricSignatureSignerContext(EdDSA)`
- AKP keys → **RuntimeException** (missing case)

The generic `generateSignedDPoPProof(…, KeyWrapper, …)` method in the same SDK area is algorithm-agnostic and works; only the `setupKeyPair()` convenience method is broken for AKP.

This is a LOW priority because the primary PQC client auth fix is on the server side (Issue 3), and SDK users can work around this by using the `KeyWrapper`-based path. However, it should be resolved before a general PQC migration guide is published, as it would otherwise be a confusing failure mode for SDK integrators.

#### Scope

**In scope:**
- Add an `AKP` case to the `switch` in `setupKeyPair()` in `JWTClientCredentialsProvider.java` that wires an `AsymmetricSignatureSignerContext` for ML-DSA.
- Unit test: `setupKeyPair()` with an ML-DSA key pair does not throw and produces a valid signer context.
- Integration test (can be added to existing `ClientAuthPostMethodTest` suite): SDK client using ML-DSA key pair authenticates successfully via `private_key_jwt`.

**Out of scope:**
- `DPoPGenerator` convenience method (separate GAP-29; low priority if callers use the generic `KeyWrapper` path).
- Server-side verifier changes (Issue 3).
- Admin CLI changes (Issue 8).

#### Acceptance Criteria

- [ ] `JWTClientCredentialsProvider.setupKeyPair()` does not throw `RuntimeException` when called with an ML-DSA (`AKP`) key pair.
- [ ] The returned `AsymmetricSignatureSignerContext` produces valid ML-DSA-signed JWTs that are accepted by a Keycloak server with ML-DSA `SignatureProviderFactory` registered (Issue 3 prerequisite).
- [ ] Existing tests for RSA, EC, and EdDSA `setupKeyPair()` paths pass unchanged.
- [ ] New unit test covers ML-DSA-44, -65, and -87 key pairs.

#### Dependencies

- **Depends on:** #43684 (ML-DSA `SignatureProviderFactory`).
- **Related:** Issue 3 (Client Asymmetric Signature Verifier — server-side counterpart; should be resolved first).
- **Related:** Issue 8 (Admin CLI `--sigalg` — same signing-side gap in a different tool).
- GAP-28 in `pqc_overview.html` (Domain 50 — Client SDK JWT).

---

## Issue Filing Checklist

Before filing each issue in GitHub:

1. **Link to parent:** Add as a sub-issue of the appropriate parent listed in each issue header.
2. **Label application:** Apply all labels listed in each issue's **Labels** field. If `pqc` label does not exist, create it as a root-level topic label.
3. **Priority label:** Apply `priority/important` for Issues 1–3; `priority/backlog` for Issues 7–9.
4. **Dependency notation:** Reference dependent issues in the issue body using `#<issue-number>` once they are filed.
5. **Filing order (recommended):**
   - File Issue 1 first (JWK Thumbprint) — it unblocks Issues 2 and partially 3.
   - File Issues 2 and 3 next (attestation auth + client verifier).
   - File Issues 4–6 (FIPS tracking, IdP broker, FAPI).
   - File Issues 7–9 (WebAuthn wiring, CLI, SDK).
6. **Spec-gated issues (Issues 4, 6, 7):** Add the `needs-spec-update` or `waiting-on-external` label if available, to distinguish from immediately actionable work.

---

## Coverage Confirmation

| Gap | Covered by | Status |
|---|---|---|
| GAP-1: JWK Thumbprint AKP | Issue 1 | ✅ New issue |
| GAP-22: Attestation auth algorithm enforcement | Issue 2 | ✅ New issue |
| GAP-30: Client signature verifier RSA guard | Issue 3 | ✅ New issue |
| GAP-6: FIPS X.509 ML-DSA blocker (core-authn scope) | Issue 4 | ✅ New issue |
| GAP-10: IdP broker hardcoded RS256 | Issue 5 | ✅ New issue |
| GAP-8/GAP-17: FAPI allowlist blocks ML-DSA at client auth | Issue 6 | ✅ New issue |
| Domain 14 (WebAuthn Keycloak-side wiring) | Issue 7 | ✅ New issue |
| GAP-13: Admin CLI hardcoded RS256 | Issue 8 | ✅ New issue |
| GAP-28: Client SDK JWTClientCredentialsProvider AKP | Issue 9 | ✅ New issue |
| Password hashing (GAP-related) | #48828 | ✅ Already closed |
| Cookies PQC | #49865 | ✅ Already closed |
| WebAuthn webauthn4j upgrade | #50085 | ✅ Existing open issue |
| WebAuthn COSE IDs in policy model | #50086 | ✅ Existing open issue |
