# PQC Readiness — Agent Brief

## Context

This work sits under the Keycloak Post-Quantum Cryptography (PQC) readiness
initiative. The relationship between the relevant GitHub issues is:

```
#43690  Post-Quantum Cryptography (PQC) readiness          ← ROOT epic
  └── #48821  PQC support for OAuth 2.0 and OpenID Connect
  └── #48819  Create inventory of cryptography in Keycloak  ← INVENTORY (this work also addresses)
  └── #48829  Review if there are any areas not identified   ← SUB-ISSUE (this work directly addresses)
        around PQC readiness
  └── #48824  Implement ML-DSA token signing (GAP-15)
  └── #48823  Operator migration guidance
  └── #48820  TLS / cluster PQC (separate track)
  └── #50292  SAML PQC
  └── #50294  SAML signing algorithm URIs
  └── #50295  SAML encryption (ML-KEM)
```

All GitHub issue links follow the pattern:
  https://github.com/keycloak/keycloak/issues/<number>

---

## What This Work Is

The `pqc_overview.html` file is the primary
deliverable for issues **#48829** (gap review) and **#48819** (crypto
inventory). It maps every asymmetric cryptography domain in the Keycloak
codebase to its current PQC state, identifies concrete gaps, and links to
per-domain implementation plans.

`Domain_1_Access_Token_ID_Token_signing.html` is the first per-domain
implementation plan, covering the code changes required to support ML-DSA
(and, by the same pattern, FN-DSA and SLH-DSA) for OAuth2/OIDC access token
and ID token signing.

---

## Files

| File | Purpose |
|------|---------|
| `pqc_overview.html` | Master overview table — all 61 domains, PQC state badges, gap tags (clickable → gap reference), issue links, links to domain files |
| `Domain_1_Access_Token_ID_Token_signing.html` | Implementation plan for Domain 1 (ML-DSA token signing) |
| `PQC_READINESS_AGENT.md` | This file — context, task list, conventions |

---

## Task List — Per Domain

For **each domain row** in the overview table, the work is:

**[1] Plain-language explanation**
   Describe what the domain does in simple terms (e.g. "this is how an
   OAuth2 access token gets signed"). No jargon beyond what is necessary.

**[2] Exact code gap**
   Identify precisely what is missing or hardcoded at the source level —
   which file, which line, what the current behaviour is, and what needs
   to change. Ground every claim in the actual codebase (read the file,
   don't speculate).

**[3] GitHub issue status**
   For each domain, determine:
   - Does an existing issue already cover this gap? (check the issue numbers
     already referenced in the overview table)
   - If yes, state the issue number and what it covers.
   - If no, flag that a new sub-issue under #43690 (or #48821 for
     OAuth2/OIDC domains) should be created, and draft the title and
     one-paragraph description.

**[4] Implementation plan (Markdown)**
   If code changes are required (i.e. PQC state is BLOCKED or PARTIAL),
   create a linked Markdown file. Markdown is preferred from Domain 4 onwards
   because it is cheaper to produce and can be pasted directly into a GitHub
   issue body.

   Naming convention:  `Domain_<N>_<Short_Name>.md`
   e.g. `Domain_4_UserInfo_Endpoint.md`

   Structure each file so it can be pasted directly into a GitHub issue:
   - H1 title matching the domain name
   - ## What is this? (plain-language explanation)
   - ## Gap (exact file + line + what needs changing)
   - ## Required Changes (summary table then per-change detail)
   - ## What does NOT need changing
   - ## Dependencies
   - Footer: `> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690)`

   Link conventions from overview:
   - Add a forward link from the overview table row's "Required Fix" cell:
     `<a href="Domain_N_Name.md" class="detail-link">→ View implementation plan</a>`

   Domains with state PENDING PROVIDERS or EXTERNAL DEPENDENCY do not need
   an implementation plan file.

---

## Domain Map — All 61 Rows

Rows are numbered by the `#` column in the overview table. Sections group
related rows. Work through them in section order.

### Section: Token Lifecycle — OAuth2 / OIDC
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 1 | Access Token / ID Token Signing | PARTIAL | GAP-4, GAP-15 | #48824 ✅ — impl plan created |
| 2 | ID Token / JARM Encryption (CEK) | BLOCKED | GAP-4 | none — new issue needed |
| 3 | Backchannel Logout Token Signing | PENDING PROVIDERS | — | #48823 (guidance only) |
| 4 | UserInfo Endpoint — Signed & Encrypted | BLOCKED | GAP-21 | none — new issue needed |
| 5 | Introspection — Embedded JWT | (empty) | — | — |
| 6 | Token Verification | PENDING PROVIDERS | — | auto when providers exist |

### Section: Authorization Request Objects — JAR / PAR / JARM
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 7 | JAR — Signed Request Object Verification | PENDING PROVIDERS | — | — |
| 8 | JAR — Encrypted Request Object Decryption | BLOCKED | GAP-20 | none — new issue needed |
| 9 | JARM — Signed Authorization Response | PENDING PROVIDERS | — | — |

### Section: Federated Client Authentication
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 54 | Federated JWT Client Authentication | PENDING PROVIDERS | — | — |

### Section: Client Authentication
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 10 | private_key_jwt Client Authentication | PENDING PROVIDERS | — | — |
| 11 | X.509 mTLS Client Authentication | SAFE | GAP-6 (FIPS only) | — |
| 12 | Attestation-Based Client Authentication | PARTIAL | GAP-22, GAP-1 | #48824 (partial) |

### Section: End-User Authentication
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 13 | X.509 Browser Authentication Flow | SAFE | GAP-6 (FIPS) | — |
| 14 | WebAuthn / Passkeys (FIDO2) | EXTERNAL DEPENDENCY | — | none (spec-blocked) |

### Section: Token Binding
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 15 | DPoP | PARTIAL | GAP-1 | none — new issue needed |

### Section: CIBA
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 16 | CIBA Signed Backchannel Auth Request | PARTIAL | GAP-8, GAP-17 | none — new issue needed |

### Section: Token Exchange Grant (RFC 8693)
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 57 | Token Exchange — Verification & Signing | PENDING PROVIDERS | — | — |

### Section: Device Authorization Grant (RFC 8628)
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 58 | Device Authorization Grant — Token Signing | PENDING PROVIDERS | — | — |

### Section: JWT Bearer Authorization Grant (RFC 7523)
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 17 | JWT Authorization Grant — Assertion Verification | PENDING PROVIDERS | — | — |

### Section: SAML Protocol
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 18 | SAML Assertion & Document Signing | BLOCKED | GAP-2, GAP-9 | #50292, #50294 (partial) |
| 19 | SAML Assertion Encryption | BLOCKED | GAP-3 | #50292, #50295 (partial) |
| 20 | SAML IdP Broker — SP Metadata & Federation | BLOCKED | GAP-9 | #50292, #50295 (partial) |

### Section: Trust Broker
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 55 | Default Trust Identity Provider | PENDING PROVIDERS | — | — |

### Section: SAML Metadata & Artifact Resolution
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 59 | SAML Metadata Public Key Loader | SAFE (loading layer) | GAP-2, GAP-9 | blocked on rows 18-20 |
| 60 | SAML Artifact Resolution | BLOCKED | GAP-2, GAP-9 | blocked on rows 18-20 |
| 61 | SAML2Signature — Hardcoded RSA-SHA1 Default | BLOCKED | GAP-2 | #50292 |

### Section: OIDC / OAuth2 Identity Provider Broker
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 21 | OIDC IdP — Token Verification & JWE Decryption | PARTIAL | GAP-10 | none — new issue needed |
| 22 | Kubernetes Identity Provider | PENDING PROVIDERS | — | — |
| 23 | SPIFFE / SVID Identity Provider | PENDING PROVIDERS | — | — |

### Section: SD-JWT — Selective Disclosure JWT
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 56 | SD-JWT Issuer Signing & Key Binding | PENDING PROVIDERS | — | — |

### Section: OID4VC — Verifiable Credentials Issuance
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 24 | JWT-VC / SD-JWT Credential Signing | PENDING PROVIDERS | — | — |
| 25 | LD-Proof Credential Signing | BLOCKED | GAP-7 | none — new issue needed |
| 26 | OID4VC Key Binding — JWT Proof Validation | PENDING PROVIDERS | — | — |
| 27 | OID4VC c_nonce JWT Signing | BLOCKED | GAP-18 | none — new issue needed |

### Section: Realm Key Management
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 28 | Generated RSA Signing Key Provider | BLOCKED | GAP-15 | #48824 |
| 29 | Generated RSA Encryption Key Provider | BLOCKED | GAP-15 | none for ML-KEM — new needed |
| 30 | Generated ECDSA Signing Key Provider | BLOCKED | GAP-15 | #48824 |
| 31 | Generated EdDSA Signing Key Provider | BLOCKED | GAP-15 | #48824 |
| 32 | Generated ECDH Encryption Key Provider | BLOCKED | — | none — new issue needed |
| 33 | Imported RSA Signing Key Provider | BLOCKED | GAP-5, GAP-16 | none — new issue needed |
| 34 | Java Keystore Key Provider | BLOCKED | GAP-5, GAP-16 | none — new issue needed |

### Section: Client Registration & Discovery
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 35 | Dynamic Client Registration Tokens | SAFE | GAP-11, GAP-12 | guidance in #48823 |
| 36 | OIDC Well-Known Discovery | PARTIAL | GAP-23 | none — new issue needed |

### Section: SSF / CAEP
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 37 | Security Event Token (SET) Signing | EXTERNAL DEPENDENCY | — | spec-blocked |

### Section: Admin CLI
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 38 | kcadm.sh / kcreg.sh — private_key_jwt | BLOCKED | GAP-13 | none — new issue needed |

### Section: Cluster / Infinispan
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 39 | JGroups ASYM_ENCRYPT | SAFE (production) | GAP-14 | #48823 (guidance) |

### Section: Organisations
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 40 | Organisation Invitation Token Verification | PENDING PROVIDERS | — | — |

### Section: FAPI / Client Policy Enforcement
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 41 | FAPI Algorithm Allowlist Enforcement | BLOCKED | GAP-8, GAP-17 | none — spec-gated, new issue needed |

### Section: Crypto Provider Backend
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 42 | CryptoProvider — BouncyCastle Default | PARTIAL | GAP-19 | none — new issue needed |
| 43 | CryptoProvider — FIPS 140-2 (BC-FIPS) | BLOCKED | GAP-6 | none — external dependency |
| 44 | CryptoProvider — WildFly Elytron | EXTERNAL DEPENDENCY | — | none — external dependency |

### Section: JWK / JWKS
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 45 | JWK Serialisation & Thumbprint | PARTIAL | GAP-1 | none — new issue needed |

### Section: Realm Bootstrap
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 46 | Default Realm Key Providers (Bootstrap) | BLOCKED | — | none — depends on #48824 |

### Section: Client Public Key Loading
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 47 | Client Public Key Loader (JWKS URL & Cert) | PARTIAL | — | auto when providers exist |

### Section: JWKS Endpoint
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 48 | Realm JWKS Endpoint — Key Serialisation | BLOCKED | — | none — new issue needed |

### Section: Admin API — Client Key Management
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 49 | Admin API — Client Certificate & Keypair Generation | BLOCKED | — | none — new issue needed |

### Section: Client SDK
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 50 | Client SDK — JWT Client Credentials Provider | PARTIAL | — | none — new issue needed |

### Section: Client SDK — DPoP
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 51 | Client SDK — DPoP Proof Generation | PARTIAL | — | none — new issue needed |

### Section: Docker Protocol
| # | Domain | PQC State | Gap tags | Existing issue |
|---|--------|-----------|----------|----------------|
| 52 | Docker Registry — Self-Signed Cert Generation | BLOCKED | — | low priority, new if needed |
| 53 | Client Asymmetric Signature Verifier Stack | BLOCKED (RSA path) | — | none — new issue needed |

---

## Progress Tracker

Mark each domain as it is completed:

- [x] Domain 1 — Access Token / ID Token Signing (impl plan: Domain_1_Access_Token_ID_Token_signing.html)
- [x] Domain 2 — ID Token / JARM Encryption (impl plan: Domain_2_ID_Token_JARM_Encryption.html)
- [x] Domain 3 — Backchannel Logout Token Signing (PENDING PROVIDERS; Domain_3_Backchannel_Logout_Token_Signing.md; guidance → #48823)
- [x] Domain 4 — UserInfo Endpoint (impl plan: Domain_4_UserInfo_Endpoint.md)
- [x] Domain 5 — Introspection Embedded JWT (no gap — resolves with Domain 1; Domain_5_Introspection_Embedded_JWT.md)
- [x] Domain 6 — Token Verification (no gap — resolves with Domain 1; Domain_6_Token_Verification.md)
- [x] Domain 7 — JAR Signed Request Object Verification (no gap — resolves with Domain 1; Domain_7_JAR_Signed_Request_Object.md)
- [x] Domain 8 — JAR Encrypted Request Object Decryption (impl plan: Domain_8_JAR_Encrypted_Request_Object.md)
- [x] Domain 9 — JARM Signed Authorization Response (PENDING PROVIDERS; Domain_9_JARM_Signed_Authorization_Response.md; no independent code change needed)
- [x] Domain 10 — private_key_jwt Client Authentication (PENDING PROVIDERS; Domain_10_Private_Key_JWT_Client_Authentication.md; no independent code change needed)
- [x] Domain 11 — X.509 mTLS Client Authentication (SAFE; Domain_11_X509_mTLS_Client_Authentication.md; GAP-6 FIPS blocker only; no Keycloak code changes needed)
- [x] Domain 12 — Attestation-Based Client Authentication (PARTIAL; Domain_12_Attestation_Based_Client_Authentication.md; GAP-22 algorithm enforcement TODO + GAP-1 JWK thumbprint)
- [x] Domain 13 — X.509 Browser Authentication Flow (SAFE; Domain_13_X509_Browser_Authentication.md; identical to Domain 11 — same CertificateValidator, same GAP-6 FIPS blocker only)
- [x] Domain 14 — WebAuthn / Passkeys (FIDO2) (EXTERNAL DEPENDENCY; Domain_14_WebAuthn_Passkeys.md; spec-blocked: FIDO Alliance + W3C + IANA + hardware vendors; no Keycloak work actionable)
- [x] Domain 15 — DPoP (Demonstrating Proof of Possession) (PARTIAL; Domain_15_DPoP.md; GAP-1 JWK thumbprint for AKP is critical blocker; signature verification is SPI-driven and ready)
- [x] Domain 16 — CIBA Signed Backchannel Auth Request (PARTIAL; Domain_16_CIBA_Signed_Backchannel_Auth.md; GAP-8 FAPI allowlist spec-gated; verification is SPI-driven and ready; internal serialization is quantum-safe)
- [x] Domain 17 — JWT Authorization Grant — Assertion Verification (PENDING PROVIDERS; Domain_17_JWT_Authorization_Grant.md; fully SPI-driven; no independent gap)
- [x] Domain 18 — SAML Assertion & Document Signing (BLOCKED; Domain_18_SAML_Signing.md; GAP-2 no ML-DSA XML URIs + GAP-9 hardcoded RS256 key selection; two-level gap)
- [x] Domain 19 — SAML Assertion Encryption (BLOCKED; Domain_19_SAML_Encryption.md; GAP-3 no ML-KEM in XMLEncryptionUtil; W3C/IETF spec-blocked)
- [x] Domain 20 — SAML IdP Broker — SP Metadata & Federation (BLOCKED; Domain_20_SAML_IdP_Broker.md; same GAP-2 + GAP-9 as Domain 18; ML-DSA keys excluded from metadata export)
- [x] Domain 21 — OIDC IdP — Token Verification & JWE Decryption (PARTIAL; Domain_21_OIDC_IdP_Broker.md; GAP-10 hardcoded RS256 outbound; inbound SPI-driven)
- [x] Domain 22 — Kubernetes Identity Provider (PENDING PROVIDERS; Domain_22_Kubernetes_IdP.md; SPI-driven)
- [x] Domain 23 — SPIFFE / SVID Identity Provider (PENDING PROVIDERS; Domain_23_SPIFFE_IdP.md; SPI-driven)
- [x] Domain 24 — JWT-VC / SD-JWT Credential Signing (PENDING PROVIDERS; Domain_24_JWT_VC_Signing.md; SPI-driven)
- [x] Domain 25 — LD-Proof Credential Signing (BLOCKED; Domain_25_LD_Proof_Signing.md; GAP-7 hardcoded Ed255192018Suite)
- [x] Domain 26 — OID4VC Key Binding JWT Proof (PENDING PROVIDERS; Domain_26_OID4VC_Key_Binding.md; SPI-driven)
- [x] Domain 27 — OID4VC c_nonce JWT Signing (BLOCKED; Domain_27_OID4VC_CNonce.md; GAP-18 hardcoded ES256/RS256)
- [x] Domain 28 — Generated RSA Signing Key Provider (BLOCKED; Domain_28_Generated_RSA_Signing_Keys.md; GAP-15 no ML-DSA keygen)
- [x] Domain 29 — Generated RSA Encryption Key Provider (BLOCKED; Domain_29_Generated_RSA_Encryption_Keys.md; no ML-KEM keygen provider)
- [x] Domain 30 — Generated ECDSA Signing Key Provider (BLOCKED; Domain_30_Generated_ECDSA_Signing_Keys.md; GAP-15)
- [x] Domain 31 — Generated EdDSA Signing Key Provider (BLOCKED; Domain_31_Generated_EdDSA_Signing_Keys.md; GAP-15)
- [x] Domain 32 — Generated ECDH Encryption Key Provider (BLOCKED; Domain_32_Generated_ECDH_Encryption_Keys.md; ML-KEM keygen needed)
- [x] Domain 33 — Imported RSA Signing Key Provider (BLOCKED; Domain_33_Imported_RSA_Signing_Keys.md; GAP-5, GAP-16)
- [x] Domain 34 — Java Keystore Key Provider (BLOCKED; Domain_34_Java_Keystore_Provider.md; GAP-5, GAP-16)
- [x] Domain 35 — Dynamic Client Registration Tokens (SAFE; Domain_35_Client_Registration_Tokens.md; GAP-11, GAP-12)
- [x] Domain 36 — OIDC Well-Known Discovery (PARTIAL; Domain_36_OIDC_Well_Known_Discovery.md; GAP-23)
- [x] Domain 37 — SSF / CAEP SET Signing (EXTERNAL DEPENDENCY; Domain_37_SET_Signing.md; CAEP spec-blocked)
- [x] Domain 38 — Admin CLI private_key_jwt (BLOCKED; Domain_38_Admin_CLI.md; GAP-13)
- [x] Domain 39 — JGroups ASYM_ENCRYPT (SAFE; Domain_39_JGroups_ASYM_ENCRYPT.md; GAP-14 operator guidance)
- [x] Domain 40 — Organisation Invitation Token (PENDING PROVIDERS; Domain_40_Organisation_Invitation_Tokens.md)
- [x] Domain 41 — FAPI Algorithm Allowlist (BLOCKED; Domain_41_FAPI_Algorithm_Allowlist.md; GAP-8, GAP-17 spec-gated)
- [x] Domain 42 — CryptoProvider BouncyCastle Default (PARTIAL; Domain_42_CryptoProvider_BouncyCastle.md; GAP-19)
- [x] Domain 43 — CryptoProvider FIPS 140-2 (BLOCKED; Domain_43_CryptoProvider_FIPS.md; GAP-6 external)
- [x] Domain 44 — CryptoProvider WildFly Elytron (EXTERNAL DEPENDENCY; Domain_44_CryptoProvider_Elytron.md)
- [x] Domain 45 — JWK Serialisation & Thumbprint (PARTIAL; Domain_45_JWK_Serialisation_Thumbprint.md; GAP-1)
- [x] Domain 46 — Realm Bootstrap (BLOCKED; Domain_46_Realm_Bootstrap.md; depends on GAP-15)
- [x] Domain 47 — Client Public Key Loader (PARTIAL; Domain_47_Client_Public_Key_Loader.md; depends on providers)
- [x] Domain 48 — Realm JWKS Endpoint (BLOCKED; Domain_48_Realm_JWKS_Endpoint.md; AKP null branch)
- [x] Domain 49 — Admin API Client Keypair Generation (BLOCKED; Domain_49_Admin_API_Client_Keypair.md)
- [x] Domain 50 — Client SDK JWT Credentials (PARTIAL; Domain_50_Client_SDK_JWT_Credentials.md; no AKP case)
- [x] Domain 51 — Client SDK DPoP Proof Generation (PARTIAL; Domain_51_Client_SDK_DPoP.md; RSA convenience only)
- [x] Domain 52 — Docker Registry Cert Generation (BLOCKED; Domain_52_Docker_Registry_Cert.md; low priority)
- [x] Domain 53 — Client Asymmetric Signature Verifier Stack (BLOCKED; Domain_53_Client_Asymmetric_Verifier.md; RSA guard + no AKP path)
- [x] Domain 54 — Federated JWT Client Authentication (PENDING PROVIDERS; Domain_54_Federated_JWT_Client_Auth.md)
- [x] Domain 55 — Default Trust Identity Provider (PENDING PROVIDERS; Domain_55_Default_Trust_IdP.md)
- [x] Domain 56 — SD-JWT Issuer Signing (PENDING PROVIDERS; Domain_56_SD_JWT_Issuer.md)
- [x] Domain 57 — Token Exchange Grant (PENDING PROVIDERS; Domain_57_Token_Exchange.md)
- [x] Domain 58 — Device Authorization Grant (PENDING PROVIDERS; Domain_58_Device_Authorization.md)
- [x] Domain 59 — SAML Metadata Public Key Loader (SAFE loading / BLOCKED end-to-end; Domain_59_SAML_Metadata_Loader.md; GAP-2, GAP-9)
- [x] Domain 60 — SAML Artifact Resolution (BLOCKED; Domain_60_SAML_Artifact_Resolution.md; GAP-2, GAP-9)
- [x] Domain 61 — SAML2Signature RSA-SHA1 Default (BLOCKED; Domain_61_SAML2Signature_Default.md; GAP-2)

---

## GitHub Issues Tracking

This section tracks which GitHub issues exist, which need updates, and which need to be created based on the domain analysis.

### Existing Issues (Under #43690)

| Issue # | Title | Covers | Status | Action Needed |
|---------|-------|--------|--------|---------------|
| [#48821](https://github.com/keycloak/keycloak/issues/48821) | PQC support for OAuth 2.0 and OpenID Connect | Core OIDC/OAuth2 PQC | Open | Parent for OIDC domains |
| [#48819](https://github.com/keycloak/keycloak/issues/48819) | Create inventory of cryptography in Keycloak | Crypto inventory | Open | Fulfilled by `pqc_overview.html` |
| [#48829](https://github.com/keycloak/keycloak/issues/48829) | Review if there are any areas not identified around PQC readiness | Gap review | Open | Fulfilled by domain analysis |
| [#48824](https://github.com/keycloak/keycloak/issues/48824) | Implement ML-DSA token signing | GAP-15: ML-DSA providers | Open | Covers Domains 1, 28-31 (key gen) |
| [#48823](https://github.com/keycloak/keycloak/issues/48823) | Operator migration guidance | Migration docs | Open | Should document: Domain 3 (logout token coupling), Domain 11/13 (CA migration), Domain 35 (RS256 special case), Domain 39 (ASYM_ENCRYPT deprecation) |
| [#48820](https://github.com/keycloak/keycloak/issues/48820) | TLS / cluster PQC | TLS, mTLS, cluster | Open | Separate track (not covered by domain analysis) |
| [#50292](https://github.com/keycloak/keycloak/issues/50292) | SAML PQC | SAML (parent) | Open | Parent for Domains 18-20, 59-61 |
| [#50294](https://github.com/keycloak/keycloak/issues/50294) | SAML signing algorithm URIs | GAP-2: XML Signature URIs | Open | Covers Domain 18, 20, 59, 60, 61 |
| [#50295](https://github.com/keycloak/keycloak/issues/50295) | SAML encryption (ML-KEM) | GAP-3: XML Encryption | Open | Covers Domain 19 |

### New Issues Needed

#### Critical / High Priority

| Gap | Title | Description | Affects Domains | Priority |
|-----|-------|-------------|-----------------|----------|
| **GAP-1** | Add AKP (ML-DSA) support to JWK thumbprint computation (RFC 7638) | `JWKSUtils.computeThumbprint()` does not support `KeyType.AKP`. Blocks DPoP and Attestation-Based Client Auth with ML-DSA keys. Fix: Add `AKP` to `JWK_THUMBPRINT_REQUIRED_MEMBERS` map with `crv` and `x` fields. | 12, 15 | **CRITICAL** |
| **GAP-4** | Implement ML-KEM CekManagementProvider for OIDC token encryption | No ML-KEM `CekManagementProviderFactory` exists. All OIDC token encryption (ID tokens, JARM, UserInfo) uses RSA-OAEP or ECDH-ES. Required before any outbound token encryption can be quantum-safe. | 2, 4, 8 | **HIGH** |
| **GAP-9** | SAML: replace hardcoded RS256 key selection with configurable algorithm | 4 call sites hardcode `Algorithm.RS256` when selecting SAML signing keys (SamlProtocol.java:544, SamlService.java:965, SAMLIdentityProvider.java:506, +1 more). Even after ML-DSA URIs exist (GAP-2), these lookups bypass ML-DSA keys. | 18, 20, 59, 60 | **HIGH** |
| **GAP-22** | Attestation-Based Client Auth: enforce asymmetric algorithm requirement | Algorithm enforcement TODO at line 418 of `AttestationBasedClientAuthenticator.java` is not implemented. Should reject symmetric algorithms (HS256, etc.) and validate algorithm type. | 12 | **MEDIUM** |

#### Medium Priority

| Gap | Title | Description | Affects Domains | Priority |
|-----|-------|-------------|-----------------|----------|
| **GAP-15** | Implement ML-DSA key generation providers | No `GeneratedAKPKeyProviderFactory` exists. `DefaultKeyManager.createFallbackKeys()` doesn't know about ML-DSA. `CryptoProvider.getKeyPairGen()` needs ML-DSA support. | 28, 30, 31 | **MEDIUM** (Covered by #48824) |
| **ML-KEM keygen** | Implement ML-KEM key generation provider | No ML-KEM equivalent of `GeneratedRsaEncKeyProviderFactory` exists. GAP-15 covers ML-DSA SIG keys only. ML-KEM ENC key generation is separate. Affects Domains 29, 32. | 29, 32 | **MEDIUM** |
| **GAP-16** | Realm key import: add ML-DSA support | Imported key providers need ML-DSA support. Needs both `ImportedRsaKeyProviderFactory` and `JavaKeystoreKeyProviderFactory` updates. Related to GAP-5. | 33, 34 | **MEDIUM** |
| **GAP-5** | Java Keystore Provider: add ML-DSA/ML-KEM import support | `JavaKeystoreKeyProviderFactory.mergedAlgorithmProperties()` only exposes RSA, EC, OKP in admin UI. BCFKS and PKCS12 can store PQC keys, but Keycloak doesn't import them. | 33, 34 | **MEDIUM** |
| **GAP-20** | JAR Encrypted Request Object: add ML-KEM support for inbound decryption | `DefaultTokenManager.decodeClientJWT()` uses realm ENC key for JWE decryption. No ML-KEM `CekManagementProviderFactory` exists. Distinct from GAP-4 (outbound). | 8 | **MEDIUM** |
| **GAP-21** | UserInfo Endpoint: per-client signature algorithm doesn't follow realm default | UserInfo uses dedicated per-client attribute `userinfo.response.signature.alg` that doesn't automatically follow realm default. Explicit per-client update required on migration. | 4 | **MEDIUM** |
| **GAP-23** | OIDC Discovery: make DEFAULT_CLIENT_AUTH_SIGNING_ALG_VALUES_SUPPORTED dynamic | Hardcoded static constant `DEFAULT_CLIENT_AUTH_SIGNING_ALG_VALUES_SUPPORTED = RS256` in `OIDCWellKnownProvider`. Should query registered asymmetric `SignatureProvider` implementations dynamically. | 36 | **MEDIUM** |

#### Lower Priority / Spec-Gated

| Gap | Title | Description | Affects Domains | Priority |
|-----|-------|-------------|-----------------|----------|
| **GAP-6** | BC-FIPS: blocked on ML-DSA certificate support | BouncyCastle FIPS 2.1.2 does not support PQC certificates. Hard blocker for FIPS deployments. External dependency on BC-FIPS release. | 11, 13, 42, 43 | **EXTERNAL** |
| **GAP-7** | LD-Proof Credential Signing: implement ML-DSA cryptographic suite | LD-Proof signing hardcoded to `Ed255192018Suite`. New ML-DSA LD suite needed (doesn't inherit from `SignatureProvider` SPI). | 25 | **LOW** |
| **GAP-8** | CIBA FAPI policy: add ML-DSA to allowed algorithms (pending FAPI 2.0 spec) | `FapiConstant.ALLOWED_ALGORITHMS` rejects ML-DSA. Cannot be fixed until FAPI 2.0 spec includes PQC. | 16, 41 | **SPEC-GATED** |
| **GAP-10** | OIDC IdP Broker: make outbound assertion algorithm configurable | `AbstractOAuth2IdentityProvider.java:697` hardcodes `Algorithm.RS256` fallback for `private_key_jwt` broker assertions. No path to select ML-DSA. | 21 | **LOW** |
| **GAP-11** | Client Registration: RS256 special-case logic for DEFAULT_SIGNATURE_ALGORITHM | `DescriptionConverter.java:416` omits `id_token_signed_response_alg` only for RS256. Needs ML-DSA handling. | 35 | **LOW** |
| **GAP-12** | Document DEFAULT_SIGNATURE_ALGORITHM fallback migration risk | When realm `DEFAULT_SIGNATURE_ALGORITHM` changes, clients without explicit config fall back to new default. Document in #48823. | 35 | **DOC** |
| **GAP-13** | Admin CLI: add algorithm parameter for private_key_jwt | `kcadm.sh` and `kcreg.sh` hardcode RS256 via `JWSBuilder.rsa256()`. Need `--sigalg` option. | 38 | **LOW** |
| **GAP-14** | JGroups ASYM_ENCRYPT: document switch to SSL_KEY_EXCHANGE | Production doesn't use `ASYM_ENCRYPT`, but operators who manually enabled it need guidance to switch to TLS. Document in #48823. | 39 | **DOC** |
| **GAP-17** | FAPI Algorithm Allowlist (broader than GAP-8?) | May be parent issue for all FAPI algorithm allowlists across domains. Verify relationship to GAP-8. | 16, 41 | **SPEC-GATED** |
| **GAP-18** | OID4VC c_nonce JWT: replace hardcoded algorithm selection | `JwtCNonceHandler.selectSigningKey()` hardcodes ES256 then RS256. Independent of realm default. | 27 | **LOW** |
| **GAP-19** | CryptoProvider BouncyCastle: verify ML-DSA provider integration | Early groundwork exists, but end-to-end ML-DSA flow needs verification. | 42 | **VERIFY** |

### Issues That May NOT Need Creation

These gaps are likely **covered by existing issues** or will be **automatically resolved** when providers exist:

- **Domains 3, 5, 6, 7, 9, 10, 17, 22, 23, 24, 26, 54-58**: PENDING PROVIDERS — covered by #48824
- **Domain 14**: EXTERNAL DEPENDENCY (FIDO Alliance/W3C) — no Keycloak issue needed
- **Domain 37**: EXTERNAL DEPENDENCY (CAEP spec) — no Keycloak issue needed
- **Domain 44**: EXTERNAL DEPENDENCY (WildFly Elytron) — no Keycloak issue needed
- **GAP-15**: Likely already covered by #48824 (ML-DSA token signing includes key generation)

### Action Items After All 61 Domains Complete

1. **Review #48824** scope: Does it cover GAP-15 (key generation) or just signing?
2. **Create new issues** for GAP-1, GAP-4, GAP-9, GAP-22 (critical/high priority gaps)
3. **Verify GAP-2/GAP-9 coverage** under #50294: Does it cover both URI registration AND hardcoded key selection?
4. **Check GAP-8/GAP-17 relationship**: Are these the same or separate issues?
5. **Update #48823** (operator guidance) with findings from Domains 3, 11, 13, 35, 39
6. **Review remaining domains** (21-61) for additional gaps

---

## Conventions

### PQC State Badges
| Badge | Meaning |
|-------|---------|
| BLOCKED | Code change definitely required; cannot use PQC today or after providers are added |
| PARTIAL | Some groundwork exists but end-to-end PQC does not work yet |
| PENDING PROVIDERS | Will work automatically once core ML-DSA/ML-KEM providers exist; no independent code gap |
| SAFE | Not quantum-vulnerable at the application layer (e.g. symmetric, or cert-chain is CA-controlled) |
| EXTERNAL DEPENDENCY | Blocked on an external spec or library outside Keycloak's control |

### When to create a new GitHub issue
Create a new issue (as a sub-issue under #43690) when:
- The domain is BLOCKED and has no existing tracking issue
- The domain is PARTIAL with a specific named gap that is not yet tracked
- The implementation work is self-contained enough to be a PR on its own

Do NOT create a new issue when:
- The domain is PENDING PROVIDERS (covered by #48824 or similar core work)
- The domain is EXTERNAL DEPENDENCY (nothing for Keycloak to do now)
- The gap is already tracked by an existing issue referenced in the table

### Algorithm scope
Per issue #48821, the scope is ML-DSA (FIPS 204), FN-DSA/Falcon (FIPS 206),
and SLH-DSA (FIPS 205) for signatures, and ML-KEM (FIPS 203) for key
encapsulation. All three signature algorithms follow the same Keycloak
implementation pattern — where Domain 1 covers ML-DSA, the same structure
applies for FN-DSA and SLH-DSA.
