# PQC Overview — Complete Accuracy Review

**Date of review:** 2026-10-30  
**Keycloak branch:** main (HEAD: `9de75aacac`, committed 2026-10-30 08:38 UTC; local clone on `main`, clean working tree, no `git fetch` run)  
**Previous review HEAD:** `f82c6402ba` (2026-10-29) — 11 commits behind current HEAD  
**Scope:** Every claim in every row, every GAP, and the "Files Evaluated" section  
**DOMAIN_COUNT:** 63 (`grep -c 'class="num"' pqc_overview.html`)  
**GAP_LIST:** GAP-1 … GAP-32 (32 GAPs — GAP-32 Kerberos/SPNEGO is new since the 2026-10-29 review)  
**FILE_LIST:** 321 distinct `*.java` tokens in the HTML (316 at the previous review; +5 non-crypto mdoc helper classes added on 29-Oct)

**Method note / evidence levels.** Pre-flight, Section A, B1, B2, B3, B4 and the Part 5 recount were all executed against the live source. Part 2 file existence was checked programmatically for every `.java` token (`git ls-files`, basenames, wildcard expanded). Algorithm / PQC-state claims were re-verified through the B2/B3 greps; rows whose prose is purely architectural (SPI-driven, "will inherit ML-DSA") were not re-derived line by line and are marked "files ✅, no contradicting change at HEAD". The full `git diff f82c6402ba..HEAD` (57 files) was read for anything touching crypto, JWK/JOSE, key providers, SAML, OID4VC/OID4VP, SD-JWT, mdoc or Kerberos. Library JARs in `~/.m2` were inspected directly for the `bc-fips` and `bcprov` claims.

**Headline result.** No gap was resolved, no status badge changes, and the Status Distribution table (27 / 16 / 12 / 5 / 3 = 63) is correct. All five corrections requested by the 2026-10-29 report (BOM label, `bc-fips` 2.1.3, GAP-22 lines, GAP-27 lines, mdoc helpers) are now applied in the docs. The only new finding is a minor imprecision in the **new GAP-32** line reference (`KerberosJdkProvider` "line 120" is the `SunJDKProvider` class declaration; the `Krb5LoginModule` entries are at 138/154 and 203/230). Upstream moved by 11 commits with no PQC-relevant change. Errors: 0 · Stale: 0 · Line shifted: 1 (GAP-32) · New domains: 0 · Resolved gaps: 0.

---

## Part 1 — Errors and Inaccuracies

### 1.1 — GAP-32: `KerberosJdkProvider` "line 120" does not point at the `Krb5LoginModule` configuration

**Severity:** LINE SHIFTED (documentation imprecision; crypto/claim unchanged — Keycloak still has no enctype handling)  
**Evidence:**
- `SPNEGOAuthenticator.establishContext()` — declared at line **160** ✅ (`GSSManager.getInstance()` 161, `KRB5_OID`/`SPNEGO_OID` 163, `acceptSecContext` 168).
- `KerberosJdkProvider.java`: line 120 is `private static class SunJDKProvider extends KerberosJdkProvider`. `createJaasConfigurationForServer` is at 124 and the `"com.sun.security.auth.module.Krb5LoginModule"` entries are at **138** (server) and **154** (username/password); the IBM variants (`com.ibm.security.auth.module.Krb5LoginModule`) are at 203 and 230.
- Enctype sweep (`enctype|rc4|aes128|aes256|des3|permitted_enctypes|default_tkt_enctypes`) over `federation/kerberos`, `SpnegoAuthenticator.java`, `KerberosJdkProvider.java`: **zero matches** ✅. B3 sweep (`enctype|pkinit`) over `federation/kerberos` + `KerberosJdkProvider.java`: **zero matches** ✅. The GAP-32 premise and LOW/external severity are unchanged.

**Documented as line 120:** `gaps/Github_issue_requirements_for_gaps.md` line 616 (GAP-32 Affected File(s)). The HTML GAP-32 rows (1857, 2475–2476) cite no line numbers.  
**Suggested wording:** "JAAS `Krb5LoginModule` configuration, line 120" → "`SunJDKProvider.createJaasConfigurationForServer()`, lines 124–139" (or drop the number). Note that this value has never been exact; the prompt's "~120" is approximate by design, so this is a low-priority tidy-up, not a regression.

### 1.2 — Items inside the ±3 tolerance (✅, flag for the next edit pass)

| Location | Documented | Actual | Shift |
|----------|-----------|--------|-------|
| `AbstractOAuth2IdentityProvider.java` `Algorithm.HS256` fallback (GAP-10) | 969 | **972** | +3 ✅ |
| `AbstractOAuth2IdentityProvider.java` `Algorithm.RS256` fallback (GAP-10; HTML 845, `Domain_21` 18/41/70/78/107, gaps doc 196/201/205) | 977 | **980** | +3 ✅ |
| `SamlService.java` `Algorithm.RS256` (GAP-9) | 1012 | **1013** | +1 ✅ |
| `DescriptionConverter.java` RS256 special-case (GAP-11) | 416 | 417 | +1 ✅ |
| `JavaKeystoreKeyProviderFactory.mergedAlgorithmProperties()` (GAP-16) | 189–202 | 191–204 | +2 ✅ |
| `OID4VPIdentityProvider.java` TODO (row 62) | 81 | 83 | +2 ✅ |

`AbstractOAuth2IdentityProvider` is unchanged from the previous run (still +3); the next added line will push it past tolerance. Recommend citing it by the `Algorithm.RS256` fallback expression rather than by line number when it is next edited.

### 1.3 — Previously flagged items now confirmed corrected

- Quarkus BOM label **3.40.1.1** in `Domain_42` line 13 and gaps doc GAP-19 (line 374) ✅; `~/.m2/.../quarkus-bom-3.40.1.1.pom` line 9514–9515 still declares `bcprov-jdk18on` **1.86**.
- `bc-fips` **2.1.3** — no `2.1.2` reference remains outside historical `findings/*.md` files (grep across all `*.md`/`*.html` in the workspace) ✅.
- GAP-22 / Domain 12 `[TODO]` references now read **423–424** (HTML 448/451, gaps doc 426/431/435, `Domain_12`) ✅; Keycloak line 423 is the first `[TODO]` ✅.
- GAP-27 / Domain 49 now read **126, 288** (gaps doc 521/526, `Domain_49` 13/21) ✅.
- mdoc non-crypto helper classes (`CborUtil`, `MdocException`, `MdocIssuerSignedDocument`, `MdocNamespacedClaims`, `MdocValidityInfo`) are now cited in the HTML ✅.
- JDK hybrid-TLS (`X25519MLKEM768`) note present in `Domain_43` line 22 ✅.

---

## Part 2 — Domain-by-Domain Verification Table

Files = every `*.java` token in the row resolved against non-test Keycloak sources. All 63 rows: **files ✅**. The only two of the 321 tokens that do not resolve by basename are explained: the glob `EcdhEsA*KwCekManagementProviderFactory.java` (extracted as `KwCekManagementProviderFactory.java`; matches `EcdhEsA128/192/256KwCekManagementProviderFactory.java` in `services/src/main/java/org/keycloak/crypto/`) → ✅, and the annotated rename `AttestationX509CertificateValidator.java` (HTML line 2127 "Previously cited as…") → ✅ RENAMED, replacement `AttestationProofValidator.java` exists in `services/.../oid4vc/issuance/keybinding/`. No file is MISSING or MOVED. Status = primary (first) badge in the HTML.

| Row | Domain | Files exist? | Algorithm claims / state verified | PQC state | Notes |
|-----|--------|--------------|-----------------------------------|-----------|-------|
| 1 | Access Token / ID Token Signing | ✅ | ✅ `ML_DSA_44/65/87` (`Algorithm.java` 58–60) + `KeyType.AKP` (25) present; no ML-DSA factory; GAP-12 fallback `DefaultTokenManager` 240 | PARTIAL | CONFIRMED |
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
| 11 | X.509 mTLS Client Authentication | ✅ | ✅ | SAFE | No change |
| 12 | Attestation-Based Client Authentication | ✅ | ✅ `[TODO]` alg enforcement at 423–424 (doc now 423–424 ✅) | PARTIAL | GAP-22; `AttestationProofValidator` present |
| 13 | X.509 Browser Authentication Flow | ✅ | ✅ | SAFE | |
| 14 | WebAuthn / Passkeys (FIDO2) | ✅ | ✅ | EXTERNAL DEPENDENCY | |
| 15 | DPoP | ✅ | ✅ | PARTIAL | |
| 16 | CIBA Signed Backchannel Auth Request | ✅ | ✅ `SecureCibaAuthenticationRequestSigningAlgorithmExecutorFactory` line 45 uses `FapiConstant.ALLOWED_ALGORITHMS` | PARTIAL | |
| 57 | Token Exchange — Verification & Signing | ✅ | ✅ | PENDING PROVIDERS | |
| 58 | Device Authorization Grant — Token Signing | ✅ | ✅ | PENDING PROVIDERS | |
| 17 | JWT Authorization Grant — Assertion Verification | ✅ | ✅ | PENDING PROVIDERS | |
| 18 | SAML Assertion & Document Signing | ✅ | ✅ RS256 at `SAMLIdentityProvider` 416/507, `SamlProtocol` 544, `SamlService` 1013 | BLOCKED | `XMLTimeUtil`/`PicketLinkLogger` changes are logging/null-guard only |
| 19 | SAML Assertion Encryption | ✅ | ✅ `XMLEncryptionUtil` RSA_OAEP_11 default (77), RSA_v1dot5 (431); `SAMLEncryptionAlgorithms` = RSA_OAEP, RSA1_5 only | BLOCKED | |
| 20 | SAML IdP Broker — SP Metadata & Federation Signing | ✅ | ✅ RS256 at 416/507 | BLOCKED | |
| 55 | Default Trust Identity Provider | ✅ | ✅ | PENDING PROVIDERS | |
| 59 | SAML Metadata Public Key Loader | ✅ | ✅ | SAFE | |
| 60 | SAML Artifact Resolution | ✅ | ✅ `DefaultSamlArtifactResolver.resolveArtifact()` line 32 (storage/routing only); SOAP `Soap.createMessage()` in `SAMLIdentityProvider` 575 (`resolveArtifact` 554); `SAMLEndpoint` 448 only calls it | BLOCKED | |
| 61 | SAML2Signature — Hardcoded RSA-SHA1 Default | ✅ | ✅ `RSA_SHA1` line 55, `DigestMethod.SHA1` line 57 | BLOCKED | |
| 62 | OID4VP IdP — Request Signing & Response Encryption | ✅ | ✅ `ACCEPTED_ALGORITHMS` 85; ES256 key lookups 210–211; `CURVE_SEC` 41; `KEY_MANAGEMENT_ALG` 38; TODO 83 | BLOCKED | GAP-24/25; all 12 oid4vp files + 4 mappers present |
| 63 | OID4VCI Credential Response Encryption | ✅ | ✅ no ML-KEM CEK provider | BLOCKED | |
| 21 | OIDC IdP — Token Sig Verification & JWE Decryption | ✅ | ✅ JWE key lookup `OIDCIdentityProvider` 675/677; outbound fallbacks 972/980 | PARTIAL | GAP-10 (+3, tolerated) |
| 22 | Kubernetes Identity Provider | ✅ | ✅ | PENDING PROVIDERS | |
| 23 | SPIFFE / SVID Identity Provider | ✅ | ✅ | PENDING PROVIDERS | |
| 56 | SD-JWT Issuer Signing & Key Binding Verification | ✅ | ✅ | PENDING PROVIDERS | |
| 24 | JWT-VC / SD-JWT Credential Signing | ✅ | ✅ | PENDING PROVIDERS | GAP-31 mdoc sibling |
| 25 | LD-Proof Credential Signing | ✅ | ✅ `Ed255192018Suite.PROOF_TYPE = "Ed25519Signature2018"` line 67 | BLOCKED | GAP-7 |
| 26 | OID4VC Key Binding — JWT Proof Validation | ✅ | ✅ | PENDING PROVIDERS | |
| 27 | OID4VC c_nonce JWT Signing | ✅ | ✅ ES256 281, RS256 287 | BLOCKED | GAP-18 |
| 28 | Generated RSA Signing Key Provider | ✅ | ✅ no `GeneratedAKP*` | BLOCKED | |
| 29 | Generated RSA Encryption Key Provider | ✅ | ✅ | BLOCKED | |
| 30 | Generated ECDSA Signing Key Provider | ✅ | ✅ | BLOCKED | |
| 31 | Generated EdDSA Signing Key Provider | ✅ | ✅ | BLOCKED | |
| 32 | Generated ECDH Encryption Key Provider | ✅ | ✅ | BLOCKED | |
| 33 | Imported RSA Signing Key Provider | ✅ | ✅ no `ImportedAKP*` | BLOCKED | |
| 34 | Java Keystore Key Provider | ✅ | ✅ `loadRSAKey` 217 / `loadECKey` 214 / OKP 205; no AKP | BLOCKED | GAP-5 |
| 35 | Dynamic Client Registration Tokens | ✅ | ✅ `DescriptionConverter` 417; `INTERNAL_SIGNATURE_ALGORITHM = HS512` (`Constants` 69) | SAFE | |
| 36 | OIDC Well-Known — Algorithm Advertisement | ✅ | ✅ `DEFAULT_CLIENT_AUTH_SIGNING_ALG_VALUES_SUPPORTED = list(RS256)` line 85 | PARTIAL | GAP-23 |
| 37 | Security Event Token (SET) Signing | ✅ | ✅ `SsfSignatureAlgorithms.ALLOWED = Set.of(RS256)` line 31, `DEFAULT = RS256` line 38 | EXTERNAL DEPENDENCY | CAEP-gated; `SsfScopes` change (client-scope protocol) is unrelated |
| 38 | kcadm.sh / kcreg.sh — private_key_jwt | ✅ | ✅ `.rsa256(` line 213 | BLOCKED | GAP-13 |
| 39 | JGroups ASYM_ENCRYPT (non-default / test) | ✅ (test resource) | ✅ | SAFE | GAP-14 |
| 40 | Organisation Invitation Token Verification | ✅ | ✅ | PENDING PROVIDERS | |
| 41 | FAPI Algorithm Allowlist Enforcement | ✅ | ✅ `FapiConstant.ALLOWED_ALGORITHMS` line 30: PS*/ES* only | BLOCKED | GAP-8, GAP-17 (factory lines 42/45 ✅) |
| 42 | CryptoProvider SPI — BouncyCastle Default | ✅ | ✅ `bcprov-jdk18on` 1.86 via Quarkus BOM 3.40.1.1; 121 `mldsa` + 29 `asymmetric/mlkem` JAR entries | PARTIAL | |
| 43 | CryptoProvider SPI — FIPS (BC-FIPS) | ✅ | ✅ `bc-fips` 2.1.3 (`pom.xml` 78); `crypto/internal/pqc` = `lms/` + generic signer/encryptor interfaces; 0 `mldsa/mlkem` JAR entries | BLOCKED | GAP-6 |
| 44 | CryptoProvider SPI — WildFly Elytron | ✅ | ✅ | EXTERNAL DEPENDENCY | |
| 45 | JWK Serialisation & Thumbprint | ✅ | ✅ `JWK_THUMBPRINT_REQUIRED_MEMBERS` (53–58) RSA/EC/OKP only; `JWKBuilder.akp()` 86, `JWKParser` AKP branch 84 | PARTIAL | GAP-1 |
| 46 | Default Realm Key Providers (Bootstrap) | ✅ | ✅ no AKP/ML-DSA in `DefaultKeyProviders` | BLOCKED | GAP-15/17 |
| 47 | Client Public Key Loader | ✅ | ✅ | PARTIAL | |
| 48 | Realm JWKS Endpoint — Key Serialisation | ✅ | ✅ `toJwk()` (50): RSA 59 / EC 61 / OKP 63; no AKP | BLOCKED | GAP-26 |
| 49 | Admin API — Client Certificate & Keypair Generation | ✅ | ✅ `generateKeyPairCertificate` 126 / 288; `KeycloakModelUtils` 247 `generateRsaKeyPair` | BLOCKED | GAP-27 |
| 50 | Client SDK — JWT Client Credentials Provider | ✅ | ✅ switch RSA 77 / EC 83 / OKP 89, default throw 96; no AKP | PARTIAL | GAP-28 |
| 51 | Client SDK — DPoP Proof Generation | ✅ | ✅ `generateRsaSignedDPoPProof` 50, TODO 49 | PARTIAL | GAP-29 |
| 52 | Docker Registry — Self-Signed Cert Generation | ✅ | ✅ `getKeyPairGen(KeyType.RSA)` 29, `initialize(2048, …)` 30 | BLOCKED | |
| 53 | Client Asymmetric Signature Verifier Context | ✅ | ✅ `Key Type is not RSA` guard lines 36–37 | BLOCKED | GAP-30 — dual-badge row (BLOCKED + PENDING PROVIDERS) |

---

## Part 3 — GAP Reference Verification

Cross-referenced against `gaps/Github_issue_requirements_for_gaps.md`. All 49 `…/src/main/java/…` paths cited in the gaps doc exist in Keycloak. Summary counts in the gaps doc (HIGH 11 / MEDIUM 16 / LOW 5 = 32; Severity field tally HIGH 11, MEDIUM 16, LOW 3, LOW (EXTERNAL) 2) are internally consistent.

| GAP | Title | Files correct? | Line numbers | Notes |
|-----|-------|----------------|--------------|-------|
| GAP-1 | JWK Thumbprint — AKP keys | ✅ | ✅ `JWKSUtils` 53–58 | CONFIRMED |
| GAP-2 | SAML Signing — no PQC entries | ✅ | ✅ `SignatureAlgorithm.java` 29–33 (RSA_SHA1/256/256_MGF1/512/512_MGF1) | CONFIRMED |
| GAP-3 | SAML Encryption — no ML-KEM | ✅ | ✅ `XMLEncryptionUtil` 77, 186, 193, 196, 431 | CONFIRMED |
| GAP-4 | JWE Key Mgmt — no ML-KEM CEK | ✅ | n/a | CONFIRMED — 7 CEK factories, no ML-KEM |
| GAP-5 | JavaKeystoreKeyProvider — ML-DSA import | ✅ | ✅ 205 / 214 / 223 | CONFIRMED |
| GAP-6 | BC-FIPS — no ML-DSA/ML-KEM | ✅ (`pom.xml` 78) | ✅ | CONFIRMED at 2.1.3 (0 `mldsa/mlkem` entries) |
| GAP-7 | LD-Proof suite hardcoded Ed25519 | ✅ | ✅ 67 | CONFIRMED |
| GAP-8 | FAPI allowlist blocks ML-DSA | ✅ | ✅ 30 | CONFIRMED |
| GAP-9 | SAML hardcoded RS256 | ✅ | ✅ 416 / 507 / 544 / 1013 (doc 1012, +1) | CONFIRMED |
| GAP-10 | IdP broker RS256/HS256 fallback | ✅ | ✅ 972 / 980 (doc 969 / 977, +3) | CONFIRMED (edge of tolerance) |
| GAP-11 | DCR RS256 special-case | ✅ | ✅ 417 (doc 416, +1) | CONFIRMED |
| GAP-12 | `DEFAULT_SIGNATURE_ALGORITHM` fallback | ✅ | ✅ `Constants` 68; `DefaultTokenManager` 240 | CONFIRMED |
| GAP-13 | AuthUtil `.rsa256()` | ✅ | ✅ 213; `JWTClientCredentialsProvider` 62 | CONFIRMED |
| GAP-14 | JGroups ASYM_ENCRYPT | ✅ | n/a | CONFIRMED |
| GAP-15 | Realm key mgmt — no ML-DSA keygen | ✅ | n/a | CONFIRMED |
| GAP-16 | JavaKeystoreKeyProviderFactory — ML-DSA excluded from UI | ✅ | ✅ `mergedAlgorithmProperties()` 191–204 (doc 189–202, +2) | CONFIRMED |
| GAP-17 | FAPI/CIBA executor factories — UI lists | ✅ | ✅ factory lines 42 / 45 | CONFIRMED; external/spec-gated (no domain cross-ref required) |
| GAP-18 | JwtCNonceHandler ES256→RS256 | ✅ | ✅ 281 / 287 | CONFIRMED |
| GAP-19 | No enforcer minimum for bcprov | ✅ | n/a | CONFIRMED — root `pom.xml` lines 200–201 only define `version.enforcer.plugin` / `extra.enforcer.rules.version`; BOM label 3.40.1.1 ✅ |
| GAP-20 | JAR encrypted request — no ML-KEM inbound | ✅ | ✅ `decodeClientJWT` 122; ENC lookup 144 / 150 | CONFIRMED |
| GAP-21 | UserInfo per-client signing/encryption | ✅ | ✅ 348 / 386–387 | CONFIRMED |
| GAP-22 | Attestation-based client auth `[TODO]` | ✅ | ✅ 423–424 (doc 423–424) | CONFIRMED |
| GAP-23 | OIDC discovery hardcoded RS256 | ✅ | ✅ 85 | CONFIRMED |
| GAP-24 | OID4VP hardcoded ES256 / `ACCEPTED_ALGORITHMS` | ✅ | ✅ 85; 210–211 | CONFIRMED |
| GAP-25 | OID4VP ECDH-ES / secp256r1 | ✅ | ✅ EphemeralKey 41; ResponseEncryption 38 | CONFIRMED; external/spec-gated — "Domains: —" intentional |
| GAP-26 | `JWKSServerUtils.toJwk()` no AKP | ✅ | ✅ 59–63 | CONFIRMED |
| GAP-27 | Admin API keypair gen hardcodes RSA | ✅ | ✅ 126 / 288 (doc 126 / 288) | CONFIRMED |
| GAP-28 | `JWTClientCredentialsProvider` missing AKP | ✅ | ✅ 77–96 | CONFIRMED |
| GAP-29 | `DPoPGenerator` no ML-DSA path | ✅ | ✅ 49–50 | CONFIRMED |
| GAP-30 | Client verifier RSA guard | ✅ | ✅ 36–37 | CONFIRMED |
| GAP-31 | `MdocAlgorithm` / `MdocDeviceKey` no ML-DSA/AKP | ✅ | ✅ `MdocAlgorithm` enum ends ES512 (41) / EDDSA (42); `MdocDeviceKey` switch EC 60 / RSA 63 / OKP 66 | CONFIRMED |
| GAP-32 | Kerberos / SPNEGO — enctypes delegated to JDK | ✅ (all 4 cited files exist) | ⚠️ `establishContext()` 160 ✅; `KerberosJdkProvider` "120" → 124 / 138 / 154 (§1.1) | **LINE SHIFTED** (minor). Enctype/PKINIT sweeps: zero matches → premise CONFIRMED; LOW/external unchanged; "Domains: —" intentional |

---

## Part 4 — "Files Evaluated" Section Verification

All `.java` filenames cited in the HTML (including the "Files Evaluated" list, HTML line 1870 onward) resolve against `git ls-files` (non-test) except the two explained tokens in Part 2. The five mdoc helper classes added on 29-Oct are present in `core/.../mdoc` and the reasons given for exclusion remain valid. Gaps-doc/Domain-43 cross-checks unchanged.

Observation (not an error): the **Kerberos** `.java` files (see Part 6.2) are not in "Files Evaluated"; GAP-32 intentionally has no numbered domain, and the HTML cites them only indirectly (GAP table rows 1857/2475–2476 name `Krb5LoginModule` but no `.java` filename). The gaps doc GAP-32 entry carries the file list.

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

Current HTML Status Distribution table (HTML line 2183, "All 63 Domains"):
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

- Non-test files mentioning `ML-DSA|ML-KEM|AKP` outside the allowed core JWK / oid4vp set: **none**. `JavaAlgorithm.java`, `DefaultKeyProviders.java`, `JWKSServerUtils.java`: no ML-DSA/AKP.
- `*SignatureProviderFactory.java`: RS256/384/512, PS256/384/512, ES256/384/512, EdDSA, HS256/384/512 — no ML-DSA.
- `*CekManagementProviderFactory.java`: ECDH-ES (+A128/192/256KW) and RSA (RSA1_5, OAEP, OAEP-256) — no ML-KEM.
- `Generated*/Imported*KeyProviderFactory.java`: Aes, Ecdh, Ecdsa, Eddsa, Hmac, RsaEnc, Rsa; Imported RsaEnc, Rsa — no AKP; no `GeneratedAKP*`.
- `JWKSUtils` thumbprint map RSA/EC/OKP only; `FapiConstant.ALLOWED_ALGORITHMS` no ML-DSA; `ClientAsymmetricSignatureVerifierContext` RSA-only guard; `SsfSignatureAlgorithms.ALLOWED` RS256 only; `OIDCWellKnownProvider` static RS256; OID4VP `ACCEPTED_ALGORITHMS = List.of(ES256)`.
- Libraries: `quarkus.version = quarkus.build.version = 3.40.1.1` (`pom.xml` 43/44, unchanged); `bcprov-jdk18on` 1.86 (BOM pom 9514–9515; 121 `mldsa` JAR entries, 29 `asymmetric/mlkem` entries); `bouncycastle.bcfips.version = 2.1.3` (`pom.xml` 78; 0 `mldsa/mlkem` entries).
- B4 discovery: oid4vp = 12 files + 4 mappers (all cited); IdPs with signing/verification: JWTAuthorizationGrant, Kubernetes, OID4VP, AbstractOAuth2/OIDC, SAML, Spiffe (+ test-framework `OAuthIdentityProvider`, test infra only) — all covered; sdjwt/oid4vc signing files — all previously covered. No renames/deletions/additions of crypto-relevant `.java` files since last review.

### 6.2 — Kerberos file sweep (GAP-32): no new Kerberos files; HTML "Files Evaluated" has no Kerberos entry

`kerberos/*.java` (non-test): `CommonKerberosConfig`, `KerberosConfig`, `KerberosFederationProvider`, `KerberosFederationProviderFactory`, `KerberosPrincipal`, `ReadOnlyKerberosUserModelDelegate`, `impl/KerberosServerSubjectAuthenticator`, `impl/KerberosUsernamePasswordAuthenticator`, `impl/SPNEGOAuthenticator`, plus `LDAPProviderKerberosConfig`. None contains cipher/enctype/algorithm/key-type logic (grep for `cipher|encrypt|algorithm|KeyType|signature|digest|rc4|aes|des` returned only an unrelated `EditMode` hit). No file added or changed in this area since 29-Oct (`git log --since` empty). Optional: add a single "Kerberos / SPNEGO (`federation/kerberos`, `KerberosJdkProvider`)" line to "Files Evaluated" pointing at GAP-32 so the sweep result is traceable from the HTML. Not required.

### 6.3 — Upstream changes since `f82c6402ba` reviewed with no PQC impact (11 commits)

- Liquibase/manual-migration empty-database lock-table fix (`LiquibaseJpaUpdaterProvider`, `QuarkusJpaUpdaterProvider`, dist tests, `migrate_db.adoc`).
- `XMLTimeUtil`/`PicketLinkLogger`/`DefaultPicketLinkLogger` (`5843d14177`): null-argument guard and logger signature change for SAML time parsing — no algorithm or key handling.
- `AuthenticationManager` (`32e4b96c4e`): logout-session `LOGGING_OUT` action guard for admin session deletion with client credentials — no crypto.
- `SsfScopes` / SSF client-scope protocol set to `openid-connect` (`124470a6d7`) — not related to `SsfSignatureAlgorithms` (still RS256-only).
- `FreeMarkerLoginFormsProvider` realm display-name placeholder resolution; Admin UI/Account UI E2E workflow and test changes; removal of `misc/test-stability` module and stability workflows; CI/operator docs. No crypto, JOSE, SAML-algorithm, OID4VC/VP, mdoc, key-provider or Kerberos code touched.

---

## Part 7 — Summary of Required Changes

### Part 7A — Corrections to existing content

| # | Severity | Location | Change required |
|---|----------|----------|-----------------|
| C1 | LINE SHIFTED (minor, optional) | `gaps/Github_issue_requirements_for_gaps.md` line 616 (GAP-32) | "`KerberosJdkProvider.java` … `Krb5LoginModule` configuration, line 120" → `SunJDKProvider.createJaasConfigurationForServer()` (lines 124–139; `Krb5LoginModule` entries at 138 / 154, IBM variants 203 / 230), or drop the line number. |
| C2 | Housekeeping | `pqc_overview.html` line 2496; `README.md` line 48 | Replace "Last updated" with 30 October 2026 and Keycloak HEAD `9de75aacac`; state "verified all 63 domains and 32 GAPs" (the current text says 31 GAPs @ `f82c6402ba`). |

No STALE or ERROR items. Tolerated drift (§1.2) needs no action; GAP-10's `AbstractOAuth2IdentityProvider` references (HTML 845, `Domain_21`, gaps doc 196/201/205) are the next likely to fall out of tolerance.

### Part 7B — Additions of new content

| # | Severity | Action |
|---|----------|--------|
| A1 | NEW (optional) | Add a Kerberos/SPNEGO entry (federation/kerberos + `KerberosJdkProvider`, "no cryptography in Keycloak; see GAP-32") to the HTML "Files Evaluated" table so the B4 Kerberos sweep is traceable (Part 6.2). |

### Part 7C — Resolved items

None. All 32 GAPs remain open; no status badge should change; the Status Distribution table (27 / 16 / 12 / 5 / 3 = 63) is correct.

### Part 7D — Prompt maintenance (for `prompts/REASSESS_PQC_READINESS_PROMPT.md`, not the HTML)

- B2 comments for GAP-22 (line ~266) and GAP-27 (line ~287) still say "documented 418-419 / 293 … pending doc update"; the docs now say 423–424 and 126 / 288. Update to "documented 423–424" and "documented 126, 288".
- B2 GAP-32 comment: `KerberosJdkProvider Krb5LoginModule ~line 120` should read "`SunJDKProvider` class at 120, `Krb5LoginModule` entries at 138/154 (IBM 203/230)".
- B2 GAP-10: documented lines 969 / 977 versus actual 972 / 980 — note the edge-of-tolerance state, or switch the docs to cite the expression instead of a line number.
- B2 GAP-16 (`JavaKeystoreKeyProviderFactory.mergedAlgorithmProperties()`, documented 189–202, now 191–204) is not in the prompt's B2 list; consider adding it.
- C1 template counts are already updated (63 rows / 32 GAPs) ✅.
- B3 the "all matching files" `find … | xargs grep` pipeline runs over a full checkout (slow, walks `node_modules`); `git ls-files '*.java' | grep -v /test/ | xargs grep -l …` is faster and equivalent for tracked source. Same for B4's `find` sweeps.
- B3 `grep "<artifactId>bcprov-jdk18on</artifactId>"` against the `~/.m2` BOM worked (the 3.40.1.1 BOM is present) ✅.

---

*Review conducted against Keycloak `main` @ `9de75aacac` on 2026-10-30. `pqc_overview.html` and all domain/gap markdown files were left unmodified, pending reviewer approval.*
