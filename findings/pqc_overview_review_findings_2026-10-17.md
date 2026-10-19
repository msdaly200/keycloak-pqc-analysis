# PQC Overview — Complete Accuracy Review

**Date of review:** 2026-10-17  
**Keycloak branch:** main (HEAD: `eb3ac9a4ba`, committed 2026-09-27; local clone reported "up to date with origin/main" — no `git fetch` was run)  
**Previous review HEAD:** `63aeb4c98a` (2026-09-23) — 330 commits behind current HEAD  
**Scope:** Every claim in every row, every GAP, and the "Files Evaluated" section  
**DOMAIN_COUNT:** 63 (`grep -c 'class="num"' pqc_overview.html`)  
**GAP_LIST:** GAP-1 … GAP-31 (31 GAPs; the prior review header said 30 — the HTML already carried GAP-31 after the 2026-09-24 edit pass)  
**FILE_LIST:** 307 distinct `*.java` tokens in the HTML

**Method note / evidence levels.** Pre-flight, Section A, B1, B2, B3, B4 and the Part 5 recount were all executed against the live source. For Part 2, *file existence* was checked programmatically for every file cited in every one of the 63 rows. *Algorithm / PQC-state claims* were re-verified through the B2/B3 greps (every row that cites a line number, hardcoded algorithm, or "not yet implemented" state is covered). Rows whose prose is purely architectural (SPI-driven, "will inherit ML-DSA") were not re-derived line by line; they are marked "files ✅, no contradicting change at HEAD".

**Important observation about the previous report.** The Part 2 table in `pqc_overview_review_findings_2026-09-24.md` uses row labels and statuses that do **not** match the HTML's actual row numbering (e.g. its "row 1 — BLOCKED" vs HTML row 1 "Access Token / ID Token Signing — PARTIAL"; its "row 28 — ML-DSA SignatureProvider" vs HTML row 28 "Generated RSA Signing Key Provider"). The table below uses the HTML's own numbering and statuses and should be treated as authoritative.

---

## Part 1 — Errors and Inaccuracies

### 1.1 — `bc-fips` version is STALE (2.1.2 → 2.1.3); the GAP-6 conclusion still holds

**Severity:** STALE  
**Evidence:** `pom.xml` line 78 is now `<bouncycastle.bcfips.version>2.1.3</bouncycastle.bcfips.version>` (commit `1bf99e3722`, "Upgrade BouncyCastle FIPS (#53182)", 2026-09-25). I inspected `~/.m2/.../bc-fips-2.1.3.jar`: the only `internal/pqc` content is `MessageSigner`, `StateAwareMessageSigner`, `Digesting*MessageSigner`, `ExchangePair*`, `MessageEncryptor` and `lms/*`. **0** entries match `mldsa|mlkem|ml_dsa|ml_kem`. So FIPS mode still has no ML-DSA / ML-KEM — GAP-6 remains open and correct.  
**Stale text locations (all say "2.1.2"):**
- `pqc_overview.html` lines 1331, 1336 (Domain 43), 2214 (GAP-6 bullet), 2327 (FIPS Hard Blocker bullet)
- `gaps/Github_issue_requirements_for_gaps.md` lines 114, 119 (GAP-6)
- `domains/Domain_43_CryptoProvider_FIPS.md` lines 11, 13, 17; `Domain_11_…md` line 47; `Domain_13_…md` line 75; `Domain_2_…md` line 236

### 1.2 — `bcprov-jdk18on` version is STALE (1.85.2 → 1.86); Quarkus pin moved to 3.40.1

**Severity:** STALE  
**Evidence:** `pom.xml` line 43 is now `<quarkus.version>3.40.1</quarkus.version>` (was `3.40.0.CR1`). The Quarkus platform BOM 3.40.1 (`quarkus-bom-3.40.1.pom` in `~/.m2`) declares `bcprov-jdk18on` **1.86**. The 3.40.0.CR1 BOM declared 1.85.2, so the previous review's number was right for that pin. `bcprov-jdk18on-1.86.jar` contains 121 `mldsa` and 29 `provider/asymmetric/mlkem` entries, so the functional claim (ML-DSA/ML-KEM present, ≥ 1.78) stands.  
**Stale text locations:**
- `pqc_overview.html` line 1315 ("`bcprov-jdk18on` 1.85.2 (in use)")
- `gaps/Github_issue_requirements_for_gaps.md` line 368 (GAP-19: "1.85.2 via Quarkus BOM 3.40.0.CR1")
- `domains/Domain_42_CryptoProvider_BouncyCastle.md` lines 13, 17

GAP-19 itself (no enforcer minimum-version rule) is **CONFIRMED**: root `pom.xml` contains only the properties `version.enforcer.plugin` / `extra.enforcer.rules.version` (lines 199-200) — no `maven-enforcer-plugin` declaration and no `bcprov` rule anywhere in a `pom.xml` enforcer/ban/require context.

### 1.3 — Row 60 (SAML Artifact Resolution): wrong class named for the SOAP back-channel fetch

**Severity:** ERROR (documentation accuracy; no change to the PQC assessment)  
**Location:** `pqc_overview.html` line 739 — "Artifact-resolved assertions (fetched over the SOAP backchannel by `SAMLEndpoint.java`, already covered in row 20)". The same wording is in the Section A fact of this prompt.  
**Actual:** The SOAP call (`Soap.createMessage()…call(artifactResolutionServiceUrl, session)`) is in `SAMLIdentityProvider.java` (~lines 562-585, `resolveArtifact(...)`). `SAMLEndpoint.java` (lines 432-458) only *invokes* `provider.resolveArtifact(...)` and then validates the `ArtifactResponse` signature. The same layout existed at the previous review's HEAD (`SAMLIdentityProvider.java:575` at `63aeb4c98a`), so this is a long-standing wording error rather than a new regression. `DefaultSamlArtifactResolver.resolveArtifact()` (line 32) is storage/routing only, as documented.  
**Required correction:** "…fetched over the SOAP backchannel by `SAMLIdentityProvider.resolveArtifact()` and validated in `SAMLEndpoint.java`…". Row 60 remains BLOCKED (GAP-2 / GAP-9).

### 1.4 — Wrong module path in the gaps document (GAP-15)

**Severity:** ERROR (gaps doc only)  
**Location:** `gaps/Github_issue_requirements_for_gaps.md` line 287 lists `core/src/main/java/org/keycloak/common/crypto/CryptoProvider.java`.  
**Actual:** `common/src/main/java/org/keycloak/common/crypto/CryptoProvider.java`. This is the only mismatch among all `*/src/main/java/...` paths cited in the gaps document (every other path exists).

### 1.5 — LINE SHIFTED items (crypto code unchanged, shift > ±3)

| # | Location | Documented | Actual | Shift | Where documented |
|---|----------|-----------|--------|-------|------------------|
| 1.5a | `AbstractOAuth2IdentityProvider.java` — `Algorithm.HS256` fallback (GAP-10) | 737 | **969** | +232 | gaps doc line 190/195; HTML line 845 |
| 1.5b | `AbstractOAuth2IdentityProvider.java` — `Algorithm.RS256` fallback (GAP-10) | 745 | **977** | +232 | gaps doc lines 190/195/199; HTML line 845 |
| 1.5c | `DefaultTokenManager.java` — `Constants.DEFAULT_SIGNATURE_ALGORITHM` fallback (GAP-12) | 233 | **240** | +7 | gaps doc lines 228/233 |
| 1.5d | `DefaultTokenManager.decodeClientJWT()` key selection by `KeyUse.ENC` (GAP-20) | 139, 144 | **144, 150** | +5/+6 | gaps doc line 387 |
| 1.5e | `SamlService.java` — `Algorithm.RS256` (GAP-9) | 992 | **1012** | +20 | gaps doc line 171; `domains/Domain_18_SAML_Signing.md` lines 62, 144 |
| 1.5f | `UserInfoEndpoint.java` — `signatureAlgorithm(TokenCategory.USERINFO)` / `cekManagementAlgorithm` / `encryptAlgorithm` (GAP-21) | 341, 379–380 | **348, 386–387** | +7 | gaps doc line 401 |

Notes: 1.5f was already at 348/386-387 at the previous review's HEAD, so it was missed before (it is not in the B2 grep list). `decodeClientJWT()` itself moved only −1 (123 → 122, ✅). Pre-existing stale reference outside this review's scope: `domains/Domain_21_OIDC_IdP_Broker.md` lines 18/78/107 still cite `AbstractOAuth2IdentityProvider.java` line **697**.

### 1.6 — Informational: `decodeClientJWT()` behaviour changed (relevant to GAP-20, no status change)

Commit `6bc5146087` (2026-10-12) now rejects a JWE with no `alg` header and selects the decryption key only when `headerAlg.equals(k.getAlgorithm())` and the key is active (or enabled when addressed by `kid`). This makes future ML-KEM key selection by algorithm name more natural, but there is still no ML-KEM `CekManagementProviderFactory` and no way to hold an ML-KEM `ENC` key, so GAP-20 stays open. Resolution-criteria wording ("ML-KEM decapsulation path present in `decodeClientJWT()`") is unaffected.

---

## Part 2 — Domain-by-Domain Verification Table

Files = every `*.java` token in the row's Key Files column resolved against non-test Keycloak sources (wildcards expanded). All 63 rows: **files ✅**. Status = primary (first) badge in the HTML.

| Row | Domain | Files exist? | Algorithm claims / state verified | PQC state | Notes |
|-----|--------|--------------|-----------------------------------|-----------|-------|
| 1 | Access Token / ID Token Signing | ✅ | ✅ `Algorithm.ML_DSA_44/65/87` + `KeyType.AKP` present; no ML-DSA factory | PARTIAL | GAP-12 fallback now line 240 (LINE SHIFTED) |
| 2 | ID Token / JARM Encryption (Outbound CEK) | ✅ | ✅ no ML-KEM CEK provider | BLOCKED | New `RSAWrapperAlgorithmProvider` wraps RSA key unwrap (see Part 6) — no PQC change |
| 3 | Backchannel Logout Token Signing | ✅ | ✅ `LOGOUT` and `ID` share `ID_TOKEN_SIGNED_RESPONSE_ALG` (DefaultTokenManager 214-216) | PENDING PROVIDERS | |
| 4 | UserInfo — Signed & Encrypted Response | ✅ | ✅ | BLOCKED | GAP-21 lines now 348, 386–387 (LINE SHIFTED) |
| 5 | Introspection — Embedded JWT Response | ✅ | ✅ | PARTIAL | |
| 6 | Token Verification (Identity & Session Tokens) | ✅ | ✅ | PENDING PROVIDERS | |
| 7 | JAR — Signed Request Object Verification | ✅ | ✅ | PENDING PROVIDERS | |
| 8 | JAR — Encrypted Request Object Decryption | ✅ | ✅ no ML-KEM path | BLOCKED | GAP-20 lines shifted; key selection now alg-bound (§1.5d, §1.6) |
| 9 | JARM — Signed Authorization Response | ✅ | ✅ | PENDING PROVIDERS | |
| 54 | Federated JWT Client Authentication | ✅ | ✅ | PENDING PROVIDERS | |
| 10 | private_key_jwt Client Authentication | ✅ | ✅ | PENDING PROVIDERS | |
| 11 | X.509 mTLS Client Authentication | ✅ | ✅ | SAFE | `X509ClientAuthenticator`, `CertificateValidator`, `OCSPProvider` modified upstream; diffs contain no algorithm/key-type changes |
| 12 | Attestation-Based Client Authentication | ✅ | ✅ `[TODO]` alg enforcement at line 418 | PARTIAL | GAP-22 ✅ |
| 13 | X.509 Browser Authentication Flow | ✅ | ✅ | SAFE | Authenticator diffs: no crypto-algorithm change |
| 14 | WebAuthn / Passkeys (FIDO2) | ✅ | ✅ | EXTERNAL DEPENDENCY | |
| 15 | DPoP | ✅ | ✅ | PARTIAL | |
| 16 | CIBA Signed Backchannel Auth Request | ✅ | ✅ | PARTIAL | |
| 57 | Token Exchange — Verification & Signing | ✅ | ✅ | PENDING PROVIDERS | `StandardTokenExchangeProvider` modified; no algorithm change |
| 58 | Device Authorization Grant — Token Signing | ✅ | ✅ | PENDING PROVIDERS | |
| 17 | JWT Authorization Grant — Assertion Verification | ✅ | ✅ | PENDING PROVIDERS | |
| 18 | SAML Assertion & Document Signing | ✅ | ✅ RS256 at SAMLIdentityProvider 416/507, SamlProtocol 544; SamlService 1012 | BLOCKED | SamlService LINE SHIFTED (992 → 1012) |
| 19 | SAML Assertion Encryption | ✅ | ✅ `XMLEncryptionUtil` RSA_OAEP_11 default (line 77), RSA_v1dot5 (431) | BLOCKED | |
| 20 | SAML IdP Broker — SP Metadata & Federation Signing | ✅ | ✅ RS256 at 416/507 | BLOCKED | |
| 55 | Default Trust Identity Provider | ✅ | ✅ | PENDING PROVIDERS | |
| 59 | SAML Metadata Public Key Loader | ✅ | ✅ | SAFE | |
| 60 | SAML Artifact Resolution | ✅ | ✅ | BLOCKED | **ERROR §1.3** — SOAP fetch is in `SAMLIdentityProvider`, not `SAMLEndpoint` |
| 61 | SAML2Signature — Hardcoded RSA-SHA1 Default | ✅ | ✅ `RSA_SHA1` line 55, `DigestMethod.SHA1` line 57 | BLOCKED | |
| 62 | OID4VP IdP — Request Signing & Response Encryption | ✅ | ✅ `ACCEPTED_ALGORITHMS` line 85; ES256 at 210–211; `CURVE_SEC` line 41; `KEY_MANAGEMENT_ALG` line 38; TODO now line 83 (doc says 81, +2 ✅) | BLOCKED | GAP-24/25 ✅; the 4 `mappers/*` files contain no crypto (Part 6) |
| 63 | OID4VCI Credential Response Encryption | ✅ | ✅ no ML-KEM CEK provider | BLOCKED | |
| 21 | OIDC IdP — Token Sig Verification & JWE Decryption | ✅ | ✅ JWE key lookup at `OIDCIdentityProvider` 675/677 unchanged | PARTIAL | GAP-10 lines shifted (§1.5a/b) |
| 22 | Kubernetes Identity Provider | ✅ | ✅ | PENDING PROVIDERS | |
| 23 | SPIFFE / SVID Identity Provider | ✅ | ✅ | PENDING PROVIDERS | |
| 56 | SD-JWT Issuer Signing & Key Binding Verification | ✅ | ✅ | PENDING PROVIDERS | `ClaimVerifier` only enforces `alg != none` (Part 6) |
| 24 | JWT-VC / SD-JWT Credential Signing | ✅ | ✅ | PENDING PROVIDERS | GAP-31 mdoc sibling — see Part 6 |
| 25 | LD-Proof Credential Signing | ✅ | ✅ `Ed25519Signature2018` proof type, `Ed255192018Suite` line 67 | BLOCKED | GAP-7 ✅ |
| 26 | OID4VC Key Binding — JWT Proof Validation | ✅ | ✅ `AttestationProofValidator.java` present | PENDING PROVIDERS | |
| 27 | OID4VC c_nonce JWT Signing | ✅ | ✅ ES256 line 281, RS256 line 287 | BLOCKED | GAP-18 ✅ |
| 28 | Generated RSA Signing Key Provider | ✅ | ✅ no `GeneratedAKP*` factory | BLOCKED | |
| 29 | Generated RSA Encryption Key Provider | ✅ | ✅ | BLOCKED | |
| 30 | Generated ECDSA Signing Key Provider | ✅ | ✅ | BLOCKED | |
| 31 | Generated EdDSA Signing Key Provider | ✅ | ✅ | BLOCKED | |
| 32 | Generated ECDH Encryption Key Provider | ✅ | ✅ | BLOCKED | |
| 33 | Imported RSA Signing Key Provider | ✅ | ✅ no `ImportedAKP*` factory | BLOCKED | |
| 34 | Java Keystore Key Provider | ✅ | ✅ branches OKP(205)/EC(214)/RSA(223); no AKP | BLOCKED | GAP-5 ✅ |
| 35 | Dynamic Client Registration Tokens | ✅ | ✅ `DescriptionConverter` RS256 special-case at 417 (doc 416, +1 ✅); `INTERNAL_SIGNATURE_ALGORITHM = HS512` | SAFE | `ClientRegistrationTokenUtils` refactor only |
| 36 | OIDC Well-Known — Algorithm Advertisement | ✅ | ✅ `DEFAULT_CLIENT_AUTH_SIGNING_ALG_VALUES_SUPPORTED = list(RS256)` line 85 | PARTIAL | GAP-23 ✅ |
| 37 | Security Event Token (SET) Signing | ✅ | ✅ `ALLOWED = Set.of(Algorithm.RS256)` (line 31), `DEFAULT = RS256` (38) | EXTERNAL DEPENDENCY | CAEP-gated |
| 38 | kcadm.sh / kcreg.sh — private_key_jwt | ✅ | ✅ `.rsa256(` line 213 | BLOCKED | GAP-13 ✅ |
| 39 | JGroups ASYM_ENCRYPT (non-default / test) | ✅ (`cache-ispn-asym-enc.xml` present) | ✅ | SAFE | GAP-14 |
| 40 | Organisation Invitation Token Verification | ✅ | ✅ | PENDING PROVIDERS | |
| 41 | FAPI Algorithm Allowlist Enforcement | ✅ | ✅ `FapiConstant.ALLOWED_ALGORITHMS` line 30: PS*/ES* only | BLOCKED | GAP-8, GAP-17 (factory lines 42/45 ✅) |
| 42 | CryptoProvider SPI — BouncyCastle Default | ✅ | ⚠️ version STALE (§1.2) | PARTIAL | Functional claim OK |
| 43 | CryptoProvider SPI — FIPS (BC-FIPS) | ✅ | ⚠️ version STALE (§1.1); still no ML-DSA/ML-KEM | BLOCKED | GAP-6 still open |
| 44 | CryptoProvider SPI — WildFly Elytron | ✅ | ✅ | EXTERNAL DEPENDENCY | |
| 45 | JWK Serialisation & Thumbprint | ✅ | ✅ `JWK_THUMBPRINT_REQUIRED_MEMBERS` (53–58) RSA/EC/OKP only | PARTIAL | GAP-1 ✅; `JWKBuilder.akp()`, `JWKParser` AKP branch (84), `AKPUtils`, `AKPPublicJWK` present |
| 46 | Default Realm Key Providers (Bootstrap) | ✅ | ✅ no AKP/ML-DSA in `DefaultKeyProviders` | BLOCKED | GAP-15/17 |
| 47 | Client Public Key Loader | ✅ | ✅ | PARTIAL | |
| 48 | Realm JWKS Endpoint — Key Serialisation | ✅ | ✅ `toJwk()` branches RSA(59)/EC(61)/OKP(63); no AKP | BLOCKED | GAP-26 ✅ |
| 49 | Admin API — Client Certificate & Keypair Generation | ✅ | ✅ `generateKeyPairCertificate` at 126, 293 | BLOCKED | GAP-27 ✅ |
| 50 | Client SDK — JWT Client Credentials Provider | ✅ | ✅ switch RSA(77)/EC(83)/OKP(89), default throw (96); no AKP | PARTIAL | GAP-28 ✅ |
| 51 | Client SDK — DPoP Proof Generation | ✅ | ✅ `generateRsaSignedDPoPProof` line 50, `// TODO: Similar for EC and EdDSA` line 49 | PARTIAL | GAP-29 ✅ |
| 52 | Docker Registry — Self-Signed Cert Generation | ✅ | ✅ `getKeyPairGen(KeyType.RSA)` 29, `initialize(2048)` 30 | BLOCKED | |
| 53 | Client Asymmetric Signature Verifier Context | ✅ | ✅ `Key Type is not RSA` guard lines 36–37 | BLOCKED | GAP-30 ✅ — **dual-badge row** (BLOCKED + PENDING PROVIDERS) |

---

## Part 3 — GAP Reference Verification

| GAP | Title | Files correct? | Line numbers | Notes |
|-----|-------|----------------|--------------|-------|
| GAP-1 | JWK Thumbprint — AKP keys | ✅ | ✅ `JWKSUtils` 53–58 | CONFIRMED — no AKP entry |
| GAP-2 | SAML Signing — no PQC entries | ✅ | ✅ `SignatureAlgorithm.java` RSA_SHA1/256/256_MGF1/512/512_MGF1 only (29–33) | CONFIRMED |
| GAP-3 | SAML Encryption — no ML-KEM | ✅ | ✅ `XMLEncryptionUtil` 77, 186, 193, 196, 431 | CONFIRMED; `SAMLEncryptionAlgorithms` = RSA_OAEP, RSA1_5 only |
| GAP-4 | JWE Key Mgmt — no ML-KEM CEK | ✅ | n/a | CONFIRMED — no `*MLKEM*CekManagement*`; 7 CEK factories, all ECDH-ES / RSA |
| GAP-5 | JavaKeystoreKeyProvider — ML-DSA import | ✅ | ✅ 205/214/223 branch structure | CONFIRMED |
| GAP-6 | BC-FIPS — no ML-DSA/ML-KEM | ✅ (`pom.xml` line 78 ✅) | ✅ | **STALE text 2.1.2 → 2.1.3** (§1.1); gap itself CONFIRMED |
| GAP-7 | LD-Proof suite hardcoded Ed25519 | ✅ | `Ed255192018Suite` PROOF_TYPE now line 67 (doc 66, +1 ✅) | CONFIRMED |
| GAP-8 | FAPI allowlist blocks ML-DSA | ✅ | ✅ line 30 | CONFIRMED |
| GAP-9 | SAML hardcoded RS256 | ✅ | SAMLIdentityProvider 416/507 ✅; SamlProtocol 544 ✅; **SamlService 992 → 1012** | LINE SHIFTED (§1.5e) |
| GAP-10 | IdP broker RS256/HS256 fallback | ✅ | **737 → 969, 745 → 977** | LINE SHIFTED (§1.5a/b) |
| GAP-11 | DCR RS256 special-case | ✅ | 416 → 417 ✅ (+1) | CONFIRMED |
| GAP-12 | `DEFAULT_SIGNATURE_ALGORITHM` fallback | ✅ | Constants line 68 ✅; **DefaultTokenManager 233 → 240** | LINE SHIFTED (§1.5c) |
| GAP-13 | AuthUtil `.rsa256()` | ✅ | ✅ 213; `JWTClientCredentialsProvider` 62 & 96 ✅ | CONFIRMED |
| GAP-14 | JGroups ASYM_ENCRYPT | ✅ | n/a (test-only XML) | CONFIRMED |
| GAP-15 | Realm key mgmt — no ML-DSA keygen | ⚠️ `CryptoProvider.java` path wrong in gaps doc | n/a | **ERROR §1.4** (correct module `common/`) |
| GAP-16 | JavaKeystoreKeyProviderFactory — ML-DSA excluded from UI | ✅ | merged algorithm list at ~188–202 (AES, EdDSA, ES*, HS*, RS*, RSA-ENC, ECDH) ✅ | CONFIRMED |
| GAP-17 | FAPI/CIBA executor factories — UI lists | ✅ | ✅ factory lines 42 / 45 | CONFIRMED |
| GAP-18 | JwtCNonceHandler ES256→RS256 | ✅ | ✅ 281 / 287 | CONFIRMED |
| GAP-19 | No enforcer minimum for bcprov | ✅ | n/a | CONFIRMED; **STALE version text 1.85.2 → 1.86** (§1.2) |
| GAP-20 | JAR encrypted request — no ML-KEM inbound | ✅ | `decodeClientJWT` 122 ✅; **ENC lookup 139/144 → 144/150** | LINE SHIFTED (§1.5d); behaviour note §1.6 |
| GAP-21 | UserInfo per-client signing/encryption | ✅ | **341, 379–380 → 348, 386–387** | LINE SHIFTED (§1.5f) |
| GAP-22 | Attestation-based client auth `[TODO]` | ✅ | ✅ 418 (HTML cites 418–419) | CONFIRMED |
| GAP-23 | OIDC discovery hardcoded RS256 | ✅ | ✅ 85 | CONFIRMED |
| GAP-24 | OID4VP hardcoded ES256 / `ACCEPTED_ALGORITHMS` | ✅ | ✅ 85; 210–211 | CONFIRMED |
| GAP-25 | OID4VP ECDH-ES / secp256r1 | ✅ | ✅ EphemeralKey 41; ResponseEncryption 38 | CONFIRMED |
| GAP-26 | `JWKSServerUtils.toJwk()` no AKP | ✅ | ✅ 59–63 (gaps doc cites 59–65, trailing lines are the closing branch) | CONFIRMED |
| GAP-27 | Admin API keypair gen hardcodes RSA | ✅ | ✅ 126 / 293 | CONFIRMED |
| GAP-28 | `JWTClientCredentialsProvider` missing AKP | ✅ | ✅ 77–96 | CONFIRMED |
| GAP-29 | `DPoPGenerator` no ML-DSA path | ✅ | ✅ 49–50 | CONFIRMED |
| GAP-30 | Client verifier RSA guard | ✅ | ✅ 36–37 | CONFIRMED |
| GAP-31 | `MdocAlgorithm` no ML-DSA COSE mapping | ✅ | n/a — enum still lists only RS/PS/ES/EdDSA | CONFIRMED; scope may be broader (§6.1) |

Gaps-doc cross-check: all `*/src/main/java/...` paths in `gaps/Github_issue_requirements_for_gaps.md` exist except the one in §1.4. Descriptions that cite stale line numbers are flagged STALE in Part 7.

---

## Part 4 — "Files Evaluated" Section Verification

161 file entries checked. All exist in non-test sources at HEAD. Two entries are glob patterns (`*SignatureProviderFactory.java`, `HS*SignatureProviderFactory.java`) and both resolve (`HS256/384/512SignatureProviderFactory.java` exist). `AttestationProofValidator.java` (renamed from `AttestationX509CertificateValidator.java`) is present, and the HTML correctly annotates the rename. The ISO mdoc entry added after the previous review (MdocAlgorithm / MdocCredentialBody / MdocCredentialSigner / CredentialSigningAlgorithmResolver) is present and accurate. Reasons given for exclusion remain valid.

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

### 6.1 — Remaining ISO mdoc classes (extends GAP-31 scope)

The previous review added `MdocAlgorithm`, `MdocCredentialBody`, `MdocCredentialSigner`, `CredentialSigningAlgorithmResolver`. Three more mdoc classes in `core/src/main/java/org/keycloak/mdoc/` and two consumers are still not mentioned in the HTML:

| File | Relevance |
|------|-----------|
| `MdocDeviceKey.java` | `fromProofJwk()` converts the *holder* proof JWK into a COSE_Key and supports only `KeyType.EC`, `RSA`, `OKP` (Ed25519); any other type (including `AKP`) throws `MdocException("Unsupported proof key type for mDoc COSE_Key")`. This is a second, holder-side ML-DSA blocker for mdoc, distinct from the issuer-side `MdocAlgorithm` mapping. |
| `MdocCose.java` | `sign1()` writes the COSE `alg` header from `MdocAlgorithm` — inherits the same allow-list. |
| `MdocCredential.java` | `resolveSigningAlgorithm()` calls `MdocAlgorithm.fromJoseAlgorithm()` (throws for non-listed algorithms); hash is fixed SHA-256 (no PQC impact). |
| `CredentialBuildConfig.java`, `SupportedCredentialConfiguration.java` | Consume `CredentialSigningAlgorithmResolver` / `MdocAlgorithm.fromJoseAlgorithm()`; no independent gap. |

**Suggestion:** extend GAP-31's description/resolution criteria to include `MdocDeviceKey` (AKP → COSE_Key) and add `MdocCose` / `MdocCredential` / `MdocDeviceKey` to the existing mdoc "Files Evaluated" entry. Severity: NEW.

### 6.2 — `RSAWrapperAlgorithmProvider.java` (new, classical-only)

`core/src/main/java/org/keycloak/jose/jwe/alg/RSAWrapperAlgorithmProvider.java` (commit `8b678aee35`, "Harden RSA1_5 JWE decryption against timing-based padding oracles"). It wraps the RSA1_5, RSA-OAEP and RSA-OAEP-256 `JWEAlgorithmProvider`s in the Default, FIPS and Elytron crypto providers and substitutes a random CEK on unwrap failure. RSA-only; no PQC gap, no status change. Candidate for the "Files Evaluated" list (related to rows 2, 8, 42–44).

### 6.3 — OID4VP claim-mapper files not named in the HTML

`broker/oid4vp/mappers/AbstractOID4VPClaimMapper.java`, `ClaimPath.java`, `OID4VPSdJwtUserAttributeMapper.java`, `OID4VPSdJwtUserSessionAttributeMapper.java` are not literally cited. A grep for `Algorithm.`, `RS256`, `ES256` and `SignatureProvider` in that directory returns nothing — claim/attribute mapping only, no PQC relevance. `core/.../sdjwt/ClaimVerifier.java` likewise only adds a negated `alg == "none"` header check (line ~432). Optional additions to "Files Evaluated"; `test-framework/.../OAuthIdentityProvider.java` is test infrastructure and out of scope.

### 6.4 — Upstream changes reviewed with no PQC impact

`BCEcdhEsAlgorithmProvider` / `BCFIPSEcdhEsAlgorithmProvider` (added EC point-on-curve validation), `SAMLEndpoint` / `SamlService` (back-channel SLO SessionIndex hardening), `OID4VCIssuerWellKnownProvider` (drops self-signed root from `x5c`, adds `issuer_info`), `ClientRegistrationTokenUtils` (refactor), `OIDCIdentityProvider` (token-exchange refactor). None change algorithm selection or key types.

### 6.5 — No resolved gaps

- `find … -name "*SignatureProviderFactory.java"`: RS*/PS*/ES*/HS*/EdDSA only — no ML-DSA.
- `find … -name "*CekManagementProviderFactory.java"`: ECDH-ES (+A128/192/256KW) and RSA variants only — no ML-KEM.
- `Generated*/Imported*KeyProviderFactory.java`: AES, ECDH, ECDSA, EdDSA, HMAC, RSA, RSA-ENC — no AKP.
- `JavaAlgorithm.java`, `DefaultKeyProviders.java`, `JWKSServerUtils.java`: no ML-DSA/AKP references.
- `JWKSUtils`: AKP/ML-DSA only absent from `JWK_THUMBPRINT_REQUIRED_MEMBERS`; `FapiConstant`: no ML-DSA; `ClientAsymmetricSignatureVerifierContext`: RSA-only guard; `SsfSignatureAlgorithms.ALLOWED`: RS256 only; `OIDCWellKnownProvider`: static RS256.
- Non-test Java files mentioning `ML-DSA|ML-KEM|AKP` outside the allowed core JWK/oid4vp set: none.

---

## Part 7 — Summary of Required Changes

### Part 7A — Corrections to existing content

| # | Severity | Location | Change required |
|---|----------|----------|-----------------|
| C1 | STALE | HTML lines 1331, 1336, 2214, 2327; gaps doc GAP-6 (lines 114, 119); Domain_43 (11, 13, 17), Domain_11 (47), Domain_13 (75), Domain_2 (236) | `bc-fips` 2.1.2 → **2.1.3** (still LMS-only; gap unchanged). Also GAP-6 `pom.xml` "bouncycastle.bcfips.version=2.1.2" → 2.1.3. |
| C2 | STALE | HTML line 1315; gaps doc GAP-19 (line 368); Domain_42 (13, 17) | `bcprov-jdk18on` 1.85.2 → **1.86**; Quarkus BOM 3.40.0.CR1 → **3.40.1**. |
| C3 | ERROR | HTML line 739 (row 60) | Replace "fetched over the SOAP backchannel by `SAMLEndpoint.java`" with `SAMLIdentityProvider.resolveArtifact()` (SAMLEndpoint calls it and validates the response). |
| C4 | ERROR | gaps doc line 287 (GAP-15) | `core/src/main/java/org/keycloak/common/crypto/CryptoProvider.java` → `common/src/main/java/org/keycloak/common/crypto/CryptoProvider.java`. |
| C5 | LINE SHIFTED | HTML line 845; gaps doc GAP-10 (190, 195, 199) | `AbstractOAuth2IdentityProvider` 737 → **969**, 745 → **977**. |
| C6 | LINE SHIFTED | gaps doc GAP-12 (228, 233) | `DefaultTokenManager` 233 → **240**. |
| C7 | LINE SHIFTED | gaps doc GAP-20 (387) | `KeyUse.ENC` lookup lines 139/144 → **144/150**. |
| C8 | LINE SHIFTED | gaps doc GAP-9 (171, 176); Domain_18 (62, 144) | `SamlService` 992 → **1012**. |
| C9 | LINE SHIFTED | gaps doc GAP-21 (401) | `UserInfoEndpoint` 341, 379–380 → **348, 386–387**. |
| C10 | LINE SHIFTED (pre-existing) | `domains/Domain_21_OIDC_IdP_Broker.md` lines 18, 78, 107 | Cites 697 → **977**. |

### Part 7B — Additions of new content

| # | Severity | Action |
|---|----------|--------|
| A1 | NEW | Extend GAP-31 (description + resolution criteria) to cover `MdocDeviceKey.fromProofJwk()` (no AKP → COSE_Key) in addition to the `MdocAlgorithm` ML-DSA entry. |
| A2 | NEW | Add `MdocCose.java`, `MdocCredential.java`, `MdocDeviceKey.java` to the existing mdoc "Files Evaluated"/domain-24 text. |
| A3 | NEW | Add `RSAWrapperAlgorithmProvider.java` to "Files Evaluated" (RSA-only hardening wrapper, no PQC gap). |
| A4 | NEW (optional) | Add the four OID4VP mapper files and `ClaimVerifier.java` to "Files Evaluated" as non-crypto. |

### Part 7C — Resolved items

None. All 31 GAPs remain open; no status badge should change; the Status Distribution table (27 / 16 / 12 / 5 / 3 = 63) is correct.

### Part 7D — Prompt maintenance (for `prompts/REASSESS_PQC_READINESS_PROMPT.md`, not the HTML)

- Section A row for `DIDUtils.java` is dead: no such file exists in the Keycloak tree and nothing in the HTML/domains/gaps references it. Remove or replace.
- Section A row for `DefaultSamlArtifactResolver.java` says the SOAP fetch path is in `SAMLEndpoint.java`; it is in `SAMLIdentityProvider.java`.
- Section A row for the Maven Enforcer says the plugin is "present in `pom.xml`"; the root `pom.xml` only defines version properties (`version.enforcer.plugin`, `extra.enforcer.rules.version`). The "no bcprov minimum rule" conclusion stands.
- Section B3 grep `quarkus.version|quarkus.platform` finds the Quarkus pin but not the resulting `bcprov` version — it must be read from the BOM in `~/.m2/.../quarkus-bom/<version>/`.
- Section B2 should add `UserInfoEndpoint` (GAP-21), `DefaultTokenManager.decodeClientJWT` (GAP-20) and `AbstractOAuth2IdentityProvider` HS256 line (GAP-10) greps; GAP-21 was silently drifting.
- C1 template says "all 30 GAPs" / "62+ rows"; now 31 GAPs / 63 rows.

---

*Review conducted against Keycloak `main` @ `eb3ac9a4ba` on 2026-10-17. `pqc_overview.html` and all domain/gap markdown files were left unmodified, pending reviewer approval.*
