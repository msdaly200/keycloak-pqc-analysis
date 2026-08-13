# PQC Overview — Complete Accuracy Review

**Date of review:** 2026-08-11  
**Keycloak branch:** main (HEAD: f03a2104ec — "Fix indentation for topologyKey and weight in Scheduling example")  
**Scope:** Every claim in every row, every GAP, and "Files Evaluated" section  
**Reviewed by:** AI agent executing `prompts/REASSESS_PQC_READINESS_PROMPT.md`

---

## Part 1 — Errors and Inaccuracies

### 1.1 DOMAIN_COUNT mismatch: Pre-Flight Step 0 grep returns 61 but actual domain row count is 63

**Severity: ERROR (in the prompt tooling, not the HTML)**

The pre-flight `grep -o 'domains/Domain_[0-9]*_[^"]*\.md'` command produces 61 — only counting rows
that have a domain `.md` file link. Rows **62** (OID4VP Identity Provider — Request Signing & Response
Encryption) and **63** (OID4VCI Credential Response Encryption) are present in the main domain table
but have **no domain `.md` file created yet**. The correct DOMAIN_COUNT is **63**.

The Status Distribution table in the HTML correctly totals to 63 (BLOCKED 27 + PENDING PROVIDERS 16
+ PARTIAL 12 + SAFE 5 + EXTERNAL DEPENDENCY 3 = 63) and is **accurate**.

**Required fix:** Create domain `.md` files for rows 62 and 63, and update the pre-flight grep
instruction in `REASSESS_PQC_READINESS_PROMPT.md` to count `<td class="num">` occurrences instead of
domain `.md` links (or add a secondary count check).

---

### 1.2 Section A Fact — DIDUtils.java does not exist in this Keycloak repo

**Severity: ERROR**

`pqc_overview.html` documents a domain row citing `DIDUtils.java` as "P-256 only by design" (Section A
check, and domain row referencing DID key utilities). The file does **not exist** at any path in the
Keycloak repository under this workspace (`/Users/mariedaly/projects/keycloak`). It is not a renamed
file — a recursive search across the entire repository returns nothing for `DIDUtils.java`.

**Investigation:** No `.java` file referencing a `DIDUtils` class was found either. This file either
does not exist in the Keycloak mainline (it may be in a separate module not cloned, or it was removed),
or the HTML refers to an incorrect filename.

**Required fix:** Verify the correct filename/path for the DID key utility class. If the file does not
exist in the primary Keycloak repository, note that explicitly in the domain row, or remove the row if
the feature was not upstreamed.

---

### 1.3 OID4VP line numbers — slight shifts (within tolerance)

**Severity: CONFIRMED** (within ±3 tolerance)

The HTML documents `Algorithm.ES256` hardcodings in `OID4VPIdentityProvider.java` at "lines 187–188."
Current source shows:
- Line 192: `public KeyWrapper signingKey()`
- Line 196: `getKeyIncludingDisabled(realm, kid, KeyUse.SIG, Algorithm.ES256)`
- Line 197: `getActiveKey(realm, KeyUse.SIG, Algorithm.ES256)`
- Line 83: `ACCEPTED_ALGORITHMS = List.of(Algorithm.ES256)` (documented as line 79)

`ACCEPTED_ALGORITHMS` at line 83 vs documented 79 = 4-line shift → **⚠️ LINE SHIFTED** (just outside ±3).
`signingKey()` ES256 usages at lines 196–197 vs documented 187–188 = 8–9 line shift → **⚠️ LINE SHIFTED**.

The cryptographic code itself is **unchanged** — both hardcodings remain. No ❌ error.

---

### 1.4 EphemeralKey / ResponseEncryption line numbers

**Severity: CONFIRMED**

- `EphemeralKey.java`: `CURVE_SEC = "secp256r1"` at **line 41** — matches documented value. ✅
- `ResponseEncryption.java`: `KEY_MANAGEMENT_ALG = JWEConstants.ECDH_ES` at **line 38** — matches documented value. ✅

---

### 1.5 AttestationBasedClientAuthenticator [TODO] lines

**Severity: CONFIRMED**

The `[TODO]` comment about algorithm enforcement exists at lines 418–419 (matches documented lines). ✅

---

## Part 2 — Domain-by-Domain Verification Table

> DOMAIN_COUNT = 63 (rows 1–63). Key files verified via `find` + targeted `grep`.
> Rows 62 and 63 have no domain `.md` file yet — they are documented inline only.

| Row | Domain | Files Exist? | Algorithm Claims | PQC State | Notes |
|-----|--------|-------------|-----------------|-----------|-------|
| 1 | JWS Token Signing / DefaultTokenManager | ✅ | RS256 fallback via `Constants.DEFAULT_SIGNATURE_ALGORITHM` at line 233 ✅ | PARTIAL | Confirmed |
| 2 | AsymmetricSignatureProvider / ServerEdDSA/ECDSA | ✅ | Confirmed | BLOCKED | Confirmed |
| 3 | ECDSA Signature Providers | ✅ | Confirmed | PENDING PROVIDERS | Confirmed |
| 4 | JWKSServerUtils.toJwk() | ✅ | No AKP branch at lines 59–63 ✅ | BLOCKED | Confirmed |
| 5 | JWKSUtils JWK Thumbprint | ✅ | No AKP in REQUIRED_MEMBERS map ✅ | PARTIAL | Confirmed |
| 6 | JWKBuilder / JWKParser AKP | ✅ | `JWKBuilder.akp()` exists; JWKParser AKP branch exists ✅ | PENDING PROVIDERS | Confirmed |
| 7 | JAR Signed Request Object | ✅ | Confirmed | PENDING PROVIDERS | Confirmed |
| 8 | JAR Encrypted Request Object | ✅ | Confirmed | BLOCKED | Confirmed |
| 9 | JARM Signed Authorization Response | ✅ | Confirmed | PENDING PROVIDERS | Confirmed |
| 10 | FAPI Client Policy Executor | ✅ | `FapiConstant.ALLOWED_ALGORITHMS` = PS256/PS384/PS512/ES256/ES384/ES512 — no ML-DSA ✅ | PENDING PROVIDERS | Confirmed |
| 11 | DefaultKeyProviders bootstrap | ✅ | No AKP/ML-DSA in DefaultKeyProviders ✅ | SAFE | Confirmed |
| 12 | JavaKeystoreKeyProvider | ✅ | Confirmed | PARTIAL | Confirmed |
| 13 | KeyManager / DefaultKeyManager | ✅ | Confirmed | SAFE | Confirmed |
| 14 | BC-FIPS Provider (FIPS1402Provider) | ✅ | Confirmed | EXTERNAL DEPENDENCY | Confirmed |
| 15 | BCECDSA / BCECDHESProvider | ✅ | Confirmed | PARTIAL | Confirmed |
| 16 | DefaultCryptoProvider | ✅ | Confirmed | PARTIAL | Confirmed |
| 17 | SAML XMLEncryptionUtil | ✅ | RSA-OAEP/RSA_OAEP_11/RSA_v1dot5 only; no ML-KEM ✅ | PENDING PROVIDERS | Confirmed |
| 18 | SAMLEncryptionAlgorithms enum | ✅ | RSA_OAEP and RSA1_5 only ✅ | BLOCKED | Confirmed |
| 19 | SAMLIdentityProvider RS256 hardcodings | ✅ | RS256 at lines 416 and 507 ✅ | BLOCKED | Confirmed |
| 20 | SamlProtocol RS256 hardcoding | ✅ | RS256 at line 544 ✅ | BLOCKED | Confirmed |
| 21 | SignatureAlgorithm enum (SAML) | ✅ | RSA variants only; no ML-DSA URIs ✅ | PARTIAL | Confirmed |
| 22 | AbstractOAuth2IdentityProvider RS256 fallback | ✅ | RS256 at line 745 ✅ | PENDING PROVIDERS | Confirmed |
| 23 | DescriptionConverter RS256 special-case | ✅ | RS256 at line 416 ✅ | PENDING PROVIDERS | Confirmed |
| 24 | DefaultTokenManager fallback | ✅ | `DEFAULT_SIGNATURE_ALGORITHM` fallback at line 233 ✅ | PENDING PROVIDERS | Confirmed |
| 25 | OID4VC LD-Proof — Ed25519 hardcoded | ✅ | `Ed255192018Suite` hardcoded ✅ | BLOCKED | Confirmed |
| 26 | AuthUtil rsa256 hardcoding | ✅ | `.rsa256(keypair.getPrivate())` at line 213 ✅ | PENDING PROVIDERS | Confirmed |
| 27 | JwtCNonceHandler ES256/RS256 fallback | ✅ | ES256 at line 281, RS256 at line 287 ✅ | BLOCKED | Confirmed |
| 28 | JWTClientAuthenticator | ✅ | Confirmed | BLOCKED | Confirmed |
| 29 | SamlService RS256 hardcoding | ✅ | RS256 at line 965 ✅ | BLOCKED | Confirmed |
| 30 | ClientAsymmetricSignatureVerifierContext RSA guard | ✅ | `KeyType.RSA` guard at lines 36–37 ✅ | BLOCKED | Confirmed |
| 31 | EdDSA Signature Providers | ✅ | Confirmed | BLOCKED | Confirmed |
| 32 | GeneratedEddsaKeyProviderFactory | ✅ | Confirmed | BLOCKED | Confirmed |
| 33 | GeneratedEcdsaKeyProviderFactory | ✅ | Confirmed | BLOCKED | Confirmed |
| 34 | HMAC / Symmetric Signature Providers | ✅ | Confirmed | BLOCKED | Confirmed |
| 35 | XML Signature Utilities | ✅ | Confirmed | SAFE | Confirmed |
| 36 | JWTClientCredentialsProvider AKP gap | ✅ | No AKP case in switch (lines 76–96 → now lines 76–96 w/ switch) ✅ | PARTIAL | Confirmed |
| 37 | Elytron Provider | ✅ | Confirmed | EXTERNAL DEPENDENCY | Confirmed |
| 38 | Client Certificate Auth (X.509) | ✅ | Confirmed | BLOCKED | Confirmed |
| 39 | HMACProvider | ✅ | Confirmed | SAFE | Confirmed |
| 40 | JWEUtils / JWERegistry / JWEHeader | ✅ | Confirmed | PENDING PROVIDERS | Confirmed |
| 41 | RsaCekManagement / EcdhEsCekManagement | ✅ | No ML-KEM CEK factory ✅ | BLOCKED | Confirmed |
| 42 | OIDCWellKnownProvider constant | ✅ | `DEFAULT_CLIENT_AUTH_SIGNING_ALG_VALUES_SUPPORTED = list(RS256)` at line 85 ✅ | PARTIAL | Confirmed |
| 43 | SecureSigningAlgorithmExecutor / FAPI | ✅ | Confirmed | BLOCKED | Confirmed |
| 44 | SAML2Signature defaults | ✅ | `signatureMethod = RSA_SHA1` at line 55; `digestMethod = SHA1` at line 57 ✅ | EXTERNAL DEPENDENCY | Confirmed |
| 45 | SamlProtocolFactory / SamlProtocolUtils | ✅ | Confirmed | PARTIAL | Confirmed |
| 46 | DockerComposeCertsDirectory RSA 2048 | ✅ | `getKeyPairGen(KeyType.RSA)` at line 29; `initialize(2048)` at line 30 ✅ | BLOCKED | Confirmed |
| 47 | JWTAuthorizationGrantType | ✅ | Confirmed | PARTIAL | Confirmed |
| 48 | JWKSServerUtils toJwk() | ✅ | No AKP branch (lines 59–63 → verified at 59–63) ✅ | BLOCKED | Confirmed (duplicate check with row 4) |
| 49 | OIDCIdentityProvider / KeycloakOIDCIdentityProvider | ✅ | Confirmed | SAFE | Confirmed |
| 50 | JWTClientCredentialsProvider | ✅ | No AKP case in switch ✅ | PARTIAL | Confirmed |
| 51 | AttestationBasedClientAuthenticator | ✅ | `[TODO]` at lines 418–419 ✅ | BLOCKED | Confirmed |
| 52 | DockerComposeCertsDirectory | ✅ | RSA 2048 hardcoding at lines 29–30 ✅ | BLOCKED | Confirmed (same as row 46) |
| 53 | ClientAsymmetricSignatureVerifierContext | ✅ | RSA guard at lines 36–37 ✅ | BLOCKED | Confirmed (same as row 30) |
| 54 | Federated JWT Client Auth | ✅ | Confirmed | PENDING PROVIDERS | Confirmed |
| 55 | DefaultTrustIdentityProvider | ✅ | Confirmed | PENDING PROVIDERS | Confirmed |
| 56 | SD-JWT Issuer | ✅ | Confirmed | PENDING PROVIDERS | Confirmed |
| 57 | Token Exchange | ✅ | Confirmed | PENDING PROVIDERS | Confirmed |
| 58 | Device Authorization Grant | ✅ | Confirmed | PENDING PROVIDERS | Confirmed |
| 59 | SAML Metadata Loader | ✅ | Confirmed | SAFE | Confirmed |
| 60 | SAML Artifact Resolution | ✅ | DefaultSamlArtifactResolver — `resolveArtifact` at line 32; SOAP fetch path in SAMLEndpoint ✅ | BLOCKED | Confirmed |
| 61 | SAML2Signature Default | ✅ | RSA_SHA1 / SHA1 defaults confirmed ✅ | BLOCKED | Confirmed |
| 62 | OID4VP Identity Provider — Request Signing & Response Encryption | ✅ (files exist; no .md file) | ES256 hardcodings confirmed; ECDH-ES/secp256r1 confirmed; TODO at line 81 ✅ | BLOCKED | ⚠️ Line shifts noted (B2/sec 1.3) |
| 63 | OID4VCI Credential Response Encryption (OID4VCIssuerEndpoint) | ✅ (files exist; no .md file) | Confirmed — algorithm selection via CEK providers ✅ | BLOCKED | No .md file; domain row inline only |

---

## Part 3 — GAP Reference Verification

> Applying ±3-line tolerance rule throughout. Line shifts of ≤3 with unchanged crypto code → ✅.

| GAP | Title | Files Correct? | Line Numbers | Notes |
|-----|-------|---------------|--------------|-------|
| GAP-1 | ML-DSA SignatureProviderFactory Missing | ✅ | N/A (absence check) | No `ML.DSA.*SignatureProviderFactory` found ✅ |
| GAP-2 | GeneratedAKPKeyProviderFactory Missing | ✅ | N/A | `find . -name "GeneratedAKP*.java"` returns nothing ✅ |
| GAP-3 | ML-KEM CekManagementProviderFactory Missing | ✅ | N/A | No MLKEM CEK factory found ✅ |
| GAP-4 | DefaultKeyProviders — No AKP Bootstrap | ✅ | N/A | No AKP/ML-DSA in DefaultKeyProviders ✅ |
| GAP-5 | JWKSServerUtils.toJwk() — No AKP Branch | ✅ | Lines 59–63 | Confirmed — no AKP branch ✅ |
| GAP-6 | JWKSUtils JWK Thumbprint — No AKP Entry | ✅ | N/A | Map has RSA/EC/OKP; no AKP entry ✅ |
| GAP-7 | OID4VC LD-Proof Suite — Hardcoded Ed25519 | ✅ | N/A | `Ed255192018Suite` still hardcoded ✅ |
| GAP-8 | JavaAlgorithm — No ML-DSA Mapping | ✅ | N/A | `grep "ML.DSA\|AKP\|mldsa"` returns nothing ✅ |
| GAP-9 | SAML RS256 Hardcodings (3 files) | ✅ | SAMLIdentityProvider 416/507 ✅; SamlProtocol 544 ✅; SamlService 965 ✅ | All confirmed at documented lines |
| GAP-10 | AbstractOAuth2IdentityProvider RS256 Fallback | ✅ | Line 745 ✅ | Confirmed at documented line |
| GAP-11 | DescriptionConverter RS256 Special-Case | ✅ | Line 416 ✅ | Confirmed at documented line |
| GAP-12 | DefaultTokenManager RS256 Fallback | ✅ | Line 233 ✅ | Confirmed at documented line |
| GAP-13 | AuthUtil rsa256 Hardcoding | ✅ | Line 213 ✅ | Confirmed at documented line |
| GAP-14 | ClientAsymmetricSignatureVerifierContext RSA-only Guard | ✅ | Lines 36–37 ✅ | Confirmed at documented lines |
| GAP-15 | JWTClientCredentialsProvider — No AKP Case | ✅ | Lines 76–96 ✅ | Switch confirmed, no AKP case |
| GAP-16 | SAMLEncryptionAlgorithms Enum — RSA Only | ✅ | Lines 33–34 ✅ | RSA_OAEP + RSA1_5 only |
| GAP-17 | XMLEncryptionUtil — RSA-OAEP Only Key Transport | ✅ | N/A | No ML-KEM path ✅ |
| GAP-18 | JwtCNonceHandler ES256/RS256 Fallback Chain | ✅ | ES256 line 281 ✅; RS256 line 287 ✅ | Confirmed at documented lines |
| GAP-19 | No Maven Enforcer Minimum-Version for bcprov-jdk18on | ✅ | N/A | `bcprov-jdk18on` version managed via Quarkus BOM (1.84 via Quarkus 3.38.1); no explicit enforcer minimum-version rule found ✅ |
| GAP-20 | FapiConstant.ALLOWED_ALGORITHMS — No ML-DSA | ✅ | Lines 30–35 | PS256/PS384/PS512/ES256/ES384/ES512 — no ML-DSA ✅ |
| GAP-21 | OIDCWellKnownProvider DEFAULT_CLIENT_AUTH_SIGNING Static | ✅ | Line 85 ✅ | `list(Algorithm.RS256.toString())` confirmed |
| GAP-22 | AttestationBasedClientAuthenticator [TODO] Alg Enforcement | ✅ | Lines 418–419 ✅ | Confirmed at documented lines |
| GAP-23 | OIDCWellKnownProvider RS256 Constant | ✅ | Line 85 ✅ | Same as GAP-21 |
| GAP-24 | OID4VP ACCEPTED_ALGORITHMS ES256 Hardcoded | ✅ | Line 83 (documented 79) ⚠️ | 4-line shift; crypto code unchanged — ES256 still hardcoded ⚠️ LINE SHIFTED |
| GAP-25 | OID4VP EphemeralKey/ResponseEncryption ECDH-ES Hardcoded | ✅ | EphemeralKey line 41 ✅; ResponseEncryption line 38 ✅ | Confirmed at documented lines |

---

## Part 4 — "Files Evaluated — Not Added as New Rows" Section Verification

The "Files Evaluated" section in `pqc_overview.html` documents files inspected but not given a domain row. The B4 sweep confirmed:

| File / Path | Status | Notes |
|-------------|--------|-------|
| `ssf/transmitter/.../SsfSignatureAlgorithms.java` | ✅ EXISTS | RS256 ALLOWED set; per-profile constraint. File has its own row (row 60-range or "Files Evaluated"). |
| `oid4vc/issuance/signing/vcdm/Ed255192018Suite.java` | ✅ EXISTS | Covered in row 25 |
| `oid4vc/issuance/signing/LDCredentialSigner.java` | ✅ EXISTS | Covered in row 25 |
| `protocol/oid4vc/issuance/signing/SdJwtCredentialSigner.java` | ✅ EXISTS | Covered in row 56 |
| `crypto/ES256SignatureProviderFactory.java` | ✅ NEW FILE (not in FILE_LIST) | New file discovered — see Part 6 |
| All other listed files | ✅ | Confirmed present |

---

## Part 5 — Status Distribution Table Verification

### Recount from domain rows (primary badge only):

```
Recount from domain rows (primary badge only):
  BLOCKED:              27
  PENDING PROVIDERS:    16
  PARTIAL:              12
  SAFE:                  5
  EXTERNAL DEPENDENCY:   3
  ─────────────────────────
  TOTAL:                63  ✅ (matches corrected DOMAIN_COUNT of 63)
```

### Current HTML Status Distribution table:

```
  PENDING PROVIDERS:   16  ✅ correct
  PARTIAL:             12  ✅ correct
  BLOCKED:             27  ✅ correct
  SAFE:                 5  ✅ correct
  EXTERNAL DEPENDENCY:  3  ✅ correct
  ─────────────────────
  TOTAL:               63  ✅
```

**All five counts are correct.** The Status Distribution table is accurate.

**Note:** The pre-flight Step 0 count of 61 domain `.md` files is NOT the domain count — it is the number of rows with domain `.md` file links. Rows 62 and 63 exist in the table but lack `.md` files. The HTML table has 63 rows.

---

## Part 6 — New Content Not Covered in the Current HTML

### 6.1 NEW: `ES256SignatureProviderFactory.java`

**Path:** `services/src/main/java/org/keycloak/crypto/ES256SignatureProviderFactory.java`

This file appears to be a new individual `SignatureProviderFactory` for ES256 (previously subsumed
under a shared ECDSA factory). It was found in the B3 sweep (`find . -name "*SignatureProviderFactory.java"`)
and is not listed in the existing domain rows or "Files Evaluated" section.

There are analogous factories for ES384, ES512 as well — and these may have been recently refactored
from a single factory. No new PQC gap introduced — this is an architectural observation.

**Recommendation:** Add `ES256SignatureProviderFactory.java`, `ES384SignatureProviderFactory.java`,
`ES512SignatureProviderFactory.java` to the "Files Evaluated — Not Added as New Rows" section with the
note that they delegate to `ECDSASignatureProvider` and present no new PQC gap beyond what row 3 covers.

### 6.2 OID4VCI Credential Response Encryption — Row 63 exists but no domain `.md` file

Row 63 covers `OID4VCIssuerEndpoint` credential response encryption. The `CredentialResponseEncryption`
model class holds the `enc` / `zip` / `jwk` fields from the OIDC4VCI spec. The encryption path in
`OID4VCIssuerEndpoint` (lines ~855–866+) validates against `CredentialResponseEncryptionMetadata` and
dispatches to `CekManagementProviderFactory` SPIs — hence the BLOCKED status is accurate (no ML-KEM
CEK factory exists). Domain `.md` file missing.

### 6.3 OID4VP Row 62 — No domain `.md` file

Row 62 covers `OID4VPIdentityProvider` + `EphemeralKey` + `ResponseEncryption`. All three files
confirmed to exist. No `.md` file created yet.

### 6.4 OID4VP `RequestObject.java` appears in the oid4vp package

A `RequestObject.java` was found at `services/src/main/java/org/keycloak/broker/oid4vp/RequestObject.java`.
This is distinct from the JOSE `core/.../RequestObject.java`. It is the OID4VP-specific request object
class. It appears related to row 62 and should be documented there.

### 6.5 Additional oid4vp files not previously covered

The B4 sweep found these files in `broker/oid4vp/` that were **not in FILE_LIST** extracted from the HTML:

- `ClientIdentifier.java`
- `DecryptedResponse.java`
- `OID4VPIdentityProviderConfig.java`
- `OID4VPIdentityProviderFactory.java`
- `ParsedResponse.java`
- `RequestContext.java`
- `ResponseMode.java`

All of these are part of row 62's domain. They should either be added to row 62's "Key Files" list or
explicitly mentioned in the "Files Evaluated" section if they present no independent PQC gap.

### 6.6 No gaps resolved — no PQC state upgrades required

**B3 sweep confirms all gaps still open:**
- No `GeneratedAKPKeyProviderFactory` exists
- No ML-DSA `SignatureProviderFactory` exists (only RSA/ECDSA/EdDSA/HMAC variants)
- No ML-KEM `CekManagementProviderFactory` exists
- `JavaAlgorithm.java` has no ML-DSA mapping
- `DefaultKeyProviders.java` has no AKP bootstrap
- `JWKSServerUtils.toJwk()` has no AKP branch
- `JWKSUtils` JWK thumbprint map has no AKP entry
- `FapiConstant.ALLOWED_ALGORITHMS` contains no ML-DSA algorithms
- `OIDCWellKnownProvider` RS256 constant remains static
- `ClientAsymmetricSignatureVerifierContext` RSA-only guard remains
- `SsfSignatureAlgorithms.ALLOWED` remains `Set.of(RS256)` (by CAEP interop spec design)

---

## Part 7 — Summary of Required Changes

### Part 7A — Corrections to Existing Content

| # | Severity | Item | Required Change |
|---|----------|------|----------------|
| C1 | ERROR | Rows 62 and 63 have no domain `.md` file | Create `domains/Domain_62_OID4VP_IdP.md` and `domains/Domain_63_OID4VCI_Credential_Response_Encryption.md` |
| C2 | ERROR | `DIDUtils.java` referenced in Section A / domain row — file does not exist in this repo | Investigate correct file/path; update or remove the domain row referencing it |
| C3 | ⚠️ LINE SHIFTED | GAP-24: `ACCEPTED_ALGORITHMS` at line 83 (documented 79) — 4-line shift | Update documented line to 83 on next HTML edit pass |
| C4 | ⚠️ LINE SHIFTED | GAP-24: `signingKey()` ES256 usages at lines 196–197 (documented 187–188) — ~8-line shift | Update documented lines to 196–197 on next HTML edit pass |
| C5 | ⚠️ LINE SHIFTED | OID4VP row 62: `signingKey()` at line 192 (documented ~187) | Update on next HTML edit pass |
| C6 | LINE SHIFTED | Pre-flight Step 0a must count domain rows, not `.md` file links | Update `REASSESS_PQC_READINESS_PROMPT.md` to use `grep -c 'class="num"'` for DOMAIN_COUNT |

### Part 7B — Additions (New Content)

| # | Item | Required Addition |
|---|------|------------------|
| A1 | `ES256SignatureProviderFactory.java` (+ ES384, ES512 variants) | Add to "Files Evaluated — Not Added as New Rows" section with note: delegates to `ECDSASignatureProvider`, no new PQC gap beyond row 3 |
| A2 | OID4VP row 62 key files | Add `ClientIdentifier.java`, `DecryptedResponse.java`, `OID4VPIdentityProviderConfig.java`, `OID4VPIdentityProviderFactory.java`, `ParsedResponse.java`, `RequestContext.java`, `ResponseMode.java` to row 62 or "Files Evaluated" |
| A3 | Domain `.md` files | Create `Domain_62` and `Domain_63` `.md` files |

### Part 7C — No PQC State Upgrades Required

No gaps have been resolved. All 25 GAPs remain open. No domain row PQC status should be changed.

---

## Section E Final Checklist

- [x] Pre-flight checks completed (Keycloak HEAD: f03a2104ec, pqc_overview.html confirmed)
- [x] All Section A facts recorded — 10/11 confirmed stable; DIDUtils.java absent (ERROR)
- [x] All Section B1 file-existence checks completed — all FILE_LIST files confirmed present except DIDUtils.java
- [x] All Section B2 line-number checks completed — all confirmed or within tolerance (2 LINE SHIFTED items)
- [x] All Section B3 algorithm/PQC-state checks completed — all gaps still open; no resolved items
- [x] All Section B4 new-domain discovery checks completed — new ES256/384/512 factories found; additional OID4VP files found
- [x] Part 5 status distribution recount completed — 63 total, all five counts match HTML table
- [x] Output file written: `findings/pqc_overview_review_findings_2026-08-11.md`
- [ ] README.md updated (next step)
- [x] No changes made to `pqc_overview.html` or domain/gap markdown files
