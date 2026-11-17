# PQC Overview — Complete Accuracy Review

**Date of review:** 2026-11-17  
**Keycloak branch:** main (HEAD: `eafe81dd79`, committed 2026-11-16; local clone reported "up to date with origin/main" — no `git fetch` was run)  
**Previous review HEAD:** `eb3ac9a4ba` (2026-09-27) — 263 commits behind current HEAD  
**Scope:** Every claim in every row, every GAP, and the "Files Evaluated" section  
**DOMAIN_COUNT:** 63 (`grep -c 'class="num"' pqc_overview.html`)  
**GAP_LIST:** GAP-1 … GAP-31 (31 GAPs)  
**FILE_LIST:** 316 distinct `*.java` tokens in the HTML (167 of them in the "Files Evaluated" section)

**Method note / evidence levels.** Pre-flight, Section A, B1, B2, B3, B4 and the Part 5 recount were all executed against the live source. *File existence* (B1) was checked programmatically against `git ls-files` for every `*.java` token in the HTML. *Line-number and algorithm/PQC-state claims* were re-verified with the B2/B3 greps (every row that cites a line number, hardcoded algorithm or "not yet implemented" state is covered). In addition, the diff `eb3ac9a4ba..HEAD` was inspected for every changed non-test file that is named in the HTML, plus the files they call into. Rows whose prose is purely architectural (SPI-driven, "will inherit ML-DSA") were not re-derived line by line; they are marked "files ✅, no contradicting change at HEAD" unless the diff review below found otherwise. JAR inspection (`bc-fips-2.1.3`, `bcprov-jdk18on-1.86`) was repeated.

**Headline result.** No GAP has been resolved; all 31 remain open and the Status Distribution table (27 / 16 / 12 / 5 / 3 = 63) is still arithmetically correct. The review found **one substantive new technical finding** (the `x5c` certificate-chain path in OID4VC / ABCA / OID4VP converts only RSA and EC leaf keys — it is not "algorithm-agnostic" as the HTML says), **three LINE SHIFTED items** (including a +47 and a +16 shift) and **one minor STALE version note**.

---

## Part 1 — Errors and Inaccuracies

### 1.1 — `X509CertificateChainValidator.toJwk()` / `X5cKeyUtils` accept only RSA and EC leaf keys — HTML calls the x5c path "algorithm-agnostic"

**Severity:** ERROR (documentation accuracy) — and a **NEW** PQC gap candidate (see Part 6.1)  
**Locations in `pqc_overview.html`:**
- Line 2127 (Files Evaluated, `X5cKeyUtils.java`): "…Algorithm-agnostic — delegates certificate parsing to the JVM. No independent PQC gap; covered under row 26."
- Line 2119 (Files Evaluated, `AttestationProofValidator.java`): "…converts the leaf certificate to a JWK via `X5cKeyUtils`. Algorithm parameter is passed in … — no hardcoded algorithm. No independent PQC gap."
- Row 26 (line ~966): "x5c/attestation proof path not fully analysed."
- Row 55 (Default Trust Identity Provider): "No independent hardcoding found… No code change required beyond core ML-DSA provider work."  
**Actual:** `core/src/main/java/org/keycloak/crypto/X509CertificateChainValidator.java` lines 116-126:
```java
public static JWK toJwk(X509Certificate leaf, String algorithm, List<X509Certificate> certificateChain) throws VerificationException {
    PublicKey key = leaf.getPublicKey();
    if (key instanceof RSAPublicKey) { return JWKBuilder.create().algorithm(algorithm).rsa(...); }
    if (key instanceof ECPublicKey)  { return JWKBuilder.create().algorithm(algorithm).ec(...);  }
    throw new VerificationException("Unsupported public key type in certificate: " + key.getClass().getName());
}
```
Any X.509 leaf that is not RSA/EC — **ML-DSA (AKP), and also Ed25519/Ed448 (OKP)** — is rejected with "Unsupported public key type in certificate". `JWKBuilder.akp(PublicKey)` (line 86) has no overload that carries an `x5c` chain either. This single static method is the key-extraction step for **four** flows:

| Flow | Call chain | HTML row |
|------|------------|----------|
| OID4VCI wallet key-binding proof carrying `x5c` | `JwtProofValidator:181-182` → `AttestationValidatorUtil.resolveJwkFromProofX5c` (339) → `X5cKeyUtils.toJwk` → `X509CertificateChainValidator.toJwk` | 26 |
| OID4VCI key-attestation `x5c` | `AttestationValidatorUtil` (337-339) → `X5cKeyUtils` | 26 |
| ABCA attester key via `x5c` header (**new since prior HEAD**, commit `f2d2da56ba8`, 2026-11-05) | `AttestationBasedClientAuthenticator.findAttesterKey` (329-332) → `TrustMaterialResolver.validateX509Chain` → `TrustMaterialIdentityProvider.validateX509Chain` (76) → `X509CertificateChainValidator.validate` → `toJwk` | 12, 55 |
| OID4VP trusted-issuer SD-JWT `x5c` | `OID4VPIdentityProvider.trustedIssuerResolver()` (187) → `TrustMaterialSdJwtIssuerResolver.resolve` (65) → same chain | 62 (and 56 verification) |

**Not a regression:** the RSA/EC-only logic already existed in `X5cKeyUtils` at commit `ee444fb72a3` (2026-07-31) and was moved verbatim into `core` by `0c54fe85f53` (2026-08-25) — both before the previous review HEAD. Prior reviews did not catch it, and the new ABCA `x5c` path (11-05) enlarges the affected surface. The `kid`/JWKS path is unaffected (`JWKSUtils.getKeyWrappersForUse` / `JWKParser` do handle AKP).  
**Required correction:** replace "algorithm-agnostic" in the two Files Evaluated entries and add a new GAP (suggested **GAP-32**, see 6.1). Re-word row 55 so that "handles AKP" applies to the JWKS path only.

### 1.2 — Row 12 / GAP-22 and Domain 12: `[TODO]` algorithm-enforcement line numbers shifted by +16

**Severity:** LINE SHIFTED (crypto code unchanged — the two `[TODO]` comments are still present and the enforcement is still not implemented)  
**Evidence:** `AttestationBasedClientAuthenticator.java` — `// [TODO] The alg JOSE Header Parameter … indicates a registered asymmetric digital signature algorithm` is now line **434** (was 418); `// [TODO] The key contained in the cnf claim … is not a private key` is now **435** (was 419). Cause: commit `f2d2da56ba8` ("Resolve ABCA attester key from x5c JOSE header") added ~16 lines to `findAttesterKey`.  
**Other lines in the same file that moved (cited in `domains/Domain_12_…md`):** `findAttesterKey()` call 396 → **412**; Client Attestation JWT verify 407/412 → **428**; PoP JWT verify 484/489 → **505**; `toPublicKeyWrapper()` 332-339 → **348-356**.  
**Where documented:** `pqc_overview.html` lines 448, 451; `gaps/Github_issue_requirements_for_gaps.md` lines 420, 425, 429; `domains/Domain_12_…md` lines 20, 22, 73-74, 78, 81, 166, 182, 186.  
**Additional prose to revise:** Domain 12 "What already works: `findAttesterKey()` … already handles AKP keys via `JWKSUtils.getKeyWrappersForUse()`" is now true only for the `kid`/JWKS branch; the new `x5c` branch cannot yield an AKP key (see 1.1).

### 1.3 — GAP-9 / Domain 18: `SamlService.java` `Algorithm.RS256` moved 1012 → 1059

**Severity:** LINE SHIFTED (+47; code unchanged: `session.keys().getKeysStream(realm, KeyUse.SIG, Algorithm.RS256)` in `getIDPMetadataDescriptor`)  
**Cause:** SOAP-artifact async handling, `maxSoapMessageRequestSize`/SAML 1.1 flags, and redirect-binding signature changes in `SamlService`.  
**Where documented:** `gaps/Github_issue_requirements_for_gaps.md` line 171 (GAP-9); `domains/Domain_18_SAML_Signing.md` lines 62, 144.  
Other GAP-9 sites: `SAMLIdentityProvider` 416 / 507 ✅ (unchanged), `SamlProtocol` 544 → 545 ✅ (+1).

### 1.4 — GAP-27 / Domain 49: `generateAndGetKeystore()` call to `generateKeyPairCertificate` moved 293 → 288

**Severity:** LINE SHIFTED (−5; code unchanged)  
**Evidence:** `ClientAttributeCertificateResource.java` line **126** (`generate()`) ✅ unchanged; line **288** (`generateAndGetKeystore()`) — was 293. Cause: commit `39e185405bd` ("Fix of getting client keystore without private pem") removed the private-key branch from `getKeystore(...)`. Hard-coded RSA key generation via `KeycloakModelUtils.generateKeyPairCertificate` → `KeyUtils.generateRsaKeyPair(keysize)` (`KeycloakModelUtils.java:247`) is unchanged.  
**Where documented:** `gaps/Github_issue_requirements_for_gaps.md` lines 515, 520; `domains/Domain_49_Admin_API_Cert_Generation.md` line 21.

### 1.5 — Quarkus pin moved 3.40.1 → 3.40.1.2 (bcprov version unchanged at 1.86)

**Severity:** STALE (minor)  
**Evidence:** `pom.xml` line 43 is now `<quarkus.version>3.40.1.2</quarkus.version>`. The `quarkus-bom-3.40.1.2.pom` in `~/.m2` declares `bcprov-jdk18on` **1.86** — identical to 3.40.1 and 3.40.1.1, so the HTML claim "bcprov-jdk18on 1.86 (in use)" is **CONFIRMED**. `bcprov-jdk18on-1.86.jar`: 121 `mldsa` entries, 31 `provider/asymmetric/mlkem` entries.  
**Stale text:** `gaps/Github_issue_requirements_for_gaps.md` line 368 ("via Quarkus BOM 3.40.1"); `domains/Domain_42_CryptoProvider_BouncyCastle.md` line 13 ("Quarkus platform BOM 3.40.1"). The historical "Last updated" footers in `pqc_overview.html` / `README.md` are changelog text and should not be edited.

### 1.6 — Items previously reported and now correct

Verified fixed since the 2026-10-17 report (no action): row 60 now names `SAMLIdentityProvider.resolveArtifact()` (HTML line 739); GAP-10 lines are 969/977 in the gaps doc and 977 in Domain 21; bc-fips is 2.1.3 and bcprov 1.86 everywhere in the HTML; GAP-15 `CryptoProvider.java` path is `common/src/main/java/…`; GAP-20/GAP-21 line numbers match source (122/144/150 and 348/386-387); the mdoc, `RSAWrapperAlgorithmProvider`, OID4VP-mapper and `ClaimVerifier` entries exist in Files Evaluated.

---

## Part 2 — Domain-by-Domain Verification Table

Files = every `*.java` token in the row, resolved against non-test sources via `git ls-files` (scripted). All 63 rows: **files ✅** (the only unresolved token is the glob `EcdhEsA*KwCekManagementProviderFactory.java` in row 2, which resolves to the three `EcdhEsA128/192/256Kw…` factories). Status = primary (first) badge in the HTML (scripted).

| Row | Domain | Files exist? | Algorithm claims / state verified | PQC state | Notes |
|-----|--------|--------------|-----------------------------------|-----------|-------|
| 1 | Access Token / ID Token Signing | ✅ | ✅ `Algorithm.ML_DSA_44/65/87` + `KeyType.AKP` present; no ML-DSA factory; GAP-12 fallback line 240 | PARTIAL | `Constants.DEFAULT_SIGNATURE_ALGORITHM` now line 70 (doc 68, +2 ✅) |
| 2 | ID Token / JARM Encryption (Outbound CEK) | ✅ | ✅ 7 CEK factories, ECDH-ES/RSA only | BLOCKED | |
| 3 | Backchannel Logout Token Signing | ✅ | ✅ `ID`/`LOGOUT` share `ID_TOKEN_SIGNED_RESPONSE_ALG` (DefaultTokenManager 214-216) | PENDING PROVIDERS | |
| 4 | UserInfo — Signed & Encrypted Response | ✅ | ✅ GAP-21 lines 348, 386–387 | BLOCKED | |
| 5 | Introspection — Embedded JWT Response | ✅ | ✅ | PARTIAL | |
| 6 | Token Verification (Identity & Session Tokens) | ✅ | ✅ | PENDING PROVIDERS | |
| 7 | JAR — Signed Request Object Verification | ✅ | ✅ | PENDING PROVIDERS | |
| 8 | JAR — Encrypted Request Object Decryption | ✅ | ✅ no ML-KEM; `decodeClientJWT` 122, ENC lookup 144/150 | BLOCKED | |
| 9 | JARM — Signed Authorization Response | ✅ | ✅ | PENDING PROVIDERS | |
| 54 | Federated JWT Client Authentication | ✅ | ✅ | PENDING PROVIDERS | `FederatedJWTClientAuthenticator` diff = secret-key accessor only |
| 10 | private_key_jwt Client Authentication | ✅ | ✅ | PENDING PROVIDERS | diff = secret-key accessor only |
| 11 | X.509 mTLS Client Authentication | ✅ | ✅ | SAFE | `X509ClientAuthenticator` diff = secret-key accessor only |
| 12 | Attestation-Based Client Authentication | ✅ | ⚠️ `[TODO]` alg enforcement now lines **434/435** (LINE SHIFTED §1.2); new `x5c` branch limited to RSA/EC leaf (§1.1) | PARTIAL | Status stays PARTIAL; prose needs revising |
| 13 | X.509 Browser Authentication Flow | ✅ | ✅ | SAFE | |
| 14 | WebAuthn / Passkeys (FIDO2) | ✅ | ✅ | EXTERNAL DEPENDENCY | `WebAuthnCredentialProvider` modified — no algorithm change |
| 15 | DPoP | ✅ | ✅ | PARTIAL | |
| 16 | CIBA Signed Backchannel Auth Request | ✅ | ✅ | PARTIAL | `CIBAAuthenticationRequest` diff = import removal |
| 57 | Token Exchange — Verification & Signing | ✅ | ✅ | PENDING PROVIDERS | `StandardTokenExchangeProvider` diff = expiry capping only |
| 58 | Device Authorization Grant — Token Signing | ✅ | ✅ | PENDING PROVIDERS | |
| 17 | JWT Authorization Grant — Assertion Verification | ✅ | ✅ | PENDING PROVIDERS | |
| 18 | SAML Assertion & Document Signing | ✅ | ✅ RS256 at SAMLIdentityProvider 416/507, SamlProtocol 545; **SamlService 1059** | BLOCKED | SamlService LINE SHIFTED (§1.3) |
| 19 | SAML Assertion Encryption | ✅ | ✅ `XMLEncryptionUtil` RSA_OAEP_11 (77), RSA_v1dot5 (431); `SAMLEncryptionAlgorithms` = RSA_OAEP, RSA1_5 | BLOCKED | |
| 20 | SAML IdP Broker — SP Metadata & Federation Signing | ✅ | ✅ RS256 at 416/507 | BLOCKED | |
| 55 | Default Trust Identity Provider | ✅ | ⚠️ JWKS path handles AKP ✅; inherited `TrustMaterialIdentityProvider.validateX509Chain` → RSA/EC-only `toJwk` (§1.1) | PENDING PROVIDERS | Candidate for PARTIAL (reviewer decision, see 7C) |
| 59 | SAML Metadata Public Key Loader | ✅ | ✅ | SAFE | |
| 60 | SAML Artifact Resolution | ✅ | ✅ SOAP fetch in `SAMLIdentityProvider.resolveArtifact` (554-585) | BLOCKED | |
| 61 | SAML2Signature — Hardcoded RSA-SHA1 Default | ✅ | ✅ `RSA_SHA1` line 55, `DigestMethod.SHA1` line 57 | BLOCKED | file modified (signature-placement fix only) |
| 62 | OID4VP IdP — Request Signing & Response Encryption | ✅ | ✅ `ACCEPTED_ALGORITHMS` 85; ES256 210–211; `CURVE_SEC` 41; `KEY_MANAGEMENT_ALG` 38; TODO line 83 (doc 81, +2 ✅) | BLOCKED | GAP-24/25 ✅; inbound `x5c` issuer path RSA/EC-only (§1.1) |
| 63 | OID4VCI Credential Response Encryption | ✅ | ✅ no ML-KEM CEK provider | BLOCKED | |
| 21 | OIDC IdP — Token Sig Verification & JWE Decryption | ✅ | ✅ RS256 fallback 980 (doc 977, +3 ✅), HS256 972 (doc 969, +3 ✅) | PARTIAL | |
| 22 | Kubernetes Identity Provider | ✅ | ✅ | PENDING PROVIDERS | |
| 23 | SPIFFE / SVID Identity Provider | ✅ | ✅ | PENDING PROVIDERS | |
| 56 | SD-JWT Issuer Signing & Key Binding Verification | ✅ | ✅ core `sdjwt/*` SPI-driven | PENDING PROVIDERS | OID4VP-side `x5c` issuer resolution is RSA/EC-only (§1.1) |
| 24 | JWT-VC / SD-JWT Credential Signing | ✅ | ✅ | PENDING PROVIDERS | GAP-31 mdoc sibling |
| 25 | LD-Proof Credential Signing | ✅ | ✅ `PROOF_TYPE = "Ed25519Signature2018"` line 67 | BLOCKED | GAP-7 ✅ |
| 26 | OID4VC Key Binding — JWT Proof Validation | ✅ | ⚠️ "x5c path not fully analysed" — now analysed: RSA/EC-only (§1.1) | PENDING PROVIDERS | Candidate for PARTIAL |
| 27 | OID4VC c_nonce JWT Signing | ✅ | ✅ ES256 281, RS256 287 | BLOCKED | GAP-18 ✅ |
| 28 | Generated RSA Signing Key Provider | ✅ | ✅ no `GeneratedAKP*` | BLOCKED | |
| 29 | Generated RSA Encryption Key Provider | ✅ | ✅ | BLOCKED | |
| 30 | Generated ECDSA Signing Key Provider | ✅ | ✅ | BLOCKED | |
| 31 | Generated EdDSA Signing Key Provider | ✅ | ✅ | BLOCKED | |
| 32 | Generated ECDH Encryption Key Provider | ✅ | ✅ | BLOCKED | |
| 33 | Imported RSA Signing Key Provider | ✅ | ✅ no `ImportedAKP*` | BLOCKED | |
| 34 | Java Keystore Key Provider | ✅ | ✅ branches OKP(205)/EC(214)/RSA(223); no AKP | BLOCKED | GAP-5 ✅ |
| 35 | Dynamic Client Registration Tokens | ✅ | ✅ `DescriptionConverter` RS256 special-case line 417; `INTERNAL_SIGNATURE_ALGORITHM = HS512` (Constants 71) | SAFE | |
| 36 | OIDC Well-Known — Algorithm Advertisement | ✅ | ✅ `DEFAULT_CLIENT_AUTH_SIGNING_ALG_VALUES_SUPPORTED = list(RS256)` line 85 | PARTIAL | file modified (PAR gating only) |
| 37 | Security Event Token (SET) Signing | ✅ | ✅ `ALLOWED = Set.of(RS256)` line 31; `DEFAULT = RS256` line 38 | EXTERNAL DEPENDENCY | |
| 38 | kcadm.sh / kcreg.sh — private_key_jwt | ✅ | ✅ `.rsa256(` line 213 | BLOCKED | GAP-13 ✅ |
| 39 | JGroups ASYM_ENCRYPT (non-default / test) | ✅ (`cache-ispn-asym-enc.xml` present) | ✅ | SAFE | GAP-14 |
| 40 | Organisation Invitation Token Verification | ✅ | ✅ | PENDING PROVIDERS | |
| 41 | FAPI Algorithm Allowlist Enforcement | ✅ | ✅ `ALLOWED_ALGORITHMS` line 30: PS*/ES* only | BLOCKED | GAP-8, GAP-17 (factory lines 42/45 ✅) |
| 42 | CryptoProvider SPI — BouncyCastle Default | ✅ | ✅ bcprov 1.86 confirmed (§1.5) | PARTIAL | `BCECDSACryptoProvider`, `BCOCSPProvider` etc. modified — stream-closing / try-with-resources only |
| 43 | CryptoProvider SPI — FIPS (BC-FIPS) | ✅ | ✅ bc-fips 2.1.3; jar has LMS only under `internal/pqc`; 0 ML-DSA/ML-KEM entries | BLOCKED | GAP-6 still open |
| 44 | CryptoProvider SPI — WildFly Elytron | ✅ | ✅ | EXTERNAL DEPENDENCY | |
| 45 | JWK Serialisation & Thumbprint | ✅ | ✅ `JWK_THUMBPRINT_REQUIRED_MEMBERS` (53–58) RSA/EC/OKP only; `JWKBuilder.akp()` (86), `JWKParser` AKP (84), `AKPUtils`, `AKPPublicJWK` present | PARTIAL | GAP-1 ✅ |
| 46 | Default Realm Key Providers (Bootstrap) | ✅ | ✅ no AKP/ML-DSA in `DefaultKeyProviders` | BLOCKED | GAP-15/17 |
| 47 | Client Public Key Loader | ✅ | ✅ | PARTIAL | |
| 48 | Realm JWKS Endpoint — Key Serialisation | ✅ | ✅ `toJwk()` RSA(59)/EC(61)/OKP(63); no AKP | BLOCKED | GAP-26 ✅ |
| 49 | Admin API — Client Certificate & Keypair Generation | ✅ | ✅ `generateKeyPairCertificate` at 126 and **288** (doc 293) | BLOCKED | GAP-27; LINE SHIFTED (§1.4) |
| 50 | Client SDK — JWT Client Credentials Provider | ✅ | ✅ switch RSA(77)/EC(83)/OKP(89), default throw (96); `setupKeyPair` RS256 (62) | PARTIAL | GAP-28 ✅ |
| 51 | Client SDK — DPoP Proof Generation | ✅ | ✅ `generateRsaSignedDPoPProof` 50, TODO 49 | PARTIAL | GAP-29 ✅ |
| 52 | Docker Registry — Self-Signed Cert Generation | ✅ | ✅ `getKeyPairGen(KeyType.RSA)` 29, `initialize(2048)` 30 | BLOCKED | |
| 53 | Client Asymmetric Signature Verifier Context | ✅ | ✅ `Key Type is not RSA` guard 36–37 | BLOCKED | GAP-30 ✅ — **dual-badge row** (BLOCKED + PENDING PROVIDERS) |

---

## Part 3 — GAP Reference Verification

| GAP | Title | Files correct? | Line numbers | Notes |
|-----|-------|----------------|--------------|-------|
| GAP-1 | JWK Thumbprint — AKP keys | ✅ | ✅ `JWKSUtils` 53–58 | CONFIRMED — no AKP entry |
| GAP-2 | SAML Signing — no PQC entries | ✅ | ✅ `SignatureAlgorithm.java` RSA_SHA1/256/256_MGF1/512/512_MGF1 only (29–33) | CONFIRMED |
| GAP-3 | SAML Encryption — no ML-KEM | ✅ | ✅ `XMLEncryptionUtil` 77, 186, 193, 196, 431 | CONFIRMED |
| GAP-4 | JWE Key Mgmt — no ML-KEM CEK | ✅ | n/a | CONFIRMED — 7 CEK factories, ECDH-ES / RSA only |
| GAP-5 | JavaKeystoreKeyProvider — ML-DSA import | ✅ | ✅ 205/214/223 | CONFIRMED |
| GAP-6 | BC-FIPS — no ML-DSA/ML-KEM | ✅ (`pom.xml` line 78 = 2.1.3) | ✅ | CONFIRMED |
| GAP-7 | LD-Proof suite hardcoded Ed25519 | ✅ | ✅ line 67 | CONFIRMED |
| GAP-8 | FAPI allowlist blocks ML-DSA | ✅ | ✅ line 30 | CONFIRMED |
| GAP-9 | SAML hardcoded RS256 | ✅ | SAMLIdentityProvider 416/507 ✅; SamlProtocol 545 ✅; **SamlService 1012 → 1059** | LINE SHIFTED (§1.3) |
| GAP-10 | IdP broker RS256/HS256 fallback | ✅ | 969 → 972, 977 → 980 (+3, within tolerance) | CONFIRMED |
| GAP-11 | DCR RS256 special-case | ✅ | ✅ 417 | CONFIRMED |
| GAP-12 | `DEFAULT_SIGNATURE_ALGORITHM` fallback | ✅ | Constants 68 → 70 (+2 ✅); DefaultTokenManager 240 ✅ | CONFIRMED |
| GAP-13 | AuthUtil `.rsa256()` | ✅ | ✅ 213; `JWTClientCredentialsProvider` 62 & 96 ✅ | CONFIRMED |
| GAP-14 | JGroups ASYM_ENCRYPT | ✅ | n/a (test-only XML present) | CONFIRMED |
| GAP-15 | Realm key mgmt — no ML-DSA keygen | ✅ | n/a | CONFIRMED (path corrected since last review) |
| GAP-16 | JavaKeystoreKeyProviderFactory — ML-DSA excluded from UI | ✅ | `mergedAlgorithmProperties()` now 191–206; gaps doc says 189–202 (+2 ✅); list still AES, EdDSA, ES*, HS*, RS*, RSA-ENC, ECDH | CONFIRMED |
| GAP-17 | FAPI/CIBA executor factories — UI lists | ✅ | ✅ factory lines 42 / 45 | CONFIRMED |
| GAP-18 | JwtCNonceHandler ES256→RS256 | ✅ | ✅ 281 / 287 | CONFIRMED |
| GAP-19 | No enforcer minimum for bcprov | ✅ | n/a | CONFIRMED — `pom.xml` has only `version.enforcer.plugin` / `extra.enforcer.rules.version` (lines 200-201); no bcprov rule. Quarkus BOM note STALE (§1.5) |
| GAP-20 | JAR encrypted request — no ML-KEM inbound | ✅ | ✅ `decodeClientJWT` 122; ENC lookup 144/150 | CONFIRMED |
| GAP-21 | UserInfo per-client signing/encryption | ✅ | ✅ 348, 386–387 | CONFIRMED |
| GAP-22 | Attestation-based client auth `[TODO]` | ✅ | **418–419 → 434–435** | LINE SHIFTED (§1.2); TODO still unimplemented |
| GAP-23 | OIDC discovery hardcoded RS256 | ✅ | ✅ 85 | CONFIRMED |
| GAP-24 | OID4VP hardcoded ES256 / `ACCEPTED_ALGORITHMS` | ✅ | ✅ 85; 210–211; TODO 83 | CONFIRMED |
| GAP-25 | OID4VP ECDH-ES / secp256r1 | ✅ | ✅ EphemeralKey 41; ResponseEncryption 38 | CONFIRMED |
| GAP-26 | `JWKSServerUtils.toJwk()` no AKP | ✅ | ✅ 59–63 | CONFIRMED |
| GAP-27 | Admin API keypair gen hardcodes RSA | ✅ | 126 ✅; **293 → 288** | LINE SHIFTED (§1.4) |
| GAP-28 | `JWTClientCredentialsProvider` missing AKP | ✅ | ✅ 77–96 | CONFIRMED |
| GAP-29 | `DPoPGenerator` no ML-DSA path | ✅ | ✅ 49–50 | CONFIRMED |
| GAP-30 | Client verifier RSA guard | ✅ | ✅ 36–37 | CONFIRMED |
| GAP-31 | `MdocAlgorithm` no ML-DSA COSE mapping | ✅ | n/a — enum lists only RS256 (33), ES256 (39), EdDSA (42) etc.; `MdocDeviceKey.fromProofJwk` still EC/RSA/OKP only (60-69) | CONFIRMED |

Gaps-doc cross-check: no GAP description contradicts the source apart from the line numbers listed above. The **new gap candidate** (§1.1 / 6.1) is not yet tracked — no existing GAP covers x5c leaf-key conversion.

---

## Part 4 — "Files Evaluated" Section Verification

167 distinct `*.java` tokens checked. All exist in non-test sources at HEAD except `AttestationX509CertificateValidator.java`, which the HTML itself annotates as renamed (now `AttestationProofValidator.java`, present). Reasons for exclusion remain valid **except** the two entries whose reasoning depends on "algorithm-agnostic" x5c handling:

- `X5cKeyUtils.java` (HTML line 2127) — **ERROR**, see §1.1 (it delegates to RSA/EC-only `X509CertificateChainValidator.toJwk`).
- `AttestationProofValidator.java` (HTML line 2119) — **ERROR / incomplete**, same reason ("No independent PQC gap").

All other entries (including the mdoc, `RSAWrapperAlgorithmProvider`, OID4VP-mapper and `ClaimVerifier` entries added after the previous review) are accurate.

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

Current HTML Status Distribution table (line 2175):
  BLOCKED:              27  (correct)
  PENDING PROVIDERS:    16  (correct)
  PARTIAL:              12  (correct)
  SAFE:                 5   (correct)
  EXTERNAL DEPENDENCY:  3   (correct)
```

No corrections required to the counts as the HTML stands. If the reviewer accepts the §1.1 recommendation to move rows 26 and 55 from PENDING PROVIDERS to PARTIAL, the table becomes 27 / **14** / **14** / 5 / 3 = 63.

---

## Part 6 — New Content Not in Current HTML

### 6.1 — NEW gap candidate (suggested GAP-32): x5c certificate-chain leaf-key conversion is RSA/EC-only

| Item | Detail |
|------|--------|
| Files | `core/src/main/java/org/keycloak/crypto/X509CertificateChainValidator.java` (`toJwk` lines 116-126, `validate` 61-91); `services/src/main/java/org/keycloak/protocol/oid4vc/issuance/keybinding/X5cKeyUtils.java` (42-48); `server-spi-private/src/main/java/org/keycloak/broker/provider/TrustMaterialIdentityProvider.java` (51-82); `…/TrustMaterialResolver.java` (80-110); `…/TrustMaterialSdJwtIssuerResolver.java` (65) |
| Behaviour | Leaf certificates holding ML-DSA (and Ed25519/Ed448) public keys are rejected with `VerificationException("Unsupported public key type in certificate: …")`. No `JWKBuilder.akp(...)` variant accepts a certificate chain. |
| Affected rows | 12 (ABCA, `x5c` branch added 2026-11-05), 26 (OID4VCI proof / key attestation `x5c`), 55 (`DefaultTrustIdentityProvider` inherits default method), 62/56 (OID4VP trusted-issuer SD-JWT with `x5c`) |
| Resolution criteria | `toJwk` handles `KeyType.AKP` (and OKP) leaf keys, using `AKPUtils`/`JWKBuilder.akp` with an `x5c` chain; accompanying test with an ML-DSA leaf certificate. Depends on BC PKIX path validation of ML-DSA-signed chains (bcprov 1.86 has the algorithms) and on GAP-1 for thumbprints. |
| Note | The crypto-provider `CertPathBuilder` (`CryptoIntegration.getProvider().getCertPathBuilder()`) and `getX509CertFactory()` are provider-routed, so chain building itself is not hardcoded; the blocker is the key-type switch. |

**Suggested state change (reviewer decision):** rows 26 and 55 → PARTIAL (a concrete code gap exists beyond "waiting for providers"). Rows 12 / 62 keep their states.

### 6.2 — Trust-material classes not named in the HTML

`TrustMaterialIdentityProvider`, `TrustMaterialResolver`, `TrustMaterialSdJwtIssuerResolver`, `X509TrustMaterial` and `core/…/crypto/X509CertificateChainValidator.java` are not cited anywhere in `pqc_overview.html` (only `TrustMaterialPublicKeyLoader` is). They should be added to the key-files column of rows 12 / 26 / 55 / 62 (or at least to Files Evaluated) together with the new GAP. They were introduced by commits `a1bd1ab855` (2026-05-25), `ee444fb72a3` (2026-07-31) and `0c54fe85f53` (2026-08-25) — i.e. also missed by earlier reviews.

### 6.3 — B4 discovery sweep results

- OID4VP package: 13 classes + 5 mappers; the only files not already named are `DcqlQuery.java` and `mappers/DcqlQueryContributor.java` (new since last review, commit range; DCQL query building/claim contribution — contain no `Algorithm.`, `RS256`, `ES256`, `SignatureProvider` or `KeyType` references). Optional Files Evaluated additions.
- `*SignatureProviderFactory`: RS*/PS*/ES*/HS*/EdDSA only — no ML-DSA. `*CekManagementProviderFactory`: ECDH-ES (+A128/192/256KW) and RSA variants only — no ML-KEM. `Generated*/Imported*KeyProviderFactory`: AES, ECDH, ECDSA, EdDSA, HMAC, RSA, RSA-ENC, plus JavaKeystore — no AKP.
- Identity providers touching signing: `JWTAuthorizationGrantIdentityProvider`, `KubernetesIdentityProvider`, `OID4VPIdentityProvider`, `AbstractOAuth2IdentityProvider`, `OIDCIdentityProvider`, `SAMLIdentityProvider`, `SpiffeIdentityProvider` — all covered by rows.
- sdjwt / oid4vc / mdoc signing-related files not named in the HTML: `CredentialBuildConfig.java`, `SupportedCredentialConfiguration.java` (both only consume `CredentialSigningAlgorithmResolver` / `MdocAlgorithm.fromJoseAlgorithm`; no independent gap — same conclusion as the previous review).

### 6.4 — Upstream changes reviewed with no PQC impact

- `ea78b1644ac` — ECDSA signature-length validation (`ECDSAAlgorithm`, `ECDSASignatureVerifierContext`, `Client/ServerECDSASignatureVerifierContext`): hardening of existing ECDSA verification; no key-type or algorithm-selection change.
- `02d0c2fcc22` (SAML issuer validation vs configured IdP entity ID), SAML redirect-binding signature now bound to message + relay state (`SamlProtocolUtils`, `SAMLEndpoint`, `SamlService`), `SAML2Signature` signature-placement fix, SOAP-artifact async pool / `maxSoapMessageRequestSize` / SAML 1.1 flag: no algorithm change.
- `39e185405bd` — `ClientAttributeCertificateResource.getKeystore` no longer exports a private key (only certificate); `generateAndGetKeystore` still generates RSA (GAP-27 unchanged).
- `BCECDSACryptoProvider`, `BCOCSPProvider`, `BCPemUtilsProvider`, `BCUserIdentityExtractorProvider` and FIPS equivalents: stream-closing refactor (`a9aad4c7a4c`); `KeystoreUtil` / `TruststoreBuilder`: optional keystore-provider parameter; `FIPS1402Provider`: annotation-only.
- `OIDCWellKnownProvider`: PAR metadata gated on feature flag; `StandardTokenExchangeProvider`: expiry capping; `GoogleIdentityProvider`, `AuthenticationManager`, authenticator `getAdapterConfigurationSecretKeys()` additions: unrelated to algorithms.

### 6.5 — No resolved gaps

- No non-test Java file outside the allowed core-JWK / oid4vp set mentions `ML-DSA|ML-KEM|AKP`.
- `JavaAlgorithm.java`, `DefaultKeyProviders.java`, `JWKSServerUtils.java`, `JWKSUtils.JWK_THUMBPRINT_REQUIRED_MEMBERS`: no ML-DSA/AKP.
- `FapiConstant.ALLOWED_ALGORITHMS`: PS256/384/512, ES256/384/512. `ClientAsymmetricSignatureVerifierContext`: RSA-only guard. `SsfSignatureAlgorithms.ALLOWED`: RS256 only. `OIDCWellKnownProvider`: static RS256. `OID4VPIdentityProvider`: `ACCEPTED_ALGORITHMS = List.of(ES256)`.
- Library state: `bc-fips` 2.1.3 (jar: 0 ML-DSA/ML-KEM entries, LMS only); `bcprov-jdk18on` 1.86 via Quarkus BOM 3.40.1.2.

---

## Part 7 — Summary of Required Changes

### Part 7A — Corrections to existing content

| # | Severity | Location | Change required |
|---|----------|----------|-----------------|
| C1 | ERROR | HTML line 2127 (`X5cKeyUtils.java` entry); HTML line 2119 (`AttestationProofValidator.java` entry) | Remove "Algorithm-agnostic" / "No independent PQC gap"; state that `X509CertificateChainValidator.toJwk` converts only RSA and EC leaf keys and cross-reference the new GAP (§6.1). |
| C2 | ERROR (prose) | HTML row 26 (~line 966), row 55 prose; `domains/Domain_26_…md` line 43, `domains/Domain_55_…md`, `domains/Domain_12_…md` ("What already works") | Replace "x5c/attestation proof path not fully analysed" with the §1.1 finding; qualify "handles AKP" as JWKS/`kid` path only; add `x5c` caveat to row 12. |
| C3 | LINE SHIFTED | HTML lines 448, 451; gaps doc GAP-22 (420, 425, 429); `domains/Domain_12_…md` (20, 22, 73-74, 78, 81, 166, 182, 186) | `[TODO]` 418 / 419 → **434 / 435**; `findAttesterKey` call 396 → 412; verify calls 407 → 428, 484 → 505; `toPublicKeyWrapper` 332-339 → 348-356. |
| C4 | LINE SHIFTED | gaps doc GAP-9 (line 171); `domains/Domain_18_SAML_Signing.md` (62, 144) | `SamlService.java` 1012 → **1059**; `SamlProtocol` 544 → 545 (optional). |
| C5 | LINE SHIFTED | gaps doc GAP-27 (515, 520); `domains/Domain_49_Admin_API_Cert_Generation.md` (21) | `generateAndGetKeystore()` call 293 → **288**. |
| C6 | STALE (minor) | gaps doc GAP-19 (line 368); `domains/Domain_42_CryptoProvider_BouncyCastle.md` (line 13) | "Quarkus BOM 3.40.1" → **3.40.1.2** (bcprov remains 1.86). |

### Part 7B — Additions of new content

| # | Severity | Action |
|---|----------|--------|
| A1 | NEW | Add **GAP-32** (x5c leaf-key conversion RSA/EC-only; files and criteria in §6.1) to the HTML GAP table, gaps doc and the affected rows (12, 26, 55, 62). Raise as a sub-issue alongside GAP-1 / GAP-28 (AKP plumbing). |
| A2 | NEW | Add `X509CertificateChainValidator.java`, `TrustMaterialIdentityProvider.java`, `TrustMaterialResolver.java`, `TrustMaterialSdJwtIssuerResolver.java`, `X509TrustMaterial.java` to the key files of rows 12 / 26 / 55 / 62 as appropriate. |
| A3 | NEW (optional) | Add `DcqlQuery.java`, `DcqlQueryContributor.java`, `CredentialBuildConfig.java`, `SupportedCredentialConfiguration.java` to "Files Evaluated" as non-crypto. |

### Part 7C — Resolved items / state changes

- **Resolved gaps:** none. All 31 GAPs remain open.
- **Possible status upgrade for reviewer approval:** rows 26 and 55 PENDING PROVIDERS → PARTIAL (new distribution 27 / 14 / 14 / 5 / 3). Not applied — requires human approval.

### Part 7D — Prompt maintenance (for `prompts/REASSESS_PQC_READINESS_PROMPT.md`, not the HTML)

Carried over from the 2026-10-17 report and still unapplied:
- Section A row for `DIDUtils.java` is dead (no such file in the tree); remove or replace.
- Section A row for `DefaultSamlArtifactResolver.java` says the SOAP fetch is in `SAMLEndpoint.java`; it is in `SAMLIdentityProvider.resolveArtifact()` (SAMLEndpoint only invokes it).
- Section A row for the Maven Enforcer says the plugin is "present in `pom.xml`"; the root `pom.xml` only defines version properties (lines 200-201).
- Section B3 should read the resulting `bcprov` version from the BOM in `~/.m2/.../quarkus-bom/<version>/` rather than only grepping `quarkus.version`.
- C1 template says "all 30 GAPs" / "62+ rows"; now 31 GAPs / 63 rows.

New this run:
- Section B2 should add greps for `UserInfoEndpoint` (GAP-21), `DefaultTokenManager.decodeClientJWT` (GAP-20) and the `AbstractOAuth2IdentityProvider` HS256 line (GAP-10).
- Section B2's GAP-22 grep (`\[TODO\].*alg`) works but should also report the `x5c` branch (`grep -n "x5c" AttestationBasedClientAuthenticator.java`).
- Section B3 should add: `grep -n "instanceof RSAPublicKey\|instanceof ECPublicKey\|AKP\|OKP" core/src/main/java/org/keycloak/crypto/X509CertificateChainValidator.java` (tracks the §1.1 / GAP-32 state) and `grep -rn "validateX509Chain" services server-spi-private core` to detect new call sites.
- Section B1 should note that FILE_LIST contains glob tokens (`EcdhEsA*Kw…`, `*SignatureProviderFactory.java`) and one documented rename (`AttestationX509CertificateValidator.java`) that are expected "missing" results.

---

*Review conducted against Keycloak `main` @ `eafe81dd79` on 2026-11-17. `pqc_overview.html` and all domain/gap markdown files were left unmodified, pending reviewer approval.*
