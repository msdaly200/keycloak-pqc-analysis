# PQC Overview — Complete Accuracy Review

**Date of review:** 2026-10-29  
**Keycloak branch:** main (HEAD: `f82c6402ba`, committed 2026-10-29; identical to `origin/main` in the local clone — no `git fetch` was run). The clone is currently checked out on local branch `53168` at that same commit, with three uncommitted non-crypto edits (`AuthenticatorUtil`, `OIDCLoginProtocol`, `LoginActionsService` — ForceAuthn/ACR work) that were reviewed and have no PQC relevance.  
**Previous review HEAD:** `eb3ac9a4ba` (2026-09-27) — 104 commits behind current HEAD  
**Scope:** Every claim in every row, every GAP, and the "Files Evaluated" section  
**DOMAIN_COUNT:** 63 (`grep -c 'class="num"' pqc_overview.html`)  
**GAP_LIST:** GAP-1 … GAP-31 (31 GAPs)  
**FILE_LIST:** 316 distinct `*.java` tokens in the HTML (up from 307 at the previous review)

**Method note / evidence levels.** Pre-flight, Section A, B1, B2, B3, B4 and the Part 5 recount were all executed against the live source. For Part 2, *file existence* was checked programmatically for every `.java` token in every one of the 63 rows (wildcards expanded). *Algorithm / PQC-state claims* were re-verified through the B2/B3 greps; rows whose prose is purely architectural (SPI-driven, "will inherit ML-DSA") were not re-derived line by line and are marked "files ✅, no contradicting change at HEAD". The `git diff eb3ac9a4ba..HEAD` file list was read for anything touching crypto, JWK/JOSE, key providers, SAML, OID4VC/OID4VP, SD-JWT or mdoc. Library JARs in `~/.m2` were inspected directly for the `bc-fips` and `bcprov` claims.

**Headline result.** No gap was resolved, no status badge changes, and the Status Distribution table (27 / 16 / 12 / 5 / 3 = 63) is correct. Required edits are documentation-only: the Quarkus BOM label moved to 3.40.1.1 (bcprov still 1.86), `bc-fips 2.1.2` is still cited in seven files the 17-Oct pass did not cover, and two line-number references drifted (GAP-22 `[TODO]` 418 → 423–424; GAP-27 second call 293 → 288).

---

## Part 1 — Errors and Inaccuracies

### 1.1 — Quarkus pin is STALE (3.40.1 → 3.40.1.1); `bcprov-jdk18on` 1.86 claim still holds

**Severity:** STALE  
**Evidence:** `pom.xml` lines 43/44 are now `<quarkus.version>3.40.1.1</quarkus.version>` / `<quarkus.build.version>3.40.1.1</quarkus.build.version>` (commit `b1920d7f82`, "Upgrade to Quarkus 3.40.1.1 (#54187)", 2026-10-21). `~/.m2/.../quarkus-bom/3.40.1.1/*.pom` line 9514 still declares `bcprov-jdk18on` **1.86**, and `bcprov-jdk18on-1.86.jar` contains 121 `mldsa` entries and 29 `provider/asymmetric/mlkem` entries. So the functional claim (ML-DSA/ML-KEM present in the default backend) and the version number 1.86 are unchanged — only the BOM label moved.  
**Stale text locations (all say "Quarkus BOM 3.40.1"):**
- `gaps/Github_issue_requirements_for_gaps.md` line 368 (GAP-19)
- `domains/Domain_42_CryptoProvider_BouncyCastle.md` line 13

`pqc_overview.html` line 1315 says only "`bcprov-jdk18on` 1.86 (in use)" and needs no change. The "Last updated" sentence at `pqc_overview.html` line 2347 and `README.md` line 48 describe the 17-Oct edit pass historically and should simply be replaced by the new date line.

### 1.2 — `bc-fips` 2.1.2 still cited in documents outside the 17-Oct edit pass (pre-existing STALE)

**Severity:** STALE (documentation only; GAP-6 conclusion unchanged)  
**Evidence:** `pom.xml` line 78 is still `<bouncycastle.bcfips.version>2.1.3</bouncycastle.bcfips.version>`. `bc-fips-2.1.3.jar` has **0** entries matching `mldsa|mlkem|ml_dsa|ml_kem`; `org/bouncycastle/crypto/internal/pqc` contains only `lms/`. GAP-6 stays open.  
**Locations still saying 2.1.2:**
- `core-authn/pqc-cross-reference-analysis.md` line 467
- `core-authn/pqc-issues-gaps-core-authn-ownership-analysis.html` line 663 (title cell) and the matching prose
- `core-authn/core-authn-issue-list.md` lines 234, 240
- `gh_issues/Github_PQC_issues_overview.md` line 158 (GAP-6 row)
- `approach/pqc_configuration_strategy.md` line 123
- `.bob/artifacts/pqc-issues-gaps-core-authn-ownership-analysis.html` line 663 (generated copy)

`pqc_overview.html`, `gaps/Github_issue_requirements_for_gaps.md` GAP-6 and the `domains/Domain_43/11/13/2` files were already corrected on 17-Oct.

### 1.3 — GAP-22 / Domain 12 line numbers: `[TODO]` algorithm enforcement moved 418 → 423

**Severity:** LINE SHIFTED (crypto code unchanged, shift +5)  
**Evidence:** `AttestationBasedClientAuthenticator.java` now has
```
423:  // [TODO] The alg JOSE Header Parameter for both JWTs indicates a registered asymmetric digital signature algorithm
424:  // [TODO] The key contained in the cnf claim of the Client Attestation JWT is not a private key
```
Commit `32d89586d9` (2026-10-19, "Mask secrets in client installations…") inserted `getAdapterConfigurationSecretKeys()` above this block. Enforcement is still unimplemented, so GAP-22 is unchanged.  
**Locations documented as 418 / 418–419:**
- `pqc_overview.html` lines 448, 451 (row 12)
- `gaps/Github_issue_requirements_for_gaps.md` GAP-22: lines 420, 425, 429
- `domains/Domain_12_Attestation_Based_Client_Authentication.md` lines 20, 22, 81, 182, 186
- `core-authn/core-authn-issue-list.md` lines 108, 127, 141
- `core-authn/pqc-cross-reference-analysis.md` line 367
- `prompts/REASSESS_PQC_READINESS_PROMPT.md` line 239 comment

Suggested wording: "line 418" → "lines 423–424" (or drop the number).

### 1.4 — GAP-27 / Domain 49 line numbers: second `generateKeyPairCertificate` call moved 293 → 288

**Severity:** LINE SHIFTED (crypto code unchanged, shift −5)  
**Evidence:** `ClientAttributeCertificateResource.java` still calls `KeycloakModelUtils.generateKeyPairCertificate(client.getClientId())` at **126** (✅) and `…generateKeyPairCertificate(client.getClientId(), keySize, calendar)` now at **288**. `KeycloakModelUtils.generateKeyPairCertificate(String,int,Calendar)` (line 246) still calls `KeyUtils.generateRsaKeyPair(keysize)` (line 247). Still RSA-only.  
**Locations documented as 293:** `gaps/Github_issue_requirements_for_gaps.md` lines 515, 520; `domains/Domain_49_Admin_API_Cert_Generation.md` lines 13, 21. (`pqc_overview.html` row 49 cites no line numbers.)

### 1.5 — Items inside tolerance (✅, but at the edge — flag for the next edit pass)

| Location | Documented | Actual | Shift |
|----------|-----------|--------|-------|
| `AbstractOAuth2IdentityProvider.java` `Algorithm.HS256` fallback (GAP-10) | 969 | **972** | +3 ✅ |
| `AbstractOAuth2IdentityProvider.java` `Algorithm.RS256` fallback (GAP-10) | 977 | **980** | +3 ✅ |
| `SamlService.java` `Algorithm.RS256` (GAP-9) | 1012 | **1013** | +1 ✅ |
| `DescriptionConverter.java` RS256 special-case (GAP-11) | 416 | 417 | +1 ✅ |
| `OID4VPIdentityProvider.java` TODO (row 62) | 81 | 83 | +2 ✅ |

`AbstractOAuth2IdentityProvider` is the next one to cross the ±3 threshold; consider citing it by method name when it is next edited.

### 1.6 — Items previously flagged and now confirmed corrected

- Row 60 SOAP back-channel class: HTML line 739 now reads `SAMLIdentityProvider.resolveArtifact()` ✅ (`Soap.createMessage()` call at `SAMLIdentityProvider.java` 575; `resolveArtifact` at 554).
- GAP-15 `CryptoProvider.java` module path in the gaps doc: now `common/src/main/java/…` ✅.
- GAP-6 `bc-fips` version text and GAP-19 `bcprov` version text in the HTML/gaps doc/domain files: ✅ at 2.1.3 / 1.86.
- GAP-9/12/20/21 line numbers in the gaps doc: ✅ (1012, 240, 144/150, 348/386–387).

---

## Part 2 — Domain-by-Domain Verification Table

Files = every `*.java` token in the row resolved against non-test Keycloak sources (`git ls-files`, wildcards expanded). All 63 rows: **files ✅** (the only two non-resolving tokens in the whole HTML are the glob `EcdhEsA*KwCekManagementProviderFactory.java`, which matches the three `EcdhEsA128/192/256Kw…` files, and the deliberately annotated former name `AttestationX509CertificateValidator.java`, now `AttestationProofValidator.java`). Status = primary (first) badge in the HTML.

| Row | Domain | Files exist? | Algorithm claims / state verified | PQC state | Notes |
|-----|--------|--------------|-----------------------------------|-----------|-------|
| 1 | Access Token / ID Token Signing | ✅ | ✅ `ML_DSA_44/65/87` + `KeyType.AKP` present; no ML-DSA factory; GAP-12 fallback `DefaultTokenManager` 240 | PARTIAL | CONFIRMED |
| 2 | ID Token / JARM Encryption (Outbound CEK) | ✅ | ✅ no ML-KEM CEK provider (7 CEK factories, all ECDH-ES/RSA) | BLOCKED | |
| 3 | Backchannel Logout Token Signing | ✅ | ✅ `case ID: case LOGOUT:` → `ID_TOKEN_SIGNED_RESPONSE_ALG` (`DefaultTokenManager` 214–216) | PENDING PROVIDERS | |
| 4 | UserInfo — Signed & Encrypted Response | ✅ | ✅ 348, 386–387 | BLOCKED | GAP-21 |
| 5 | Introspection — Embedded JWT Response | ✅ | ✅ | PARTIAL | |
| 6 | Token Verification (Identity & Session Tokens) | ✅ | ✅ | PENDING PROVIDERS | |
| 7 | JAR — Signed Request Object Verification | ✅ | ✅ | PENDING PROVIDERS | |
| 8 | JAR — Encrypted Request Object Decryption | ✅ | ✅ `decodeClientJWT` 122; `KeyUse.ENC` lookup 144/150; no ML-KEM path | BLOCKED | GAP-20 |
| 9 | JARM — Signed Authorization Response | ✅ | ✅ | PENDING PROVIDERS | |
| 54 | Federated JWT Client Authentication | ✅ | ✅ | PENDING PROVIDERS | |
| 10 | private_key_jwt Client Authentication | ✅ | ✅ | PENDING PROVIDERS | |
| 11 | X.509 mTLS Client Authentication | ✅ | ✅ | SAFE | No change since last review |
| 12 | Attestation-Based Client Authentication | ✅ | ✅ `[TODO]` alg enforcement still present — **now lines 423–424** | PARTIAL | **LINE SHIFTED §1.3**; `AttestationValidatorUtil` still enforces `FapiConstant.ALLOWED_ALGORITHMS` (PS*/ES*) at line 312 |
| 13 | X.509 Browser Authentication Flow | ✅ | ✅ | SAFE | |
| 14 | WebAuthn / Passkeys (FIDO2) | ✅ | ✅ | EXTERNAL DEPENDENCY | |
| 15 | DPoP | ✅ | ✅ | PARTIAL | |
| 16 | CIBA Signed Backchannel Auth Request | ✅ | ✅ factory uses `FapiConstant.ALLOWED_ALGORITHMS` (line 45) | PARTIAL | |
| 57 | Token Exchange — Verification & Signing | ✅ | ✅ | PENDING PROVIDERS | |
| 58 | Device Authorization Grant — Token Signing | ✅ | ✅ | PENDING PROVIDERS | |
| 17 | JWT Authorization Grant — Assertion Verification | ✅ | ✅ | PENDING PROVIDERS | |
| 18 | SAML Assertion & Document Signing | ✅ | ✅ RS256 at `SAMLIdentityProvider` 416/507, `SamlProtocol` 544, `SamlService` 1013 | BLOCKED | SAML OneTimeUse / Recipient / ForceAuthn commits touch `SAMLEndpoint`/`SamlService` — no algorithm change (Part 6) |
| 19 | SAML Assertion Encryption | ✅ | ✅ `XMLEncryptionUtil` RSA_OAEP_11 default (77), RSA_v1dot5 (431) | BLOCKED | `SAMLEncryptionAlgorithms` = RSA_OAEP, RSA1_5 only |
| 20 | SAML IdP Broker — SP Metadata & Federation Signing | ✅ | ✅ RS256 at 416/507 | BLOCKED | |
| 55 | Default Trust Identity Provider | ✅ | ✅ | PENDING PROVIDERS | |
| 59 | SAML Metadata Public Key Loader | ✅ | ✅ | SAFE | |
| 60 | SAML Artifact Resolution | ✅ | ✅ `DefaultSamlArtifactResolver.resolveArtifact()` line 32 storage/routing only; SOAP call in `SAMLIdentityProvider` 575 — HTML wording correct | BLOCKED | Previous ERROR now fixed |
| 61 | SAML2Signature — Hardcoded RSA-SHA1 Default | ✅ | ✅ `RSA_SHA1` line 55, `DigestMethod.SHA1` line 57 | BLOCKED | |
| 62 | OID4VP IdP — Request Signing & Response Encryption | ✅ | ✅ `ACCEPTED_ALGORITHMS` 85; ES256 at 210–211; `CURVE_SEC` 41; `KEY_MANAGEMENT_ALG` 38; TODO 83 | BLOCKED | GAP-24/25; all 12 oid4vp files + 4 mappers cited |
| 63 | OID4VCI Credential Response Encryption | ✅ | ✅ no ML-KEM CEK provider | BLOCKED | |
| 21 | OIDC IdP — Token Sig Verification & JWE Decryption | ✅ | ✅ JWE key lookup `OIDCIdentityProvider` 675/677 unchanged; outbound fallbacks 972/980 | PARTIAL | GAP-10 (+3, tolerated) |
| 22 | Kubernetes Identity Provider | ✅ | ✅ | PENDING PROVIDERS | |
| 23 | SPIFFE / SVID Identity Provider | ✅ | ✅ | PENDING PROVIDERS | |
| 56 | SD-JWT Issuer Signing & Key Binding Verification | ✅ | ✅ | PENDING PROVIDERS | |
| 24 | JWT-VC / SD-JWT Credential Signing | ✅ | ✅ | PENDING PROVIDERS | GAP-31 mdoc sibling |
| 25 | LD-Proof Credential Signing | ✅ | ✅ `Ed255192018Suite.PROOF_TYPE = "Ed25519Signature2018"` line 67 | BLOCKED | GAP-7 |
| 26 | OID4VC Key Binding — JWT Proof Validation | ✅ | ✅ `AttestationProofValidator` present | PENDING PROVIDERS | |
| 27 | OID4VC c_nonce JWT Signing | ✅ | ✅ ES256 281, RS256 287 | BLOCKED | GAP-18 |
| 28 | Generated RSA Signing Key Provider | ✅ | ✅ no `GeneratedAKP*` | BLOCKED | |
| 29 | Generated RSA Encryption Key Provider | ✅ | ✅ | BLOCKED | |
| 30 | Generated ECDSA Signing Key Provider | ✅ | ✅ | BLOCKED | |
| 31 | Generated EdDSA Signing Key Provider | ✅ | ✅ | BLOCKED | |
| 32 | Generated ECDH Encryption Key Provider | ✅ | ✅ | BLOCKED | |
| 33 | Imported RSA Signing Key Provider | ✅ | ✅ no `ImportedAKP*` | BLOCKED | |
| 34 | Java Keystore Key Provider | ✅ | ✅ branches OKP(205)/EC(214)/RSA(223); no AKP | BLOCKED | GAP-5 |
| 35 | Dynamic Client Registration Tokens | ✅ | ✅ `DescriptionConverter` 417; `INTERNAL_SIGNATURE_ALGORITHM = HS512` (`Constants` 69) | SAFE | |
| 36 | OIDC Well-Known — Algorithm Advertisement | ✅ | ✅ `DEFAULT_CLIENT_AUTH_SIGNING_ALG_VALUES_SUPPORTED = list(RS256)` line 85 | PARTIAL | GAP-23 |
| 37 | Security Event Token (SET) Signing | ✅ | ✅ `SsfSignatureAlgorithms.ALLOWED = Set.of(RS256)` line 31, `DEFAULT = RS256` line 38 | EXTERNAL DEPENDENCY | CAEP-gated; `SecurityEventTokenMapper` change is javadoc only |
| 38 | kcadm.sh / kcreg.sh — private_key_jwt | ✅ | ✅ `.rsa256(` line 213 | BLOCKED | GAP-13 |
| 39 | JGroups ASYM_ENCRYPT (non-default / test) | ✅ (`quarkus/tests/integration/src/test/resources/cache-ispn-asym-enc.xml` present) | ✅ | SAFE | GAP-14 |
| 40 | Organisation Invitation Token Verification | ✅ | ✅ | PENDING PROVIDERS | |
| 41 | FAPI Algorithm Allowlist Enforcement | ✅ | ✅ `FapiConstant.ALLOWED_ALGORITHMS` line 30: PS256/384/512, ES256/384/512 only | BLOCKED | GAP-8, GAP-17 (factory lines 42/45 ✅) |
| 42 | CryptoProvider SPI — BouncyCastle Default | ✅ | ✅ `bcprov-jdk18on` 1.86 via Quarkus BOM 3.40.1.1 | PARTIAL | Domain_42 doc says BOM 3.40.1 (§1.1); `RSAWrapperAlgorithmProvider` already noted |
| 43 | CryptoProvider SPI — FIPS (BC-FIPS) | ✅ | ✅ `bc-fips` 2.1.3, LMS-only PQC | BLOCKED | GAP-6; see §6.2 on JDK hybrid TLS group |
| 44 | CryptoProvider SPI — WildFly Elytron | ✅ | ✅ | EXTERNAL DEPENDENCY | |
| 45 | JWK Serialisation & Thumbprint | ✅ | ✅ `JWK_THUMBPRINT_REQUIRED_MEMBERS` (53–58) RSA/EC/OKP only; `JWKBuilder.akp()` 86, `JWKParser` AKP branch 84 | PARTIAL | GAP-1 |
| 46 | Default Realm Key Providers (Bootstrap) | ✅ | ✅ no AKP/ML-DSA in `DefaultKeyProviders` | BLOCKED | GAP-15/17 |
| 47 | Client Public Key Loader | ✅ | ✅ | PARTIAL | |
| 48 | Realm JWKS Endpoint — Key Serialisation | ✅ | ✅ `toJwk()` branches RSA(59)/EC(61)/OKP(63); no AKP | BLOCKED | GAP-26 |
| 49 | Admin API — Client Certificate & Keypair Generation | ✅ | ✅ RSA-only keygen (`KeycloakModelUtils` 247) | BLOCKED | GAP-27; **second call now line 288 (was 293) — LINE SHIFTED §1.4** |
| 50 | Client SDK — JWT Client Credentials Provider | ✅ | ✅ switch RSA(77)/EC(83)/OKP(89), default throw (96); no AKP | PARTIAL | GAP-28 |
| 51 | Client SDK — DPoP Proof Generation | ✅ | ✅ `generateRsaSignedDPoPProof` 50, TODO 49 | PARTIAL | GAP-29 |
| 52 | Docker Registry — Self-Signed Cert Generation | ✅ | ✅ `getKeyPairGen(KeyType.RSA)` 29, `initialize(2048, …)` 30 | BLOCKED | |
| 53 | Client Asymmetric Signature Verifier Context | ✅ | ✅ `Key Type is not RSA` guard lines 36–37 | BLOCKED | GAP-30 — **dual-badge row** (BLOCKED + PENDING PROVIDERS) |

---

## Part 3 — GAP Reference Verification

| GAP | Title | Files correct? | Line numbers | Notes |
|-----|-------|----------------|--------------|-------|
| GAP-1 | JWK Thumbprint — AKP keys | ✅ | ✅ `JWKSUtils` 53–58 | CONFIRMED |
| GAP-2 | SAML Signing — no PQC entries | ✅ | ✅ `SignatureAlgorithm.java` RSA_SHA1/256/256_MGF1/512/512_MGF1 (29–33) | CONFIRMED |
| GAP-3 | SAML Encryption — no ML-KEM | ✅ | ✅ `XMLEncryptionUtil` 77, 186, 193, 196, 431 | CONFIRMED |
| GAP-4 | JWE Key Mgmt — no ML-KEM CEK | ✅ | n/a | CONFIRMED — 7 CEK factories, no ML-KEM |
| GAP-5 | JavaKeystoreKeyProvider — ML-DSA import | ✅ | ✅ 205/214/223 | CONFIRMED |
| GAP-6 | BC-FIPS — no ML-DSA/ML-KEM | ✅ (`pom.xml` 78) | ✅ | CONFIRMED at 2.1.3; STALE 2.1.2 text elsewhere (§1.2) |
| GAP-7 | LD-Proof suite hardcoded Ed25519 | ✅ | ✅ 67 | CONFIRMED |
| GAP-8 | FAPI allowlist blocks ML-DSA | ✅ | ✅ 30 | CONFIRMED |
| GAP-9 | SAML hardcoded RS256 | ✅ | ✅ 416 / 507 / 544 / 1013 (doc 1012, +1) | CONFIRMED |
| GAP-10 | IdP broker RS256/HS256 fallback | ✅ | ✅ 972 / 980 (doc 969 / 977, +3) | CONFIRMED (edge of tolerance) |
| GAP-11 | DCR RS256 special-case | ✅ | ✅ 417 | CONFIRMED |
| GAP-12 | `DEFAULT_SIGNATURE_ALGORITHM` fallback | ✅ | ✅ `Constants` 68; `DefaultTokenManager` 240 | CONFIRMED |
| GAP-13 | AuthUtil `.rsa256()` | ✅ | ✅ 213; `JWTClientCredentialsProvider` 62 | CONFIRMED |
| GAP-14 | JGroups ASYM_ENCRYPT | ✅ | n/a | CONFIRMED |
| GAP-15 | Realm key mgmt — no ML-DSA keygen | ✅ (`common/…/CryptoProvider.java`) | n/a | CONFIRMED (previous path error fixed) |
| GAP-16 | JavaKeystoreKeyProviderFactory — ML-DSA excluded from UI | ✅ | ✅ `mergedAlgorithmProperties()` 191–204 (doc 189–202, +2) | CONFIRMED |
| GAP-17 | FAPI/CIBA executor factories — UI lists | ✅ | ✅ factory lines 42 / 45 | CONFIRMED |
| GAP-18 | JwtCNonceHandler ES256→RS256 | ✅ | ✅ 281 / 287 | CONFIRMED |
| GAP-19 | No enforcer minimum for bcprov | ✅ | n/a | CONFIRMED — root `pom.xml` has only `version.enforcer.plugin` / `extra.enforcer.rules.version` (lines 200–201); STALE BOM label 3.40.1 → 3.40.1.1 (§1.1) |
| GAP-20 | JAR encrypted request — no ML-KEM inbound | ✅ | ✅ `decodeClientJWT` 122; ENC lookup 144/150 | CONFIRMED |
| GAP-21 | UserInfo per-client signing/encryption | ✅ | ✅ 348 / 386–387 | CONFIRMED |
| GAP-22 | Attestation-based client auth `[TODO]` | ✅ | ⚠️ **418–419 → 423–424** | **LINE SHIFTED §1.3** |
| GAP-23 | OIDC discovery hardcoded RS256 | ✅ | ✅ 85 | CONFIRMED |
| GAP-24 | OID4VP hardcoded ES256 / `ACCEPTED_ALGORITHMS` | ✅ | ✅ 85; 210–211 | CONFIRMED |
| GAP-25 | OID4VP ECDH-ES / secp256r1 | ✅ | ✅ EphemeralKey 41; ResponseEncryption 38 | CONFIRMED |
| GAP-26 | `JWKSServerUtils.toJwk()` no AKP | ✅ | ✅ 59–63 | CONFIRMED |
| GAP-27 | Admin API keypair gen hardcodes RSA | ✅ | ⚠️ 126 ✅; **293 → 288** | **LINE SHIFTED §1.4** |
| GAP-28 | `JWTClientCredentialsProvider` missing AKP | ✅ | ✅ 77–96 | CONFIRMED |
| GAP-29 | `DPoPGenerator` no ML-DSA path | ✅ | ✅ 49–50 | CONFIRMED |
| GAP-30 | Client verifier RSA guard | ✅ | ✅ 36–37 | CONFIRMED |
| GAP-31 | `MdocAlgorithm` / `MdocDeviceKey` no ML-DSA/AKP | ✅ | ✅ `MdocAlgorithm` enum ends at ES512/EDDSA (41–42); `MdocDeviceKey` switch EC(60)/RSA(63)/OKP(66) | CONFIRMED |

Gaps-doc cross-check: every `*/src/main/java/...` path in `gaps/Github_issue_requirements_for_gaps.md` exists. Descriptions with stale line numbers: GAP-22 and GAP-27 (flagged STALE in Part 7).

---

## Part 4 — "Files Evaluated" Section Verification

All `.java` filenames cited in the HTML (including the "Files Evaluated" list) were resolved against `git ls-files` (non-test). Everything resolves except the two tokens explained in Part 2. `RSAWrapperAlgorithmProvider.java`, `MdocCose.java`, `MdocCredential.java`, `MdocDeviceKey.java` (added on 17-Oct) are present. Reasons given for exclusion remain valid. The remaining `core/.../mdoc` files not cited (`CborUtil`, `MdocException`, `MdocIssuerSignedDocument`, `MdocNamespacedClaims`, `MdocValidityInfo`) contain no algorithm, key-type, signature or digest references (grep returned nothing) — optional additions only.

---

## Part 5 — Status Distribution Table Verification

Method: for each of the 63 `<tr>` containing `class="num"`, took the first `class="pqc pqc-*"` badge (scripted). Exactly one row has a secondary badge: **row 53** (BLOCKED + PENDING PROVIDERS) — counted as BLOCKED.

```
Recount from domain rows (primary badge only):
  BLOCKED:              27
  PENDING PROVIDERS:    16
  PARTIAL:              12
  SAFE:                 5
  EXTERNAL DEPENDENCY:  3
  ──────────────────────
  TOTAL:                63  ← equals DOMAIN_COUNT

Current HTML Status Distribution table:
  BLOCKED:              27  (correct)
  PENDING PROVIDERS:    16  (correct)
  PARTIAL:              12  (correct)
  SAFE:                 5   (correct)
  EXTERNAL DEPENDENCY:  3   (correct)
```

No corrections required.

---

## Part 6 — New Content Not in Current HTML

### 6.1 — B3 / B4 state checks: no gap resolved, no new crypto domain

- `*SignatureProviderFactory.java` (non-test): RS256/384/512, PS256/384/512, ES256/384/512, EdDSA, HS256/384/512 — no ML-DSA.
- `*CekManagementProviderFactory.java`: ECDH-ES (+A128/192/256KW) and RSA variants — no ML-KEM.
- `Generated*/Imported*KeyProviderFactory.java`: AES, ECDH, ECDSA, EdDSA, HMAC, RSA, RSA-ENC — no AKP.
- No non-test Java file mentions `ML-DSA|ML-KEM|AKP` outside the allowed core JWK / oid4vp set. `JavaAlgorithm.java`, `DefaultKeyProviders.java`, `JWKSServerUtils.java`: no ML-DSA/AKP.
- `JWKSUtils` thumbprint map: RSA/EC/OKP only. `FapiConstant`: no ML-DSA. `ClientAsymmetricSignatureVerifierContext`: RSA-only guard. `SsfSignatureAlgorithms.ALLOWED`: RS256 only. `OIDCWellKnownProvider`: static RS256.
- New non-test Java files since last review: workflow-engine classes (`UserDisabledBy*LockoutWorkflowEvent*`, `LogoutUserStepProvider*`), `AcrNotFulfilledException`, `ClientInstallationUtils` (secret masking only). None relate to PQC.
- Renames/deletions of `.java` files since last review: none.

### 6.2 — NEW (informational): JDK hybrid TLS group `X25519MLKEM768` breaks FIPS CI

Commit `cfe4a411c5` (2026-10-28, "Disable X25519MLKEM768 for Maven in FIPS tests (#54399)") sets `-Djdk.tls.namedGroups=secp256r1,secp384r1,secp521r1,x25519,ffdhe2048,ffdhe3072` in `.github/scripts/run-fips-it.sh` and `run-fips-ut.sh`, because JDK 25.0.5+ offers the ML-KEM hybrid TLS group by default and it fails in FIPS mode when Maven resolves from Central. This is the first place Keycloak's FIPS test setup has met a PQC primitive implicitly via the JDK. It is a transport-layer (JSSE) matter, not a Keycloak token/JWT/SAML code path, so no row or GAP is affected, but it reinforces row 43 / GAP-6: BC-FIPS 2.1.3 cannot supply ML-KEM and the FIPS configuration actively disables it. Candidate one-line note in Domain_43.

### 6.3 — Upstream changes reviewed with no PQC impact

- **SAML broker hardening:** `641c549949` (OneTimeUse replay protection — uses `HashUtils.sha256UrlEncodedHash`, SHA-256 only), `e4cc3ef7fe` (Recipient bound to target client), `25488cd90d` (reject empty principal), `ff5df30e90` (ForceAuthn stored as client note), `b42bee42bf` (artifact-resolve receiver Destination, test util). Files: `SAMLEndpoint`, `SAMLIdentityProviderConfig` (adds `oneTimeUseMaxAgeSeconds`), `ConditionsValidator`, `SubjectConfirmationDataValidator`, `SamlService`, `SamlProtocol`. `SamlProtocol` change is a one-line `getAuthNote` → `getClientNote` swap for ForceAuthn; the RS256 call site (line 544) is unchanged.
- **`KeystoreUtil.loadKeyStore(..., provider)`** (`895e8045cd`, "use configured crypto provider to read generated truststores"): adds an optional JCE provider argument so truststores are read through the active `CryptoProvider` (relevant to FIPS truststore types). No algorithm or key-type change.
- **Client installation secret masking** (`32d89586d9`, `ClientInstallationUtils`, `getAdapterConfigurationSecretKeys`): no crypto change; responsible for the §1.3 line shift.
- **`decodeClientJWT()` JWE `alg` binding** (`6bc5146087`, previously noted 17-Oct): still in place; GAP-20 unchanged.
- **Quarkus 3.40.1.1** (`b1920d7f82`) and `jsoup 1.23.2`: no `bcprov` change.
- `AccessTokenContext` null-check typo fix, `SecurityEventTokenMapper` javadoc, `OutboxStore`/JPA dialects, Admin UI, workflows, group-cycle check: none relevant.

---

## Part 7 — Summary of Required Changes

### Part 7A — Corrections to existing content

| # | Severity | Location | Change required |
|---|----------|----------|-----------------|
| C1 | STALE | `gaps/Github_issue_requirements_for_gaps.md` line 368 (GAP-19); `domains/Domain_42_CryptoProvider_BouncyCastle.md` line 13 | "Quarkus BOM 3.40.1" → **3.40.1.1** (bcprov stays 1.86). |
| C2 | STALE | `core-authn/pqc-cross-reference-analysis.md` 467; `core-authn/core-authn-issue-list.md` 234, 240; `core-authn/pqc-issues-gaps-core-authn-ownership-analysis.html` 663 (+ prose); `.bob/artifacts/…ownership-analysis.html` 663; `gh_issues/Github_PQC_issues_overview.md` 158; `approach/pqc_configuration_strategy.md` 123 | `BC-FIPS 2.1.2` → **2.1.3** (still LMS-only). |
| C3 | LINE SHIFTED | `pqc_overview.html` 448, 451; gaps doc GAP-22 (420, 425, 429); `domains/Domain_12_…md` 20, 22, 81, 182, 186; `core-authn/core-authn-issue-list.md` 108, 127, 141; `core-authn/pqc-cross-reference-analysis.md` 367 | `[TODO]` algorithm enforcement "line 418" / "418–419" → **423–424**. |
| C4 | LINE SHIFTED | gaps doc GAP-27 (515, 520); `domains/Domain_49_Admin_API_Cert_Generation.md` 13, 21 | `generateKeyPairCertificate` second call line 293 → **288**. |
| C5 | Housekeeping | `pqc_overview.html` line 2347; `README.md` line 48 | Update "Last updated" to 29 October 2026 and HEAD `f82c6402ba` when the above are applied. |

### Part 7B — Additions of new content

| # | Severity | Action |
|---|----------|--------|
| A1 | NEW (optional) | One-line note in `Domain_43` (and optionally GAP-6 text): FIPS CI disables JDK `X25519MLKEM768` hybrid TLS group (`cfe4a411c5`); BC-FIPS 2.1.3 cannot provide ML-KEM. |
| A2 | NEW (optional) | Add `CborUtil`, `MdocException`, `MdocIssuerSignedDocument`, `MdocNamespacedClaims`, `MdocValidityInfo` to the mdoc "Files Evaluated" entry as non-crypto. |

### Part 7C — Resolved items

None. All 31 GAPs remain open; no status badge should change; the Status Distribution table (27 / 16 / 12 / 5 / 3 = 63) is correct.

### Part 7D — Prompt maintenance (for `prompts/REASSESS_PQC_READINESS_PROMPT.md`, not the HTML)

Items from the 2026-10-17 report that are still **not** applied to the prompt:
- Section A row for `DIDUtils.java` is dead (no such file in the tree; nothing in HTML/domains/gaps references it). Remove.
- Section A row for `DefaultSamlArtifactResolver.java` says the SOAP fetch is in `SAMLEndpoint.java`; it is in `SAMLIdentityProvider.resolveArtifact()` (called and validated from `SAMLEndpoint`).
- Section A Maven Enforcer row says the plugin is "present in `pom.xml`"; the root `pom.xml` only defines version properties (`version.enforcer.plugin`, `extra.enforcer.rules.version`).
- Section B3 `quarkus.version|quarkus.platform` grep finds the pin but not the resulting `bcprov` version — read it from `~/.m2/.../quarkus-bom/<version>/*.pom`.
- Section B2 should add `UserInfoEndpoint` (GAP-21), `DefaultTokenManager.decodeClientJWT` (GAP-20) and the `AbstractOAuth2IdentityProvider` HS256 line (GAP-10).
- C1 template says "all 30 GAPs" / "62+ rows"; now 31 GAPs / 63 rows.

New observations from this run:
- **B1 `find .` times out** on a full Keycloak checkout (>50 s) — use `git ls-files '*.java' | grep -v /test/` and compare basenames instead.
- **B1 false MISSING entries:** the glob token `EcdhEsA*KwCekManagementProviderFactory.java` and the annotated rename `AttestationX509CertificateValidator.java` always appear unresolved. Tell the agent to expand globs and to ignore tokens that the HTML itself marks as "previously cited as".
- **B2 `ClientAttributeCertificateResource` grep points at the wrong file** (`integration/admin-client-core/.../ClientAttributeCertificateResource.java`, the REST client, which has no matches). Correct path: `services/src/main/java/org/keycloak/services/resources/admin/ClientAttributeCertificateResource.java`, grep for `generateKeyPairCertificate`; documented lines are 126 / 288 (the prompt's "119, 258" is long out of date).
- **B2 `AuthUtil` grep** `\.rsa256\(` is invalid in basic regex (unmatched `\(`); use `grep -nE "\.rsa256\("`.
- **B2 GAP-22 grep** `\[TODO\].*alg|alg.*\[TODO\]` needs `-E`; update the "documented" lines to 423–424. Several other greps in B2 use `\|`/`|` inconsistently — standardise on `grep -nE`.
- **B2 Domain 62/OID4VP duplicates** the GAP-24/25 greps; they can be collapsed.
- **Section A "Constants" row:** `INTERNAL_SIGNATURE_ALGORITHM` is now line 69 (was 68 + 1).
- **B4 `find` for key provider factories** has a precedence bug (`-o` without parentheses before `! -path`); use `git ls-files | grep -E "(Generated|Imported).*KeyProviderFactory.java"`.

---

*Review conducted against Keycloak `main` @ `f82c6402ba` on 2026-10-29. `pqc_overview.html` and all domain/gap markdown files were left unmodified, pending reviewer approval.*
