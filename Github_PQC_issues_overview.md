# GitHub PQC Issues Overview

**Parent Feature Issue:** [#43690 - Post-Quantum Cryptography (PQC) readiness](https://github.com/keycloak/keycloak/issues/43690)

**Last Updated:** 2026-07-09

---

## Complete Issue Hierarchy

### **[#43690](https://github.com/keycloak/keycloak/issues/43690) - Post-Quantum Cryptography (PQC) readiness** 
**Status:** <span style="color: #d97706; font-weight: bold;">🔵 OPEN</span> | **Type:** feature | **Progress:** 0 of 10

Top-level feature issue tracking full PQC readiness for Keycloak.

**Direct Sub-Issues: 10**

| Issue | Title | Status | Type | Progress | Sub-Issues | Gaps |
|-------|-------|--------|------|----------|------------|------|
| [#43691](https://github.com/keycloak/keycloak/issues/43691) | Hybrid key exchange in TLS 1.3 | <span style="color: #d97706; font-weight: bold;">🔵 OPEN</span> | milestone | N/A | 0 | |
| [#45168](https://github.com/keycloak/keycloak/issues/45168) | Review what is needed for PQC readiness in Keycloak | <span style="color: #d97706; font-weight: bold;">🔵 OPEN</span> | spike | 4 of 12 | **12** | |
| ↳ [#48819](https://github.com/keycloak/keycloak/issues/48819) | Create inventory of cryptography in Keycloak | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| ↳ [#48820](https://github.com/keycloak/keycloak/issues/48820) | Investigate and plan what is required for TLS PQC support | <span style="color: #7c3aed; font-weight: bold;">🟣 CLOSED</span> | task | N/A | 0 | |
| ↳ [#48822](https://github.com/keycloak/keycloak/issues/48822) | Investigate and plan what is required for truststore and keystores to support PQC | <span style="color: #7c3aed; font-weight: bold;">🟣 CLOSED</span> | task | N/A | 0 | |
| ↳ [#48823](https://github.com/keycloak/keycloak/issues/48823) | Investigate and plan what is needed for production readiness around PQC | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| ↳ [#48824](https://github.com/keycloak/keycloak/issues/48824) | Investigate and plan what is needed for OpenID Connect and OAuth 2.0 to be PQC ready | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| ↳ [#48825](https://github.com/keycloak/keycloak/issues/48825) | Investigate and plan what is needed for SAML 2.0 to be PQC ready | <span style="color: #7c3aed; font-weight: bold;">🟣 CLOSED</span> | task | N/A | 0 | |
| ↳ [#48826](https://github.com/keycloak/keycloak/issues/48826) | Investigate what is required for WebAuthn/passkeys to be PQC ready | <span style="color: #7c3aed; font-weight: bold;">🟣 CLOSED</span> | task | N/A | 0 | |
| ↳ [#48827](https://github.com/keycloak/keycloak/issues/48827) | Investigate what is required for cookies to be PQC ready | <span style="color: #7c3aed; font-weight: bold;">🟣 CLOSED</span> | task | N/A | **(2)** | |
| &nbsp;&nbsp;&nbsp;&nbsp;↳ [#49858](https://github.com/keycloak/keycloak/issues/49858) | Increase AES default from 128 to 256 in KC_RESTART cookie | <span style="color: #7c3aed; font-weight: bold;">🟣 CLOSED</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;↳ [#49860](https://github.com/keycloak/keycloak/issues/49860) | Use SHA384 or SHA512 for hashing for AUTH_SESSION_ID_HASH and SESSION cookies | <span style="color: #7c3aed; font-weight: bold;">🟣 CLOSED</span> | task | N/A | 0 | |
| ↳ [#48828](https://github.com/keycloak/keycloak/issues/48828) | Investigate and plan what is needed for password hashing to be PQC ready | <span style="color: #7c3aed; font-weight: bold;">🟣 CLOSED</span> | task | N/A | 0 | |
| ↳ [#48829](https://github.com/keycloak/keycloak/issues/48829) | Review if there are any areas not identified around PQC readiness | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| ↳ [#48830](https://github.com/keycloak/keycloak/issues/48830) | Create a plan for documentation around PQC readiness | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| ↳ [#49851](https://github.com/keycloak/keycloak/issues/49851) | PQC modes | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| [#46333](https://github.com/keycloak/keycloak/issues/46333) | Audit and Upgrade Cryptographic Defaults | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | milestone | 0 of 1 | **1** | |
| ↳ [#46336](https://github.com/keycloak/keycloak/issues/46336) | Cryptographic Inventory for Keycloak | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| [#48821](https://github.com/keycloak/keycloak/issues/48821) | PQC support for OAuth 2.0 and OpenID Connect | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | milestone | 0 of 4 | **4** | |
| ↳ [#43693](https://github.com/keycloak/keycloak/issues/43693) | Support FN-DSA for OAuth and OpenID Connect | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | feature | N/A | 0 | |
| ↳ [#43692](https://github.com/keycloak/keycloak/issues/43692) | Support ML-DSA for OAuth and OpenID Connect | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | feature | 1 of 5 | **(5)** | |
| &nbsp;&nbsp;&nbsp;&nbsp;↳ [#44141](https://github.com/keycloak/keycloak/issues/44141) | JWK Algorithm Key Pair support | <span style="color: #7c3aed; font-weight: bold;">🟣 CLOSED</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;↳ [#44142](https://github.com/keycloak/keycloak/issues/44142) | Add generated keys provider for ML-DSA | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;↳ [#43684](https://github.com/keycloak/keycloak/issues/43684) | Add ML-DSA SignatureProviderFactory and ClientSignatureVerifierProviderFactory | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;↳ [#44143](https://github.com/keycloak/keycloak/issues/44143) | Support issuing tokens with ML-DSA | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;↳ [#44144](https://github.com/keycloak/keycloak/issues/44144) | Support verifying tokens issued by clients signed with ML-DSA | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| ↳ [#50299](https://github.com/keycloak/keycloak/issues/50299) | Support HPKE (Hybrid Public Key Encryption) algorithms for JWE | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | feature | N/A | 0 | |
| ↳ [#50304](https://github.com/keycloak/keycloak/issues/50304) | Support PQ/T Hybrid Composite Signatures for JOSE and COSE | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | feature | N/A | 0 | |
| [#49865](https://github.com/keycloak/keycloak/issues/49865) | Milestone for cookies to be PQC ready | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | milestone | 2 of 2 | **(2)** | |
| ↳ [#49858](https://github.com/keycloak/keycloak/issues/49858) | Increase AES default from 128 to 256 in KC_RESTART cookie | <span style="color: #7c3aed; font-weight: bold;">🟣 CLOSED</span> | task | N/A | 0 | |
| ↳ [#49860](https://github.com/keycloak/keycloak/issues/49860) | Use SHA384 or SHA512 for hashing for AUTH_SESSION_ID_HASH and SESSION cookies | <span style="color: #7c3aed; font-weight: bold;">🟣 CLOSED</span> | task | N/A | 0 | |
| [#50084](https://github.com/keycloak/keycloak/issues/50084) | PQC support for WebAuthn/Passkeys | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | milestone | 0 of 2 | **(2)** | |
| ↳ [#50085](https://github.com/keycloak/keycloak/issues/50085) | Upgrade webauthn4j to a version with ML-DSA support | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| ↳ [#50086](https://github.com/keycloak/keycloak/issues/50086) | Add ML-DSA COSE algorithm IDs to WebAuthn policies | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| [#50292](https://github.com/keycloak/keycloak/issues/50292) | PQC support for SAML 2.0 | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | milestone | 0 of 2 | **(2)** | |
| ↳ [#50294](https://github.com/keycloak/keycloak/issues/50294) | Add ML-DSA algorithm support for XMLSignatureUtil and XMLEncryptionUtil | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | feature | N/A | 0 | |
| ↳ [#50295](https://github.com/keycloak/keycloak/issues/50295) | Dual hybrid support for SAML assertions and their validations | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | feature | N/A | 0 | |
| [#50678](https://github.com/keycloak/keycloak/issues/50678) | Add ML-DSA JCE algorithm mapping to JavaAlgorithm | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| [#50679](https://github.com/keycloak/keycloak/issues/50679) | Support loading ML-DSA keys from Java keystores | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| [#50680](https://github.com/keycloak/keycloak/issues/50680) | Add test coverage for truststore loading with PQC certificates | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |

---

## Additional Issues (Not in Main Hierarchy)

These issues exist but are not tracked as sub-issues in the main tree. They may be:
- Standalone implementation tasks
- Infrastructure/dependency issues
- Recently created issues not yet linked

### Infrastructure & Dependencies

| Issue # | Status | Title | Domains |
|---------|--------|-------|---------|
| [50674](https://github.com/keycloak/keycloak/issues/50674) | OPEN | Adopt Quarkus PQC TLS support and adapt Keycloak configuration | TLS integration |
| [50675](https://github.com/keycloak/keycloak/issues/50675) | OPEN | Determine Infinispan PQC path for Hot Rod client and server transport | Infinispan cluster |
| [50676](https://github.com/keycloak/keycloak/issues/50676) | OPEN | Add database TLS test coverage and verify JDBC driver PQC compatibility | Database TLS |
| [50677](https://github.com/keycloak/keycloak/issues/50677) | OPEN | Add OpenSSL libraries to Keycloak container image | Container dependencies |
| [50678](https://github.com/keycloak/keycloak/issues/50678) | OPEN | Add ML-DSA JCE algorithm mapping to JavaAlgorithm | Domain 1 (algorithm mapping) |
| [50679](https://github.com/keycloak/keycloak/issues/50679) | OPEN | Support loading ML-DSA keys from Java keystores | Domains 33-34 (GAP-5, GAP-16) |
| [50680](https://github.com/keycloak/keycloak/issues/50680) | OPEN | Add test coverage for truststore loading with PQC certificates | Domain 34 (testing) |
| [49968](https://github.com/keycloak/keycloak/issues/49968) | OPEN | Switch internal HTTP client from Apache to Vert.x/Netty | HTTP client |
| [50355](https://github.com/keycloak/keycloak/issues/50355) | OPEN | Post-Quantum Keys choice for signing/encryption | UI/UX configuration |

### OID4VCI Specific

| Issue # | Status | Title | Domains |
|---------|--------|-------|---------|
| [48415](https://github.com/keycloak/keycloak/issues/48415) | CLOSED | [OID4VCI] Review JwtProofValidator.JWK_PRIVATE_KEY_CLAIMS | Domain 26 |

---

## Domains Missing GitHub Issues

Based on the comprehensive analysis of 61 domains in `pqc_overview.html`, the following domains and gaps **require new GitHub issues**:

### CRITICAL Priority - New Issues Needed

| GAP/Domain | Title | Reason | Affected Domains |
|------------|-------|--------|------------------|
| **GAP-1** | JWK Thumbprint for AKP (ML-DSA) Keys | Blocks DPoP and attestation-based client auth | Domains 12, 15, 45 |
| **GAP-4** | ML-KEM CekManagementProvider for OIDC Token Encryption | No quantum-safe JWE encryption path exists | Domains 2, 4, 8 |
| **GAP-9** | SAML: Replace Hardcoded RS256 Key Selection | 4 call sites bypass ML-DSA keys even when present | Domains 18, 20, 59-61 |

### HIGH Priority - New Issues Needed

| GAP/Domain | Title | Reason | Affected Domains |
|------------|-------|--------|------------------|
| **Domain 48** | JWKS Endpoint: Add AKP Branch to JWKSServerUtils.toJwk() | ML-DSA keys return null, never appear in public JWKS | Domain 48 |
| **Domain 53** | Client Signature Verifier: Add AKP Support / Remove RSA-Only Guard | ClientAsymmetricSignatureVerifierContext rejects all non-RSA keys | Domain 53 |
| **Domain 46** | Realm Bootstrap: Add ML-DSA/ML-KEM Default Key Providers | New realms never get PQC keys by default | Domain 46 |

### MEDIUM Priority - New Issues Needed

| GAP/Domain | Title | Reason | Affected Domains |
|------------|-------|--------|------------------|
| **GAP-5 + GAP-16** | Keystore Import: Add ML-DSA Support | Cannot import ML-DSA keys from BCFKS/PKCS12 | Domains 33-34 |
| **Domains 29, 32** | ML-KEM Key Generation Provider | No ML-KEM equivalent of GAP-15 (encryption keys) | Domains 29, 32 |
| **GAP-19** | Maven Enforcer: bcprov-jdk18on ≥ 1.78 Minimum Version | Silent ML-DSA regression risk | Domain 42 |
| **GAP-20** | JAR Encrypted Request Object Decryption - ML-KEM Inbound Path | No inbound ML-KEM decryption (distinct from GAP-4 outbound) | Domain 8 |
| **GAP-21** | UserInfo Endpoint: Per-Client Signing Attribute & Encryption Gap | Doesn't inherit realm default, requires explicit migration | Domain 4 |
| **GAP-22** | Attestation-Based Client Auth: Enforce Asymmetric Algorithm Requirement | Algorithm enforcement TODO not implemented | Domain 12 |
| **GAP-23** | OIDC Discovery: Make DEFAULT_CLIENT_AUTH_SIGNING_ALG_VALUES_SUPPORTED Dynamic | Hardcoded RS256 constant for CIBA discovery | Domain 36 |
| **Domain 49** | Admin API: Client Keypair Generation Algorithm Parameter | Hardcoded RSA generation | Domain 49 |
| **GAP-10** | IdP Broker: Configurable Outbound Assertion Algorithm | Hardcoded RS256 fallback for private_key_jwt | Domain 21 |
| **GAP-13** | Admin CLI: Add --sigalg Parameter for Client Assertions | kcadm.sh/kcreg.sh hardcoded RS256 | Domain 38 |
| **GAP-18** | OID4VC c_nonce JWT: Replace Hardcoded ES256/RS256 Selection | Bypasses realm default algorithm | Domain 27 |

### LOW Priority - New Issues Needed

| GAP/Domain | Title | Reason | Affected Domains |
|------------|-------|--------|------------------|
| **Domains 50-51** | Client SDK: Add AKP Support to JWTClientCredentialsProvider & DPoPGenerator | Client-side SDK missing ML-DSA support | Domains 50-51 |
| **GAP-11** | Dynamic Client Registration: RS256 Special-Case Logic | RS256 omitted from DCR response, no ML-DSA equivalent | Domain 35 |
| **Domain 52** | Docker Registry: Configurable Certificate Algorithm | Hardcoded RSA-2048 for test/dev path | Domain 52 |

### EXTERNAL DEPENDENCIES - No Action Needed (Document Only)

| GAP/Domain | Title | Blocker | Affected Domains |
|------------|-------|---------|------------------|
| **GAP-6** | FIPS 140-2 Backend — ML-DSA Availability Blocked | BC-FIPS 2.1.2 lacks ML-DSA/ML-KEM | Domains 11, 13, 43 |
| **GAP-7** | OID4VC Linked Data Proof Suite — Hardcoded Ed25519 | Spec-level work needed | Domain 25 |
| **GAP-8** | FAPI / Client Policy Allowlist — ML-DSA Actively Blocked | Spec-gated on FAPI 2.0 | Domains 16, 41 |
| **Domain 37** | Security Event Token (SET) Signing | CAEP spec gated | Domain 37 |
| **Domain 44** | CryptoProvider SPI — WildFly Elytron Backend | WildFly Elytron roadmap | Domain 44 |

---

## Domain Coverage Summary

### Domains Covered by Existing Issues

The following domains are covered by issues in the hierarchy:

**OAuth 2.0 / OIDC Core (#43692, #48821, #48824):**
- Domains 1-17: Token signing, encryption, client authentication, JAR/PAR/JARM, CIBA, DPoP, etc.
- Domains 35-36: Client registration and OIDC discovery
- Domains 54-58: Additional OAuth flows (federated auth, token exchange, device flow, etc.)

**SAML (#48825, #50292, #50294, #50295):**
- Domains 18-20: SAML signing, encryption, IdP broker
- Domains 59-61: SAML metadata, artifact resolution

**WebAuthn (#48826, #50084, #50085, #50086):**
- Domain 14: WebAuthn/FIDO2 authentication

**Key Management (#44142, #50678, #50679, #50680):**
- Domains 28-34: Realm key generation and import
- Domain 45: JWK serialization and thumbprint

**OID4VC:**
- Domains 24-27, 56: Verifiable credentials issuance and verification

**Infrastructure:**
- Domain 39: Cluster/Infinispan
- Domains 42-44: Crypto provider backends

**Other:**
- Domain 40: Organizations
- Domain 41: FAPI policy enforcement
- Domain 47: Client public key loader

### Domains NOT Covered (23 New Issues Needed)

- **CRITICAL (3):** GAP-1, GAP-4, GAP-9
- **HIGH (3):** Domain 48, Domain 53, Domain 46
- **MEDIUM (10):** GAP-5/16, Domains 29/32, GAP-19, GAP-20, GAP-21, GAP-22, GAP-23, Domain 49, GAP-10, GAP-13, GAP-18
- **LOW (3):** Domains 50-51, GAP-11, Domain 52
- **EXTERNAL (4):** GAP-6, GAP-7, GAP-8, Domain 37, Domain 44

---

## Status Distribution (61 Domains)

| Status | Count | Description |
|--------|-------|-------------|
| **PENDING PROVIDERS** | 20 | Will work automatically once ML-DSA/ML-KEM providers exist |
| **PARTIAL** | 10 | Mostly ready; specific gaps to fix |
| **BLOCKED** | 24 | Require code changes before PQC can work |
| **SAFE** | 4 | Quantum-safe (symmetric algorithms, algorithm-agnostic cert validation) |
| **EXTERNAL DEPENDENCY** | 3 | Blocked on external specs (FAPI 2.0, CAEP, FIDO Alliance, BC-FIPS) |

---

## Next Steps

1. **Review the 23 new issues** listed in "Domains Missing GitHub Issues" section
2. **Create new GitHub issues** for CRITICAL and HIGH priority gaps first
3. **Link new issues** to appropriate parent issues in the hierarchy:
   - ML-DSA signing issues → under #43692
   - ML-KEM encryption issues → under #50299 or new encryption parent
   - SAML issues → under #50292
   - Keystore issues → reference #48822
4. **Update tracking** in #48829 (gap analysis) as new areas are identified
5. **Document EXTERNAL DEPENDENCY blockers** with no immediate action items

---

**Analysis Source:** `pqc_overview.html` (61 domains analyzed)  
**GitHub Issue Hierarchy:** Verified from GitHub UI screenshots (2026-07-08)