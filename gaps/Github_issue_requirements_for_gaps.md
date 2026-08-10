# PQC Gap Reference - Complete Table

| Gap ID | Title | Severity | GitHub Issue | Domains Affected | Affected File(s) | Description & Impact |
|--------|-------|----------|--------------|------------------|------------------|---------------------|
| **GAP-1** | JWK Thumbprint — AKP (ML-DSA) Keys | **HIGH** | Check [#44141](https://github.com/keycloak/keycloak/issues/44141) (CLOSED) or [#43692](https://github.com/keycloak/keycloak/issues/43692) | 12, 15, 45 | `core/…/util/JWKSUtils.java` | `AKP` is absent from `JWK_THUMBPRINT_REQUIRED_MEMBERS`. Calling `computeThumbprint()` on an ML-DSA JWK throws `UnsupportedOperationException`. Directly breaks DPoP `dpop_jkt` binding (row 15) and attestation-based client auth (row 12) for ML-DSA keys. Fix: add AKP member set and implement thumbprint computation for AKP JWKs. |
| **GAP-2** | SAML Signing — No PQC Algorithm Entries | **HIGH** | [#50294](https://github.com/keycloak/keycloak/issues/50294) | 18, 20, 59, 60, 61 | `saml-core/…/saml/SignatureAlgorithm.java`<br/>`saml-core/…/util/XMLSignatureUtil.java` | `SignatureAlgorithm` enum contains only RSA and DSA variants; no PQC XML Digital Signature URIs are registered. `XMLSignatureUtil` uses `javax.xml.crypto.dsig` with standard algorithms only. SAML assertion signing cannot use ML-DSA at the algorithm layer. Tracked under [#50292](https://github.com/keycloak/keycloak/issues/50292) / [#50294](https://github.com/keycloak/keycloak/issues/50294) (partial). |
| **GAP-3** | SAML Encryption — No ML-KEM Key Wrapping | **HIGH** | [#50295](https://github.com/keycloak/keycloak/issues/50295) | 19 | `saml-core/…/util/XMLEncryptionUtil.java` | `XMLEncryptionUtil` supports only RSA-OAEP and RSA1_5 for key transport. No ML-KEM key-encapsulation path exists. SAML assertion encryption cannot use quantum-safe key wrapping. Tracked under [#50292](https://github.com/keycloak/keycloak/issues/50292) / [#50295](https://github.com/keycloak/keycloak/issues/50295) (partial). |
| **GAP-4** | JWE Key Management — No ML-KEM CEK Provider | **HIGH** | [#48821](https://github.com/keycloak/keycloak/issues/48821) (create sub-issue) | 2, 4, 8 | `crypto/RsaCekManagementProvider.java`<br/>`crypto/EcdhEsCekManagementProvider.java`<br/>`jose/jws/DefaultTokenManager.java` | All `CekManagementProviderFactory` implementations use RSA-OAEP or ECDH-ES. No ML-KEM provider exists. Clients requesting encrypted ID tokens, encrypted UserInfo responses, or encrypted authorisation responses have no quantum-safe key encapsulation option. Blocks rows 2, 4, 8, 19. Must be implemented before any outbound token encryption can be quantum-safe. Related: [#50299](https://github.com/keycloak/keycloak/issues/50299) (HPKE alternative approach). |
| **GAP-5** | JavaKeystoreKeyProvider — ML-DSA Key Import | **MEDIUM** | [#50679](https://github.com/keycloak/keycloak/issues/50679) | 33, 34 | `keys/JavaKeystoreKeyProviderFactory.java`<br/>`keys/JavaKeystoreKeyProvider.java` | `JavaKeystoreKeyProviderFactory` supports RSA, EC, and OKP/EdDSA import paths only. BCFKS and PKCS12 keystores can physically store ML-DSA keys, but the provider's import logic has no AKP branch. Operators cannot import ML-DSA keys from a keystore file. See also GAP-16. |
| **GAP-6** | FIPS 140-2 Backend — ML-DSA Availability Blocked | **HIGH** | *External Dependency* | 11, 13, 43 | `crypto/fips1402/…/FIPS1402Provider.java`<br/>`crypto/fips1402/…/BCFIPSCertificateUtilsProvider.java` | BC-FIPS 2.1.2 (pinned in root `pom.xml`) confirmed by JAR inspection to contain only LMS under `org.bouncycastle.crypto.internal.pqc`. No ML-DSA and no ML-KEM classes are present. FIPS-mode Keycloak deployments cannot use ML-DSA or ML-KEM. Confirmed hard blocker for regulated environments targeting FIPS 140-2/3 compliance. Unblocked only when BC-FIPS 2.x achieves NIST FIPS 140-3 validation for ML-DSA/ML-KEM. Affects X.509 cert validation with PQC certificates in FIPS mode (rows 11, 13, 43). |
| **GAP-7** | OID4VC Linked Data Proof Suite — Hardcoded Ed25519 | **MEDIUM** | *External Dependency* | 25 | `oid4vc/issuance/signing/LDCredentialSigner.java`<br/>`oid4vc/issuance/signing/vcdm/Ed255192018Suite.java` | LD-Proof credential signing is hardcoded to `Ed25519Signature2018`. It does not use the `SignatureProvider` SPI path and will not inherit ML-DSA support automatically. A new ML-DSA LD cryptographic suite would need to be implemented separately if LD-Proofs are required with PQC. No existing tracking issue covers this. |
| **GAP-8** | FAPI / Client Policy Allowlist — ML-DSA Actively Blocked | **HIGH** | *Spec-Gated* | 16, 41 | `clientpolicy/executor/FapiConstant.java`<br/>`clientpolicy/executor/SecureSigningAlgorithmExecutor.java`<br/>`clientpolicy/executor/SecureSigningAlgorithmForSignedJwtExecutor.java`<br/>`ciba/clientpolicy/executor/SecureCibaAuthenticationRequestSigningAlgorithmExecutor.java` | `FapiConstant.ALLOWED_ALGORITHMS` is hardcoded to `{PS256, PS384, PS512, ES256, ES384, ES512}`. `SecureSigningAlgorithmExecutor.isSecureAlgorithm()` and the equivalent CIBA executor call `ALLOWED_ALGORITHMS.contains(sigAlg)` — ML-DSA will be *actively rejected* even after providers exist. When a FAPI client policy is applied, PQC algorithms are blocked at runtime and in the admin UI. Blocked on FAPI 2.0 / FAPI profiles including PQC algorithms. |
| **GAP-9** | SAML Protocol — Hardcoded RS256 Key Lookup | **HIGH** | [#50292](https://github.com/keycloak/keycloak/issues/50292) (create sub-issue) | 18, 20, 59, 60, 61 | `broker/saml/SAMLIdentityProvider.java` (lines 415, 506)<br/>`protocol/saml/SamlProtocol.java` (line 544)<br/>`protocol/saml/SamlService.java` (line 965) | Four hardcoded RS256 key-selection call sites in SAML code: SP metadata signing key export (line 415), SP metadata document signing (line 506), artifact resolve response signing (line 544), and IDP metadata descriptor redirect-binding export (line 965). Even if GAP-2 is fixed at the algorithm layer, ML-DSA support can never be reached because these call sites will never select an AKP key. Both layers (GAP-2 and GAP-9) must be fixed. Requires a third sub-issue under [#50292](https://github.com/keycloak/keycloak/issues/50292). |
| **GAP-10** | IdP Broker — Hardcoded RS256 / HS256 Fallback | **MEDIUM** | [#48821](https://github.com/keycloak/keycloak/issues/48821) (create sub-issue) | 21 | `broker/oidc/AbstractOAuth2IdentityProvider.java` (lines 689, 697) | Line 689 hardcodes `Algorithm.HS256` as the fallback when `client_secret_jwt` has no configured algorithm. Line 697 hardcodes `Algorithm.RS256` as the fallback for all other client assertion signing (e.g., `private_key_jwt`). An IdP broker configured for `private_key_jwt` will always fall back to RS256 unless the admin explicitly sets `clientAssertionSigningAlg`. No path to select ML-DSA as the outbound default even after providers exist. |
| **GAP-11** | Dynamic Client Registration — RS256 Special-Case Logic | **LOW** | [#48821](https://github.com/keycloak/keycloak/issues/48821) or [#48823](https://github.com/keycloak/keycloak/issues/48823) | 35 | `clientregistration/oidc/DescriptionConverter.java` (line 416) | `if (Algorithm.RS256.equals(defaultSignatureAlgorithm) || StringUtil.isBlank(defaultSignatureAlgorithm))` — RS256 is explicitly special-cased so that `id_token_signed_response_alg` is omitted from the DCR response when the realm default is RS256. No equivalent treatment is defined for ML-DSA. Minor inconsistency; will need updating when ML-DSA becomes a valid realm default. Low priority - could be addressed in operator guidance. |
| **GAP-12** | Constants.DEFAULT_SIGNATURE_ALGORITHM — RS256 Fallback Propagation | **MEDIUM** | [#48823](https://github.com/keycloak/keycloak/issues/48823) (document) | Multiple | `server-spi-private/…/models/Constants.java`<br/>`jose/jws/DefaultTokenManager.java` (line 232) | `DEFAULT_SIGNATURE_ALGORITHM = Algorithm.RS256` is the ultimate fallback for all token signing when neither client nor realm has configured an algorithm. Any incomplete PQC migration (realm default updated but client attributes not set) will silently revert to classical RS256. Operators and documentation must explicitly flag this transition risk. |
| **GAP-13** | Admin CLI — Hardcoded RS256 Client Assertion Signing | **MEDIUM** | [#48821](https://github.com/keycloak/keycloak/issues/48821) or [#48823](https://github.com/keycloak/keycloak/issues/48823) (create sub-issue) | 38 | `client-cli/admin-cli/…/util/AuthUtil.java` (line 213)<br/>`core/…/jose/jws/JWSBuilder.java` | `getSignedRequestToken()` calls `new JWSBuilder().jsonContent(reqToken).rsa256(keypair.getPrivate())`, hardcoding RS256. No `--sigalg` parameter exists in the CLI. Both `kcadm.sh` and `kcreg.sh` share this code path. If a realm requires ML-DSA for client assertions (e.g., via a FAPI policy), both CLI tools will be unable to authenticate via the keystore-based `private_key_jwt` flow. Fix: add an algorithm parameter to `getSignedRequestToken()` and plumb a `--sigalg` option through the credential configuration command chain. |
| **GAP-14** | JGroups ASYM_ENCRYPT — RSA Cluster Key Exchange (Operator Guidance) | **LOW** | [#48823](https://github.com/keycloak/keycloak/issues/48823) (document) | 39 | `quarkus/tests/integration/…/cache-ispn-asym-enc.xml` | JGroups' `ASYM_ENCRYPT` protocol uses RSA-2048 for AES group-key distribution to joining cluster nodes — quantum-vulnerable under a Harvest Now, Decrypt Later attack. Risk is confined to operators who manually copy the example `ASYM_ENCRYPT` config; production Keycloak uses mTLS (`JGroupsCertificateProvider`) which will inherit quantum-safety from PQC-capable TLS ([#48820](https://github.com/keycloak/keycloak/issues/48820)). No code change needed — deployment/operator guidance only; document under [#48823](https://github.com/keycloak/keycloak/issues/48823). |
| **GAP-15** | Realm Key Management — No ML-DSA Key Generation or Rotation | **HIGH** | [#44142](https://github.com/keycloak/keycloak/issues/44142) | 28, 30, 31, 46 | `keys/GeneratedRsaKeyProviderFactory.java`<br/>`keys/GeneratedEcdsaKeyProviderFactory.java`<br/>`keys/GeneratedEddsaKeyProviderFactory.java`<br/>`keys/DefaultKeyManager.java`<br/>`common/…/crypto/CryptoProvider.java` | No `GeneratedAKPKeyProviderFactory` (or equivalent) exists for ML-DSA key generation. `DefaultKeyManager.createFallbackKeys()` does not know about ML-DSA. `CryptoProvider.getKeyPairGen(String algorithm)` would need to support ML-DSA algorithm names. This is the foundational gap — until ML-DSA key providers exist, no other ML-DSA signing work can be completed end-to-end. Tracked under [#48824](https://github.com/keycloak/keycloak/issues/48824) / [#44142](https://github.com/keycloak/keycloak/issues/44142). |
| **GAP-16** | JavaKeystoreKeyProviderFactory — ML-DSA Excluded from Admin UI | **MEDIUM** | [#50679](https://github.com/keycloak/keycloak/issues/50679) | 33, 34 | `keys/JavaKeystoreKeyProviderFactory.java` | `mergedAlgorithmProperties()` builds the selectable algorithm list shown in the admin UI. ML-DSA algorithm names are absent from this list. Even if GAP-5 (import logic) is fixed, the admin cannot select an ML-DSA algorithm for a keystore-based key provider in the UI. Both the provider logic (GAP-5) and the configuration surface (GAP-16) must be fixed together. |
| **GAP-17** | FAPI / CIBA Executor Factories — Admin UI Algorithm Lists | **MEDIUM** | *Depends on GAP-8* | 16, 41 | `clientpolicy/executor/SecureSigningAlgorithmExecutorFactory.java`<br/>`ciba/clientpolicy/executor/SecureCibaAuthenticationRequestSigningAlgorithmExecutorFactory.java` | Both executor factories build their selectable option lists directly from `FapiConstant.ALLOWED_ALGORITHMS`. ML-DSA is blocked both at runtime (GAP-8) and in the admin configuration UI. The UI-level block means an administrator cannot even configure a FAPI realm to accept ML-DSA before the algorithm constant is updated. Depends on GAP-8 being resolved first. |
| **GAP-18** | JwtCNonceHandler — OID4VC c_nonce JWT Signing Hardcoded to ES256 / RS256 | **MEDIUM** | *None* (create new OID4VC issue) | 27 | `oid4vc/issuance/keybinding/JwtCNonceHandler.java` | `selectSigningKey()` first attempts ES256, then falls back to RS256. This is independent of `Constants.DEFAULT_SIGNATURE_ALGORITHM` and completely bypasses the realm default algorithm. c_nonce JWTs will not use an active ML-DSA key even after ML-DSA providers and realm keys exist. Fix: replace the hardcoded lookup with a configurable or SPI-driven selection. |
| **GAP-19** | No Minimum-Version Constraint on bcprov-jdk18on — Silent ML-DSA Regression Risk | **MEDIUM** | [#46333](https://github.com/keycloak/keycloak/issues/46333) or [#46336](https://github.com/keycloak/keycloak/issues/46336) (create sub-issue) | 42 | `pom.xml` (root) | `bcprov-jdk18on` 1.84 (inherited via Quarkus BOM) is confirmed to contain ML-DSA and ML-KEM support (`org.bouncycastle.pqc.crypto.mldsa.*`, `org.bouncycastle.jcajce.provider.asymmetric.mlkem.*`). ML-DSA classes first appeared in version 1.78. However, no Maven Enforcer minimum-version rule exists. A downstream build with an older BOM version could silently drop ML-DSA support without a build-time failure. Fix: add an Enforcer rule pinning `bcprov-jdk18on ≥ 1.78`. |
| **GAP-20** | JAR Encrypted Request Object Decryption — No ML-KEM Inbound Path | **MEDIUM** | [#48821](https://github.com/keycloak/keycloak/issues/48821) (create sub-issue) | 8 | `jose/jws/DefaultTokenManager.java` (line 123)<br/>`endpoints/request/AuthzEndpointRequestObjectParser.java`<br/>`par/endpoints/request/ParEndpointRequestObjectParser.java` | `decodeClientJWT()` selects a realm private key by `KeyUse.ENC` and passes it directly to the JWE engine. No ML-KEM `CekManagementProviderFactory` exists and the realm cannot hold an ML-KEM ENC key. A client wishing to send an encrypted JAR using ML-KEM key encapsulation cannot do so. Distinct from GAP-4 (outbound) — this is *inbound* decryption. Fix: extend `decodeClientJWT()` to support ML-KEM key decapsulation, contingent on GAP-4. |
| **GAP-21** | UserInfo Endpoint — Per-Client Signing Attribute & Encryption Gap | **MEDIUM** | [#48821](https://github.com/keycloak/keycloak/issues/48821) (create sub-issue), document in [#48823](https://github.com/keycloak/keycloak/issues/48823) | 4 | `protocol/oidc/endpoints/UserInfoEndpoint.java` (lines 341, 379–398) | Signing: `signatureAlgorithm(TokenCategory.USERINFO)` resolves only the per-client `userinfo.response.signature.alg` attribute, then falls back to the realm default (`RS256`). Unlike ID tokens, there is no automatic inheritance — each client's explicit attribute must be updated independently during migration. Encryption: `jweFromContent()` uses `CekManagementProvider` SPI (no ML-KEM provider, same as GAP-4) but in a separate class and code path. UserInfo encryption is a confirmed hard gap requiring explicit testing once GAP-4 is resolved. |
| **GAP-22** | Attestation-Based Client Authentication — Algorithm Enforcement & JWK Thumbprint | **MEDIUM** | [#43684](https://github.com/keycloak/keycloak/issues/43684) or combine with GAP-28 | 12 | `authenticators/client/AttestationBasedClientAuthenticator.java` (lines 418, 483–489) | Two concerns: (1) An in-code `[TODO]` at line 418 notes that algorithm-type enforcement (must be a registered asymmetric algorithm) is not yet implemented — a security hardening gap independent of PQC, compounded by it. (2) If the `cnf.jwk` claim contains an ML-DSA (AKP) key, `JWKSUtils.computeThumbprint()` will throw `UnsupportedOperationException` (dependent on GAP-1). Could be addressed alongside GAP-28 under [#43684](https://github.com/keycloak/keycloak/issues/43684). |
| **GAP-23** | OIDC Discovery — Hardcoded RS256 for backchannel_authentication_request_signing_alg_values_supported | **LOW** | [#48821](https://github.com/keycloak/keycloak/issues/48821) (create sub-issue) | 36 | `protocol/oidc/OIDCWellKnownProvider.java` (line 85) | `DEFAULT_CLIENT_AUTH_SIGNING_ALG_VALUES_SUPPORTED = list(Algorithm.RS256.toString())` is a static constant used to advertise supported CIBA backchannel authentication request signing algorithms. Unlike other discovery fields (which are dynamically populated from registered `SignatureProvider` instances), this one remains hardcoded to RS256 even after ML-DSA providers are registered. A conformant CIBA client relying on this field will never attempt ML-DSA-signed requests. Fix: replace with a dynamic lookup merging registered asymmetric `SignatureProvider` algorithm names — same pattern as `id_token_signing_alg_values_supported`. |
| **GAP-24** | JWKS Endpoint — Add AKP Branch to JWKSServerUtils.toJwk() | **HIGH** | Check [#44141](https://github.com/keycloak/keycloak/issues/44141) (CLOSED) or [#43692](https://github.com/keycloak/keycloak/issues/43692) | 48 | `protocol/oidc/utils/JWKSServerUtils.java` | `JWKSServerUtils.toJwk()` has explicit `if/else if` branches for `KeyType.RSA`, `KeyType.EC`, and `KeyType.OKP` only. An AKP (ML-DSA) key falls through all branches and returns `null`, meaning ML-DSA realm keys will be silently omitted from the public JWKS endpoint (`/protocol/openid-connect/certs`). Clients will never discover the realm's ML-DSA keys. Fix: add an `AKP` branch in `toJwk()` calling `JWKBuilder.akp()`. Straightforward one-branch addition; `JWKBuilder.akp()` already exists. Verify if [#44141](https://github.com/keycloak/keycloak/issues/44141) covered this. |
| **GAP-25** | Admin API — Client Certificate & Keypair Generation Algorithm Parameter | **MEDIUM** | [#48821](https://github.com/keycloak/keycloak/issues/48821) (create sub-issue) | 49 | `services/resources/admin/ClientAttributeCertificateResource.java` | Both `generate()` and `generateAndGetKeystore()` call `KeycloakModelUtils.generateKeyPairCertificate()`, which hardcodes RSA key generation (`KeyUtils.generateRsaKeyPair(keysize)`). No algorithm selection is exposed in the API or UI. Admin-generated client keypairs will always be RSA regardless of realm PQC configuration. Fix: expose an algorithm parameter in the `generateAndGetKeystore` endpoint and update `KeycloakModelUtils.generateKeyPairCertificate()` to support ML-DSA key generation once providers exist. |
| **GAP-26** | Client SDK — Add AKP Support to JWTClientCredentialsProvider | **MEDIUM** | [#48821](https://github.com/keycloak/keycloak/issues/48821) (create sub-issue, could combine with GAP-27) | 50 | `core/…/client/authentication/JWTClientCredentialsProvider.java` | `setupKeyPair()` defaults to `Algorithm.RS256`. The algorithm-aware overload routes RSA, EC, and OKP (EdDSA) key types to the correct signer context. However the `switch` on `JavaAlgorithm.getKeyType()` has no `AKP` case — passing an ML-DSA key pair will throw `RuntimeException("Invalid KeyPair algorithm")`. Distinct from the server-side verifier (row 10) and the admin CLI path (row 38). Fix: add an `AKP` case to `setupKeyPair()` wiring an `AsymmetricSignatureSignerContext` for ML-DSA, once ML-DSA providers exist. |
| **GAP-27** | Client SDK — Add AKP Support to DPoPGenerator | **MEDIUM** | [#48821](https://github.com/keycloak/keycloak/issues/48821) (create sub-issue, could combine with GAP-26) | 51 | `core/…/util/DPoPGenerator.java` | The generic `generateSignedDPoPProof(…, KeyWrapper, …)` method is algorithm-agnostic and will support ML-DSA once providers exist. However, the convenience method `generateRsaSignedDPoPProof()` is hardcoded to RSA and includes an explicit `TODO` noting EC and EdDSA equivalents are missing. No ML-DSA convenience path exists. Distinct from server-side DPoP verification (row 15). Fix: add ML-DSA convenience method or update callers to use the generic `KeyWrapper` path. Low priority if callers already use the generic path. |
| **GAP-28** | Client Asymmetric Signature Verifier Context & Providers — RSA Guard Blocks AKP | **HIGH** | [#43684](https://github.com/keycloak/keycloak/issues/43684) | 53 | `services/crypto/ClientAsymmetricSignatureVerifierContext.java`<br/>`services/crypto/AsymmetricClientSignatureVerifierProvider.java`<br/>`services/crypto/ClientECDSASignatureVerifierContext.java`<br/>`services/crypto/ClientEdDSASignatureVerifierContext.java`<br/>`services/crypto/ECDSAClientSignatureVerifierProvider.java`<br/>`services/crypto/EdDSAClientSignatureVerifierProvider.java`<br/>ES*/EdDSA provider factories | Full client-side signature verifier stack for `private_key_jwt` client authentication. RSA path: `ClientAsymmetricSignatureVerifierContext.getKey()` at line 36 explicitly throws `VerificationException("Key Type is not RSA")` for any non-RSA key — ECDSA, EdDSA, and ML-DSA (AKP) client keys are all unconditionally rejected through the RSA path. The ECDSA and EdDSA paths route through their own contexts and are SPI-driven — they will support ML-DSA automatically once providers exist. ML-DSA (AKP) client assertions currently have no routed path at all. Fix: remove or generalise the `KeyType.RSA` guard in `ClientAsymmetricSignatureVerifierContext` to accept any asymmetric key type, or add a dedicated AKP client verifier context + provider factory following the same pattern as the ECDSA/EdDSA paths. **Covered by [#43684](https://github.com/keycloak/keycloak/issues/43684) - "Add ML-DSA ClientSignatureVerifierProviderFactory"**. |

---

## Gap Summary by Severity

### CRITICAL Priority (0 gaps)
*No gaps classified as CRITICAL severity*

### HIGH Priority (8 gaps)
- **GAP-1** - JWK Thumbprint for AKP Keys
- **GAP-2** - SAML Signing Algorithm URIs
- **GAP-3** - SAML Encryption ML-KEM
- **GAP-4** - ML-KEM CEK Provider
- **GAP-6** - BC-FIPS Blocker (External)
- **GAP-8** - FAPI Allowlist (Spec-Gated)
- **GAP-9** - SAML Hardcoded RS256 Key Selection
- **GAP-15** - ML-DSA Key Generation
- **GAP-24** - JWKS Endpoint AKP Branch
- **GAP-28** - Client Signature Verifier RSA Guard

### MEDIUM Priority (15 gaps)
- **GAP-5** - Keystore ML-DSA Import
- **GAP-7** - LD-Proof Suite (External)
- **GAP-10** - IdP Broker Fallback
- **GAP-12** - DEFAULT_SIGNATURE_ALGORITHM Fallback (Documentation)
- **GAP-13** - Admin CLI Hardcoded RS256
- **GAP-16** - Keystore Admin UI
- **GAP-17** - FAPI Executor Factories (Depends on GAP-8)
- **GAP-18** - OID4VC c_nonce Hardcoded
- **GAP-19** - bcprov Minimum Version
- **GAP-20** - JAR Encrypted Request Decryption
- **GAP-21** - UserInfo Per-Client Signing
- **GAP-22** - Attestation Algorithm Enforcement
- **GAP-25** - Admin API Client Keypair Generation
- **GAP-26** - Client SDK JWT Credentials
- **GAP-27** - Client SDK DPoP Generation

### LOW Priority (3 gaps)
- **GAP-11** - DCR RS256 Special-Case
- **GAP-14** - JGroups ASYM_ENCRYPT (Documentation)
- **GAP-23** - OIDC Discovery Hardcoded Constant

---

## Gaps by GitHub Issue Status

### ✅ With Existing GitHub Issues (6 gaps)
- **GAP-2** → [#50294](https://github.com/keycloak/keycloak/issues/50294) (SAML signing URIs)
- **GAP-3** → [#50295](https://github.com/keycloak/keycloak/issues/50295) (SAML encryption)
- **GAP-5** → [#50679](https://github.com/keycloak/keycloak/issues/50679) (Keystore import)
- **GAP-15** → [#44142](https://github.com/keycloak/keycloak/issues/44142) (ML-DSA key generation)
- **GAP-16** → [#50679](https://github.com/keycloak/keycloak/issues/50679) (same as GAP-5)
- **GAP-28** → [#43684](https://github.com/keycloak/keycloak/issues/43684) (ClientSignatureVerifierProviderFactory)

### ⚠️ Need Verification (2 gaps)
- **GAP-1** → Check if [#44141](https://github.com/keycloak/keycloak/issues/44141) (CLOSED) covered JWK thumbprint; if not, create sub-issue under [#43692](https://github.com/keycloak/keycloak/issues/43692)
- **GAP-24** → Check if [#44141](https://github.com/keycloak/keycloak/issues/44141) (CLOSED) covered JWKS endpoint; if not, create sub-issue under [#43692](https://github.com/keycloak/keycloak/issues/43692)

### 🆕 Need New Issues - Under [#48821](https://github.com/keycloak/keycloak/issues/48821) OAuth/OIDC (11 gaps)
- **GAP-4** - ML-KEM CEK Provider (HIGH)
- **GAP-10** - IdP Broker Fallback (MEDIUM)
- **GAP-11** - DCR RS256 Special-Case (LOW) *or document in #48823*
- **GAP-13** - Admin CLI (MEDIUM) *or under #48823*
- **GAP-20** - JAR Encrypted Request Decryption (MEDIUM)
- **GAP-21** - UserInfo Per-Client Signing (MEDIUM) *also document in #48823*
- **GAP-23** - OIDC Discovery Constant (LOW)
- **GAP-25** - Admin API Client Keypair (MEDIUM)
- **GAP-26** - Client SDK JWT Credentials (MEDIUM) *could combine with GAP-27*
- **GAP-27** - Client SDK DPoP (MEDIUM) *could combine with GAP-26*

### 🆕 Need New Issues - Under [#50292](https://github.com/keycloak/keycloak/issues/50292) SAML (1 gap)
- **GAP-9** - SAML Hardcoded RS256 Key Selection (HIGH)

### 🆕 Need New Issues - Under [#46333](https://github.com/keycloak/keycloak/issues/46333) Crypto Defaults (1 gap)
- **GAP-19** - bcprov Minimum Version (MEDIUM)

### 🆕 Need New Issues - Standalone or Under [#43684](https://github.com/keycloak/keycloak/issues/43684) (1 gap)
- **GAP-22** - Attestation Algorithm Enforcement (MEDIUM) *could combine with GAP-28*

### 🆕 Need New Issues - Standalone OID4VC (1 gap)
- **GAP-18** - OID4VC c_nonce Hardcoded Algorithm (MEDIUM)

### 📝 External Dependencies / Documentation Only (4 gaps)
- **GAP-6** - BC-FIPS external dependency (document only)
- **GAP-7** - LD-Proof spec dependency (document only)
- **GAP-8** - FAPI 2.0 spec dependency (document in #48823)
- **GAP-12, GAP-14** - Documentation/operator guidance (document in #48823)
- **GAP-17** - Resolves when GAP-8 is addressed (no separate issue needed)

---

## Domain Coverage

| Domain # | Gap ID | Status |
|----------|--------|--------|
| 2, 4, 8 | GAP-4 | No issue |
| 4 | GAP-21 | No issue |
| 8 | GAP-20 | No issue |
| 11, 13, 43 | GAP-6 | External |
| 12 | GAP-1, GAP-22 | No issue |
| 15 | GAP-1 | No issue |
| 16, 41 | GAP-8, GAP-17 | Spec-gated |
| 18, 20, 59, 60, 61 | GAP-2, GAP-9 | GAP-2: #50294, GAP-9: No issue |
| 19 | GAP-3 | #50295 |
| 21 | GAP-10 | No issue |
| 25 | GAP-7 | External |
| 27 | GAP-18 | No issue |
| 28, 30, 31, 46 | GAP-15 | #44142 |
| 33, 34 | GAP-5, GAP-16 | #50679 |
| 35 | GAP-11 | No issue |
| 36 | GAP-23 | No issue |
| 38 | GAP-13 | No issue |
| 39 | GAP-14 | Documentation |
| 42 | GAP-19 | No issue |
| 45 | GAP-1 | No issue |
| 48 | GAP-24 | No issue |
| 49 | GAP-25 | No issue |
| 50 | GAP-26 | No issue |
| 51 | GAP-27 | No issue |
| 53 | GAP-28 | No issue |
| Multiple | GAP-12 | Documentation |

---

**Total Gaps:** 28  
**With Existing GitHub Issues:** 6  
**Need Verification:** 2  
**Need New Issues:** 16 (could be 14-15 if combined)  
**External/Documentation:** 5
