# PQC Domain & Gap Cross-Reference Analysis

**Source files:** `pqc_overview.html` (63 domains, 30 gaps) ↔ `gh_issues/Github_PQC_issues_overview.md` (55 tracked issues)  
**Analysis date:** 2026-08-21  
**Analyst note:** This is an independent analysis. It does not reproduce the existing markdown summary — it re-examines every domain and gap from first principles and identifies errors and omissions in the existing coverage claims.

---

## Key Findings — Errors in Existing Markdown

The existing `Github_PQC_issues_overview.md` makes several coverage claims that do not hold up under direct inspection:

1. **Domains 1–17 claimed "covered" by `#48821`, `#43692`, `#48824`** — `#48824` is a **closed investigation spike** ("Investigate and plan what is needed for OIDC..."), not an implementation issue. Claiming it covers 17 domains is incorrect. Many of those domains are SPI-driven and will inherit ML-DSA automatically, but that is not the same as having active tracking.

2. **GAP-4 claimed addressed by `#50299`** — `#50299` tracks HPKE (Hybrid Public Key Encryption), which is a distinct algorithm. ML-KEM `CekManagementProviderFactory` (the fix for GAP-4) is an entirely different construct and is untracked.

3. **GAP-9 (4 hardcoded RS256 key-selection call sites)** — `#50294` covers adding ML-DSA algorithm support to `XMLSignatureUtil`/`XMLEncryptionUtil` but does not explicitly track fixing the 4 hardcoded RS256 call sites in `SamlProtocol.java`, `SamlService.java`, and `SAMLIdentityProvider.java`. GAP-9 is not covered.

4. **Domain 62 (OID4VP IdP)** — Entirely absent from the markdown. No GitHub issue exists. Two independent hardcodings: ES256 signing (GAP-24) and ECDH-ES/secp256r1 response encryption (GAP-25), one of which actively rejects ML-DSA at runtime.

5. **Domain 63 (OID4VCI credential response encryption)** — Missing from the markdown coverage section. Blocked by GAP-4.

6. **GAP-15 / Domain 46** — `#44142` adds ML-DSA key generation provider but does not track updating `DefaultKeyProviders.createProviders()` to bootstrap ML-DSA/ML-KEM keys for new realms. Domain 46 is untracked.

7. **GAP-26, GAP-30** — Two HIGH-severity runtime blockers (JWKS endpoint returns null for ML-DSA keys; client signature verifier throws "Key Type is not RSA" for AKP keys) have **no GitHub issues**.

8. **GAP-2 scope ambiguity** — The markdown lists GAP-2 as covered by `#50294` but that issue's scope focuses on algorithm implementations; the distinct requirement to register ML-DSA XML signature URIs in `SignatureAlgorithm.java` is not explicitly tracked.

---

## All 63 Domains — Extracted from `pqc_overview.html`

### Token Lifecycle — OAuth2 / OIDC
| # | Domain | PQC State |
|---|--------|-----------|
| 1 | Access Token / ID Token Signing | PARTIAL |
| 2 | ID Token / JARM Encryption (Outbound CEK) | BLOCKED |
| 3 | Backchannel Logout Token Signing | PENDING PROVIDERS |
| 4 | UserInfo Endpoint — Signed & Encrypted Response | BLOCKED |
| 5 | Introspection — Embedded JWT Response | PARTIAL |
| 6 | Token Verification (Identity & Session Tokens) | PENDING PROVIDERS |
| 7 | JAR — Signed Request Object Verification (Inbound) | PENDING PROVIDERS |
| 8 | JAR — Encrypted Request Object Decryption (Inbound) | BLOCKED |
| 9 | JARM — Signed Authorization Response | PENDING PROVIDERS |
| 10 | private_key_jwt Client Authentication | PENDING PROVIDERS |
| 11 | X.509 mTLS Client Authentication | SAFE (FIPS blocked) |
| 12 | Attestation-Based Client Authentication | PARTIAL |
| 13 | X.509 Browser Authentication Flow | SAFE (FIPS blocked) |
| 14 | WebAuthn / Passkeys (FIDO2) | EXTERNAL DEPENDENCY |
| 15 | DPoP (Demonstrating Proof of Possession) | PARTIAL |
| 16 | CIBA Signed Backchannel Auth Request | PARTIAL |
| 17 | JWT Authorization Grant — Assertion Verification | PENDING PROVIDERS |
| 54 | Federated JWT Client Authentication | PENDING PROVIDERS |
| 57 | Token Exchange — Subject/Actor Token Verification & Exchanged Token Signing | PENDING PROVIDERS |
| 58 | Device Authorization Grant — Token Signing | PENDING PROVIDERS |

### SAML Protocol
| # | Domain | PQC State |
|---|--------|-----------|
| 18 | SAML Assertion & Document Signing | BLOCKED |
| 19 | SAML Assertion Encryption | BLOCKED |
| 20 | SAML IdP Broker — SP Metadata & Federation Signing | BLOCKED |
| 59 | SAML Metadata Public Key Loader | SAFE (at key-loading layer) |
| 60 | SAML Artifact Resolution | BLOCKED |
| 61 | SAML2Signature — Hardcoded RSA-SHA1 Default | BLOCKED |

### Identity Provider Brokers
| # | Domain | PQC State |
|---|--------|-----------|
| 21 | OIDC IdP — Token Signature Verification & JWE Decryption | PARTIAL |
| 22 | Kubernetes Identity Provider | PENDING PROVIDERS |
| 23 | SPIFFE / SVID Identity Provider | PENDING PROVIDERS |
| 55 | Default Trust Identity Provider (Trust Broker) | PENDING PROVIDERS |

### OID4VC — Verifiable Credentials
| # | Domain | PQC State |
|---|--------|-----------|
| 24 | JWT-VC / SD-JWT Credential Signing | PENDING PROVIDERS |
| 25 | LD-Proof Credential Signing (Linked Data) | BLOCKED |
| 26 | OID4VC Key Binding — JWT Proof Validation | PENDING PROVIDERS |
| 27 | OID4VC c_nonce JWT Signing | BLOCKED |
| 56 | SD-JWT Issuer Signing & Key Binding Verification | PENDING PROVIDERS |
| 62 | OID4VP Identity Provider — Request Signing & Response Encryption | BLOCKED |
| 63 | OID4VCI Credential Response Encryption | BLOCKED |

### Realm Key Management
| # | Domain | PQC State |
|---|--------|-----------|
| 28 | Generated RSA Signing Key Provider | BLOCKED |
| 29 | Generated RSA Encryption Key Provider | BLOCKED |
| 30 | Generated ECDSA Signing Key Provider | BLOCKED |
| 31 | Generated EdDSA Signing Key Provider | BLOCKED |
| 32 | Generated ECDH Encryption Key Provider | BLOCKED |
| 33 | Imported RSA Signing Key Provider | BLOCKED |
| 34 | Java Keystore Key Provider (PKCS12 / BCFKS) | BLOCKED |

### Client Registration & Discovery
| # | Domain | PQC State |
|---|--------|-----------|
| 35 | Dynamic Client Registration Tokens | SAFE (with gaps) |
| 36 | OIDC Well-Known Discovery — Algorithm Advertisement | PARTIAL |

### SSF / CAEP
| # | Domain | PQC State |
|---|--------|-----------|
| 37 | Security Event Token (SET) Signing | EXTERNAL DEPENDENCY |

### Admin CLI
| # | Domain | PQC State |
|---|--------|-----------|
| 38 | kcadm.sh / kcreg.sh — private_key_jwt Auth | BLOCKED |

### Cluster / Infinispan / Organisations / FAPI
| # | Domain | PQC State |
|---|--------|-----------|
| 39 | JGroups ASYM_ENCRYPT (non-default / test config) | SAFE (production) |
| 40 | Organisation Invitation Token Verification | PENDING PROVIDERS |
| 41 | FAPI Algorithm Allowlist Enforcement | BLOCKED |

### Crypto Provider Backend
| # | Domain | PQC State |
|---|--------|-----------|
| 42 | CryptoProvider SPI — BouncyCastle Default Backend | PARTIAL |
| 43 | CryptoProvider SPI — FIPS 140-2/3 Backend (BC-FIPS) | BLOCKED |
| 44 | CryptoProvider SPI — WildFly Elytron Backend | EXTERNAL DEPENDENCY |

### JWK / JWKS / Bootstrap / Client Key Management / Client SDK / Docker
| # | Domain | PQC State |
|---|--------|-----------|
| 45 | JWK Serialisation & Thumbprint | PARTIAL |
| 46 | Default Realm Key Providers (Bootstrap) | BLOCKED |
| 47 | Client Public Key Loader (JWKS URL & Stored Cert) | PARTIAL |
| 48 | Realm JWKS Endpoint — Key Serialisation | BLOCKED |
| 49 | Admin API — Client Certificate & Keypair Generation | BLOCKED |
| 50 | Client SDK — JWT Client Credentials Provider | PARTIAL |
| 51 | Client SDK — DPoP Proof Generation | PARTIAL |
| 52 | Docker Registry — Self-Signed Certificate Generation | BLOCKED |
| 53 | Client Asymmetric Signature Verifier Context & Providers | BLOCKED |

---

## All 30 Gaps — Extracted from `pqc_overview.html`

| Gap ID | Title | Severity | Affected Domains |
|--------|-------|----------|-----------------|
| GAP-1 | JWK Thumbprint — AKP (ML-DSA) Keys | HIGH | 12, 15, 45 |
| GAP-2 | SAML Signing — No PQC Algorithm Entries | HIGH | 18, 20, 59, 60, 61 |
| GAP-3 | SAML Encryption — No ML-KEM Key Wrapping | HIGH | 19 |
| GAP-4 | JWE Key Management — No ML-KEM CEK Provider | HIGH | 2, 4, 8, 63 |
| GAP-5 | JavaKeystoreKeyProvider — ML-DSA Key Import | MEDIUM | 33, 34 |
| GAP-6 | FIPS 140-2 Backend — ML-DSA Availability Blocked | HIGH (EXTERNAL) | 11, 13, 43 |
| GAP-7 | OID4VC Linked Data Proof Suite — Hardcoded Ed25519 | MEDIUM (EXTERNAL) | 25 |
| GAP-8 | FAPI / Client Policy Allowlist — ML-DSA Actively Blocked | HIGH | 16, 41 |
| GAP-9 | SAML Protocol — Hardcoded RS256 Key Lookup | HIGH | 18, 20, 59, 60, 61 |
| GAP-10 | IdP Broker — Hardcoded RS256 / HS256 Fallback | MEDIUM | 21 |
| GAP-11 | Dynamic Client Registration — RS256 Special-Case Logic | LOW | 35 |
| GAP-12 | Constants.DEFAULT_SIGNATURE_ALGORITHM — RS256 Fallback | MEDIUM | 35 |
| GAP-13 | Admin CLI & Authz Client — Hardcoded RS256 | MEDIUM | 38 |
| GAP-14 | JGroups ASYM_ENCRYPT — RSA Cluster Key Exchange | LOW | 39 |
| GAP-15 | Realm Key Management — No ML-DSA Key Generation | HIGH | 1, 28, 29, 30, 31, 46 |
| GAP-16 | JavaKeystoreKeyProviderFactory — ML-DSA Excluded from Admin UI | MEDIUM | 33, 34 |
| GAP-17 | FAPI / CIBA Executor Factories — Admin UI Algorithm Lists | MEDIUM | 16, 41 |
| GAP-18 | JwtCNonceHandler — c_nonce Signing Hardcoded ES256/RS256 | MEDIUM | 27 |
| GAP-19 | No Minimum-Version Constraint on bcprov-jdk18on | MEDIUM | 42 |
| GAP-20 | JAR Encrypted Request Object — No ML-KEM Inbound Path | MEDIUM | 8 |
| GAP-21 | UserInfo Endpoint — Per-Client Signing & Encryption Gap | MEDIUM | 4 |
| GAP-22 | Attestation-Based Client Auth — Algorithm Enforcement | MEDIUM | 12 |
| GAP-23 | OIDC Discovery — Hardcoded RS256 CIBA Constant | LOW | 36 |
| GAP-24 | OID4VP — Hardcoded ES256 Signing / ACCEPTED_ALGORITHMS | HIGH | 62 |
| GAP-25 | OID4VP — ECDH-ES/secp256r1 Response Encryption (HAIP spec-gated) | LOW (EXTERNAL) | 62 |
| GAP-26 | JWKS Endpoint — AKP Branch in JWKSServerUtils.toJwk() | HIGH | 48 |
| GAP-27 | Admin API — Client Keypair Generation Hardcodes RSA | MEDIUM | 49 |
| GAP-28 | Client SDK — JWTClientCredentialsProvider Missing AKP | MEDIUM | 50 |
| GAP-29 | Client SDK — DPoPGenerator No ML-DSA Convenience Path | MEDIUM | 51 |
| GAP-30 | Client Signature Verifier — RSA Guard Blocks AKP Keys | HIGH | 53 |

---

## Domain-by-Domain Cross-Reference

### ✅ Covered by Existing GitHub Issues

These domains are addressed by explicit GitHub issues. Rationale is given for each.

#### Explicit Implementation Tracking

| Domain(s) | Gap(s) | GitHub Issue(s) | Coverage Assessment |
|-----------|--------|-----------------|---------------------|
| 1, 28, 30, 31 | GAP-15 | `#43692`, `#44142`, `#43684`, `#44143`, `#44144`, `#50678` | ML-DSA signing providers, key pair generation, token issuance and verification, `JavaAlgorithm` mappings. Solid coverage. |
| 18, 20, 59–61 | GAP-2, GAP-9 (partial — see note) | `#50292`, `#50294`, `#51421`, `#51422` | SAML ML-DSA signing tracked. **GAP-9 (4 hardcoded RS256 key-selection call sites) is NOT explicitly covered** — see "Requires New Issue". |
| 19 | GAP-3 | `#50292`, `#50295` | ML-KEM for SAML assertion encryption tracked via dual hybrid support issue. |
| 14 | — | `#50084`, `#50085`, `#50086` | WebAuthn ML-DSA: upgrade webauthn4j + add ML-DSA COSE algorithm IDs. Spec-gated but tracked. |
| 33, 34 | GAP-5 (partial — see note) | `#50679`, `#50680` | ML-DSA keystore loading and truststore test coverage tracked. **GAP-16 (admin UI option list not updated) is NOT explicitly covered.** |
| TLS/Network | — | `#43691`, `#50674`, `#50675`, `#50676`, `#50677`, `#49968`, `#51255` | TLS 1.3 hybrid key exchange, Quarkus PQC TLS adoption, OpenSSL container image, database TLS, Infinispan Hot Rod transport, HTTP client migration, OTLP exporter. Well-covered. |
| Infinispan (embedded) | — | `#50868` | Embedded Infinispan PQC support tracked. |
| Production perf/testing | — | `#50938`, `#50940`, `#50941` | Load testing, CloudNativePG TLS, proxy quickstarts. Tracked. |
| LDAP federation | — | `#50789` | LDAP federation on PQC-capable JDK tracked. |

#### External Dependencies (no Keycloak issue needed or possible)

| Domain(s) | Gap(s) | Assessment |
|-----------|--------|------------|
| 11, 13, 43 | GAP-6 | BC-FIPS 2.x with FIPS 140-3 validated ML-DSA/ML-KEM required. Hard blocker. No Keycloak issue can unblock this. |
| 44 | — | WildFly Elytron roadmap dependency. No Keycloak issue can unblock this. |
| 37 | — | CAEP Interoperability Profile pins transmitters to RS256. No Keycloak action until spec moves. |
| 25 | GAP-7 | LD-Proof cryptographic suite for ML-DSA requires spec-level work. No Keycloak issue can unblock this. |
| 62 (encryption only) | GAP-25 | HAIP spec mandates ECDH-ES/P-256 for `direct_post.jwt`. No Keycloak action until spec changes. |

#### SPI-Driven / No Independent Gap (will auto-support ML-DSA once core providers exist)

These domains are correctly noted as "PENDING PROVIDERS" in the HTML and have no independent code gap. They do not need their own GitHub issues — they inherit the ML-DSA support delivered by `#43692` and its sub-tasks.

| Domain(s) | Inherits From |
|-----------|---------------|
| 3, 5, 6, 7, 9, 10, 17 | Domain 1 / `#43692` — token signing/verification via `DefaultTokenManager` and `SignatureProvider` SPI |
| 22, 23 | Domain 21 / OIDC broker — SPI-driven JWT-SVID / Kubernetes token verification |
| 24, 26 | Domain 1 / `#43692` — OID4VC credential signing via `SignatureProvider` SPI |
| 40 | Domain 1 / `#43692` — organisation invitation token verification |
| 47 | Domain 1 / `#43692` — `JWKSUtils.getKeyWrappersForUse()` already handles AKP keys |
| 54, 55 | Domain 1 / `#43692` — federated JWT auth and trust broker both SPI-driven |
| 56, 57, 58 | Domain 1 / `#43692` — SD-JWT signing, token exchange, device flow all delegate to SPI paths |
| 59 | Domains 18–20 / `#50292` — SAML metadata key loading is algorithm-agnostic; fix depends on GAP-2/GAP-9 |
| 39 | `#48823` (operator migration docs) — JGroups ASYM_ENCRYPT is non-default; documentation-only action |

---

### ❌ Requires a New GitHub Issue

The following domains and gaps have **no existing GitHub issue** and need new issues created. Each entry explains exactly why no existing issue covers it.

---

#### 🔴 CRITICAL Priority

---

**1. GAP-1 — JWK Thumbprint for AKP (ML-DSA) Keys**
- **Affected domains:** 12, 15, 45
- **The problem:** `JWKSUtils.computeThumbprint()` throws `UnsupportedOperationException` for AKP key types. `JWK_THUMBPRINT_REQUIRED_MEMBERS` has no AKP entry.
- **Why not covered:** No existing issue tracks this. `#43692` tracks ML-DSA provider factories but does not mention JWK thumbprint computation.
- **Runtime impact:** DPoP `dpop_jkt` binding fails for ML-DSA keys. Attestation-based client auth fails when `cnf.jwk` is an AKP key.
- **Suggested parent:** Sub-issue under `#43692` or `#48821`

---

**2. GAP-4 — ML-KEM CekManagementProvider (JWE Key Management)**
- **Affected domains:** 2, 4, 8, 63
- **The problem:** No `CekManagementProviderFactory` for ML-KEM (FIPS 203) exists. All JWE key management uses RSA-OAEP or ECDH-ES. Clients requesting encrypted ID tokens, UserInfo responses, or encrypted JAR objects have no quantum-safe path.
- **Why not covered:** `#50299` (HPKE for JWE) is **a different algorithm** (Hybrid Public Key Encryption). ML-KEM direct encapsulation as a CEK management provider is a separate implementation and is not tracked anywhere.
- **Runtime impact:** All outbound JWE encryption remains quantum-vulnerable. Domain 63 (OID4VCI credential response encryption) also blocked.
- **Suggested parent:** New milestone under `#43690` or link under `#50299` as a sibling

---

**3. GAP-9 — SAML: Replace 4 Hardcoded RS256 Key-Selection Call Sites**
- **Affected domains:** 18, 20, 59, 60
- **The problem:** Even after GAP-2 adds ML-DSA XML signature URIs to `SignatureAlgorithm.java`, the following 4 call sites bypass the algorithm selection and directly request RS256 keys:
  - `SamlProtocol.java` (2 sites)
  - `SamlService.java` (1 site)
  - `SAMLIdentityProvider.java` line 507 (1 site)
- **Why not covered:** `#50294` ("Add ML-DSA algorithm support for XMLSignatureUtil and XMLEncryptionUtil") addresses the algorithm layer but its sub-issues (`#51421`, `#51422`) focus on key provider generation and protocol review — neither explicitly targets these call sites.
- **Runtime impact:** ML-DSA keys exist in realm but are never selected for SAML signing because key lookup always returns the RS256 key.
- **Suggested parent:** Sub-issue under `#50294`

---

#### 🟠 HIGH Priority

---

**4. GAP-26 — JWKS Endpoint: Add AKP Branch to `JWKSServerUtils.toJwk()`**
- **Affected domain:** 48
- **The problem:** `JWKSServerUtils.toJwk()` has `if/else if` branches only for `KeyType.RSA`, `KeyType.EC`, and `KeyType.OKP`. An AKP (ML-DSA) key falls through all branches and returns `null` — ML-DSA realm keys are silently omitted from the `/protocol/openid-connect/certs` JWKS endpoint.
- **Why not covered:** No existing issue mentions `JWKSServerUtils` or the public JWKS endpoint serialisation path. `#50678` adds `JavaAlgorithm` JCE mappings; `#50679` loads from keystores — neither touches endpoint serialisation.
- **Runtime impact:** Clients fetching the JWKS endpoint for token verification will never see ML-DSA keys. Token verification with ML-DSA will fail for all remote clients.
- **Suggested parent:** Sub-issue under `#43692` or `#48821`

---

**5. GAP-30 — Client Signature Verifier: Remove RSA-Only Guard**
- **Affected domain:** 53
- **The problem:** `ClientAsymmetricSignatureVerifierContext.getKey()` at line 36 throws `VerificationException("Key Type is not RSA")` for any non-RSA key. There is no AKP client verifier context or provider factory.
- **Why not covered:** No existing issue tracks this class. `#43692` adds ML-DSA server-side signing/verification, but client-side authentication uses a different code path (`ClientSignatureVerifier` SPI, not the token signing `SignatureProvider` SPI).
- **Runtime impact:** Any client attempting `private_key_jwt` authentication with an ML-DSA key will receive a hard `VerificationException` regardless of whether ML-DSA providers are registered.
- **Suggested parent:** Sub-issue under `#43692` or `#48821`

---

**6. Domain 46 — Realm Bootstrap: Add ML-DSA/ML-KEM Default Key Providers**
- **The problem:** `DefaultKeyProviders.createProviders()` hardcodes `rsa-generated` (SIG) and `rsa-enc-generated` (ENC, RSA-OAEP) as the only providers bootstrapped for every new realm. Even after `GeneratedAKPKeyProviderFactory` exists (tracked by `#44142`), new realms will not receive PQC keys by default unless this bootstrap logic is updated.
- **Why not covered:** `#44142` adds the ML-DSA key provider factory but does not track `DefaultKeyProviders`. These are distinct code changes in different files.
- **Runtime impact:** Every realm created after ML-DSA is available will still default to RSA-only keys. Operators must manually add PQC key providers per realm.
- **Suggested parent:** Sub-issue under `#44142` or `#48823`

---

**7. GAP-2 — SAML Signing: Add ML-DSA XML Signature URIs to `SignatureAlgorithm.java`**
- **Affected domains:** 18, 20, 59, 60, 61
- **The problem:** `SignatureAlgorithm.java` has no PQC algorithm URIs registered. The XML Digital Signatures layer is entirely classical.
- **Why not covered (ambiguity):** `#50294` is titled "Add ML-DSA algorithm support for XMLSignatureUtil and XMLEncryptionUtil" which implies this is in scope, but its sub-issues (`#51421`, `#51422`) focus on key generation and final review, not on registering new XML algorithm URIs in `SignatureAlgorithm.java`. If `#50294` does not explicitly include updating `SignatureAlgorithm.java`, a new sub-issue is required.
- **Recommended action:** Review `#50294` acceptance criteria and explicitly confirm or create a sub-issue for `SignatureAlgorithm.java` URI registration.
- **Suggested parent:** Sub-issue under `#50294`

---

**8. GAP-24 — OID4VP Identity Provider: Replace Hardcoded ES256 and `ACCEPTED_ALGORITHMS`**
- **Affected domain:** 62
- **The problem:** Two independent hardcodings:
  - `OID4VPIdentityProvider.signingKey()` hardcodes `Algorithm.ES256` at lines 196–197
  - `ACCEPTED_ALGORITHMS = List.of(Algorithm.ES256)` at line 83 is used by `requireAcceptedAlgorithm()` to actively **reject all non-ES256 algorithms**, including ML-DSA
- **Why not covered:** Domain 62 is **entirely absent from the existing markdown**. No issue exists at any level for OID4VP PQC readiness.
- **Runtime impact:** Even after ML-DSA providers exist, VP tokens signed with ML-DSA will be rejected at runtime by the OID4VP endpoint. This is a BLOCKED domain with an active runtime rejection.
- **Suggested parent:** New feature issue under `#43690`

---

**9. GAP-8 — FAPI Allowlist: Extend `FapiConstant.ALLOWED_ALGORITHMS` with ML-DSA**
- **Affected domains:** 16, 41
- **The problem:** `FapiConstant.ALLOWED_ALGORITHMS` contains only `PS256`, `PS384`, `PS512`, `ES256`, `ES384`, `ES512`. `SecureSigningAlgorithmExecutor.isSecureAlgorithm()` calls `ALLOWED_ALGORITHMS.contains(sigAlg)` — ML-DSA is actively rejected even after providers exist.
- **Why not covered:** `#48824` ("Investigate and plan what is needed for OIDC and OAuth 2.0 to be PQC ready") is **OPEN** as an investigation task. No implementation issue has been created as a follow-on for the FAPI allowlist fix.
- **Runtime impact:** All FAPI-constrained realms will reject ML-DSA for token signing, JAR, CIBA, and `private_key_jwt` — regardless of whether ML-DSA providers are registered.
- **Suggested parent:** Sub-issue under `#43692` or new issue linked to `#48824`

---

#### 🟡 MEDIUM Priority

---

**10. GAP-16 — JavaKeystoreKeyProviderFactory: ML-DSA Excluded from Admin UI**
- **Affected domains:** 33, 34
- **The problem:** `JavaKeystoreKeyProviderFactory.mergedAlgorithmProperties()` does not expose ML-DSA in the admin UI option list. `#50679` ("Support loading ML-DSA keys from Java keystores") tracks the provider-level import logic but does not mention the admin UI surface.
- **Suggested parent:** Sub-issue under `#50679`

---

**11. Domains 29, 32 — ML-KEM Key Generation Provider**
- **The problem:** No ML-KEM equivalent of `GeneratedRsaEncKeyProviderFactory` or `GeneratedEcdhKeyProviderFactory` exists. GAP-15 and `#44142` address **ML-DSA SIG key generation only**. ML-KEM ENC key generation is entirely untracked.
- **Why critical:** Without ML-KEM ENC key generation providers, the realm cannot hold ML-KEM keys — which means GAP-4 (ML-KEM CEK management) has no keys to operate on even after the provider is implemented.
- **Suggested parent:** New issue under `#43690` or sibling to `#44142`

---

**12. GAP-19 — Maven Enforcer: bcprov-jdk18on ≥ 1.78 Minimum Version**
- **Affected domain:** 42
- **The problem:** No Maven enforcer minimum-version constraint exists for `bcprov-jdk18on`. A downstream build with an older BOM (pre-1.78) could silently lose all ML-DSA support with no build-time failure.
- **Suggested parent:** Infrastructure issue under `#43690` or `#46333`

---

**13. GAP-20 — JAR Encrypted Request Object: No ML-KEM Inbound Decryption Path**
- **Affected domain:** 8
- **The problem:** `DefaultTokenManager.decodeClientJWT()` selects a realm private key by `KeyUse.ENC` for inbound JAR decryption. This is **distinct from GAP-4** (outbound token encryption) — inbound decryption requires three separate fixes: (1) `JWEAlgorithmProvider` for ML-KEM in `CryptoIntegration`, (2) `CekManagementProviderFactory` for ML-KEM, and (3) a realm ML-KEM ENC key provider. No issue tracks this inbound direction.
- **Suggested parent:** Sub-issue under `#43692` or `#48821`

---

**14. GAP-21 — UserInfo Endpoint: Per-Client Signing Attribute Migration**
- **Affected domain:** 4
- **The problem:** The `userinfo.response.signature.alg` client attribute does not automatically follow the realm default — each client must be explicitly updated during migration. There is no migration tooling or documentation tracking issue for this.
- **Suggested parent:** Sub-issue under `#48823` (production readiness)

---

**15. GAP-22 — Attestation-Based Client Auth: Algorithm Enforcement TODO**
- **Affected domain:** 12
- **The problem:** `AttestationBasedClientAuthenticator.java` has an unimplemented `[TODO]` at line 418 for algorithm-type enforcement. The check that client attestations must use an asymmetric algorithm is not yet coded.
- **Suggested parent:** Sub-issue under `#48824` or `#43692`

---

**16. GAP-23 — OIDC Discovery: Hardcoded RS256 CIBA Constant**
- **Affected domain:** 36
- **The problem:** `OIDCWellKnownProvider.DEFAULT_CLIENT_AUTH_SIGNING_ALG_VALUES_SUPPORTED` is a static constant hardcoded to RS256. This affects 4 OIDC discovery fields including CIBA signing algorithm support advertisement. The fix is to replace the constant with a dynamic lookup over registered asymmetric `SignatureProvider` algorithms.
- **Suggested parent:** Sub-issue under `#48824` or `#43692`

---

**17. Domain 49 — Admin API: Client Keypair Generation Algorithm Parameter**
- **The problem:** `ClientAttributeCertificateResource.generate()` and `generateAndGetKeystore()` both call `KeycloakModelUtils.generateKeyPairCertificate()` which hardcodes `KeyUtils.generateRsaKeyPair(keysize)`. No algorithm parameter is exposed in the Admin API or UI.
- **Suggested parent:** Sub-issue under `#43692` or `#48821`

---

**18. GAP-10 — IdP Broker: Configurable Outbound Assertion Algorithm**
- **Affected domain:** 21
- **The problem:** `AbstractOAuth2IdentityProvider.java` line 745 hardcodes `Algorithm.RS256` as the fallback for outbound `private_key_jwt` broker assertions. Line 737 hardcodes `Algorithm.HS256` for `client_secret_jwt`. No path exists to select ML-DSA as the outbound assertion algorithm.
- **Suggested parent:** Sub-issue under `#43692` or `#48824`

---

**19. GAP-13 — Admin CLI: Add `--sigalg` Parameter to `kcadm.sh` / `kcreg.sh`**
- **Affected domain:** 38
- **The problem:** `AuthUtil.getSignedRequestToken()` always produces RS256-signed JWTs via `new JWSBuilder().rsa256(keypair.getPrivate())`. No `--sigalg` parameter exists. In a realm requiring ML-DSA for client assertions, both CLI tools will be unable to authenticate via keystore-based flow. Note: `authz-client` library (`AuthzClientCryptoProvider.java`) has the same hardcoded RS256 pattern and is also untracked.
- **Suggested parent:** Sub-issue under `#43692` or `#48824`

---

**20. GAP-18 — OID4VC c_nonce JWT: Replace Hardcoded ES256/RS256 Selection**
- **Affected domain:** 27
- **The problem:** `JwtCNonceHandler.selectSigningKey()` hardcodes ES256 then RS256 fallback. It never consults `Constants.DEFAULT_SIGNATURE_ALGORITHM` or the realm's active key configuration. c_nonce JWTs will not use ML-DSA keys even after providers exist.
- **Suggested parent:** Sub-issue under `#43692` or the OID4VC tracking hierarchy

---

**21. GAP-12 — Constants.DEFAULT_SIGNATURE_ALGORITHM RS256 Fallback**
- **Affected domain:** 35
- **The problem:** `Constants.DEFAULT_SIGNATURE_ALGORITHM` is RS256. `DescriptionConverter.java` line 416 has RS256-specific special-case logic that omits `id_token_signed_response_alg` when the realm default is RS256; no equivalent treatment for ML-DSA. Clients using dynamic client registration will receive incorrect algorithm metadata when the realm default is ML-DSA.
- **Suggested parent:** Sub-issue under `#43692` or `#48824`

---

**22. GAP-17 — FAPI / CIBA Executor Factories: Admin UI Algorithm Lists**
- **Affected domains:** 16, 41
- **The problem:** `SecureSigningAlgorithmExecutorFactory` and `SecureCibaAuthenticationRequestSigningAlgorithmExecutorFactory` have hardcoded admin UI option lists (the set of algorithms offered to administrators). Even if `FapiConstant.ALLOWED_ALGORITHMS` is extended (GAP-8), the UI dropdown will not surface ML-DSA unless these factory option lists are also updated.
- **Note:** This is distinct from GAP-8. GAP-8 is runtime enforcement; GAP-17 is admin UI exposure.
- **Suggested parent:** Sub-issue under GAP-8's new issue

---

**23. Database Schema — REALM_ATTRIBUTE.VALUE Column Limit**
- **The problem:** The `REALM_ATTRIBUTE.VALUE` column is 255 characters. Concatenated PQC algorithm identifier strings (e.g. for hybrid composite algorithms) may exceed this limit. No issue tracks expanding the column.
- **Note:** The existing markdown acknowledges "Need to create" but no issue exists.
- **Suggested parent:** Infrastructure issue under `#43690`

---

#### 🟢 LOW Priority

---

**24. GAP-28 + GAP-29 (Domains 50–51) — Client SDK: Add AKP Support**
- **The problem:** Two independent SDK gaps:
  - `JWTClientCredentialsProvider.setupKeyPair()` has no `AKP` case — passing an ML-DSA key pair throws `RuntimeException("Invalid KeyPair algorithm")`
  - `DPoPGenerator.generateRsaSignedDPoPProof()` is hardcoded to RSA with a `TODO` noting missing EC/EdDSA equivalents; no ML-DSA path
- **Suggested parent:** Sub-issues under `#43692` or `#48821`

---

**25. GAP-11 — Dynamic Client Registration: RS256 Special-Case Logic**
- **Affected domain:** 35
- **The problem:** `DescriptionConverter.java` omits `id_token_signed_response_alg` from DCR responses when the realm default is RS256. No equivalent ML-DSA treatment exists. Clients registering via DCR in an ML-DSA-default realm will receive incomplete metadata.
- **Suggested parent:** Sub-issue under `#43692` or `#48824`

---

**26. Domain 52 — Docker Registry: Configurable Certificate Algorithm**
- **The problem:** `DockerComposeCertsDirectory.java` hardcodes `CryptoIntegration.getProvider().getKeyPairGen(KeyType.RSA)` with `keyGen.initialize(2048)`. Docker registry certificates will always be RSA-2048.
- **Note:** Low urgency — Docker compose is a developer/test installation path only.
- **Suggested parent:** New issue under `#43690`

---

**27. GAP-14 — JGroups ASYM_ENCRYPT: Operator Migration Documentation**
- **Affected domain:** 39
- **The problem:** The example `cache-ispn-asym-enc.xml` test configuration uses RSA-2048 key exchange. Operators who copy this configuration to production need explicit guidance to switch to `SSL_KEY_EXCHANGE`.
- **Note:** `#48823` (production readiness) may address this via documentation but has no explicit sub-issue.
- **Suggested parent:** Documentation sub-issue under `#48823`

---

#### ⚪ External Dependencies (Issues Needed for Tracking, Not Implementation)

---

**28. GAP-6 — FIPS 140-2 Backend: BC-FIPS 2.x with ML-DSA/ML-KEM**
- BC-FIPS 2.1.2 contains only LMS under PQC; no ML-DSA or ML-KEM classes. Awaiting BC-FIPS release with FIPS 140-3 validated PQC algorithms. Keycloak cannot unblock this. A tracking issue to monitor BC-FIPS releases and coordinate the version upgrade would be valuable.

**29. GAP-7 — OID4VC Linked Data Proof Suite: ML-DSA Cryptographic Suite**
- A new ML-DSA LD cryptographic suite would need to be implemented. Currently spec-level work. Requires a dedicated tracking issue once the W3C Data Integrity spec defines a PQC suite.

**30. GAP-25 — OID4VP Response Encryption: HAIP Spec Gate**
- HAIP mandates ECDH-ES/P-256 for `direct_post.jwt`. Blocked on HAIP spec evolution. A tracking issue to monitor the HAIP working group would be valuable.

---

## Summary Statistics

| Category | Count |
|----------|-------|
| Total domains in `pqc_overview.html` | 63 |
| Total gaps in `pqc_overview.html` | 30 |
| Domains with direct GitHub issue coverage | ~18 |
| Domains that are SPI-driven / auto-inherit ML-DSA | ~22 |
| Domains that are external dependencies | ~5 |
| **Domains / gaps requiring new GitHub issues** | **30** |
| — CRITICAL | 3 |
| — HIGH | 6 |
| — MEDIUM | 13 |
| — LOW | 4 |
| — External tracking issues | 3 |
| Errors identified in existing markdown coverage claims | 8 |

---

## Recommended New Issue Creation Order

1. **GAP-1** — JWK thumbprint for AKP keys (blocks DPoP and attestation auth at runtime)
2. **GAP-4** — ML-KEM `CekManagementProvider` (blocks all JWE encryption, OID4VCI)
3. **GAP-9** — SAML hardcoded RS256 key selection (SAML will silently ignore ML-DSA keys)
4. **GAP-26** — JWKS endpoint AKP branch (ML-DSA keys invisible to all remote clients)
5. **GAP-30** — Client signature verifier RSA guard (hard runtime failure for ML-DSA client auth)
6. **Domain 46** — Realm bootstrap ML-DSA/ML-KEM providers (new realms never get PQC keys)
7. **GAP-24** — OID4VP hardcoded ES256 (active rejection of ML-DSA VP tokens)
8. **GAP-8** — FAPI allowlist extension (FAPI realms actively block ML-DSA)
9. **Domains 29, 32** — ML-KEM key generation provider (prerequisite for GAP-4 to function)
10. **GAP-2** — Confirm/create sub-issue for `SignatureAlgorithm.java` XML URI registration
11. Remaining MEDIUM priority gaps in order of dependency
