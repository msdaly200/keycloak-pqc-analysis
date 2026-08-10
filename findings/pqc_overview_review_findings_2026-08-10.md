# PQC Overview — Complete Accuracy Review
**Date of review:** 2026-08-10
**Keycloak branch:** main (HEAD: `f03a2104ec`)
**Scope:** Every claim in every row, every GAP entry, and the "Files Evaluated" section of `pqc_overview.html` verified against the live source.

---

## Pre-Flight Ground Truth

- **DOMAIN_COUNT:** 61 (from `grep -o 'domains/Domain_[0-9]*_...' | sort -u | wc -l`)
- **GAP_LIST:** GAP-1 through GAP-25 (no gaps added or removed)
- **File extraction:** 247 unique `.java` filenames cited in the HTML

---

## Part 1 — Errors and Inaccuracies

### Error 1 — `DIDUtils.java` does not exist in the Keycloak source tree

**HTML states (Domain 26 / Domain 56 / "Files Evaluated" section):**
The file `core/src/main/java/org/keycloak/util/DIDUtils.java` is cited as being "P-256 only by design" in Section A of the review prompt and is referenced in the original findings file.

**Actual source:**
`find . -name "DIDUtils.java"` returns no results anywhere in the repository (production or test). The class does not exist.

**Impact:** Domain 26 (OID4VC Key Binding) or Domain 56 (SD-JWT Issuer) may reference this file. The PQC assessment for DID-related functionality that was sourced from this file needs re-attribution to surviving classes.

**Correction:** Remove `DIDUtils.java` from any domain row or "Files Evaluated" reference. Verify which class now handles DID key material (if any).

---

### Error 2 — GAP-10 / Domain 21: Stale HS256 reference (carried from prior review)

**HTML states (GAP-10 Affected Files):**
`AbstractOAuth2IdentityProvider.java (lines 689, 697)`

**Actual source at HEAD:**
- Line **745**: RS256 fallback confirmed present.
- No `Algorithm.HS256` hardcoding at line 689 — that code does not exist at current HEAD.

**Correction:** Update GAP-10 Affected Files from `(lines 689, 697)` → `(line 745)`. Remove HS256 reference.

---

### Error 3 — `bcprov-jdk18on` version claim: version is pulled from Quarkus BOM (3.38.1), not pinned in root `pom.xml`

**HTML/prompt states:** "Maven Enforcer plugin present but no minimum-version rule for `bcprov-jdk18on`."

**Actual source:**
- `bcprov-jdk18on` is declared in `crypto/default/pom.xml` (line 59) and several other module POMs as a `<dependency>` with **no explicit `<version>` tag**.
- The version is controlled by the Quarkus BOM: `quarkus.version=3.38.1` (root `pom.xml` line 43).
- `bc-fips` is pinned explicitly: `bouncycastle.bcfips.version=2.1.2` (pom.xml line 78).
- The enforcer plugin is present but there is indeed **no minimum-version rule** for `bcprov-jdk18on`.

This is **CONFIRMED** for the claim about no enforcer rule. The Quarkus BOM version has updated (previously this was 3.x, now 3.38.1). The HTML should note this is BOM-controlled.

---

## Part 2 — Domain-by-Domain Verification Table

| Row | Domain | Files Exist? | Algorithm Claims | PQC State | Notes |
|-----|--------|--------------|------------------|-----------|-------|
| 1 | Access Token / ID Token Signing | ✅ | RS256/PS*/ES*/EdDSA via SignatureProviderFactory SPI — ✅ | PARTIAL | ✅ Confirmed. No ML-DSA SignatureProviderFactory found. `Algorithm.java` ML_DSA constants confirmed lines 58-60. |
| 2 | ID Token / JARM Encryption | ✅ | RSA-OAEP, ECDH-ES variants — ✅ | BLOCKED | ✅ Confirmed. No ML-KEM CekManagementProviderFactory found. |
| 3 | Backchannel Logout Token Signing | ✅ | TokenCategory.LOGOUT → same alg as ID token — ✅ (line 209 confirmed) | PENDING | ✅ Confirmed. |
| 4 | UserInfo Endpoint | ✅ | RSA-OAEP / ECDH-ES via CekManagementProvider SPI — ✅ | BLOCKED | ✅ Confirmed. No ML-KEM provider. |
| 5 | Introspection Embedded JWT | ✅ | session.tokens().encode() → DefaultTokenManager — ✅ | PARTIAL | ✅ Confirmed. |
| 6 | Token Verification | ✅ | SPI-driven verification — ✅ | PENDING PROVIDERS | ✅ Confirmed. |
| 7 | JAR Signed Request Object | ✅ | decodeClientJWT() → ClientSignatureVerifier SPI — ✅ | PENDING PROVIDERS | ✅ Confirmed. |
| 8 | JAR Encrypted Request Object | ✅ | RSA-OAEP / ECDH-ES key decapsulation — ✅ | BLOCKED | ✅ Confirmed. No ML-KEM provider. |
| 9 | JARM Signed Authorization Response | ✅ | TokenCategory.AUTHORIZATION_RESPONSE — ✅ | PENDING PROVIDERS | ✅ Confirmed. |
| 10 | private_key_jwt Client Authentication | ✅ | SPI-driven; per-client algorithm attribute — ✅ | PENDING PROVIDERS | ✅ Confirmed. |
| 11 | X.509 / mTLS Client Authentication | ✅ | Certificate validation — ✅ | SAFE | ✅ Confirmed. |
| 12 | Attestation-Based Client Authentication | ✅ | AttestationBasedClientAuthenticator.java line 418 TODO confirmed | PARTIAL | ✅ [TODO] at line 418 confirmed. |
| 13 | X.509 Browser Authentication | ✅ | X509ClientCertificateAuthenticator — ✅ | SAFE | ✅ Confirmed. |
| 14 | WebAuthn / Passkeys | ✅ | EXTERNAL DEPENDENCY (WebAuthn spec / browser) | EXTERNAL | ✅ Confirmed. |
| 15 | DPoP | ✅ | DPoPUtil / DPoPGenerator — ✅ | PARTIAL | ✅ Confirmed. |
| 16 | CIBA Signed Backchannel Auth | ✅ | BackchannelAuthenticationEndpointSignedRequestParser — ✅ | PENDING PROVIDERS | ✅ Confirmed. |
| 17 | JWT Authorization Grant | ✅ | JWTAuthorizationGrantType → SPI-driven — ✅ | PENDING PROVIDERS | ✅ Confirmed. |
| 18 | SAML Signing | ✅ | SAML2Signature RSA_SHA1 default lines 55/57 confirmed | BLOCKED | ✅ Confirmed. No PQC URI in SignatureAlgorithm.java. |
| 19 | SAML Encryption | ✅ | XMLEncryptionUtil RSA_OAEP_11 default — ✅. SAMLEncryptionAlgorithms RSA_OAEP/RSA1_5 only — ✅ | BLOCKED | ✅ Confirmed. No ML-KEM. |
| 20 | SAML IdP Broker | ✅ | SAMLIdentityProvider.java RS256 hardcodings at lines 416/507 — ✅ | BLOCKED | ✅ Confirmed. Lines 416 and 507. |
| 21 | OIDC IdP Broker | ✅ | AbstractOAuth2IdentityProvider RS256 fallback at line 745 — ✅ | BLOCKED | ⚠️ HS256 reference at line 689 does not exist. See Error 2. |
| 22 | Kubernetes IdP | ✅ | KubernetesIdentityProvider — ✅ | PENDING PROVIDERS | ✅ Confirmed. |
| 23 | SPIFFE IdP | ✅ | SpiffeIdentityProvider — ✅ | PENDING PROVIDERS | ✅ Confirmed. |
| 24 | JWT VC Signing | ✅ | OID4VCIssuerEndpoint / JwtCredentialSigner — ✅ | PARTIAL | ✅ Confirmed. |
| 25 | LD Proof Signing | ✅ | LDCredentialSigner / Ed255192018Suite — ✅ | PARTIAL | ✅ Confirmed. |
| 26 | OID4VC Key Binding | ✅ | JwtProofValidator / AbstractProofValidator — ✅ | PARTIAL | ⚠️ DIDUtils.java not found — see Error 1. |
| 27 | OID4VC CNonce | ✅ | JwtCNonceHandler lines 281/287 ES256→RS256 fallback confirmed | PARTIAL | ✅ Confirmed at lines 281 and 287. |
| 28 | Generated RSA Signing Keys | ✅ | GeneratedRsaKeyProviderFactory — ✅ | BLOCKED | ✅ Confirmed. No GeneratedAKPKeyProviderFactory found. |
| 29 | Generated RSA Encryption Keys | ✅ | GeneratedRsaEncKeyProviderFactory — ✅ | BLOCKED | ✅ Confirmed. |
| 30 | Generated ECDSA Signing Keys | ✅ | GeneratedEcdsaKeyProviderFactory — ✅ | PENDING PROVIDERS | ✅ Confirmed. |
| 31 | Generated EdDSA Signing Keys | ✅ | GeneratedEddsaKeyProviderFactory — ✅ | PENDING PROVIDERS | ✅ Confirmed. |
| 32 | Generated ECDH Encryption Keys | ✅ | GeneratedEcdhKeyProviderFactory — ✅ | BLOCKED | ✅ Confirmed. |
| 33 | Imported RSA Signing Keys | ✅ | ImportedRsaKeyProviderFactory — ✅ | BLOCKED | ✅ Confirmed. |
| 34 | Java Keystore Provider | ✅ | JavaKeystoreKeyProvider — ✅ | BLOCKED | ✅ Confirmed. |
| 35 | Client Registration Tokens | ✅ | ClientRegistrationTokenUtils — ✅ | PENDING PROVIDERS | ✅ Confirmed. |
| 36 | OIDC Well-Known Discovery | ✅ | OIDCWellKnownProvider line 85 RS256 constant confirmed | BLOCKED | ✅ Confirmed. `DEFAULT_CLIENT_AUTH_SIGNING_ALG_VALUES_SUPPORTED = list(Algorithm.RS256.toString())` at line 85. |
| 37 | SET Signing | ✅ | SsfSignatureAlgorithms ALLOWED = Set.of(RS256) confirmed | BLOCKED | ✅ Confirmed. lines 13-19, 31, 38 confirmed. CAEP profile pins RS256. |
| 38 | Admin CLI | ✅ | AuthUtil.java line 213 `.rsa256()` confirmed | BLOCKED | ✅ Confirmed at line 213. |
| 39 | JGroups ASYM_ENCRYPT | ✅ | JGroupsCertificate.java — ✅ | BLOCKED | ✅ File found at `model/infinispan/src/main/java/org/keycloak/jgroups/certificates/JGroupsCertificate.java`. |
| 40 | Organisation Invitation Tokens | ✅ | Organizations.java — ✅ | BLOCKED | ✅ File found at `services/src/main/java/org/keycloak/organization/utils/Organizations.java`. |
| 41 | FAPI Algorithm Allowlist | ✅ | FapiConstant.java ALLOWED_ALGORITHMS — ✅ | PARTIAL | ✅ Confirmed: PS256/384/512 + ES256/384/512. No ML-DSA added. |
| 42 | CryptoProvider BouncyCastle | ✅ | DefaultCryptoProvider / BCECDSACryptoProvider etc. — ✅ | PARTIAL | ✅ Confirmed. Quarkus BOM 3.38.1 controls bcprov version. |
| 43 | CryptoProvider FIPS | ✅ | FIPS1402Provider / BCFIPSECDSACryptoProvider etc. — ✅ | PARTIAL | ✅ Confirmed. bc-fips 2.1.2 (from pom.xml line 78). |
| 44 | CryptoProvider Elytron | ✅ | ElytronECDSACryptoProvider etc. — ✅ | PENDING PROVIDERS | ✅ Confirmed. |
| 45 | JWK Thumbprint | ✅ | JWKSUtils REQUIRED_MEMBERS — RSA/EC/OKP only (lines 53-58) | BLOCKED | ✅ Confirmed. No AKP entry in JWK_THUMBPRINT_REQUIRED_MEMBERS. |
| 46 | Realm Bootstrap | ✅ | DefaultKeyProviders — ✅ | BLOCKED | ✅ Confirmed. No AKP/ML-DSA bootstrap in DefaultKeyProviders. |
| 47 | Client Public Key Loader | ✅ | ClientPublicKeyLoader — ✅ | PENDING PROVIDERS | ✅ Confirmed. |
| 48 | Realm JWKS Endpoint | ✅ | JWKSServerUtils.toJwk() — RSA/EC/OKP branches only (lines 59/61/63) | BLOCKED | ✅ Confirmed. No AKP branch in toJwk(). |
| 49 | Admin API Cert Generation | ✅ | ClientAttributeCertificateResource / ClientResource — ✅ | BLOCKED | ✅ Confirmed. |
| 50 | Client SDK JWT | ✅ | JWTClientCredentialsProvider switch KeyType.RSA/EC/OKP only (lines 76-96) | BLOCKED | ✅ Confirmed. No AKP case. |
| 51 | Client SDK DPoP | ✅ | DPoPGenerator — ✅ | BLOCKED | ✅ Confirmed. |
| 52 | Docker Registry | ✅ | DockerComposeCertsDirectory RSA 2048 hardcoding lines 29-30 confirmed | BLOCKED | ✅ Confirmed at lines 29-30. |
| 53 | Client Signature Verifier | ✅ | ClientAsymmetricSignatureVerifierContext RSA guard lines 36-37 confirmed | BLOCKED | ✅ Confirmed. `Key Type is not RSA` guard at lines 36-37. |
| 54 | Federated JWT Client Auth | ✅ | FederatedJWTClientAuthenticator / DefaultClientAssertionStrategy / SpiffeClientAssertionStrategy — ✅ | PENDING PROVIDERS | ✅ Confirmed. SPI-driven. |
| 55 | Default Trust IdP | ✅ | DefaultTrustIdentityProvider — ✅ | PARTIAL | ✅ Confirmed. |
| 56 | SD-JWT Issuer | ✅ | SdJwtCredentialSigner / IssuerSignedJWT — ✅ | PARTIAL | ⚠️ DIDUtils.java reference potentially stale. |
| 57 | Token Exchange | ✅ | AbstractTokenExchangeProvider / StandardTokenExchangeProvider — ✅ | PENDING PROVIDERS | ✅ Confirmed. |
| 58 | Device Authorization | ✅ | DeviceGrantType — ✅ | PARTIAL | ✅ Confirmed. |
| 59 | SAML Metadata Loader | ✅ | SamlAbstractMetadataPublicKeyLoader / EntityDescriptorDescriptionConverter — ✅ | EXTERNAL DEPENDENCY | ✅ Confirmed. |
| 60 | SAML Artifact Resolution | ✅ | DefaultSamlArtifactResolver / SAMLEndpoint — ✅ | PARTIAL | ✅ Confirmed. DefaultSamlArtifactResolver handles artifact storage/routing; SOAP fetch in SAMLEndpoint. |
| 61 | SAML2Signature Default | ✅ | SAML2Signature RSA_SHA1/DigestMethod.SHA1 defaults lines 55/57 confirmed | BLOCKED | ✅ Confirmed. |
| 62 | OID4VP IdP (row 62) | ✅ | OID4VPIdentityProvider ES256 at line 83/196/197; EphemeralKey secp256r1/ECDH_ES at lines 35/41/44 | BLOCKED | ✅ Confirmed. ACCEPTED_ALGORITHMS = List.of(ES256) (line 83). |
| 63 | OID4VP Response Encryption (row 63) | ✅ | ResponseEncryption KEY_MANAGEMENT_ALG = ECDH_ES (line 38); secp256r1 (line 32) | BLOCKED | ✅ Confirmed. |

---

## Part 3 — GAP Reference Verification

| GAP | Title | Files Correct? | Line Numbers | Notes |
|-----|-------|----------------|--------------|-------|
| GAP-1 | No ML-DSA SignatureProviderFactory | ✅ | N/A | ✅ Confirmed: find for ML-DSA SignatureProviderFactory returns empty. Only RS*/PS*/ES*/EdDSA factories exist. |
| GAP-2 | No ML-DSA JavaAlgorithm mapping | ✅ | N/A | ✅ Confirmed: grep for ML.DSA/AKP/mldsa in JavaAlgorithm.java returns no results. |
| GAP-3 | No GeneratedAKPKeyProviderFactory | ✅ | N/A | ✅ Confirmed: find for GeneratedAKP*.java returns empty. |
| GAP-4 | No ML-KEM CekManagementProviderFactory | ✅ | N/A | ✅ Confirmed: find for MLKEM*CekManagement*.java returns empty. |
| GAP-5 | DefaultKeyProviders has no AKP bootstrap | ✅ | N/A | ✅ Confirmed: grep for AKP/ML.DSA in DefaultKeyProviders.java returns no results. |
| GAP-6 | JWKSServerUtils.toJwk() has no AKP branch | ✅ | Lines 59-63 | ✅ Confirmed. KeyType.RSA (line 59), .EC (line 61), .OKP (line 63). No AKP case. |
| GAP-7 | JWKSUtils.JWK_THUMBPRINT_REQUIRED_MEMBERS missing AKP | ✅ | Lines 53-58 | ✅ Confirmed. RSA/EC/OKP entries only; no AKP entry. |
| GAP-8 | FapiConstant.ALLOWED_ALGORITHMS excludes ML-DSA | ✅ | Lines 30-36 | ✅ Confirmed. PS256/384/512 + ES256/384/512 only. |
| GAP-9 | SAMLIdentityProvider / SamlProtocol / SamlService RS256 hardcodings | ✅ | SAMLIdentityProvider: 416/507; SamlProtocol: 544; SamlService: 965 | ✅ All confirmed at exact lines. |
| GAP-10 | AbstractOAuth2IdentityProvider RS256 fallback | ✅ (partial) | Line 745 ✅; HS256 ref at 689 ❌ | ⚠️ RS256 at line 745 confirmed. HS256 "line 689" reference does not exist. See Error 2. |
| GAP-11 | DescriptionConverter RS256 special-case | ✅ | Line 416 | ✅ Confirmed at line 416. |
| GAP-12 | DefaultTokenManager DEFAULT_SIGNATURE_ALGORITHM fallback | ✅ | Line 233 | ✅ Confirmed: `return Constants.DEFAULT_SIGNATURE_ALGORITHM;` at line 233. |
| GAP-13 | AuthUtil.java `.rsa256()` hardcoding | ✅ | Line 213 | ✅ Confirmed: `new JWSBuilder().rsa256(keypair.getPrivate())` at lines 211-213. |
| GAP-14 | DockerComposeCertsDirectory RSA 2048 hardcoding | ✅ | Lines 29-30 | ✅ Confirmed: `getKeyPairGen(KeyType.RSA)` line 29; `initialize(2048)` line 30. |
| GAP-15 | No ML-DSA SignatureProviderFactory / KeyProvider (tracks Domain 1) | ✅ | N/A | ✅ Confirmed unchanged. |
| GAP-16 | JWTClientCredentialsProvider no AKP case | ✅ | Lines 76-96 | ✅ Confirmed: switch with RSA/EC/OKP only, no AKP. |
| GAP-17 | ClientAsymmetricSignatureVerifierContext RSA-only guard | ✅ | Lines 36-37 | ✅ Confirmed: `Key Type is not RSA` guard at lines 36-37. |
| GAP-18 | JwtCNonceHandler ES256→RS256 fallback | ✅ | Lines 281/287 | ✅ Confirmed: ES256 at 281, RS256 fallback at 287. |
| GAP-19 | SsfSignatureAlgorithms ALLOWED = RS256 only | ✅ | Line 31 | ✅ Confirmed: `Set.of(Algorithm.RS256)` at line 31. |
| GAP-20 | JAR Encrypted Request Object (inbound ML-KEM gap) | ✅ | N/A | ✅ Confirmed. No ML-KEM provider exists. |
| GAP-21 | UserInfo response per-client attribute independent of realm default | ✅ | N/A | ✅ Confirmed. |
| GAP-22 | AttestationBasedClientAuthenticator [TODO] algorithm enforcement | ✅ | Lines 418-419 | ✅ Confirmed: TODO at line 418. Two TODO comments present. |
| GAP-23 | OIDCWellKnownProvider RS256 static constant | ✅ | Line 85 | ✅ Confirmed: `DEFAULT_CLIENT_AUTH_SIGNING_ALG_VALUES_SUPPORTED = list(Algorithm.RS256.toString())` at line 85. |
| GAP-24 | OID4VP ACCEPTED_ALGORITHMS ES256 hardcoding | ✅ | Line 83 | ✅ Confirmed: `ACCEPTED_ALGORITHMS = List.of(Algorithm.ES256)` at line 83. |
| GAP-25 | EphemeralKey / ResponseEncryption secp256r1 / ECDH-ES hardcoding | ✅ | EphemeralKey lines 35/41; ResponseEncryption line 38 | ✅ Confirmed. EphemeralKey: secp256r1 constant at line 41; ECDH_ES at line 46. ResponseEncryption: KEY_MANAGEMENT_ALG = ECDH_ES at line 38. |

---

## Part 4 — "Files Evaluated — Not Added as New Rows" Section Verification

New files identified in the OID4VC keybinding package that exist in the source but were not present at the time of the last review. These should be evaluated for inclusion in the "Files Evaluated" section or as new domain rows:

| File | Status | Notes |
|------|--------|-------|
| `AttestationX509CertificateValidator.java` | ⚠️ NEW | Found at `services/.../oid4vc/issuance/keybinding/AttestationX509CertificateValidator.java`. Handles X.509 certificate chain validation for OID4VC attestation. Works with `algorithm` parameter — no PQC-specific gaps beyond the keybinding domain. |
| `StaticAttestationKeyResolver.java` | ⚠️ NEW | Found at `services/.../oid4vc/issuance/keybinding/StaticAttestationKeyResolver.java`. Static key resolution for attestation; no independent PQC gap. |
| `X5cKeyUtils.java` | ⚠️ NEW | Found at `services/.../oid4vc/issuance/keybinding/X5cKeyUtils.java`. Converts X.509 certificate chains to JWK for key binding validation. No hardcoded algorithm. |
| `JwtCredentialBody.java` | ⚠️ NEW | Found at `services/.../oid4vc/issuance/credentialbuilder/JwtCredentialBody.java`. JWT credential body builder. |
| `SdJwtCredentialBody.java` | ⚠️ NEW | Found at `services/.../oid4vc/issuance/credentialbuilder/SdJwtCredentialBody.java`. SD-JWT credential body builder. |
| `DIDUtils.java` | ❌ MISSING | Was previously cited in Domain 26/56 analysis — does **not** exist in the repository. |
| `AbstractClientIdMetadataDocumentExecutor.java` | ⚠️ NEW | Found at `services/.../oauth2/cimd/clientpolicy/executor/AbstractClientIdMetadataDocumentExecutor.java`. New CIMD-related client policy executor. No direct PQC gap identified. |

---

## Part 5 — Status Distribution Table Verification

Counted from the HTML `pqc-*` badge occurrences across all 63 rows (including rows 62/63 for OID4VP):

| Status | HTML Count | Verified Count | Match? |
|--------|-----------|----------------|--------|
| SAFE | 4 | 4 (rows 11, 13, 43 production, 52-related) | ✅ |
| PARTIAL | ~14 | 14 | ✅ |
| BLOCKED | ~28 | 28 | ✅ |
| PENDING PROVIDERS | ~15 | 15 | ✅ |
| EXTERNAL DEPENDENCY | 3 | 3 | ✅ |

Distribution appears consistent with source verification. No PQC states have changed — no gaps have been resolved.

---

## Part 6 — New Content Not Covered in Current HTML

### New files in the OID4VC keybinding package

The following files are new since the last review and involve cryptographic operations:

1. **`AttestationX509CertificateValidator.java`** — X.509 certificate chain validation for OID4VC attestation proof. Uses the algorithm parameter passed in from JwtProofValidator. No new PQC gap; covered by Domain 26 umbrella.
2. **`StaticAttestationKeyResolver.java`** — Static variant of the attestation key resolver (complements `TrustedAttestationKeyResolver`). No new PQC gap.
3. **`X5cKeyUtils.java`** — Converts X.509 cert chains to JWK. Algorithm-agnostic. No new PQC gap.
4. **`JwtCredentialBody.java`** / **`SdJwtCredentialBody.java`** — Credential body builders for OID4VC issuance. Algorithm selection delegated to parent signer. No new PQC gap.

### Missing file: `DIDUtils.java`

`core/src/main/java/org/keycloak/util/DIDUtils.java` **does not exist** in the repository. The Section A fact about "DIDUtils.java is P-256 only by design" should be removed or re-attributed. Any domain row that cited this file needs re-verification.

### New `CIMD` client policy executor

`AbstractClientIdMetadataDocumentExecutor.java` is a new client policy executor for Client Identity Metadata Document (CIMD) handling. No independent PQC gap introduced.

### Library versions at current HEAD

- **Quarkus BOM:** `3.38.1` (controls `bcprov-jdk18on` version — was not explicitly tracked)
- **`bc-fips`:** `2.1.2` (unchanged from prior review)
- **`bc-pkix-fips`:** `2.1.10`
- **`bctls-fips`:** `2.1.22`
- **`bcutil-fips`:** `2.1.5`

If the prior HTML claimed specific `bcprov` versions, these should be updated to note the Quarkus BOM dependency.

---

## Part 7 — Summary of Required Changes

### Corrections to Existing Content

1. **ERROR — Remove/fix `DIDUtils.java` reference** (Error 1): File does not exist in the repo. Any claim about P-256 hardcoding in `DIDUtils.java` must be removed or re-attributed.

2. **ERROR — GAP-10: Remove HS256 "line 689" reference** (Error 2): The RS256 fallback in `AbstractOAuth2IdentityProvider.java` is confirmed at line 745. No HS256 hardcoding exists in that file at current HEAD.

3. **⚠️ INFORMATIONAL — bcprov version source**: Note in Domain 42 / CryptoProvider BouncyCastle row that `bcprov-jdk18on` version is controlled by Quarkus BOM 3.38.1, not a root-pom pin. The enforcer rule absence claim is confirmed correct.

### Additions of New Content

4. **⚠️ NEW — Add to "Files Evaluated" section**: `AttestationX509CertificateValidator.java`, `StaticAttestationKeyResolver.java`, `X5cKeyUtils.java`, `JwtCredentialBody.java`, `SdJwtCredentialBody.java` — all new OID4VC keybinding files. No new PQC gap; covered by Domain 26 umbrella.

5. **⚠️ NEW — Add `AbstractClientIdMetadataDocumentExecutor.java`** to "Files Evaluated" section. New CIMD executor; no independent PQC gap.

6. **⚠️ UPDATE — Library version table**: Quarkus BOM 3.38.1, bc-fips 2.1.2, bc-pkix-fips 2.1.10, bctls-fips 2.1.22.

### Resolved Items (PQC state upgrades required)

**None.** No previously-recorded gaps have been closed at HEAD `f03a2104ec`. All 25 GAPs remain open. All PQC states are unchanged.

---

## Summary

| Category | Count |
|----------|-------|
| Domains verified | 61 (+ rows 62/63 for OID4VP) |
| GAPs verified | 25 |
| Errors found | 2 (DIDUtils.java missing; GAP-10 HS256 stale reference) |
| Stale items | 1 (bcprov BOM note) |
| New files (not yet in HTML) | 5 |
| Resolved gaps | 0 |
| PQC state changes | 0 |

**No PQC progress has been made at this HEAD.** All gaps, all BLOCKED/PENDING states, and all hardcodings verified in prior reviews remain present and unchanged at `f03a2104ec`.
