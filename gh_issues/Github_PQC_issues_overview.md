# GitHub PQC Issues Overview

**Parent Feature Issue:** [#43690 - Post-Quantum Cryptography (PQC) readiness](https://github.com/keycloak/keycloak/issues/43690)

**Last Updated:** 2026-09-15

---

## Status Dashboard

📊 **[View interactive dashboard →](../pqc_dashboard.html)** — issue status donuts, domain readiness breakdown, new issues summary.

| Tracked Issues | 🟢 Open | 🟣 Closed | Domains Analysed | 🔴 New Issues Needed |
|:-:|:-:|:-:|:-:|:-:|
| **55** | **42** | **13** | **63** | **30** |

---

## Complete Issue Hierarchy

### **[#43690](https://github.com/keycloak/keycloak/issues/43690) - Post-Quantum Cryptography (PQC) readiness**
**Status:** <span style="color: #d97706; font-weight: bold;">🔵 OPEN</span> | **Type:** feature | **Progress:** 1 of 15

Top-level feature issue tracking full PQC readiness for Keycloak.

**Direct Sub-Issues: 15**

| Issue | Title | Status | Type | Progress | Sub-Issues | Gaps |
|-------|-------|--------|------|----------|------------|------|
| [#43691](https://github.com/keycloak/keycloak/issues/43691) | Hybrid key exchange in TLS 1.3 | <span style="color: #d97706; font-weight: bold;">🔵 OPEN</span> | milestone | N/A | **6** | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#50674](https://github.com/keycloak/keycloak/issues/50674) | Adopt Quarkus PQC TLS support and adapt Keycloak configuration | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#50677](https://github.com/keycloak/keycloak/issues/50677) | Add OpenSSL libraries to Keycloak container image | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#50676](https://github.com/keycloak/keycloak/issues/50676) | Add database TLS test coverage and verify JDBC driver PQC compatibility | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#50675](https://github.com/keycloak/keycloak/issues/50675) | Determine Infinispan PQC path for Hot Rod client and server transport | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#49968](https://github.com/keycloak/keycloak/issues/49968) | Switch internal HTTP client from Apache to Vert.x/Netty | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#51255](https://github.com/keycloak/keycloak/issues/51255) | PQC for OTLP exporter - support hybrid key exchange and enforcing of PQC modes | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| [#45168](https://github.com/keycloak/keycloak/issues/45168) | Review what is needed for PQC readiness in Keycloak | <span style="color: #d97706; font-weight: bold;">🔵 OPEN</span> | spike | 7 of 12 | **12** | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#48819](https://github.com/keycloak/keycloak/issues/48819) | Create inventory of cryptography in Keycloak | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#48820](https://github.com/keycloak/keycloak/issues/48820) | Investigate and plan what is required for TLS PQC support | <span style="color: #7c3aed; font-weight: bold;">🟣 CLOSED</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#48822](https://github.com/keycloak/keycloak/issues/48822) | Investigate and plan what is required for truststore and keystores to support PQC | <span style="color: #7c3aed; font-weight: bold;">🟣 CLOSED</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#48823](https://github.com/keycloak/keycloak/issues/48823) | Investigate and plan what is needed for production readiness around PQC | <span style="color: #7c3aed; font-weight: bold;">🟣 CLOSED</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#48824](https://github.com/keycloak/keycloak/issues/48824) | Investigate and plan what is needed for OpenID Connect and OAuth 2.0 to be PQC ready | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#48825](https://github.com/keycloak/keycloak/issues/48825) | Investigate and plan what is needed for SAML 2.0 to be PQC ready | <span style="color: #7c3aed; font-weight: bold;">🟣 CLOSED</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#48826](https://github.com/keycloak/keycloak/issues/48826) | Investigate what is required for WebAuthn/passkeys to be PQC ready | <span style="color: #7c3aed; font-weight: bold;">🟣 CLOSED</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#48827](https://github.com/keycloak/keycloak/issues/48827) | Investigate what is required for cookies to be PQC ready | <span style="color: #7c3aed; font-weight: bold;">🟣 CLOSED</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#48828](https://github.com/keycloak/keycloak/issues/48828) | Investigate and plan what is needed for password hashing to be PQC ready | <span style="color: #7c3aed; font-weight: bold;">🟣 CLOSED</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#48829](https://github.com/keycloak/keycloak/issues/48829) | Review if there are any areas not identified around PQC readiness | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#48830](https://github.com/keycloak/keycloak/issues/48830) | Create a plan for documentation around PQC readiness | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#49851](https://github.com/keycloak/keycloak/issues/49851) | PQC modes | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | **(1)** | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#51132](https://github.com/keycloak/keycloak/issues/51132) | PQC Config Option design | <span style="color: #7c3aed; font-weight: bold;">🟣 CLOSED</span> | spike | N/A | 0 | |
| [#46333](https://github.com/keycloak/keycloak/issues/46333) | Audit and Upgrade Cryptographic Defaults | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | milestone | 0 of 1 | **1** | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#46336](https://github.com/keycloak/keycloak/issues/46336) | Cryptographic Inventory for Keycloak | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| [#48821](https://github.com/keycloak/keycloak/issues/48821) | PQC support for OAuth 2.0 and OpenID Connect | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | milestone | 0 of 4 | **4** | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#43693](https://github.com/keycloak/keycloak/issues/43693) | Support FN-DSA for OAuth and OpenID Connect | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | feature | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#43692](https://github.com/keycloak/keycloak/issues/43692) | Support ML-DSA for OAuth and OpenID Connect | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | feature | 1 of 5 | **(5)** | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#44141](https://github.com/keycloak/keycloak/issues/44141) | JWK Algorithm Key Pair support | <span style="color: #7c3aed; font-weight: bold;">🟣 CLOSED</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#44142](https://github.com/keycloak/keycloak/issues/44142) | Add generated keys provider for ML-DSA | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#43684](https://github.com/keycloak/keycloak/issues/43684) | Add ML-DSA SignatureProviderFactory and ClientSignatureVerifierProviderFactory | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#44143](https://github.com/keycloak/keycloak/issues/44143) | Support issuing tokens with ML-DSA | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#44144](https://github.com/keycloak/keycloak/issues/44144) | Support verifying tokens issued by clients signed with ML-DSA | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#50299](https://github.com/keycloak/keycloak/issues/50299) | Support HPKE (Hybrid Public Key Encryption) algorithms for JWE | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | feature | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#50304](https://github.com/keycloak/keycloak/issues/50304) | Support PQ/T Hybrid Composite Signatures for JOSE and COSE | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | feature | N/A | 0 | |
| [#49865](https://github.com/keycloak/keycloak/issues/49865) | Milestone for cookies to be PQC ready | <span style="color: #7c3aed; font-weight: bold;">🟣 CLOSED</span> | milestone | 2 of 2 | **(2)** | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#49858](https://github.com/keycloak/keycloak/issues/49858) | Increase AES default from 128 to 256 in KC_RESTART cookie | <span style="color: #7c3aed; font-weight: bold;">🟣 CLOSED</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#49860](https://github.com/keycloak/keycloak/issues/49860) | Use SHA384 or SHA512 for hashing for AUTH_SESSION_ID_HASH and SESSION cookies | <span style="color: #7c3aed; font-weight: bold;">🟣 CLOSED</span> | task | N/A | 0 | |
| [#50084](https://github.com/keycloak/keycloak/issues/50084) | PQC support for WebAuthn/Passkeys | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | milestone | 0 of 2 | **(2)** | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#50085](https://github.com/keycloak/keycloak/issues/50085) | Upgrade webauthn4j to a version with ML-DSA support | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#50086](https://github.com/keycloak/keycloak/issues/50086) | Add ML-DSA COSE algorithm IDs to WebAuthn policies | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| [#50292](https://github.com/keycloak/keycloak/issues/50292) | PQC support for SAML 2.0 | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | milestone | 0 of 2 | **(2)** | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#50294](https://github.com/keycloak/keycloak/issues/50294) | Add ML-DSA algorithm support for XMLSignatureUtil and XMLEncryptionUtil | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | feature | N/A | **(2)** | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#51421](https://github.com/keycloak/keycloak/issues/51421) | Add generated keys provider for ML-DSA | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#51422](https://github.com/keycloak/keycloak/issues/51422) | Final review of ML-DSA in SAML 2.0 protocol | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳ [#50295](https://github.com/keycloak/keycloak/issues/50295) | Dual hybrid support for SAML assertions and their validations | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | feature | N/A | 0 | |
| [#50678](https://github.com/keycloak/keycloak/issues/50678) | Add ML-DSA JCE algorithm mapping to JavaAlgorithm | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| [#50679](https://github.com/keycloak/keycloak/issues/50679) | Support loading ML-DSA keys from Java keystores | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| [#50680](https://github.com/keycloak/keycloak/issues/50680) | Add test coverage for truststore loading with PQC certificates | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| [#50789](https://github.com/keycloak/keycloak/issues/50789) | Verify LDAP federation compatibility on PQC-capable JDK | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| [#50868](https://github.com/keycloak/keycloak/issues/50868) | Make embedded Infinispan support PQC | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| [#50938](https://github.com/keycloak/keycloak/issues/50938) | Loadtesting with PQC enabled | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| [#50940](https://github.com/keycloak/keycloak/issues/50940) | Test CloudNativePG database connection for PQC readiness | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |
| [#50941](https://github.com/keycloak/keycloak/issues/50941) | Update the proxy quickstarts to use PQC | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | task | N/A | 0 | |

---

## Additional Issues (Not in Main Hierarchy)

These issues exist but are not tracked as sub-issues in the main tree. They may be:
- Standalone implementation tasks
- Infrastructure/dependency issues
- Recently created issues not yet linked

### Infrastructure & Dependencies

| Issue # | Status | Title | Domains |
|---------|--------|-------|---------|
| [50355](https://github.com/keycloak/keycloak/issues/50355) | OPEN | Post-Quantum Keys choice for signing/encryption | UI/UX configuration |
| *Need to create* | — | **Database Schema: Expand REALM_ATTRIBUTE.VALUE for PQC Algorithm Strings** | Database schema (should be under [#48823](https://github.com/keycloak/keycloak/issues/48823)) |

### OID4VCI Specific

| Issue # | Status | Title | Domains |
|---------|--------|-------|---------|
| [48415](https://github.com/keycloak/keycloak/issues/48415) | CLOSED | [OID4VCI] Review JwtProofValidator.JWK_PRIVATE_KEY_CLAIMS | Domain 26 |

---

## Domains Missing GitHub Issues

Based on the comprehensive analysis of 63 domains in `pqc_overview.html`, the following domains and gaps **may require new GitHub issues**:

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
| **Database Schema** | Database Schema: Expand REALM_ATTRIBUTE.VALUE for PQC Algorithm Strings | 255-character column limit may be insufficient for concatenated PQC algorithm identifiers | Infrastructure |

### LOW Priority - New Issues Needed

| GAP/Domain | Title | Reason | Affected Domains |
|------------|-------|--------|------------------|
| **Domains 50-51** | Client SDK: Add AKP Support to JWTClientCredentialsProvider & DPoPGenerator | Client-side SDK missing ML-DSA support | Domains 50-51 |
| **GAP-11** | Dynamic Client Registration: RS256 Special-Case Logic | RS256 omitted from DCR response, no ML-DSA equivalent | Domain 35 |
| **Domain 52** | Docker Registry: Configurable Certificate Algorithm | Hardcoded RSA-2048 for test/dev path | Domain 52 |

### EXTERNAL DEPENDENCIES

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

**TLS / Network (#43691, #50674, #50675, #50676, #50677, #49968, #51255):**
- Domain 40: TLS 1.3 hybrid key exchange
- HTTP client, database TLS, OTLP exporter, container dependencies, Infinispan transport

**Other:**
- Domain 40: Organizations
- Domain 41: FAPI policy enforcement
- Domain 47: Client public key loader

### Domains NOT Covered (25 New Issues Needed)

- **CRITICAL (3):** GAP-1, GAP-4, GAP-9
- **HIGH (3):** Domain 48, Domain 53, Domain 46
- **MEDIUM (12):** GAP-5/16, Domains 29/32, GAP-19, GAP-20, GAP-21, GAP-22, GAP-23, Domain 49, GAP-10, GAP-13, GAP-18, Database Schema
- **LOW (3):** Domains 50-51, GAP-11, Domain 52
- **EXTERNAL (5):** GAP-6, GAP-7, GAP-8, Domain 37, Domain 44

---

## Next Steps

1. **Review the 25 new issues** listed in "Domains Missing GitHub Issues" section
2. **Create new GitHub issues** for CRITICAL and HIGH priority gaps first
3. **Link new issues** to appropriate parent issues in the hierarchy:
   - ML-DSA signing issues → under #43692
   - ML-KEM encryption issues → under #50299 or new encryption parent
   - SAML issues → under #50292
   - Keystore issues → reference #48822
4. **Update tracking** in #48829 (gap analysis) as new areas are identified
5. **Document EXTERNAL DEPENDENCY blockers** with no immediate action items

---

**Analysis Source:** `pqc_overview.html` (63 domains analyzed)  
**GitHub Issue Hierarchy:** Verified from GitHub API (2026-09-15)
