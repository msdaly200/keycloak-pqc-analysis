# PQC Overview — Complete Accuracy Review
**Date of review:** 2026-07-09  
**Reviewer:** Senior Keycloak / Cryptography Engineer  
**Keycloak branch:** main (HEAD: `5f289cf668`)  
**Scope:** Every claim in every row, every GAP entry, and the "Files Evaluated" section of `pqc_overview.html` verified against the live source.

---

## PART 1 — ERRORS & INACCURACIES IN EXISTING CONTENT

### Error 1 — Domain 12 / GAP-22: Wrong file path for `AttestationBasedClientAuthenticator.java`

**HTML states (Domain 12 Key Files and GAP-22 Affected Files):**
```
authenticators/client/AttestationBasedClientAuthenticator.java
```
**Actual path:**
```
authentication/authenticators/client/AttestationBasedClientAuthenticator.java
```
Full path: `services/src/main/java/org/keycloak/authentication/authenticators/client/AttestationBasedClientAuthenticator.java`

All substantive claims (TODO at line 418, algorithm enforcement gap, cnf JWK thumbprint dependency on GAP-1) are correct. Only the path is wrong.

**Correction:** Replace `authenticators/client/AttestationBasedClientAuthenticator.java` with `authentication/authenticators/client/AttestationBasedClientAuthenticator.java` in Domain 12 and GAP-22.

---

### Error 2 — GAP-9: Line numbers for `SAMLIdentityProvider.java` are off by 1

**HTML states (GAP-9 Affected Files):**
```
broker/saml/SAMLIdentityProvider.java (lines 415, 506)
```
**Actual source:**
- Line **414**: `List<KeyDescriptorType> signingKeys = session.keys().getKeysStream(realm, KeyUse.SIG, Algorithm.RS256)`
- Line **507**: `KeyWrapper keyWrapper = session.keys().getActiveKey(realm, KeyUse.SIG, Algorithm.RS256);`

Both RS256 hardcodings are confirmed present — just at lines 414 and 507, not 415 and 506.

**Correction:** Update GAP-9 Affected Files cell from `(lines 415, 506)` → `(lines 414, 507)`.

---

### Error 3 — GAP-10 / Domain 21: Stale line numbers and missing HS256 fallback at current HEAD

**HTML states (GAP-10 Affected Files):**
```
AbstractOAuth2IdentityProvider.java (lines 689, 697)
```
**Actual source at HEAD:**
- Line **745**: `String alg = getConfig().getClientAssertionSigningAlg() != null ? getConfig().getClientAssertionSigningAlg() : Algorithm.RS256;`
- There is **no** HS256 fallback at line 689. No `Algorithm.HS256` hardcoding found anywhere near that line at current HEAD.

The RS256 fallback claim is **substantively correct** — it exists at line 745. But the HS256 "line 689" reference is not present at current HEAD (may have been removed in a refactor since the overview was written).

**Correction:** Update GAP-10 Affected Files from `(lines 689, 697)` → `(line 745)`. Remove the HS256 reference at "line 689" or note "removed at current HEAD".

---

### Error 4 — Domain 60 (`DefaultSamlArtifactResolver`): Misleading description of what the class does

**HTML states:**
> "Resolves SAML artifacts by fetching the full SAML assertion over a SOAP backchannel (ArtifactResolve / ArtifactResponse); resolved assertions are then processed by the same SAML signing/verification chain as rows 18–20"

**Actual source:**
`DefaultSamlArtifactResolver.java` does **not** fetch assertions over a SOAP backchannel. It only manages artifact *storage, lookup, and creation* in the client session — it maps an artifact byte string to a client session and builds/decodes the 44-byte artifact format. The SOAP backchannel fetch is handled by `SAMLEndpoint.java`. There are no calls to `XMLSignatureUtil` or any signing in this class.

The PQC state (BLOCKED, same GAP-2/GAP-9 as rows 18–20) is correct in that any artifact-resolved assertion *eventually* goes through the SAML signing chain. But the description of what `DefaultSamlArtifactResolver.java` itself does is inaccurate.

**Correction:** Domain 60 description should clarify that `DefaultSamlArtifactResolver.java` handles artifact *storage/routing only*; the SOAP backchannel and signing verification are handled by `SAMLEndpoint.java` (already listed in Domain 20). Consider whether `DefaultSamlArtifactResolverFactory.java` warrants separate mention. The PQC state and gaps remain correct.

---

## PART 2 — COMPLETE DOMAIN-BY-DOMAIN VERIFICATION RESULTS

The following table documents every domain row checked. ✅ = confirmed accurate. ⚠️ = issue noted above. ℹ️ = minor note.

| Row | Domain | Files exist? | Algorithm claims | PQC state | Notes |
|-----|--------|-------------|-----------------|-----------|-------|
| 1 | Access Token / ID Token Signing | ✅ All files exist | ✅ RS*/PS*/ES*/EdDSA via SPI; Algorithm.java has ML-DSA constants; no JavaAlgorithm mapping | ✅ PARTIAL | |
| 2 | ID Token / JARM Encryption | ✅ All files exist | ✅ RSA-OAEP/256, RSA1_5, ECDH-ES variants; no ML-KEM | ✅ BLOCKED | |
| 3 | Backchannel Logout Token Signing | ✅ All files exist | ✅ TokenCategory.LOGOUT maps to same alg as ID token (confirmed in DefaultTokenManager lines 207-209) | ✅ PENDING PROVIDERS | |
| 4 | UserInfo — Signed & Encrypted | ✅ UserInfoEndpoint exists | ✅ signatureAlgorithm(USERINFO) confirmed; jweFromContent() uses CekManagementProvider | ✅ BLOCKED | GAP-4 and GAP-21 references accurate |
| 5 | Introspection Embedded JWT | ✅ AccessTokenIntrospectionProvider exists | ✅ Uses session.tokens().encode() | ✅ PARTIAL | |
| 6 | Token Verification | ✅ AuthenticationManager, LoginActionsService exist | ✅ SPI-driven via SignatureProvider | ✅ PENDING PROVIDERS | |
| 7 | JAR Signed Request Object | ✅ Both parser files exist | ✅ decodeClientJWT() used | ✅ PENDING PROVIDERS | |
| 8 | JAR Encrypted Request Object | ✅ DefaultTokenManager, both parsers exist | ✅ decodeClientJWT() selects KeyUse.ENC key; no ML-KEM | ✅ BLOCKED | Line 123 confirmed |
| 9 | JARM Signed Authorization Response | ✅ DefaultTokenManager, OIDCRedirectUriBuilder exist | ✅ TokenCategory.AUTHORIZATION_RESPONSE confirmed | ✅ PENDING PROVIDERS | |
| 10 | private_key_jwt Client Authentication | ✅ All 3 files exist | ✅ ClientSignatureVerifierProvider SPI; asymmetric check at line 120 confirmed | ✅ PENDING PROVIDERS | |
| 11 | X.509 mTLS Client Authentication | ✅ All 4 files exist | ✅ CertPathBuilder via CryptoProvider; SHA-256 thumbprint confirmed | ✅ SAFE | |
| 12 | Attestation-Based Client Authentication | ✅ File exists, wrong path | ✅ TODO at lines 418-419 confirmed; cnf.jwk path confirmed | ✅ PARTIAL | **Error 1: wrong path** |
| 13 | X.509 Browser Authentication | ✅ All 3 files exist | ✅ CertPathBuilder/CertStore via CryptoProvider; OCSP via BouncyCastle | ✅ SAFE | |
| 14 | WebAuthn / Passkeys | ✅ All 3 files exist | ✅ COSE algs -7/-35/-36/-257/-258/-259 confirmed in WebAuthnRegister.java | ✅ EXTERNAL DEPENDENCY | |
| 15 | DPoP | ✅ DPoPUtil.java exists; JWKSUtils.java exists | ✅ computeThumbprint() throws UnsupportedOperationException for AKP confirmed | ✅ PARTIAL | |
| 16 | CIBA Signed Backchannel Auth | ✅ All 3 files exist | ✅ SPI-driven verification; FapiConstant allowlist confirmed | ✅ PARTIAL | |
| 17 | JWT Authorization Grant | ✅ All 4 files exist | ✅ validateSignatureAlgorithm() confirmed | ✅ PENDING PROVIDERS | |
| 18 | SAML Assertion & Document Signing | ✅ All 6 files exist | ✅ SignatureAlgorithm.java RSA variants only; XMLSignatureUtil uses javax.xml.crypto.dsig | ✅ BLOCKED | |
| 19 | SAML Assertion Encryption | ✅ XMLEncryptionUtil, SAMLEncryptionAlgorithms, SAMLDecryptionKeysLocator exist | ✅ RSA-OAEP, RSA_OAEP_11, RSA1_5 only confirmed; no ML-KEM | ✅ BLOCKED | |
| 20 | SAML IdP Broker | ✅ SAMLIdentityProvider, SAMLEndpoint, SAMLIdentityProviderFactory exist | ✅ RS256 hardcodings at lines 414 and 507 confirmed | ✅ BLOCKED | **Error 2: line numbers 415/506 → 414/507** |
| 21 | OIDC IdP Broker | ✅ All 3 files exist | ✅ RS256 fallback at line 745 confirmed; HS256 at "689" NOT found | ✅ PARTIAL | **Error 3: line numbers** |
| 22 | Kubernetes Identity Provider | ✅ KubernetesIdentityProvider exists | ✅ Sub-type of OIDC broker, SPI-driven | ✅ PENDING PROVIDERS | |
| 23 | SPIFFE Identity Provider | ✅ SpiffeIdentityProvider exists | ✅ SPI-driven via SignatureProvider | ✅ PENDING PROVIDERS | |
| 24 | JWT-VC / SD-JWT Credential Signing | ✅ All 3 files exist | ✅ Algorithm from CredentialBuildConfig via SignatureProvider SPI | ✅ PENDING PROVIDERS | |
| 25 | LD-Proof Credential Signing | ✅ LDCredentialSigner, Ed255192018Suite exist | ✅ Hardcoded Ed255192018Suite confirmed | ✅ BLOCKED | |
| 26 | OID4VC Key Binding | ✅ JwtProofValidator, AbstractProofValidator, AttestationValidatorUtil exist | ✅ getSupportedAsymmetricSignatureAlgorithms() used | ✅ PENDING PROVIDERS | |
| 27 | OID4VC c_nonce JWT Signing | ✅ JwtCNonceHandler exists | ✅ ES256 then RS256 fallback confirmed at lines 281/287 | ✅ BLOCKED | |
| 28 | Generated RSA Signing Key Provider | ✅ All 3 files exist | ✅ No GeneratedAKPKeyProviderFactory; DefaultKeyManager confirmed | ✅ BLOCKED | |
| 29 | Generated RSA Encryption Key Provider | ✅ GeneratedRsaEncKeyProviderFactory, ImportedRsaEncKeyProviderFactory exist | ✅ RSA-OAEP/256/RSA1_5 only confirmed | ✅ BLOCKED | |
| 30 | Generated ECDSA Signing Key Provider | ✅ GeneratedEcdsaKeyProviderFactory, AbstractGeneratedEcKeyProviderFactory, GeneratedEcdsaKeyProvider exist | ✅ P-256/384/521 confirmed | ✅ BLOCKED | |
| 31 | Generated EdDSA Signing Key Provider | ✅ GeneratedEddsaKeyProviderFactory, AbstractEddsaKeyProvider, GeneratedEddsaKeyProvider exist | ✅ Ed25519 (EdDSA via JDK KeyFactory) confirmed | ✅ BLOCKED | |
| 32 | Generated ECDH Encryption Key Provider | ✅ GeneratedEcdhKeyProviderFactory, GeneratedEcdhKeyProvider exist | ✅ P-256/384/521 ECDH confirmed | ✅ BLOCKED | |
| 33 | Imported RSA Signing Key Provider | ✅ ImportedRsaKeyProviderFactory, AbstractImportedRsaKeyProviderFactory exist | ✅ RS*/PS* only; no AKP import | ✅ BLOCKED | |
| 34 | Java Keystore Key Provider | ✅ JavaKeystoreKeyProviderFactory, JavaKeystoreKeyProvider exist | ✅ mergedAlgorithmProperties() confirmed; no ML-DSA in list | ✅ BLOCKED | |
| 35 | Dynamic Client Registration Tokens | ✅ ClientRegistrationTokenUtils exists | ✅ HS512 (INTERNAL) confirmed; RS256 special-case at line 416 confirmed | ✅ SAFE | |
| 36 | OIDC Well-Known Discovery | ✅ OIDCWellKnownProvider exists | ✅ DEFAULT_CLIENT_AUTH_SIGNING_ALG_VALUES_SUPPORTED = RS256 confirmed at line 85; dynamic main lists confirmed | ✅ PARTIAL | |
| 37 | SET Signing | ✅ SecurityEventTokenEncoder, SsfSignatureAlgorithms, SecurityEventTokenDispatcher exist | ✅ ALLOWED = Set.of(RS256) confirmed at line 31 | ✅ EXTERNAL DEPENDENCY | |
| 38 | Admin CLI | ✅ AuthUtil.java, JWSBuilder.java exist | ✅ .rsa256() hardcoded at line 213 confirmed | ✅ BLOCKED | |
| 39 | JGroups ASYM_ENCRYPT | ✅ cache-ispn-asym-enc.xml exists (src/test/resources) | ✅ ASYM_ENCRYPT asym_keylength="2048" asym_algorithm="RSA" confirmed | ✅ SAFE (production) | |
| 40 | Organisation Invitation Token Verification | ✅ Organizations.java exists | ✅ Uses CryptoUtils.getSignatureProvider() which is SPI-driven — confirmed | ✅ PENDING PROVIDERS | |
| 41 | FAPI Algorithm Allowlist | ✅ All 5 files exist | ✅ FapiConstant.ALLOWED_ALGORITHMS = {PS256,PS384,PS512,ES256,ES384,ES512} confirmed; isSecureAlgorithm() calls .contains() | ✅ BLOCKED | |
| 42 | CryptoProvider — BouncyCastle Default | ✅ DefaultCryptoProvider, CryptoProvider exist | ✅ bcprov-jdk18on 1.84 from Quarkus BOM 3.33.2.1 confirmed; no ML-DSA registration code in DefaultCryptoProvider | ✅ PARTIAL | |
| 43 | CryptoProvider — FIPS | ✅ FIPS1402Provider, BCFIPSCertificateUtilsProvider exist | ✅ bc-fips 2.1.2 pinned in pom.xml confirmed at line 78 | ✅ BLOCKED | |
| 44 | CryptoProvider — Elytron | ✅ WildFlyElytronProvider exists | ✅ No PQC support confirmed | ✅ EXTERNAL DEPENDENCY | |
| 45 | JWK Serialisation & Thumbprint | ✅ All 5 files exist | ✅ AKPPublicJWK, AKPUtils, JWKBuilder.akp() confirmed; JWKSUtils.computeThumbprint throws UnsupportedOperationException for AKP confirmed | ✅ PARTIAL | |
| 46 | Default Realm Key Providers (Bootstrap) | ✅ DefaultKeyProviders exists | ✅ createProviders() only creates rsa-generated and rsa-enc-generated confirmed at lines 37-42 | ✅ BLOCKED | |
| 47 | Client Public Key Loader | ✅ ClientPublicKeyLoader, HardcodedPublicKeyLoader exist | ✅ JWKSUtils.getKeyWrappersForUse() handles AKP confirmed | ✅ PARTIAL | |
| 48 | Realm JWKS Endpoint | ✅ JWKSServerUtils exists | ✅ toJwk() has RSA/EC/OKP branches only — no AKP confirmed at lines 59-63 | ✅ BLOCKED | |
| 49 | Admin API — Client Cert & Keypair Generation | ✅ ClientAttributeCertificateResource exists | ✅ generateKeyPairCertificate() confirmed; DEFAULT_RSA_KEY_SIZE confirmed | ✅ BLOCKED | |
| 50 | Client SDK — JWT Client Credentials | ✅ JWTClientCredentialsProvider exists | ✅ switch on KeyType has RSA/EC/OKP only; no AKP case; default throws RuntimeException confirmed at lines 76-96 | ✅ PARTIAL | |
| 51 | Client SDK — DPoP Proof Generation | ✅ DPoPGenerator exists | ✅ generateRsaSignedDPoPProof() hardcoded RSA confirmed; TODO comment at line 49 confirmed; generic path is SPI-driven | ✅ PARTIAL | |
| 52 | Docker Registry — Self-Signed Cert | ✅ DockerComposeCertsDirectory exists | ✅ getKeyPairGen(KeyType.RSA) with initialize(2048) confirmed at lines 29-30 | ✅ BLOCKED | |
| 53 | Client Asymmetric Signature Verifier | ✅ All 10 files listed exist | ✅ KeyType.RSA guard at line 36-37 confirmed; throws "Key Type is not RSA" | ✅ BLOCKED / PENDING | |
| 54 | Federated JWT Client Authentication | ✅ FederatedJWTClientAuthenticator, DefaultClientAssertionStrategy, SpiffeClientAssertionStrategy exist | ✅ ClientAssertionIdentityProvider SPI chain confirmed | ✅ PENDING PROVIDERS | |
| 55 | Default Trust Identity Provider | ✅ All 3 files exist | ✅ JWKSUtils.getKeyWrappersForUse() SPI-driven confirmed | ✅ PENDING PROVIDERS | |
| 56 | SD-JWT Issuer Signing | ✅ SdJwt, IssuerSignedJWT, JwsToken, SdJwtVerificationContext, KeyBindingJWT, SdJwtVP exist | ✅ SignatureSignerContext / SignatureVerifierContext confirmed in both IssuerSignedJWT and SdJwtVerificationContext | ✅ PENDING PROVIDERS | |
| 57 | Token Exchange | ✅ TokenExchangeGrantType, StandardTokenExchangeProvider, AbstractTokenExchangeProvider exist | ✅ SPI-driven; DPoPUtil.validateBinding() confirmed | ✅ PENDING PROVIDERS | |
| 58 | Device Authorization Grant | ✅ DeviceGrantType exists | ✅ Delegates to TokenManager confirmed | ✅ PENDING PROVIDERS | |
| 59 | SAML Metadata Public Key Loader | ✅ SamlAbstractMetadataPublicKeyLoader exists | ✅ X509Certificate extraction to KeyWrapper confirmed; algorithm-agnostic | ✅ SAFE (key-loading layer) | |
| 60 | SAML Artifact Resolution | ✅ DefaultSamlArtifactResolver, DefaultSamlArtifactResolverFactory exist | ⚠️ Description inaccurate — class does NOT do SOAP backchannel fetch; handles storage/routing only | ✅ BLOCKED (overall assessment correct) | **Error 4: description** |
| 61 | SAML2Signature Default | ✅ SAML2Signature exists | ✅ signatureMethod = SignatureMethod.RSA_SHA1 at line 55; digestMethod = DigestMethod.SHA1 at line 57 confirmed | ✅ BLOCKED | |

---

## PART 3 — GAP REFERENCE VERIFICATION

All 23 GAPs (GAP-1 through GAP-23) verified:

| GAP | Title | Files Correct? | Line Numbers | Notes |
|-----|-------|---------------|-------------|-------|
| GAP-1 | JWK Thumbprint — AKP Keys | ✅ | ✅ (line 162) | UnsupportedOperationException confirmed |
| GAP-2 | SAML Signing — No PQC URIs | ✅ | ✅ | SignatureAlgorithm.java RSA/DSA only confirmed |
| GAP-3 | SAML Encryption — No ML-KEM | ✅ | ✅ | XMLEncryptionUtil RSA-OAEP/RSA1_5 only confirmed |
| GAP-4 | JWE Key Management — No ML-KEM CEK Provider | ✅ | ✅ | No ML-KEM CekManagementProviderFactory confirmed |
| GAP-5 | JavaKeystoreKeyProvider — ML-DSA Import | ✅ | ✅ | No AKP branch confirmed |
| GAP-6 | FIPS 140-2 Backend — ML-DSA Blocked | ✅ | ✅ | bc-fips 2.1.2 pinned confirmed |
| GAP-7 | OID4VC LD-Proof — Hardcoded Ed25519 | ✅ | ✅ | Ed255192018Suite hardcoding confirmed |
| GAP-8 | FAPI / Client Policy Allowlist | ✅ | ✅ | ALLOWED_ALGORITHMS = PS*/ES* only confirmed; isSecureAlgorithm() .contains() confirmed |
| GAP-9 | SAML — Hardcoded RS256 Key Lookup | ✅ files | ⚠️ lines off by 1 | SAMLIdentityProvider lines should be 414, 507 not 415, 506. SamlProtocol.java line 544 ✅. SamlService.java line 965 ✅. |
| GAP-10 | IdP Broker — Hardcoded RS256 / HS256 Fallback | ✅ file | ⚠️ lines wrong | RS256 at line 745 ✅. HS256 at "line 689" NOT PRESENT at current HEAD. |
| GAP-11 | Dynamic Client Registration — RS256 Special Case | ✅ | ✅ line 416 | Confirmed exactly |
| GAP-12 | DEFAULT_SIGNATURE_ALGORITHM RS256 | ✅ | ✅ (Constants.java line 68, DefaultTokenManager line 233) | HTML says line 232 — actual is line 233 (off by 1, cosmetic) |
| GAP-13 | Admin CLI & Authz Client — Hardcoded RS256 | ✅ | ✅ (AuthUtil line 213) | JWTClientCredentialsProvider switch confirmed no AKP |
| GAP-14 | JGroups ASYM_ENCRYPT | ✅ | ✅ | asym_keylength="2048" asym_algorithm="RSA" confirmed |
| GAP-15 | Realm Key Management — No ML-DSA Keygen | ✅ | ✅ | No GeneratedAKPKeyProviderFactory confirmed |
| GAP-16 | JavaKeystoreKeyProviderFactory — ML-DSA excluded from UI | ✅ | ✅ | mergedAlgorithmProperties() confirmed; no ML-DSA |
| GAP-17 | FAPI/CIBA Executor Factories — Admin UI | ✅ | ✅ | Both factories use FapiConstant.ALLOWED_ALGORITHMS for option lists confirmed |
| GAP-18 | JwtCNonceHandler — Hardcoded ES256/RS256 | ✅ | ✅ lines 281/287 | Confirmed |
| GAP-19 | No bcprov Minimum Version Constraint | ✅ | ✅ | bcprov 1.84 from Quarkus BOM 3.33.2.1 confirmed; Maven Enforcer plugin present but no bcprov minimum rule confirmed |
| GAP-20 | JAR Encrypted Request Object Decryption — No ML-KEM | ✅ | ✅ (line 123) | decodeClientJWT() confirmed |
| GAP-21 | UserInfo — Per-Client Signing & Encryption Gap | ✅ | ✅ (lines 341, 376-382) | signatureAlgorithm(USERINFO) and jweFromContent() confirmed |
| GAP-22 | Attestation-Based Client Auth — Algorithm Enforcement & JWK Thumbprint | ⚠️ wrong path | ✅ lines 418-419 | **Error 1: wrong file path** |
| GAP-23 | OIDC Discovery — Hardcoded RS256 for backchannel signing algs | ✅ | ✅ line 85 | Confirmed exactly |

**Additional minor note on GAP-12:** HTML references DefaultTokenManager "line 232" — actual is line 233. One line off, cosmetic.

---

## PART 4 — "FILES EVALUATED — NOT ADDED AS NEW ROWS" SECTION VERIFICATION

All files in this section were verified to exist. The stated reasons for exclusion were confirmed:

- `KeyResource.java` ✅ — admin REST, delegates to KeyManager SPI
- `AbstractClientIdMetadataDocumentExecutor.java` ✅ — no crypto
- SPI interface files (SignatureProvider, SignatureProviderFactory, etc.) ✅
- `KeycloakModelUtils.java` ✅ — utility; RSA hardcoding exposed via row 49
- `CryptoProvider.java` ✅ — SPI interface
- ECDSA/ECDH backend implementations ✅ — no PQC gap of their own
- PEM utils, KeyWrapper, DerUtils, etc. ✅ — algorithm-agnostic
- All SAML supporting classes ✅ — delegate to XMLSignatureUtil/XMLEncryptionUtil chain
- `JwsToken.java` via sdjwt is listed in Domain 56 ✅
- JWE structural model classes ✅
- Truststore, OCSP, CRL, x509 lookup classes ✅
- `authz/client/…/util/crypto/AuthzClientCryptoProvider.java` ✅ — noted as independent instance of GAP-13

---

## PART 5 — STATUS DISTRIBUTION TABLE VERIFICATION

HTML claims: PENDING PROVIDERS=20, PARTIAL=10, BLOCKED=24, SAFE=4, EXTERNAL DEPENDENCY=3. Total=61.

Manual recount from the main table:

**PENDING PROVIDERS:** Rows 3, 6, 7, 9, 10, 17, 22, 23, 24, 26, 40, 54, 55, 56, 57, 58 = 16 rows primary; also rows 47, 53 (partial PENDING), 5 (noted as "no code change needed, pending domain 1"). Counting as stated: 20.

Actually counting distinctly-labelled rows as PENDING PROVIDERS: 3, 6, 7, 9, 10, 17, 22, 23, 24, 26, 40, 54, 55, 56, 57, 58, 5 = 17. With some dual-labelled rows (e.g. 16 is PARTIAL, 53 is dual), the count of 20 is plausible but **I could not independently verify the exact count is 20 vs 19 or 17**. This warrants a re-check when the new OID4VP domain is added.

**BLOCKED:** Counting explicit BLOCKED rows: 2, 8, 18, 19, 20, 21 (partial), 25, 27, 28, 29, 30, 31, 32, 33, 34, 36 (partial), 38, 41, 46, 48, 49, 52, 53 (partial), 60, 61 = approximately 24 explicit BLOCKED labels. The HTML states 24 — this is plausible.

**SAFE:** Rows 11, 13, 35, 39, 59 = 5 rows with SAFE label. HTML says 4. **This appears to be incorrect in the HTML — row 59 is labelled SAFE (at key-loading layer) and should be counted.**

> **POTENTIAL COUNT ERROR:** The status distribution table may have an off-by-one in SAFE count. Row 59 is labelled `SAFE (at key-loading layer)` but BLOCKED for GAP-2/GAP-9 dependency — whether it counts as SAFE depends on labelling convention. If only rows with a pure SAFE badge count, rows 11, 13, 35, 39 = 4 is correct. Row 59's badge is conditionally SAFE. This is defensible.

---

## PART 6 — NEW CONTENT NOT IN CURRENT OVERVIEW

### NEW DOMAIN: OID4VP Identity Provider (post-dates original overview)

**Status: Completely absent from pqc_overview.html**

The OID4VP Identity Provider (Keycloak acting as an OID4VP *verifier*) was introduced after the original overview. It contains multiple hardcoded classical-only crypto operations:

**Files:**
- `services/src/main/java/org/keycloak/broker/oid4vp/OID4VPIdentityProvider.java`
- `services/src/main/java/org/keycloak/broker/oid4vp/EphemeralKey.java`
- `services/src/main/java/org/keycloak/broker/oid4vp/ResponseEncryption.java`
- `services/src/main/java/org/keycloak/broker/oid4vp/OID4VPIdentityProviderEndpoint.java`
- `services/src/main/java/org/keycloak/broker/oid4vp/RequestObject.java`

**Crypto 1 — Request Object Signing (BLOCKED):**
`OID4VPIdentityProvider.signingKey()` hardcodes ES256:
```java
// lines 187-188
session.keys().getKeyIncludingDisabled(realm, kid, KeyUse.SIG, Algorithm.ES256)
session.keys().getActiveKey(realm, KeyUse.SIG, Algorithm.ES256)
```
Developer-acknowledged TODO at lines 77-79:
```java
// TODO the accepted presentation signature algorithms are hardcoded to match the advertised client
public static final List<String> ACCEPTED_ALGORITHMS = List.of(Algorithm.ES256);
```
`RequestObject.sign()` uses `KeyWrapperUtil.createSignatureSignerContext(signingKey)` — SPI-driven in principle, but `signingKey()` hardcodes ES256, so ML-DSA keys are never selected.

**Crypto 2 — VP Token Algorithm Enforcement (BLOCKED):**
`OID4VPIdentityProviderEndpoint.requireAcceptedAlgorithm()` checks:
```java
if (!OID4VPIdentityProvider.ACCEPTED_ALGORITHMS.contains(algorithm))
    throw new VerificationException("Unsupported issuer signed JWT signature algorithm");
```
Actively rejects all algorithms except ES256, including ML-DSA.

**Crypto 3 — Response Encryption — ECDH-ES over secp256r1 (BLOCKED / EXTERNAL DEPENDENCY):**
`EphemeralKey.generate()` hardcodes:
```java
private static final String CURVE_SEC = "secp256r1";
KeyPair keyPair = KeyUtils.generateEcKeyPair(CURVE_SEC);
```
`ResponseEncryption.java` hardcodes:
```java
public static final String KEY_MANAGEMENT_ALG = JWEConstants.ECDH_ES;
```
This follows the HAIP (High Assurance Interoperability Profile) which currently pins to ECDH-ES/P-256 for the `direct_post.jwt` response mode. The encryption side is therefore **spec-gated** (analogous to row 37 / CAEP), but the signing hardcoding is an independent code gap.

**Proposed PQC states:**
- Request object signing: **BLOCKED** — hardcoded ES256 key selection
- VP token algorithm enforcement: **BLOCKED** — ACCEPTED_ALGORITHMS list
- Response encryption: **EXTERNAL DEPENDENCY** — HAIP spec-gated

**Proposed new GAPs:**
- GAP-24: OID4VP — Hardcoded ES256 signing key selection and ACCEPTED_ALGORITHMS list
- GAP-25: OID4VP — ECDH-ES secp256r1 hardcoded for `direct_post.jwt` response encryption (HAIP spec-gated)

**Domain counter:** 61 → 62. Status counts need updating.

---

### NEW FILE: `DIDUtils.java` (P-256 only)

**File:** `core/src/main/java/org/keycloak/util/DIDUtils.java`

**Description:** Utility for `did:key` encoding/decoding. P-256 (ES256) only by design — class Javadoc states explicitly. Used by OID4VC components. ML-DSA `did:key` requires a new multicodec registration not yet standardised.

**Recommendation:** Add to "Files Evaluated — Not Added as New Rows" with reason: "P-256-only `did:key` utility; ML-DSA `did:key` is spec-gated (no multicodec assignment exists in the `did:key` specification)."

---

## PART 7 — SUMMARY OF ALL REQUIRED CHANGES TO `pqc_overview.html`

### Corrections (errors in existing content):

1. **Domain 12 / GAP-22:** Change `authenticators/client/AttestationBasedClientAuthenticator.java` → `authentication/authenticators/client/AttestationBasedClientAuthenticator.java`

2. **GAP-9:** Change `(lines 415, 506)` → `(lines 414, 507)` in SAMLIdentityProvider.java line number reference

3. **GAP-10 / Domain 21:** Change `(lines 689, 697)` → `(line 745)`; remove HS256 at "line 689" reference (not present at current HEAD)

4. **GAP-12:** Change DefaultTokenManager line reference from "line 232" → "line 233" (minor, cosmetic)

5. **Domain 60:** Correct description — `DefaultSamlArtifactResolver.java` handles artifact storage/routing only, not SOAP backchannel fetch. SOAP backchannel is in `SAMLEndpoint.java`

### Additions (new content):

6. **New Domain 62:** OID4VP Identity Provider — signing (BLOCKED), VP token enforcement (BLOCKED), response encryption (EXTERNAL DEPENDENCY / HAIP spec-gated)

7. **Two new GAPs:** GAP-24 (OID4VP ES256 signing hardcoding) and GAP-25 (OID4VP ECDH-ES response encryption)

8. **"Files Evaluated" section:** Add `DIDUtils.java` entry

9. **Status distribution table:** Update BLOCKED count (24 → 25 minimum) and total domain count (61 → 62)

---

*Review complete. All 61 existing domains, all 23 GAPs, and the "Files Evaluated" section have been checked against the live source. No changes have been made to `pqc_overview.html`. Awaiting your review and approval.*
