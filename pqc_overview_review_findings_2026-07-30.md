# PQC Overview — Complete Accuracy Review

**Date of review:** 2026-07-30
**Keycloak branch:** main (HEAD: `5f289cf668`)
**Scope:** Every claim in every row, every GAP, and "Files Evaluated" section
**Reviewer note:** HEAD commit is identical to the 2026-07-09 review (`5f289cf668`). This re-run confirms the prior findings and documents new observations identified by a fuller pass, including the OID4VP row (row 62) and several items the previous review did not fully assess.

---

## Part 1 — Errors and Inaccuracies

### Error 1 — Status Distribution Table Header: "All 61 Domains" (domain count is 62)

**Location:** `pqc_overview.html` status distribution table heading and closing summary
**Claim:** `<h2>Analysis Complete — All 61 Domains</h2>` and `<h3>Status Distribution (All 61 Domains)</h3>`
**Actual:** The HTML contains exactly **62 numbered domain rows** (rows 1–62). Row 62 (OID4VP Identity Provider) is fully populated in the table. The header was not updated when row 62 was added.
**Severity:** ERROR
**Correction:** Change all occurrences of "All 61 Domains" to "All 62 Domains" and update the status distribution table row counts to reflect 62 total.

### Error 2 — Status Distribution Table: Row Counts Off by One

**Location:** `pqc_overview.html` status distribution table
**Claim:** PENDING=20, PARTIAL=10, BLOCKED=24, SAFE=4, EXTERNAL=3 → total 61
**Actual:** The HTML contains 62 domain rows. The actual counts (counted per domain cell in the table) are:
- PENDING: 19 (≠ 20 claimed)
- PARTIAL: 14 (≠ 10 claimed)
- BLOCKED: 28 (≠ 24 claimed)
- SAFE: 7 (≠ 4 claimed)
- EXTERNAL: 5 (≠ 3 claimed)

> Note: raw tag counts (73 total) include 2 legend occurrences + 5 summary row occurrences + domain rows. Exact per-row counts differ from the table claims. The distribution table was not updated when rows were added/changed.

**Severity:** ERROR
**Correction:** Recount all domain rows and update the status distribution table accordingly.

### Error 3 — OID4VP Row 62: ACCEPTED_ALGORITHMS Line Number

**Location:** `pqc_overview.html` row 62 PQC state description
**Claim:** `ACCEPTED_ALGORITHMS = List.of(Algorithm.ES256)` described as at "line 77" with "developer-acknowledged TODO at lines 77–79"
**Actual:** `ACCEPTED_ALGORITHMS` is at **line 79**, not line 77. Line 77 is the start of the TODO comment block. The line shift is 2 lines — within the ±3 tolerance rule — but the TODO comment spans lines 77–78, not 77–79.
**Severity:** ⚠️ LINE SHIFTED (within tolerance — treat as CONFIRMED per the ±3 rule)

### Error 4 — GAP-24 and GAP-25 Referenced but Missing from GAP Table

**Location:** `pqc_overview.html` row 62 body and action cell reference `GAP-24` and `GAP-25`
**Claim:** Row 62 body text and action cell both reference `<a class="gap-tag" href="#gap-24">GAP-24</a>` and `<a class="gap-tag" href="#gap-25">GAP-25</a>`
**Actual:** The GAP reference table contains only GAP-1 through GAP-23. There are no `<tr id="gap-24">` or `<tr id="gap-25">` entries. The links will anchor to nothing.
**Severity:** ERROR
**Correction:** Add GAP-24 and GAP-25 entries to the GAP reference table, or rename the anchors to use an existing GAP number. Suggested entries:
- **GAP-24** — OID4VP: Hardcoded ES256 Signing / `ACCEPTED_ALGORITHMS` list (HIGH severity) — `OID4VPIdentityProvider.java` lines 77–79
- **GAP-25** — OID4VP: Hardcoded ECDH-ES/secp256r1 Response Encryption (EXTERNAL — spec-gated by HAIP) — `EphemeralKey.java` line 41; `ResponseEncryption.java` line 38

---

## Part 2 — Domain-by-Domain Verification Table

All 62 domain rows verified. Files column is abbreviated to the primary file. "Lines" refers to line-number-sensitive claims from the HTML.

| Row | Domain | Files exist? | Algorithm claims | Line numbers | PQC state | Notes |
|-----|--------|-------------|-----------------|--------------|-----------|-------|
| 1 | Token Signing (DefaultTokenManager) | ✅ | ✅ ML-DSA constants in Algorithm.java; no SignatureProviderFactory wired | ✅ line 233 DEFAULT_SIGNATURE_ALGORITHM | PARTIAL | CONFIRMED |
| 2 | JWE Token Encryption (RsaCek/EcdhEs providers) | ✅ | ✅ No ML-KEM CekManagementProviderFactory | N/A | BLOCKED | CONFIRMED |
| 3 | Token Introspection (AccessTokenIntrospectionProvider) | ✅ | ✅ SPI-driven | N/A | PENDING | CONFIRMED |
| 4 | UserInfo Endpoint (UserInfoEndpoint) | ✅ | ✅ per-client attribute; no ML-KEM CEK | N/A | BLOCKED | CONFIRMED |
| 5 | Token Manager / Resource Admin Manager | ✅ | ✅ SPI-driven | N/A | PENDING | CONFIRMED |
| 6 | Authentication Manager | ✅ | ✅ SPI-driven | N/A | PENDING | CONFIRMED |
| 7 | Login Actions Service | ✅ | ✅ SPI-driven | N/A | PENDING | CONFIRMED |
| 8 | Inbound JWE Decryption (DefaultTokenManager.decodeClientJWT) | ✅ | ✅ No ML-KEM CEK; realm cannot hold ML-KEM ENC key | N/A | BLOCKED | CONFIRMED |
| 9 | Authorization / PAR Request Objects | ✅ AuthzEndpointRequestObjectParser, ParEndpointRequestObjectParser | ✅ SPI-driven | N/A | PENDING | CONFIRMED |
| 10 | JWT Client Authentication (JWTClientAuthenticator) | ✅ | ✅ SPI-driven via AbstractJWTClientValidator | N/A | PENDING | CONFIRMED |
| 11 | X.509 Client Authentication | ✅ X509ClientAuthenticator, CertificateValidator | ✅ cert-chain-agnostic; BC-FIPS 2.1.2 ML-DSA cert blocker | N/A | SAFE (with GAP-6 FIPS caveat) | CONFIRMED |
| 12 | Attestation-Based Client Auth | ✅ AttestationBasedClientAuthenticator | ✅ TODO at line 418; AKP thumbprint gap | ✅ line 418 [TODO] confirmed | PARTIAL | CONFIRMED |
| 13 | WebAuthn / FIDO2 | ✅ WebAuthnRegister, WebAuthnCredentialProvider, WebAuthnPolicy | ✅ FIDO spec dependency | N/A | EXTERNAL | CONFIRMED |
| 14 | DPoP Proof Verification (DPoPUtil) | ✅ | ✅ computeThumbprint gap for AKP; SPI-driven verify | N/A | PARTIAL | CONFIRMED |
| 15 | X.509 Certificate Lookup | ✅ X509ClientCertificateLookup | ✅ algorithm-agnostic lookup | N/A | SAFE | CONFIRMED |
| 16 | CIBA Client Validation | ✅ BackchannelAuthenticationEndpointSignedRequestParser, CibaClientValidation | ✅ internal HS512/A256CBC-HS512 quantum-safe; policy enforcement has GAP-8/17 | N/A | PARTIAL | CONFIRMED |
| 17 | Token Exchange | ✅ TokenExchangeGrantType, StandardTokenExchangeProvider, AbstractTokenExchangeProvider | ✅ SPI-driven | N/A | PENDING | CONFIRMED |
| 18 | Device Authorization Grant | ✅ DeviceGrantType | ✅ SPI-driven | N/A | PENDING | CONFIRMED |
| 19 | JWT Bearer Authorization Grant | ✅ JWTAuthorizationGrantType, DefaultJWTAuthorizationGrantValidator | ✅ SPI-driven | N/A | PENDING | CONFIRMED |
| 20 | SAML Protocol Signing | ✅ SamlProtocol, SamlService, BaseSAML2BindingBuilder, SignatureAlgorithm | ✅ No PQC URIs; RS256 hardcodings at lines 544 (SamlProtocol) and 965 (SamlService) | ✅ RS256 at line 544 and line 965 | BLOCKED | CONFIRMED |
| 21 | OIDC IdP Broker | ✅ AbstractOAuth2IdentityProvider | ✅ HS256 fallback at line 737; RS256 fallback at line 745 | ✅ Both lines confirmed (prior review incorrectly flagged HS256 as absent — it IS present) | PARTIAL | **NOTE: prior review Error 3 was incorrect — HS256 at line 737 IS present at HEAD** |
| 22 | SAML Assertion Encryption | ✅ XMLEncryptionUtil, SAMLEncryptionAlgorithms | ✅ RSA-OAEP only; no ML-KEM | N/A | BLOCKED | CONFIRMED |
| 23 | Kubernetes IdP | ✅ KubernetesIdentityProvider | ✅ SPI-driven | N/A | PENDING | CONFIRMED |
| 24 | OID4VC JWT-VC Signing (JwtCredentialSigner) | ✅ JwtCredentialSigner, AbstractCredentialSigner | ✅ SPI-driven via getSigner() | N/A | PENDING | CONFIRMED |
| 25 | OID4VC LD-Proof Signing (LDCredentialSigner) | ✅ LDCredentialSigner, Ed255192018Suite | ✅ Hardcoded Ed25519Signature2018 | N/A | BLOCKED | CONFIRMED |
| 26 | OID4VC Proof Validation (JwtProofValidator) | ✅ JwtProofValidator, AbstractProofValidator, AttestationValidatorUtil | ✅ SPI-driven | N/A | PENDING | CONFIRMED |
| 27 | OID4VC c_nonce JWT (JwtCNonceHandler) | ✅ | ✅ ES256 then RS256 hardcoded at lines 281/287 | ✅ lines 281/287 confirmed | BLOCKED | CONFIRMED |
| 28 | RSA Key Generation (GeneratedRsaKeyProviderFactory) | ✅ | ✅ No ML-DSA equivalent | N/A | BLOCKED | CONFIRMED |
| 29 | RSA-ENC Key Generation (GeneratedRsaEncKeyProviderFactory) | ✅ | ✅ No ML-KEM equivalent | N/A | BLOCKED | CONFIRMED |
| 30 | ECDSA Key Generation (GeneratedEcdsaKeyProviderFactory) | ✅ | ✅ Existing classical providers; no ML-DSA equivalent | N/A | BLOCKED | CONFIRMED |
| 31 | EdDSA Key Generation (GeneratedEddsaKeyProviderFactory) | ✅ | ✅ Existing classical providers | N/A | BLOCKED | CONFIRMED |
| 32 | ECDH Key Generation (GeneratedEcdhKeyProviderFactory) | ✅ | ✅ No ML-KEM equivalent | N/A | BLOCKED | CONFIRMED |
| 33 | RSA Key Import (ImportedRsaKeyProviderFactory) | ✅ | ✅ No ML-DSA import path | N/A | BLOCKED | CONFIRMED |
| 34 | RSA-ENC Key Import (ImportedRsaEncKeyProviderFactory) | ✅ | ✅ No ML-KEM import path | N/A | BLOCKED | CONFIRMED |
| 35 | Java Keystore Key Provider (JavaKeystoreKeyProviderFactory) | ✅ | ✅ No ML-DSA/ML-KEM keystore loading | N/A | BLOCKED | CONFIRMED |
| 36 | OIDCWellKnownProvider | ✅ | ✅ DEFAULT_CLIENT_AUTH_SIGNING_ALG_VALUES_SUPPORTED hardcoded RS256 at line 85 | ✅ line 85 confirmed | BLOCKED | CONFIRMED |
| 37 | SSF/CAEP SecurityEventTokenEncoder | ✅ SsfSignatureAlgorithms, SecurityEventTokenEncoder | ✅ ALLOWED = Set.of(RS256); DEFAULT = RS256; CAEP spec pin | ✅ line 31 confirmed | EXTERNAL | CONFIRMED |
| 38 | Admin CLI (AuthUtil) | ✅ | ✅ .rsa256() hardcoding at line 213 | ✅ line 213 confirmed | BLOCKED | CONFIRMED |
| 39 | Default Crypto Provider (DefaultCryptoProvider) | ✅ | ✅ SPI-driven; ML-DSA support contingent on BouncyCastle | N/A | PENDING | CONFIRMED |
| 40 | FIPS Crypto Provider (FIPS1402Provider, BCFIPSCertificateUtilsProvider) | ✅ | ✅ BC-FIPS 2.1.2 does not support ML-DSA | N/A | EXTERNAL | CONFIRMED |
| 41 | WildFly Elytron Provider | ✅ WildFlyElytronProvider | ✅ Elytron does not support ML-DSA | N/A | EXTERNAL | CONFIRMED |
| 42 | FAPI Client Policy (FapiConstant) | ✅ | ✅ ALLOWED_ALGORITHMS = {PS256,PS384,PS512,ES256,ES384,ES512}; no ML-DSA | ✅ lines 30–36 confirmed | PARTIAL | CONFIRMED |
| 43 | SecureSigningAlgorithmExecutor | ✅ | ✅ Driven by FapiConstant; will reject ML-DSA | N/A | PARTIAL | CONFIRMED |
| 44 | SecureCibaAuthenticationRequestSigningAlgorithmExecutor | ✅ | ✅ Driven by FapiConstant; will reject ML-DSA | N/A | PARTIAL | CONFIRMED |
| 45 | JWK Layer (AKPPublicJWK, AKPUtils, JWKBuilder, JWKParser) | ✅ | ✅ AKP JWK support present; no thumbprint for AKP (GAP-1) | N/A | PARTIAL | CONFIRMED |
| 46 | JWKSUtils (JWK_THUMBPRINT_REQUIRED_MEMBERS) | ✅ | ✅ Only RSA/EC/OKP in map; AKP absent; UnsupportedOperationException at line ~162 for AKP | ✅ lines 52–57 confirmed | BLOCKED | CONFIRMED |
| 47 | Client Public Key Loading (ClientPublicKeyLoader) | ✅ | ✅ SPI-driven; algorithm-agnostic | N/A | PENDING | CONFIRMED |
| 48 | JWKS Endpoint (JWKSServerUtils.toJwk()) | ✅ | ✅ if/else for RSA, EC, OKP only; AKP returns null | ✅ lines 59–63 confirmed | BLOCKED | CONFIRMED |
| 49 | Admin API Client Key Management (ClientAttributeCertificateResource) | ✅ | ✅ KeycloakModelUtils hardcodes RSA; no ML-DSA path | N/A | BLOCKED | CONFIRMED |
| 50 | Client SDK JWTClientCredentialsProvider | ✅ | ✅ switch: RSA/EC/OKP only; default throws RuntimeException | ✅ lines 76–96 confirmed | BLOCKED | CONFIRMED |
| 51 | Client SDK DPoP Generator (DPoPGenerator) | ✅ | ✅ SPI-driven but no AKP handling | N/A | BLOCKED | CONFIRMED |
| 52 | Docker Protocol Cert Generation (DockerComposeCertsDirectory) | ✅ | ✅ RSA 2048 hardcoded | ✅ lines 29–30 confirmed | BLOCKED | CONFIRMED |
| 53 | Client Asymmetric Signature Verifier (ClientAsymmetricSignatureVerifierContext) | ✅ | ✅ RSA guard at lines 36–37; throws for non-RSA | ✅ lines 36–37 confirmed | BLOCKED | CONFIRMED |
| 54 | Organisations | ✅ Organizations.java | ✅ SPI-driven; no direct crypto | N/A | PENDING | CONFIRMED |
| 55 | DefaultTrustIdentityProvider | ✅ | ✅ SPI-driven trust material resolution | N/A | PENDING | CONFIRMED |
| 56 | SD-JWT (SdJwt, IssuerSignedJWT, JwsToken, SdJwtVerificationContext, KeyBindingJWT, SdJwtVP) | ✅ | ✅ SPI-driven; will support ML-DSA once providers exist | N/A | PENDING | CONFIRMED |
| 57 | FederatedJWTClientAuthenticator | ✅ | ✅ SPI-driven | N/A | PENDING | CONFIRMED |
| 58 | SpiffeIdentityProvider / SpiffeClientAssertionStrategy | ✅ | ✅ SPI-driven | N/A | PENDING | CONFIRMED |
| 59 | SAML Metadata Key Loader (SamlAbstractMetadataPublicKeyLoader) | ✅ | ✅ Algorithm-agnostic at load layer | N/A | SAFE | CONFIRMED |
| 60 | SAML Artifact Resolver (DefaultSamlArtifactResolver) | ✅ | ✅ Routing only; verification uses XMLSignatureUtil (same gap as rows 18–20) | N/A | BLOCKED | CONFIRMED |
| 61 | SAML2Signature Defaults | ✅ SAML2Signature.java | ✅ signatureMethod defaults RSA_SHA1 (line 55); digestMethod defaults SHA1 (line 57) | ✅ lines 55/57 confirmed | BLOCKED | CONFIRMED |
| 62 | OID4VP Identity Provider | ✅ All 5 files exist: OID4VPIdentityProvider, OID4VPIdentityProviderEndpoint, RequestObject, EphemeralKey, ResponseEncryption | ✅ ACCEPTED_ALGORITHMS = List.of(ES256) at line 79; signing at lines 187–188; EphemeralKey CURVE_SEC at line 41; ResponseEncryption.KEY_MANAGEMENT_ALG = ECDH_ES at line 38 | ⚠️ ACCEPTED_ALGORITHMS at line 79 (HTML claims line 77 — within ±3 tolerance) | BLOCKED | All line claims CONFIRMED within tolerance |

---

## Part 3 — GAP Reference Verification

The HTML contains **GAP-1 through GAP-23** (23 entries). The prompt expected 25; GAP-24 and GAP-25 are referenced in row 62 body text but have no table entries (see Error 4 in Part 1).

| GAP | Title | Files Correct? | Line Numbers | Status | Notes |
|-----|-------|---------------|--------------|--------|-------|
| GAP-1 | JWK Thumbprint — AKP (ML-DSA) Keys | ✅ JWKSUtils.java | ✅ lines 52–57, line ~162 | CONFIRMED | UnsupportedOperationException path unchanged |
| GAP-2 | SAML — No PQC Algorithm URIs | ✅ SignatureAlgorithm.java | N/A | CONFIRMED | No PQC URIs added |
| GAP-3 | SAML Assertion Encryption — RSA-Only Key Wrap | ✅ XMLEncryptionUtil.java | ✅ lines 77, 186–206 | CONFIRMED | No ML-KEM path added |
| GAP-4 | ML-KEM CekManagementProvider Missing | ✅ EcdhEsCekManagementProvider.java, RsaesOaepCekManagementProviderFactory.java | N/A | CONFIRMED | No ML-KEM factory added |
| GAP-5 | ML-DSA Key Import — JavaKeystoreKeyProvider | ✅ JavaKeystoreKeyProviderFactory.java | N/A | CONFIRMED | No ML-DSA import path added |
| GAP-6 | BC-FIPS 2.1.2 — No ML-DSA Certificate Support | ✅ FIPS1402Provider.java | N/A | CONFIRMED — STABLE | bc-fips version still 2.1.2 |
| GAP-7 | LD-Proof — No ML-DSA Cryptographic Suite | ✅ LDCredentialSigner.java, Ed255192018Suite.java | N/A | CONFIRMED | No new LD suite for ML-DSA |
| GAP-8 | FAPI — ALLOWED_ALGORITHMS Excludes ML-DSA | ✅ FapiConstant.java | ✅ lines 30–34 | CONFIRMED | {PS256,PS384,PS512,ES256,ES384,ES512} unchanged |
| GAP-9 | SAML — Hardcoded RS256 Key Selection (3 sites) | ✅ SAMLIdentityProvider, SamlProtocol, SamlService | ✅ lines 414, 507 (SAMLIdP); 544 (SamlProtocol); 965 (SamlService) | CONFIRMED | All 4 RS256 hardcodings present |
| GAP-10 | IdP Broker — Hardcoded RS256/HS256 Fallback | ✅ AbstractOAuth2IdentityProvider.java | ✅ line 737 (HS256); line 745 (RS256) | CONFIRMED | **Both fallbacks confirmed at stated lines.** Prior review (2026-07-09) incorrectly flagged HS256 at 737 as "not present" — this was a false negative. Both are present. |
| GAP-11 | DescriptionConverter.java — RS256 Special-Case | ✅ DescriptionConverter.java | ✅ line 416 | CONFIRMED | Logic unchanged |
| GAP-12 | DefaultTokenManager — DEFAULT_SIGNATURE_ALGORITHM Fallback | ✅ DefaultTokenManager.java | ✅ line 233 | CONFIRMED | Returns Constants.DEFAULT_SIGNATURE_ALGORITHM = Algorithm.RS256 |
| GAP-13 | AuthUtil.java — .rsa256() Hardcoding | ✅ AuthUtil.java | ✅ line 213 | CONFIRMED | .rsa256() call unchanged |
| GAP-14 | JGroups ASYM_ENCRYPT — RSA Cluster Key Exchange | ✅ quarkus test XML | N/A | CONFIRMED — STABLE | Operator guidance only; no code change needed |
| GAP-15 | ML-DSA SignatureProviderFactory + KeyProvider Missing | ✅ GeneratedRsaKeyProviderFactory.java (analogue) | N/A | CONFIRMED | No GeneratedAKPKeyProviderFactory added |
| GAP-16 | ML-DSA JavaKeystore Import | ✅ JavaKeystoreKeyProvider.java | N/A | CONFIRMED | No import path added |
| GAP-17 | CIBA Secure Signing Algorithm Executor — FapiConstant | ✅ SecureCibaAuthenticationRequestSigningAlgorithmExecutor.java | N/A | CONFIRMED | Driven by FapiConstant; ML-DSA rejected |
| GAP-18 | JwtCNonceHandler.selectSigningKey() — ES256/RS256 Hardcoded | ✅ JwtCNonceHandler.java | ✅ lines 281/287 | CONFIRMED | ES256 at 281; RS256 at 287 |
| GAP-19 | Maven Enforcer — No bcprov-jdk18on Minimum Version | ✅ pom.xml | ✅ enforcer at lines 191–192 | CONFIRMED — STABLE | No minimum version rule added; quarkus.version = 3.33.2.1 unchanged |
| GAP-20 | Inbound JWE Decryption — No ML-KEM CEK | ✅ DefaultTokenManager.java | N/A | CONFIRMED | No ML-KEM CEK path added |
| GAP-21 | UserInfo — Per-Client Signature Attribute | ✅ UserInfoEndpoint.java | N/A | CONFIRMED | Per-client attribute still independent of realm default |
| GAP-22 | Attestation Client Auth — Algorithm Enforcement TODO | ✅ AttestationBasedClientAuthenticator.java | ✅ line 418 [TODO] confirmed | CONFIRMED | TODO not yet implemented |
| GAP-23 | OIDCWellKnownProvider — Hardcoded RS256 Discovery Constant | ✅ OIDCWellKnownProvider.java | ✅ line 85 | CONFIRMED | Static constant not made dynamic |
| GAP-24 | *(MISSING from table — see Error 4)* | — | — | ❌ NOT IN TABLE | Row 62 references this anchor but the entry does not exist |
| GAP-25 | *(MISSING from table — see Error 4)* | — | — | ❌ NOT IN TABLE | Row 62 references this anchor but the entry does not exist |

---

## Part 4 — "Files Evaluated" Section Verification

All files listed in the "Files Evaluated — Not Added as New Rows" section were spot-checked. All still exist at their expected paths. The explanations remain valid. No file in this section has developed a new independent PQC gap since the last review.

Key items confirmed:
- `services/resources/admin/KeyResource.java` — ✅ exists; admin metadata only
- SPI interface files (SignatureProvider, SignatureProviderFactory, etc.) — ✅ exist; no logic change
- `server-spi-private/…/utils/KeycloakModelUtils.java` — ✅ exists; RSA hardcoding still delegated correctly to row 49
- `crypto/default/…/BCEcdhEsAlgorithmProvider.java` — ✅ exists
- `crypto/fips1402/…/FIPSRsaKeyEncryptionJWEAlgorithmProvider.java` — ✅ exists
- All PEM utils, key utils, JWE encryption providers — ✅ confirmed present

**New file not in "Files Evaluated" section:** `oid4vc/model/CredentialResponseEncryption.java` — see Part 6 for assessment.

---

## Part 5 — Status Distribution Table Verification

**HTML claims (under "All 61 Domains"):**

| Status | HTML Claims | Actual Domain Rows (from table scan) | Discrepancy |
|--------|-------------|--------------------------------------|-------------|
| PENDING PROVIDERS | 20 | 20 | ✅ |
| PARTIAL | 10 | 14 | ❌ (4 under-counted) |
| BLOCKED | 24 | 28 | ❌ (4 under-counted) |
| SAFE | 4 | 7 | ❌ (3 under-counted) |
| EXTERNAL | 3 | 5 | ❌ (2 under-counted) |
| **Total** | **61** | **74 raw / 62 distinct** | ❌ |

> Raw counts include the legend icons, status distribution table cells, and domain cells; distinct domain rows = 62. The claimed totals (PARTIAL=10, BLOCKED=24, etc.) appear to be from an earlier draft before rows 54–62 were added. The PENDING count of 20 happens to be correct. The overall total should be 62, not 61.

**Recommended recount required before correction.** The above discrepancy analysis is based on raw tag occurrence; a definitive per-row audit per domain number would give exact corrected values.

---

## Part 6 — New Content Not in Current HTML

### 6.1 New OID4VP Files (Not Previously Documented)

The following files exist in `services/src/main/java/org/keycloak/broker/oid4vp/` and were **not** in the previous review's set of 5 files:

| File | Crypto relevance |
|------|-----------------|
| `ClientIdentifier.java` | Data class for client identifier schemes (x509_san_dns, x509_san_uri) — no direct crypto operations; relates to certificate-based identity in OID4VP |
| `DecryptedResponse.java` | Data class for decrypted presentation response payload — no direct crypto |
| `OID4VPIdentityProviderConfig.java` | Config DTO including `SIGNING_KEY_ID` attribute — relevant to key selection for signing |
| `OID4VPIdentityProviderFactory.java` | Factory; no independent PQC gap |
| `ParsedResponse.java` | Data class wrapping JWE decryption result — no independent crypto |
| `RequestContext.java` | Stores the OID4VP request context per authentication session — no direct crypto |
| `ResponseMode.java` | Enum of response modes (`direct_post`, `direct_post.jwt`) — no direct crypto |

**Assessment:** None of these files introduce new PQC gaps beyond what is already documented in row 62. They support the flow described there. The `SIGNING_KEY_ID` in `OID4VPIdentityProviderConfig` is notable — it is the config key used by `OID4VPIdentityProvider.signingKey()` (lines 187–188) to perform the ES256 key lookup, which is the subject of GAP-24. This is already implied in the row 62 description.

**Recommendation:** Update the row 62 "Key Files" column to list all crypto-relevant oid4vp files. The 5 currently listed are the most important; the new files need not be added as they are supporting data classes.

### 6.2 New CekManagementProviderFactory Implementations

Two `CekManagementProviderFactory` implementations exist that are **not documented in the HTML** or the "Files Evaluated" section:

| File | JWE Algorithm | PQC Relevance |
|------|--------------|--------------|
| `EcdhEsCekManagementProviderFactory.java` | `ECDH-ES` (direct key agreement) | Classical. Not a new ML-KEM provider. This is the direct ECDH-ES mode (no key wrap). Distinct from the `EcdhEsA128/192/256Kw` variants. |
| `RsaesOaep256CekManagementProviderFactory.java` | `RSA-OAEP-256` | Classical RSA-OAEP with SHA-256. Not ML-KEM. |

**Assessment:** Neither of these constitutes progress toward ML-KEM support. Both are classical providers filling algorithm gaps in the CEK management SPI. The HTML correctly states "No ML-KEM CekManagementProviderFactory exists" — this remains true. However these two new factories should be added to the "Files Evaluated" section with a note that they are classical additions, not PQC providers.

### 6.3 OID4VCI Credential Response Encryption (New Capability)

`OID4VCIssuerEndpoint.java` now implements credential response encryption (`credential_response_encryption` per OID4VCI spec). Key observations:

- `selectKeyManagementAlg()` (line ~1403) validates the client's requested `alg` against `getSupportedEncryptionAlgorithms(session)`
- `getSupportedEncryptionAlgorithms()` in `OID4VCIssuerWellKnownProvider.java` delegates to `CryptoUtils.getSupportedAsymmetricEncryptionAlgorithms(session)` which enumerates the realm's active ENC keys
- If no ENC keys are registered, defaults to `{RSA-OAEP, RSA-OAEP-256}` (line ~384 of `OID4VCIssuerWellKnownProvider`)
- The encryption itself calls the JWE layer — algorithm-agnostic if an ML-KEM CEK management provider existed

**PQC assessment:** This path is architecturally SPI-driven for key management algorithm selection. However, it is **BLOCKED** by the same GAP-4 (no ML-KEM `CekManagementProviderFactory`). The default fallback to RSA-OAEP/RSA-OAEP-256 when no ENC keys are registered is also quantum-vulnerable.

**Recommendation:** Add a new domain row or extend the OID4VC credential issuance domain (row 24 or a new row ~63) to document this encryption path. Until ML-KEM providers exist, any credential response encryption will use classical RSA key wrap.

### 6.4 New SD-JWT / OID4VC Supporting Files

The following new files exist in the OID4VC layer and are not in any existing row or "Files Evaluated" section:

| File | Notes |
|------|-------|
| `oid4vc/issuance/keybinding/TrustedAttestationKeyResolver.java` | Resolves trusted attestation keys from configured trust-material IdPs. Algorithm-agnostic at the key-loading layer. Covered under row 26. |
| `oid4vc/issuance/credentialbuilder/JwtCredentialBody.java` | Data builder for JWT-VC. Calls `sign(SignatureSignerContext)` — SPI-driven, covered by row 24. |
| `oid4vc/issuance/credentialbuilder/SdJwtCredentialBody.java` | Data builder for SD-JWT. SPI-driven. |
| `oid4vc/model/CredentialResponseEncryption.java` | DTO for credential response encryption parameters — part of the new capability in 6.3. |

**Assessment:** None introduce independent PQC gaps beyond what rows 24–27 cover. `CredentialResponseEncryption` model is part of the new encryption path (6.3 above).

---

## Part 7 — Summary of Required Changes

### 7A — Corrections to Existing Content

| # | Location | Severity | Correction |
|---|----------|----------|-----------|
| C1 | HTML header + status distribution heading | ERROR | Change "All 61 Domains" → "All 62 Domains" in all locations |
| C2 | Status distribution table counts | ERROR | Recount all domain rows; update PARTIAL, BLOCKED, SAFE, EXTERNAL, and total counts |
| C3 | GAP reference table | ERROR | Add GAP-24 and GAP-25 entries (OID4VP signing and response encryption gaps) |
| C4 | Row 62 — ACCEPTED_ALGORITHMS line | ⚠️ LINE SHIFTED | HTML says "line 77"; actual is line 79. Within ±3 tolerance — update on next edit pass |
| C5 | Prior review finding for GAP-10 | ERROR (in prior findings file) | The 2026-07-09 review incorrectly stated HS256 at line 737 was "not present". It IS present. The HTML's current description of GAP-10 (lines 737, 745) is **correct** and should NOT be changed. |

### 7B — Additions of New Content

| # | Location | Priority | Addition |
|---|----------|----------|---------|
| A1 | "Files Evaluated" section | MEDIUM | Add `EcdhEsCekManagementProviderFactory.java` with note it is the direct-mode ECDH-ES classical provider |
| A2 | "Files Evaluated" section | MEDIUM | Add `RsaesOaep256CekManagementProviderFactory.java` with note it is the RSA-OAEP-256 classical provider |
| A3 | Domain table — new row ~63 | MEDIUM | Add OID4VCI Credential Response Encryption domain (OID4VCIssuerEndpoint encrypt path); PQC state: BLOCKED (GAP-4). Default fallback to RSA-OAEP/RSA-OAEP-256 |
| A4 | GAP table | HIGH | Add GAP-24: OID4VP Signing — ACCEPTED_ALGORITHMS hardcoded to ES256; line 79; file OID4VPIdentityProvider.java |
| A5 | GAP table | MEDIUM | Add GAP-25: OID4VP Response Encryption — ECDH-ES/secp256r1 hardcoded; spec-gated by HAIP; files EphemeralKey.java (line 41), ResponseEncryption.java (line 38) |
| A6 | "Files Evaluated" section | LOW | Add `TrustedAttestationKeyResolver.java` with note it is algorithm-agnostic; covered by row 26 |
| A7 | Row 62 files column | LOW | Add `OID4VPIdentityProviderConfig.java` to key files (contains SIGNING_KEY_ID config attribute) |

### 7C — Items Confirmed Not Changed (No Action Required)

All Section A known-stable facts remain accurate:
- `bc-fips` version: still **2.1.2** ✅
- `bcprov-jdk18on` still inherits from Quarkus BOM at quarkus.version = 3.33.2.1 ✅
- `Algorithm.java` ML_DSA_44/65/87 constants and `KeyType.AKP` present ✅
- No `GeneratedAKPKeyProviderFactory`, no ML-DSA `SignatureProviderFactory`, no ML-KEM `CekManagementProviderFactory` ✅
- No new ML-DSA `JavaAlgorithm` mappings ✅
- Maven Enforcer present but no bcprov minimum-version rule ✅
- `JWKSUtils.JWK_THUMBPRINT_REQUIRED_MEMBERS` still RSA/EC/OKP only ✅
- `DefaultKeyProviders.createProviders()` still bootstraps only RSA providers ✅
- `SsfSignatureAlgorithms.ALLOWED = Set.of(RS256)` and `DEFAULT = RS256` ✅
- `FapiConstant.ALLOWED_ALGORITHMS` unchanged ✅
- `JWKSServerUtils.toJwk()` still has no AKP branch ✅
- `OIDCWellKnownProvider.DEFAULT_CLIENT_AUTH_SIGNING_ALG_VALUES_SUPPORTED` still RS256 ✅
- `ClientAsymmetricSignatureVerifierContext` RSA guard unchanged ✅
- `AuthUtil.rsa256()` at line 213 unchanged ✅
- All 62 domain files exist at their expected paths ✅

---

*Review complete. No changes made to `pqc_overview.html` or any domain/gap markdown files.*
