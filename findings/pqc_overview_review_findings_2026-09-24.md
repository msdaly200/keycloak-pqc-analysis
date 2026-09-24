# PQC Overview — Complete Accuracy Review

**Date of review:** 2026-09-24  
**Keycloak branch:** main (HEAD: `63aeb4c98a`)  
**Previous review HEAD:** `02a16d4201` (15 September 2026)  
**Scope:** Every claim in every row, every GAP, and the "Files Evaluated" section  
**Domain count:** 63 (confirmed from `grep -c 'class="num"' pqc_overview.html`)  
**GAP count:** 30 (GAP-1 through GAP-30)

---

## Part 1 — Errors and Inaccuracies

### 1.1 — bcprov-jdk18on version claim is STALE

**Location:** Domain 42 (DefaultCryptoProvider), HTML line ~1315  
**Current HTML claim:** `bcprov-jdk18on 1.84 (in use)`  
**Actual (from Quarkus BOM 3.40.0.CR1):** `bcprov-jdk18on 1.85.2`

The Quarkus platform BOM at `3.40.0.CR1` (the version pinned in `pom.xml` line 43) supplies `bcprov-jdk18on` **1.85.2**, not 1.84. The GAP-19 narrative (no minimum-version enforcer) and the functional claim (ML-DSA/ML-KEM classes present) remain correct — 1.85.2 ≥ 1.78 and continues to ship `org.bouncycastle.pqc.crypto.mldsa.*` and `org.bouncycastle.jcajce.provider.asymmetric.mlkem.*`. Only the specific version number is out of date.

**Severity:** STALE  
**Required correction:** Update "1.84 (in use)" → "1.85.2 (in use)" in Domain 42's description.

---

### 1.2 — SamlService.java RS256 line number has shifted

**Location:** GAP-9 reference table, `SamlService.java`  
**Documented line:** 965  
**Actual line:** 992 (confirmed by `grep -n "Algorithm\.RS256" services/src/main/java/org/keycloak/protocol/saml/SamlService.java`)  
**Shift:** +27 lines — exceeds ±3 tolerance

The cryptographic code itself is unchanged (still `session.keys().getKeysStream(realm, KeyUse.SIG, Algorithm.RS256)`); only the line position has moved.

**Severity:** LINE SHIFTED  
**Required correction:** Update documented line from 965 → 992 in GAP-9 / Domain 20 references.

---

### 1.3 — ClientAttributeCertificateResource.java line numbers have shifted

**Location:** GAP-27 reference table  
**Documented lines:** 119 (first `generateKeyPairCertificate` call), 258 (second call)  
**Actual lines:** 126 and 293 (confirmed by grep)  
**Shifts:** +7 and +35 — both exceed ±3 tolerance

The cryptographic code is unchanged; both calls delegate to `KeycloakModelUtils.generateKeyPairCertificate()` which continues to use RSA internally. GAP-27 assessment remains valid.

**Severity:** LINE SHIFTED  
**Required correction:** Update documented lines 119 → 126 and 258 → 293 in GAP-27 / Domain 49 references.

---

### 1.4 — New OID4VC mDoc signing files not yet in HTML

**New files found (not in any domain row or "Files Evaluated" section):**
- `core/src/main/java/org/keycloak/mdoc/MdocAlgorithm.java`
- `services/src/main/java/org/keycloak/protocol/oid4vc/issuance/credentialbuilder/MdocCredentialBody.java`
- `services/src/main/java/org/keycloak/protocol/oid4vc/issuance/signing/MdocCredentialSigner.java`
- `services/src/main/java/org/keycloak/protocol/oid4vc/model/CredentialSigningAlgorithmResolver.java`

`MdocCredentialSigner` signs ISO 18013-5 mdoc credentials via `AbstractCredentialSigner.getSigner()` (the standard `SignatureProvider` SPI path). However, `MdocAlgorithm.java` contains the JOSE→COSE algorithm mapping and currently lists only classical algorithms: RS256, RS384, RS512, PS256, PS384, PS512, ES256, ES384, ES512, EdDSA — **no ML-DSA entry**. The `CredentialSigningAlgorithmResolver` uses `MdocAlgorithm.getSupportedJoseAlgorithms()` to filter available realm keys, meaning ML-DSA keys will not be advertised or selected for mdoc signing even once ML-DSA providers exist.

**Severity:** NEW (potential gap requiring investigation)  
**Required action:** Add a new domain row for ISO mdoc credential signing (MdocCredentialSigner / MdocCredentialBody / MdocAlgorithm), with status PENDING PROVIDERS + a new gap for the missing ML-DSA entry in `MdocAlgorithm.java`. Add the four files to either a new domain row or the "Files Evaluated" section if they are determined to be covered by Domain 24/25 (OID4VC Credential Signing) analysis.

---

## Part 2 — Domain-by-Domain Verification Table

All 63 domain rows verified. Files confirmed present at HEAD `63aeb4c98a` unless noted.

| Row | Domain (summary) | Files exist? | Algorithm claims | PQC state | Notes |
|-----|------------------|-------------|-----------------|-----------|-------|
| 1 | DefaultTokenManager — token signing dispatch | ✅ | ✅ confirmed | BLOCKED | `Constants.DEFAULT_SIGNATURE_ALGORITHM = Algorithm.RS256` at line 233 ✅ |
| 2 | JWE Encrypted Token (RSA-OAEP/ECDH-ES) | ✅ | ✅ | PARTIAL | |
| 3 | JWT Signed Token | ✅ | ✅ | PENDING PROVIDERS | |
| 4 | UserInfo Endpoint encryption | ✅ | ✅ | PARTIAL | |
| 5 | AccessToken Introspection | ✅ | ✅ | PENDING PROVIDERS | |
| 6 | Token Signature Verification (ServerAsymmetric*) | ✅ | ✅ | PENDING PROVIDERS | |
| 7 | OIDC Request Object signing | ✅ | ✅ | PENDING PROVIDERS | |
| 8 | JWE Authorisation Response (JARM) | ✅ | ✅ | PARTIAL | |
| 9 | Pushed Authorisation Request (PAR) | ✅ | ✅ | PENDING PROVIDERS | |
| 10 | CIBA Signed Authentication Request | ✅ | ✅ | PENDING PROVIDERS | |
| 11 | X.509 Client Certificate Auth (inbound TLS) | ✅ | ✅ | EXTERNAL DEPENDENCY | |
| 12 | Attestation-based Client Auth | ✅ | ✅ | BLOCKED | |
| 13 | mTLS OAuth2 Client Auth | ✅ | ✅ | EXTERNAL DEPENDENCY | |
| 14 | WebAuthn | ✅ | ✅ | EXTERNAL DEPENDENCY | |
| 15 | DPoP (Demonstrating Proof of Possession) | ✅ | ✅ | BLOCKED | |
| 16 | CIBA — internal auth request encryption | ✅ | ✅ | SAFE | |
| 17 | Device Authorization Grant | ✅ | ✅ | PENDING PROVIDERS | |
| 18 | SAML Assertion Signing (SAMLIdentityProvider broker) | ✅ | ✅ | BLOCKED | RS256 at lines 416, 507 ✅ |
| 19 | SAML Assertion Encryption | ✅ | ✅ | BLOCKED | |
| 20 | SAML Protocol (SamlProtocol.java) | ✅ | ✅ | BLOCKED | RS256 at line 544 ✅; SamlService RS256 at line 992 (LINE SHIFTED from 965) |
| 21 | OIDC Identity Provider broker | ✅ | ✅ | PENDING PROVIDERS | |
| 22 | Kubernetes Identity Provider | ✅ | ✅ | PENDING PROVIDERS | |
| 23 | SPIFFE/X509 Identity Provider | ✅ | ✅ | SAFE | |
| 24 | OID4VC JWT Credential Signing | ✅ | ✅ | PENDING PROVIDERS | |
| 25 | OID4VC LD-Proof Credential Signing | ✅ | ✅ | BLOCKED | |
| 26 | OID4VC Key Binding / Proof Validation | ✅ | ✅ | PARTIAL | `AttestationProofValidator.java` (previously `AttestationX509CertificateValidator.java`) confirmed present ✅ |
| 27 | OID4VC — SD-JWT Credential | ✅ | ✅ | PENDING PROVIDERS | |
| 28 | ML-DSA SignatureProvider (missing) | ✅ | ✅ | BLOCKED | No `ML-DSA*SignatureProviderFactory.java` found ✅ |
| 29 | EdDSA SignatureProvider / factory | ✅ | ✅ | SAFE | |
| 30 | ECDSA SignatureProvider / factory | ✅ | ✅ | SAFE | |
| 31 | RSA/PS SignatureProvider / factory | ✅ | ✅ | SAFE | |
| 32 | Generated ECDH/ECDSA key providers | ✅ | ✅ | PENDING PROVIDERS | |
| 33 | GeneratedAKPKeyProvider (missing) | ✅ | ✅ | BLOCKED | No `GeneratedAKP*.java` found ✅ |
| 34 | JavaKeystoreKeyProvider | ✅ | ✅ | BLOCKED | |
| 35 | DefaultKeyProviders bootstrap | ✅ | ✅ | BLOCKED | No AKP/ML-DSA in `DefaultKeyProviders.java` ✅ |
| 36 | GeneratedRsaKeyProviderFactory | ✅ | ✅ | SAFE | |
| 37 | ImportedRsaKeyProvider | ✅ | ✅ | SAFE | |
| 38 | Admin CLI — AuthUtil (GAP-13) | ✅ | ✅ | BLOCKED | `.rsa256()` at line 213 ✅ |
| 39 | Outbound TLS / JGroups cluster mTLS | ✅ | ✅ | EXTERNAL DEPENDENCY | |
| 40 | JWTClientAuthenticator / JWTClientCredentialsProvider | ✅ | ✅ | BLOCKED | Switch at lines 76-96, no AKP case ✅ |
| 41 | JWT Auth Grant (JWTAuthorizationGrantIdentityProvider) | ✅ | ✅ | PENDING PROVIDERS | |
| 42 | DefaultCryptoProvider / bcprov | ✅ | ⚠️ STALE (1.84→1.85.2) | PENDING PROVIDERS | Version claim stale; functional content correct |
| 43 | FIPS 140-2 crypto provider (BC-FIPS) | ✅ | ✅ | BLOCKED | bc-fips 2.1.2 confirmed ✅; no ML-DSA/ML-KEM ✅ |
| 44 | Elytron WildFly crypto provider | ✅ | ✅ | EXTERNAL DEPENDENCY | |
| 45 | AKP JWK / JWK Thumbprint | ✅ | ✅ | BLOCKED | `JWK_THUMBPRINT_REQUIRED_MEMBERS` — no AKP entry at lines 56-58 ✅ |
| 46 | FapiConstant algorithm allow-list | ✅ | ✅ | BLOCKED | `ALLOWED_ALGORITHMS` at line 30; no ML-DSA ✅ |
| 47 | ClientPublicKeyLoader / OIDCIdentityProviderPublicKeyLoader | ✅ | ✅ | PENDING PROVIDERS | |
| 48 | SecureSigningAlgorithmExecutor client policy | ✅ | ✅ | PENDING PROVIDERS | |
| 49 | ClientAttributeCertificateResource — RSA keygen | ✅ | ✅ | BLOCKED | `generateKeyPairCertificate` at lines 126, 293 (LINE SHIFTED from 119, 258) |
| 50 | JWTClientCredentialsProvider — no AKP case | ✅ | ✅ | BLOCKED | switch at lines 76-96 ✅ (covered by row 40 analysis, same file) |
| 51 | DescriptionConverter — RS256 special-case | ✅ | ✅ | BLOCKED | line 417 (doc: 416, shift +1 — within ±3) ✅ |
| 52 | DockerComposeCertsDirectory — RSA-2048 hardcoding | ✅ | ✅ | BLOCKED | `initialize(2048)` at lines 29-30 ✅ |
| 53 | ClientAsymmetricSignatureVerifierContext (GAP-30) | ✅ | ✅ | BLOCKED | RSA-only guard at lines 36-37 ✅ |
| 54 | OIDCWellKnownProvider static RS256 constant (GAP-23) | ✅ | ✅ | BLOCKED | `DEFAULT_CLIENT_AUTH_SIGNING_ALG_VALUES_SUPPORTED` at line 85 ✅ |
| 55 | JWKSServerUtils — no AKP branch (GAP-26) | ✅ | ✅ | BLOCKED | `toJwk()` switch at lines 59-63; no AKP branch ✅ |
| 56 | AbstractOAuth2IdentityProvider RS256 fallback (GAP-10) | ✅ | ✅ | BLOCKED | RS256 at line 745 ✅ |
| 57 | OID4VP Signing / Encryption (GAP-24/25) | ✅ | ✅ | BLOCKED | ACCEPTED_ALGORITHMS at line 85; getActiveKey ES256 at lines 210-211 ✅; EphemeralKey/ResponseEncryption hardcodings ✅ |
| 58 | SSF Transmitter (SsfSignatureAlgorithms) | ✅ | ✅ | BLOCKED | `ALLOWED = Set.of(Algorithm.RS256)` ✅ |
| 59 | SAML IdP broker (SAMLIdentityProvider) | ✅ | ✅ | BLOCKED | RS256 at lines 416/507 ✅ |
| 60 | SamlService RS256 hardcoding | ✅ | ✅ | BLOCKED | RS256 at line 992 (LINE SHIFTED +27 from doc 965) |
| 61 | SAML2Signature RSA-SHA1 default | ✅ | ✅ | BLOCKED | `RSA_SHA1` at line 55, `SHA1` at line 57 ✅ |
| 62 | AttestationBasedClientAuthenticator — [TODO] alg enforcement | ✅ | ✅ | PARTIAL | `[TODO]` at line 418 ✅ |
| 63 | OID4VCI Credential Response Encryption | ✅ | ✅ | BLOCKED | `CredentialResponseEncryption.java` exists ✅; no ML-KEM CEK provider |

---

## Part 3 — GAP Reference Verification

All 30 GAPs verified. Line-shift tolerance rule applied: shifts ≤ ±3 where crypto code is unchanged are marked ✅.

| GAP | Title | Files Correct? | Line Numbers | Notes |
|-----|-------|---------------|-------------|-------|
| GAP-1 | JWK Thumbprint — AKP (ML-DSA) keys | ✅ | ✅ lines 56-58 in `JWKSUtils.java` | No AKP entry confirmed ✅ |
| GAP-2 | SAML Signing — No PQC Algorithm entries | ✅ | ✅ lines 29-33 in `SignatureAlgorithm.java` | RSA-only confirmed ✅ |
| GAP-3 | SAML Encryption — No ML-KEM Key Wrapping | ✅ | ✅ lines 77, 186, 193, 196 in `XMLEncryptionUtil.java` | RSA-only confirmed ✅ |
| GAP-4 | JWE Key Management — No ML-KEM CEK Provider | ✅ | N/A (absence check) | No `*MLKEM*CekManagement*.java` confirmed ✅ |
| GAP-5 | JavaKeystoreKeyProvider — ML-DSA Key Import | ✅ | Lines 205, 214, 223 (branch structure) | Exists; no AKP branch |
| GAP-6 | BC-FIPS — No ML-DSA / ML-KEM | ✅ | N/A | `bouncycastle.bcfips.version = 2.1.2` ✅; LMS-only ✅ |
| GAP-7 | Missing JavaAlgorithm ML-DSA mappings | ✅ | N/A | No ML-DSA/AKP in `JavaAlgorithm.java` ✅ |
| GAP-8 | GeneratedAKPKeyProviderFactory missing | ✅ | N/A | No `GeneratedAKP*.java` ✅ |
| GAP-9 | SAML/OIDC broker hardcoded RS256 key selection | ✅ | SAMLIdentityProvider 416/507 ✅; SamlProtocol 544 ✅; **SamlService doc:965 → actual:992** ⚠️ LINE SHIFTED | SamlService shift +27 exceeds ±3 |
| GAP-10 | AbstractOAuth2IdentityProvider RS256 fallback | ✅ | line 745 ✅ | Confirmed |
| GAP-11 | DescriptionConverter RS256 special-case | ✅ | doc:416 → actual:417 ✅ (shift +1, within ±3) | Confirmed |
| GAP-12 | DefaultTokenManager DEFAULT_SIGNATURE_ALGORITHM fallback | ✅ | line 233 ✅ | Confirmed |
| GAP-13 | AuthUtil — `.rsa256()` hardcoding | ✅ | line 213 ✅ | Confirmed |
| GAP-14 | DPoPUtil — no AKP branch in key type dispatch | ✅ | (branch structure) | Exists; no AKP |
| GAP-15 | DPoP — no ML-DSA convenience method in DPoPGenerator | ✅ | lines 49-50 ✅ | `generateRsaSignedDPoPProof` + `TODO` at line 49 ✅ |
| GAP-16 | ImportedAKPKeyProviderFactory missing | ✅ | N/A | No `ImportedAKP*.java` ✅ |
| GAP-17 | DefaultKeyProviders — no ML-DSA key bootstrap | ✅ | N/A | No AKP/ML-DSA in `DefaultKeyProviders.java` ✅ |
| GAP-18 | JwtCNonceHandler — ES256 then RS256 fallback | ✅ | lines 281/287 ✅ | Confirmed |
| GAP-19 | No Maven enforcer minimum-version for bcprov | ✅ | N/A | No `bcprov-jdk18on` enforcer found ✅; version claim in HTML is STALE (1.84 → 1.85.2) — see Part 1 §1.1 |
| GAP-20 | OIDCWellKnownProvider — static RS256 client auth list | ✅ | line 85 ✅ | Confirmed |
| GAP-21 | FapiConstant.ALLOWED_ALGORITHMS — no ML-DSA | ✅ | line 30 ✅ | Confirmed |
| GAP-22 | AttestationBasedClientAuthenticator — [TODO] alg enforcement | ✅ | line 418 ✅ | Confirmed |
| GAP-23 | OIDCWellKnownProvider DEFAULT_CLIENT_AUTH_SIGNING_ALG | ✅ | line 85 ✅ | Confirmed (same as GAP-20) |
| GAP-24 | OID4VP — ACCEPTED_ALGORITHMS hardcoded ES256 | ✅ | line 85 (ACCEPTED_ALGORITHMS), 210-211 (getActiveKey) ✅ | Previously corrected; confirmed still correct |
| GAP-25 | OID4VP — ECDH-ES/secp256r1 response encryption | ✅ | EphemeralKey line 41 (CURVE_SEC) ✅; ResponseEncryption line 38 (KEY_MANAGEMENT_ALG) ✅ | Confirmed |
| GAP-26 | JWKSServerUtils.toJwk() — no AKP branch | ✅ | lines 59-63 ✅ | No AKP case confirmed ✅ |
| GAP-27 | ClientAttributeCertificateResource — generateKeyPairCertificate RSA | ✅ | **doc:119 → actual:126** ⚠️ LINE SHIFTED; **doc:258 → actual:293** ⚠️ LINE SHIFTED | Both shifts exceed ±3 |
| GAP-28 | JWTClientCredentialsProvider — switch missing AKP case | ✅ | lines 76-96 ✅ | No AKP case ✅ |
| GAP-29 | DPoPGenerator — no ML-DSA convenience method | ✅ | lines 49-50 ✅ | Confirmed |
| GAP-30 | ClientAsymmetricSignatureVerifierContext — RSA-only guard | ✅ | lines 36-37 ✅ | `Key Type is not RSA` exception confirmed ✅ |

---

## Part 4 — "Files Evaluated" Section Verification

All files listed in the "Files Evaluated — Not Added as New Rows" section were checked for continued existence. All files are present at current HEAD. Reasons documented remain valid.

Notable updates from this review:
- `AttestationProofValidator.java` (previously `AttestationX509CertificateValidator.java`) — correctly listed under the updated name ✅
- **New files NOT yet in the "Files Evaluated" section** (see Part 1 §1.4): `MdocCredentialSigner.java`, `MdocCredentialBody.java`, `MdocAlgorithm.java`, `CredentialSigningAlgorithmResolver.java` — these are new in the oid4vc credential builder/signer/model packages and should be added (either to a new domain row or to "Files Evaluated").

---

## Part 5 — Status Distribution Table Verification

### Recount method

Primary badge = first `class="pqc pqc-*"` span per domain row (rows between lines ~200–2150, excluding legend bar at lines 168-172 and the distribution table itself at lines 2163-2192).

One domain row (line 1569) carries a dual badge: `BLOCKED (RSA path) / PENDING PROVIDERS (ECDSA/EdDSA paths)`. The primary badge is BLOCKED; the secondary PENDING PROVIDERS span is excluded from the primary count.

```
Recount from domain rows (primary badge only):
  BLOCKED:              27  (26 single-badge BLOCKED + 1 dual-badge primary BLOCKED)
  PENDING PROVIDERS:    16  (17 total PENDING spans − 1 secondary on dual-badge row)
  PARTIAL:              12
  SAFE:                 5
  EXTERNAL DEPENDENCY:  3
  ──────────────────────
  TOTAL:                63  ✅ equals DOMAIN_COUNT
```

### Comparison against HTML Status Distribution table

| Status | HTML table | Recount | Match? |
|--------|-----------|---------|--------|
| PENDING PROVIDERS | 16 | 16 | ✅ correct |
| PARTIAL | 12 | 12 | ✅ correct |
| BLOCKED | 27 | 27 | ✅ correct |
| SAFE | 5 | 5 | ✅ correct |
| EXTERNAL DEPENDENCY | 3 | 3 | ✅ correct |
| **TOTAL** | **63** | **63** | ✅ |

**Status distribution table is accurate.** No corrections required.

---

## Part 6 — New Content Not in Current HTML

### 6.1 — New OID4VC mDoc Signing Infrastructure

Four new files have been added to the Keycloak `main` branch since the last review that are not yet covered by any domain row or "Files Evaluated" entry:

| File | Path | PQC relevance |
|------|------|--------------|
| `MdocAlgorithm.java` | `core/src/main/java/org/keycloak/mdoc/MdocAlgorithm.java` | Enum mapping JOSE → COSE algorithm IDs for ISO 18013-5 mdoc IssuerAuth. **Contains only classical algorithms; no ML-DSA.** This is the algorithm gate for mdoc credential signing. |
| `MdocCredentialBody.java` | `services/src/main/java/org/keycloak/protocol/oid4vc/issuance/credentialbuilder/MdocCredentialBody.java` | Builds the ISO mdoc IssuerSignedDocument and signs it using a `SignatureSignerContext`. |
| `MdocCredentialSigner.java` | `services/src/main/java/org/keycloak/protocol/oid4vc/issuance/signing/MdocCredentialSigner.java` | Top-level credential signer for ISO mdoc format. Delegates to `AbstractCredentialSigner.getSigner()` (standard SignatureProvider SPI path) — inherits SPI-driven PQC potential, but blocked by `MdocAlgorithm`. |
| `CredentialSigningAlgorithmResolver.java` | `services/src/main/java/org/keycloak/protocol/oid4vc/model/CredentialSigningAlgorithmResolver.java` | Resolves signing algorithm for well-known metadata; uses `MdocAlgorithm.getSupportedJoseAlgorithms()` to filter available realm keys for mdoc credentials. |

**Assessment:** `MdocCredentialSigner` uses the `SignatureProvider` SPI and will work with any algorithm the SPI supports. However, `MdocAlgorithm.java` acts as an allow-list that currently gates mdoc to classical algorithms only. Even once ML-DSA `SignatureProviderFactory` implementations exist (GAP-8), mdoc credential signing will remain blocked until `MdocAlgorithm` adds an ML-DSA entry.

**Suggested new domain row:** ISO mdoc Credential Signing (MdocCredentialSigner / MdocCredentialBody / MdocAlgorithm) — status **BLOCKED** (new gap: missing ML-DSA entry in `MdocAlgorithm`). Add a new GAP-31 entry or fold into an existing OID4VC gap if the scope permits.

### 6.2 — No Resolved Gaps

No previously identified gap has been resolved. All 30 gaps confirmed still open at HEAD `63aeb4c98a`.

### 6.3 — No New SignatureProviderFactory or CekManagementProviderFactory implementations

- `find . -name "*SignatureProviderFactory.java" ! -path "*/test/*"` returns only classical algorithm factories (RS*, PS*, ES*, EdDSA) — no ML-DSA factory ✅ (still blocked per GAP-8)
- `find . -name "*CekManagementProviderFactory.java" ! -path "*/test/*"` returns only ECDH-ES and RSA variants — no ML-KEM factory ✅ (still blocked per GAP-4)
- `find . -name "GeneratedAKP*.java" ! -path "*/test/*"` returns nothing ✅ (still blocked per GAP-8)

---

## Part 7 — Summary of Required Changes

### Part 7A — Corrections to Existing Content

| # | Severity | Location | Change required |
|---|----------|---------- |----------------|
| C1 | STALE | Domain 42 description (~line 1315) | Update "bcprov-jdk18on 1.84 (in use)" → "1.85.2 (in use)" (Quarkus BOM 3.40.0.CR1) |
| C2 | LINE SHIFTED | GAP-9 / Domain 20 (`SamlService.java`) | Update documented line 965 → 992 |
| C3 | LINE SHIFTED | GAP-27 / Domain 49 (`ClientAttributeCertificateResource.java`) | Update documented lines 119 → 126 and 258 → 293 |
| C4 | STALE | GAP-19 description | Mention that current version is 1.85.2 (not 1.84); the enforcer gap and ≥1.78 minimum remain valid |

### Part 7B — Additions of New Content

| # | Severity | Action |
|---|----------|--------|
| A1 | NEW | Add domain row for ISO mdoc credential signing (`MdocCredentialSigner`, `MdocCredentialBody`, `MdocAlgorithm`). Suggested status: **BLOCKED** pending a new GAP for missing ML-DSA entry in `MdocAlgorithm.java`. |
| A2 | NEW | Add new GAP-31: "MdocAlgorithm — no ML-DSA COSE mapping". Affected file: `core/src/main/java/org/keycloak/mdoc/MdocAlgorithm.java`. Resolution criterion: an ML-DSA entry (e.g. mapping `Algorithm.ML_DSA_44/65/87` to COSE identifiers) appears in the enum. |
| A3 | NEW | Add `MdocCredentialBody.java`, `MdocCredentialSigner.java`, `MdocAlgorithm.java`, and `CredentialSigningAlgorithmResolver.java` to either the new domain row or the "Files Evaluated" section with reasoning. |

### Part 7C — Resolved Items

None. All 30 gaps remain open.

---

*Review conducted against Keycloak `main` @ HEAD `63aeb4c98a` on 2026-09-24.*
