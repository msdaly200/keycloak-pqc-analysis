# REASSESS_PQC_READINESS_PROMPT

## Purpose

This prompt instructs an AI agent to perform a complete accuracy re-assessment of
`pqc_overview.html` against the live Keycloak source code. It replicates the methodology
used in the original review (`pqc_overview_review_findings.md`) and produces an updated
findings report.

**Scope:** Every domain row (currently 62), every GAP entry (currently 25), and the
"Files Evaluated — Not Added as New Rows" section of `pqc_overview.html`.

**Output:** A new findings file (see naming convention below). Do NOT modify
`pqc_overview.html` or any other file until the human reviewer has read and approved
the findings report.

---

## Instructions for the AI Agent

You are a Senior Keycloak engineer with deep knowledge of cryptographic algorithms and
the Keycloak codebase.

Your task is to verify that every factual claim in `pqc_overview.html` is accurate
against the Keycloak source at `/Users/mariedaly/projects/keycloak` (main branch).

Work through each section in order. For each item, verify it against the actual source.
Record your findings in a new file named:

```
pqc_overview_review_findings_YYYY-MM-DD.md
```

where `YYYY-MM-DD` is today's date.

Use the structure in `pqc_overview_review_findings.md` as your template. Do NOT copy
it — produce fresh findings by re-running every check. Your report must have these
sections:

- **Part 1** — Errors and inaccuracies (correcting existing wrong content)
- **Part 2** — Complete domain-by-domain verification table (all rows)
- **Part 3** — Complete GAP-by-GAP verification table (all GAPs)
- **Part 4** — "Files Evaluated" section verification
- **Part 5** — Status distribution table verification
- **Part 6** — New content not covered in the current HTML (new domains, new files)
- **Part 7** — Full list of required changes, separated into corrections vs. additions

After completing the findings file, update `README.md` to record when this prompt was
last run and add a link to the new findings file. See the README update instructions
at the bottom of this prompt.

**Do NOT modify `pqc_overview.html` until the human reviewer approves.**

---

## Pre-Flight Checks (do once before starting)

```bash
# 1. Confirm Keycloak repo location and HEAD commit
cd /Users/mariedaly/projects/keycloak && git log --oneline -3

# 2. Confirm pqc_analysis workspace
ls /Users/mariedaly/projects/pqc_analysis/pqc_overview.html
```

---

## Section A — Known-Stable Facts (do NOT re-verify, just copy forward)

These facts are structurally stable and extremely unlikely to change. Record them as
"confirmed stable" in your report without re-running the checks.

| Fact | Where to find it if you must check |
|------|-------------------------------------|
| `bcprov-jdk18on` version is **NOT** set in any Keycloak-managed `pom.xml` — it inherits from the Quarkus BOM. At the time of last review it resolved to **1.84** via `quarkus-bom:3.33.2.1`. | `~/.m2/repository/io/quarkus/platform/quarkus-bom/3.33.2.1/quarkus-bom-3.33.2.1.pom` — grep for `bcprov-jdk18on` |
| `bc-fips` version is pinned to **2.1.2** in the root `pom.xml` at property `bouncycastle.bcfips.version` (line ~78). | `/Users/mariedaly/projects/keycloak/pom.xml` lines 77–80 |
| `Algorithm.java` has `ML_DSA_44`, `ML_DSA_65`, `ML_DSA_87` constants; `KeyType.AKP = "AKP"` exists. | `core/src/main/java/org/keycloak/crypto/Algorithm.java` lines 58–60; `KeyType.java` line 25 |
| `AKPPublicJWK.java`, `AKPUtils.java`, `JWKBuilder.akp()`, and `JWKParser` AKP parsing all exist in the core JWK layer. | `core/src/main/java/org/keycloak/jose/jwk/` |
| No `GeneratedAKPKeyProviderFactory`, no ML-DSA `SignatureProviderFactory`, and no ML-KEM `CekManagementProviderFactory` exist anywhere in the non-test source. | `find /Users/mariedaly/projects/keycloak -name "*.java" ! -path "*/test/*" \| xargs grep -l "ML.DSA\|ML.KEM\|AKP" 2>/dev/null` — confirmed to return only `Algorithm.java`, `KeyType.java`, `AKPPublicJWK.java`, `AKPUtils.java`, `JWKBuilder.java`, `JWKParser.java` |
| Maven Enforcer plugin is present in `pom.xml` (versions at lines 191–192) but there is **no minimum-version rule** for `bcprov-jdk18on`. | `pom.xml` lines 191–192 |
| `JWKSUtils.JWK_THUMBPRINT_REQUIRED_MEMBERS` map contains entries for `KeyType.RSA`, `KeyType.EC`, and `KeyType.OKP` only — `AKP` is absent. Calling `computeThumbprint()` for an AKP key throws `UnsupportedOperationException` at line 162. | `core/src/main/java/org/keycloak/util/JWKSUtils.java` lines 52–57, 162 |
| `TokenCategory.LOGOUT` and `TokenCategory.ID` resolve to the same algorithm via `DefaultTokenManager.signatureAlgorithm()` — both use `ID_TOKEN_SIGNED_RESPONSE_ALG`. | `services/src/main/java/org/keycloak/jose/jws/DefaultTokenManager.java` lines 199–213 |
| `Constants.DEFAULT_SIGNATURE_ALGORITHM = Algorithm.RS256` and `Constants.INTERNAL_SIGNATURE_ALGORITHM = Algorithm.HS512`. | `server-spi-private/src/main/java/org/keycloak/models/Constants.java` lines 68–69 |
| `SAML2Signature.signatureMethod` defaults to `SignatureMethod.RSA_SHA1` (line 55); `digestMethod` defaults to `DigestMethod.SHA1` (line 57). | `saml-core/src/main/java/org/keycloak/saml/processing/api/saml/v2/sig/SAML2Signature.java` |
| `DefaultKeyProviders.createProviders()` bootstraps only `rsa-generated` (SIG) and `rsa-enc-generated` (ENC) — no ML-DSA or ML-KEM provider. | `server-spi-private/src/main/java/org/keycloak/models/utils/DefaultKeyProviders.java` lines 36–42 |
| `SsfSignatureAlgorithms.ALLOWED = Set.of(Algorithm.RS256)` (CAEP interop profile pins RS256). | `ssf/transmitter/src/main/java/org/keycloak/ssf/transmitter/event/SsfSignatureAlgorithms.java` line 31 |
| `FapiConstant.ALLOWED_ALGORITHMS` contains only `{PS256, PS384, PS512, ES256, ES384, ES512}`. | `services/src/main/java/org/keycloak/services/clientpolicy/executor/FapiConstant.java` lines 30–34 |
| `JWKSServerUtils.toJwk()` has `if/else if` branches for `KeyType.RSA`, `KeyType.EC`, and `KeyType.OKP` only — an AKP key returns `null`. | `services/src/main/java/org/keycloak/protocol/oidc/utils/JWKSServerUtils.java` lines 59–63 |
| `OIDCWellKnownProvider.DEFAULT_CLIENT_AUTH_SIGNING_ALG_VALUES_SUPPORTED = list(Algorithm.RS256.toString())` (static hardcoded constant). | `services/src/main/java/org/keycloak/protocol/oidc/OIDCWellKnownProvider.java` line 85 |
| `ClientAsymmetricSignatureVerifierContext.getKey()` throws `VerificationException("Key Type is not RSA: " + key.getType())` at line 37 for any non-RSA key. | `services/src/main/java/org/keycloak/crypto/ClientAsymmetricSignatureVerifierContext.java` lines 36–37 |
| `AuthUtil.getSignedRequestToken()` calls `.rsa256(keypair.getPrivate())` at line 213. | `integration/client-cli/admin-cli/src/main/java/org/keycloak/client/cli/util/AuthUtil.java` line 213 |
| `JwtCNonceHandler.selectSigningKey()` tries `Algorithm.ES256` (line 281) then `Algorithm.RS256` (line 287). | `services/src/main/java/org/keycloak/protocol/oid4vc/issuance/keybinding/JwtCNonceHandler.java` lines 281/287 |
| `JWTClientCredentialsProvider.setupKeyPair()` switch at lines 76–96 covers `KeyType.RSA`, `KeyType.EC`, `KeyType.OKP` only — no `AKP` case; default throws `RuntimeException("Invalid KeyPair algorithm")`. | `core/src/main/java/org/keycloak/protocol/oidc/client/authentication/JWTClientCredentialsProvider.java` lines 76–96 |
| `SignatureAlgorithm.java` (SAML) contains only RSA variants (RSA_SHA1, RSA_SHA256, RSA_SHA256_MGF1, RSA_SHA512, RSA_SHA512_MGF1). No PQC URIs. | `saml-core/src/main/java/org/keycloak/saml/SignatureAlgorithm.java` lines 29–33 |
| `XMLEncryptionUtil` supports only RSA-OAEP (RSA_OAEP_11 as default), RSA_OAEP, and RSA_v1dot5 key transport. | `saml-core/src/main/java/org/keycloak/saml/processing/core/util/XMLEncryptionUtil.java` lines 77, 186–206 |
| `SAMLEncryptionAlgorithms` enum: `RSA_OAEP(RSA_OAEP, RSA_OAEP_11)` and `RSA1_5(RSA1_5)` only. | `services/src/main/java/org/keycloak/protocol/saml/SAMLEncryptionAlgorithms.java` lines 33–34 |
| `DescriptionConverter.java` line 416 RS256 special-case: `if (Algorithm.RS256.equals(defaultSignatureAlgorithm) \|\| StringUtil.isBlank(defaultSignatureAlgorithm))` | `services/src/main/java/org/keycloak/services/clientregistration/oidc/DescriptionConverter.java` line 416 |
| `DPoPUtil` calls `JWKSUtils.computeThumbprint(jwk)` at line 242 — this will throw for AKP keys. | `services/src/main/java/org/keycloak/services/util/DPoPUtil.java` line 242 |
| `DockerComposeCertsDirectory` calls `getKeyPairGen(KeyType.RSA)` and `keyGen.initialize(2048)` at lines 29–30. | `services/src/main/java/org/keycloak/protocol/docker/installation/compose/DockerComposeCertsDirectory.java` lines 29–30 |
| `AttestationBasedClientAuthenticator.java` has `[TODO]` at lines 418–419 for algorithm enforcement. Full path: `authentication/authenticators/client/AttestationBasedClientAuthenticator.java`. | `services/src/main/java/org/keycloak/authentication/authenticators/client/AttestationBasedClientAuthenticator.java` |
| `AbstractOAuth2IdentityProvider.java` RS256 fallback is at **line 745** only (no HS256 at line 689). | `services/src/main/java/org/keycloak/broker/oidc/AbstractOAuth2IdentityProvider.java` line 745 |
| `SAMLIdentityProvider.java` RS256 hardcodings are at **line 414** and **line 507** (not 415/506 as was incorrectly stated previously). `SamlProtocol.java` RS256 at line 544. `SamlService.java` RS256 at line 965. | Verified in last review |
| `DefaultSamlArtifactResolver.java` handles artifact storage/routing only — it does NOT perform SOAP backchannel assertion fetching. The SOAP fetch path is in `SAMLEndpoint.java`. | `services/src/main/java/org/keycloak/protocol/saml/DefaultSamlArtifactResolver.java` |
| OID4VP domain is entirely absent from `pqc_overview.html`. Files: `OID4VPIdentityProvider.java`, `EphemeralKey.java`, `ResponseEncryption.java`, `OID4VPIdentityProviderEndpoint.java`, `RequestObject.java` in `services/src/main/java/org/keycloak/broker/oid4vp/`. Crypto: ES256 hardcoded for request object signing; ECDH-ES/secp256r1 hardcoded for `direct_post.jwt` response encryption (HAIP spec-gated). | `services/src/main/java/org/keycloak/broker/oid4vp/` |
| `DIDUtils.java` is P-256 only by design — only handles `did:key` for P-256/ES256. | `core/src/main/java/org/keycloak/util/DIDUtils.java` |

---

## Section B — Items to Verify Actively on Every Run

For each item below, grep or read the referenced file and confirm the stated fact still
holds. Mark ✅ (confirmed), ❌ (incorrect — state what changed), or ⚠️ (partially correct
— describe).

### B1 — All 62 Domain Rows: File Existence Check

Run a single find command to confirm every file referenced in the Key Files column still
exists at its expected path. For efficiency, batch them:

```bash
cd /Users/mariedaly/projects/keycloak

# Token Lifecycle (rows 1–9)
find . \( \
  -name "DefaultTokenManager.java" \
  -o -name "AsymmetricSignatureProvider.java" \
  -o -name "RS256SignatureProviderFactory.java" \
  -o -name "PS256SignatureProviderFactory.java" \
  -o -name "ES256SignatureProviderFactory.java" \
  -o -name "EdDSASignatureProviderFactory.java" \
  -o -name "RsaCekManagementProvider.java" \
  -o -name "RsaesOaepCekManagementProviderFactory.java" \
  -o -name "RsaesPkcs1CekManagementProviderFactory.java" \
  -o -name "EcdhEsCekManagementProvider.java" \
  -o -name "EcdhEsA128KwCekManagementProviderFactory.java" \
  -o -name "EcdhEsA192KwCekManagementProviderFactory.java" \
  -o -name "EcdhEsA256KwCekManagementProviderFactory.java" \
  -o -name "TokenManager.java" \
  -o -name "ResourceAdminManager.java" \
  -o -name "AccessTokenIntrospectionProvider.java" \
  -o -name "AuthenticationManager.java" \
  -o -name "LoginActionsService.java" \
  -o -name "AuthzEndpointRequestObjectParser.java" \
  -o -name "ParEndpointRequestObjectParser.java" \
  -o -name "OIDCRedirectUriBuilder.java" \
  -o -name "UserInfoEndpoint.java" \
\) ! -path "*/test/*" 2>/dev/null | sort

# Client & End-User Auth (rows 10–16)
find . \( \
  -name "JWTClientAuthenticator.java" \
  -o -name "AbstractJWTClientValidator.java" \
  -o -name "AbstractBaseJWTValidator.java" \
  -o -name "X509ClientAuthenticator.java" \
  -o -name "CertificateValidator.java" \
  -o -name "MtlsHoKTokenUtil.java" \
  -o -name "X509ClientCertificateLookup.java" \
  -o -name "AttestationBasedClientAuthenticator.java" \
  -o -name "WebAuthnRegister.java" \
  -o -name "WebAuthnCredentialProvider.java" \
  -o -name "WebAuthnPolicy.java" \
  -o -name "DPoPUtil.java" \
  -o -name "BackchannelAuthenticationEndpointSignedRequestParser.java" \
  -o -name "CibaClientValidation.java" \
  -o -name "SecureCibaAuthenticationRequestSigningAlgorithmExecutor.java" \
\) ! -path "*/test/*" 2>/dev/null | sort

# SAML & IdP Broker (rows 17–23, 55, 59–61)
find . \( \
  -name "JWTAuthorizationGrantType.java" \
  -o -name "DefaultJWTAuthorizationGrantValidator.java" \
  -o -name "IDJWTAuthorizationGrantValidator.java" \
  -o -name "JWTAuthorizationGrantIdentityProvider.java" \
  -o -name "SignatureAlgorithm.java" \
  -o -name "XMLSignatureUtil.java" \
  -o -name "XMLEncryptionUtil.java" \
  -o -name "BaseSAML2BindingBuilder.java" \
  -o -name "SamlProtocol.java" \
  -o -name "SamlService.java" \
  -o -name "SamlProtocolUtils.java" \
  -o -name "SAMLEncryptionAlgorithms.java" \
  -o -name "SAMLDecryptionKeysLocator.java" \
  -o -name "SAMLIdentityProvider.java" \
  -o -name "SAMLEndpoint.java" \
  -o -name "SAMLIdentityProviderFactory.java" \
  -o -name "DefaultTrustIdentityProvider.java" \
  -o -name "DefaultTrustIdentityProviderConfig.java" \
  -o -name "DefaultTrustIdentityProviderFactory.java" \
  -o -name "OIDCIdentityProvider.java" \
  -o -name "AbstractOAuth2IdentityProvider.java" \
  -o -name "KeycloakOIDCIdentityProvider.java" \
  -o -name "KubernetesIdentityProvider.java" \
  -o -name "SpiffeIdentityProvider.java" \
  -o -name "SamlAbstractMetadataPublicKeyLoader.java" \
  -o -name "DefaultSamlArtifactResolver.java" \
  -o -name "DefaultSamlArtifactResolverFactory.java" \
  -o -name "SAML2Signature.java" \
\) ! -path "*/test/*" 2>/dev/null | sort

# OID4VC, SD-JWT, DPoP, Token Exchange (rows 24–27, 51, 56–58)
find . \( \
  -name "AbstractCredentialSigner.java" \
  -o -name "JwtCredentialSigner.java" \
  -o -name "SdJwtCredentialSigner.java" \
  -o -name "LDCredentialSigner.java" \
  -o -name "Ed255192018Suite.java" \
  -o -name "JwtProofValidator.java" \
  -o -name "AbstractProofValidator.java" \
  -o -name "AttestationValidatorUtil.java" \
  -o -name "JwtCNonceHandler.java" \
  -o -name "SdJwt.java" \
  -o -name "IssuerSignedJWT.java" \
  -o -name "JwsToken.java" \
  -o -name "SdJwtVerificationContext.java" \
  -o -name "KeyBindingJWT.java" \
  -o -name "SdJwtVP.java" \
  -o -name "DPoPGenerator.java" \
  -o -name "TokenExchangeGrantType.java" \
  -o -name "StandardTokenExchangeProvider.java" \
  -o -name "AbstractTokenExchangeProvider.java" \
  -o -name "DeviceGrantType.java" \
\) ! -path "*/test/*" 2>/dev/null | sort

# Key Management, JWKS, Admin (rows 28–36, 45–53)
find . \( \
  -name "GeneratedRsaKeyProviderFactory.java" \
  -o -name "AbstractGeneratedRsaKeyProviderFactory.java" \
  -o -name "AbstractRsaKeyProvider.java" \
  -o -name "GeneratedRsaEncKeyProviderFactory.java" \
  -o -name "ImportedRsaEncKeyProviderFactory.java" \
  -o -name "GeneratedEcdsaKeyProviderFactory.java" \
  -o -name "AbstractGeneratedEcKeyProviderFactory.java" \
  -o -name "GeneratedEcdsaKeyProvider.java" \
  -o -name "GeneratedEddsaKeyProviderFactory.java" \
  -o -name "AbstractEddsaKeyProvider.java" \
  -o -name "GeneratedEddsaKeyProvider.java" \
  -o -name "GeneratedEcdhKeyProviderFactory.java" \
  -o -name "GeneratedEcdhKeyProvider.java" \
  -o -name "ImportedRsaKeyProviderFactory.java" \
  -o -name "AbstractImportedRsaKeyProviderFactory.java" \
  -o -name "JavaKeystoreKeyProviderFactory.java" \
  -o -name "JavaKeystoreKeyProvider.java" \
  -o -name "ClientRegistrationTokenUtils.java" \
  -o -name "JWKBuilder.java" \
  -o -name "JWKParser.java" \
  -o -name "AKPPublicJWK.java" \
  -o -name "AKPUtils.java" \
  -o -name "JWKSServerUtils.java" \
  -o -name "DefaultKeyProviders.java" \
  -o -name "ClientPublicKeyLoader.java" \
  -o -name "HardcodedPublicKeyLoader.java" \
  -o -name "ClientAttributeCertificateResource.java" \
  -o -name "JWTClientCredentialsProvider.java" \
\) ! -path "*/test/*" 2>/dev/null | sort

# Crypto Backends, FAPI, Admin CLI, Cluster, Org (rows 37–44, 54)
find . \( \
  -name "SecurityEventTokenEncoder.java" \
  -o -name "SsfSignatureAlgorithms.java" \
  -o -name "SecurityEventTokenDispatcher.java" \
  -o -name "AuthUtil.java" \
  -o -name "DefaultCryptoProvider.java" \
  -o -name "FIPS1402Provider.java" \
  -o -name "BCFIPSCertificateUtilsProvider.java" \
  -o -name "WildFlyElytronProvider.java" \
  -o -name "FapiConstant.java" \
  -o -name "SecureSigningAlgorithmExecutor.java" \
  -o -name "SecureSigningAlgorithmForSignedJwtExecutor.java" \
  -o -name "SecureSigningAlgorithmExecutorFactory.java" \
  -o -name "SecureCibaAuthenticationRequestSigningAlgorithmExecutorFactory.java" \
  -o -name "FederatedJWTClientAuthenticator.java" \
  -o -name "DefaultClientAssertionStrategy.java" \
  -o -name "SpiffeClientAssertionStrategy.java" \
  -o -name "Organizations.java" \
  -o -name "OIDCWellKnownProvider.java" \
\) ! -path "*/test/*" 2>/dev/null | sort

# OID4VP (row 62 — new domain)
find . \( \
  -name "OID4VPIdentityProvider.java" \
  -o -name "EphemeralKey.java" \
  -o -name "ResponseEncryption.java" \
  -o -name "OID4VPIdentityProviderEndpoint.java" \
  -o -name "RequestObject.java" \
\) ! -path "*/test/*" -path "*/oid4vp/*" 2>/dev/null | sort
```

For any file that is NOT returned by these searches, flag it as **MISSING** and note
whether the row's PQC assessment still holds (perhaps it was renamed or merged).

---

### B2 — Line-Number-Sensitive Claims to Verify

#### Line-shift tolerance rule

**If a `grep -n` shows a line number has moved by ±3 lines or fewer from the value
recorded in `pqc_overview.html`, AND the cryptographic code itself is unchanged, treat
it as ✅ (confirmed) — do NOT raise it as an error.** Only flag a line-number change as
an issue when:

- The cryptographic code at that line has been substantively altered or removed (report
  as ❌ with the new content), **or**
- The line has shifted by more than 3 positions from the documented value (report as
  ⚠️ with the new line number so the HTML can be updated on the next edit pass).

This avoids noise from trivial additions like import statements or comment blocks that
shift line numbers without changing the PQC-relevant code.

#### How to check

Use `grep -n` to search for the **distinctive code fragment** rather than reading a
fixed line number directly. The grep tells you both that the code still exists and its
current line number, so you can apply the tolerance rule in one step.

Report each item as:
- ✅ — code present, line within ±3 of documented value (or exact match)
- ⚠️ LINE SHIFTED — code present but line has moved by more than 3; note new line
- ❌ CODE CHANGED — the cryptographic logic at this location has been altered or removed

```bash
cd /Users/mariedaly/projects/keycloak

# GAP-9: SAMLIdentityProvider.java — two RS256 hardcodings (documented: lines 414, 507)
grep -n "Algorithm\.RS256" services/src/main/java/org/keycloak/broker/saml/SAMLIdentityProvider.java

# GAP-9: SamlProtocol.java — RS256 hardcoding (documented: line 544)
grep -n "Algorithm\.RS256" services/src/main/java/org/keycloak/protocol/saml/SamlProtocol.java

# GAP-9: SamlService.java — RS256 hardcoding (documented: line 965)
grep -n "Algorithm\.RS256" services/src/main/java/org/keycloak/protocol/saml/SamlService.java

# GAP-10: AbstractOAuth2IdentityProvider.java — RS256 fallback (documented: line 745)
grep -n "Algorithm\.RS256" services/src/main/java/org/keycloak/broker/oidc/AbstractOAuth2IdentityProvider.java

# GAP-11: DescriptionConverter.java — RS256 special-case (documented: line 416)
grep -n "Algorithm\.RS256\|RS256.*equals\|equals.*RS256" services/src/main/java/org/keycloak/services/clientregistration/oidc/DescriptionConverter.java

# GAP-12: DefaultTokenManager.java — DEFAULT_SIGNATURE_ALGORITHM fallback (documented: ~line 233)
grep -n "DEFAULT_SIGNATURE_ALGORITHM\|Constants\.DEFAULT" services/src/main/java/org/keycloak/jose/jws/DefaultTokenManager.java

# GAP-13: AuthUtil.java — rsa256 hardcoding (documented: line 213)
grep -n "\.rsa256\(" integration/client-cli/admin-cli/src/main/java/org/keycloak/client/cli/util/AuthUtil.java

# GAP-18: JwtCNonceHandler.java — ES256 then RS256 fallback (documented: lines 281/287)
grep -n "Algorithm\.ES256\|Algorithm\.RS256\|getActiveKey" services/src/main/java/org/keycloak/protocol/oid4vc/issuance/keybinding/JwtCNonceHandler.java

# GAP-22: AttestationBasedClientAuthenticator.java — [TODO] algorithm enforcement (documented: lines 418-419)
grep -n "\[TODO\].*alg\|alg.*\[TODO\]" services/src/main/java/org/keycloak/authentication/authenticators/client/AttestationBasedClientAuthenticator.java

# GAP-23: OIDCWellKnownProvider.java — hardcoded RS256 constant (documented: line 85)
grep -n "DEFAULT_CLIENT_AUTH_SIGNING_ALG_VALUES_SUPPORTED" services/src/main/java/org/keycloak/protocol/oidc/OIDCWellKnownProvider.java

# Domain 48: JWKSServerUtils.toJwk() — no AKP branch (documented: lines 59-63)
grep -n "KeyType\.\|toJwk\b" services/src/main/java/org/keycloak/protocol/oidc/utils/JWKSServerUtils.java

# Domain 53: ClientAsymmetricSignatureVerifierContext — RSA guard (documented: lines 36-37)
grep -n "not RSA\|Key Type is not RSA\|KeyType\.RSA" services/src/main/java/org/keycloak/crypto/ClientAsymmetricSignatureVerifierContext.java

# Domain 61: SAML2Signature — RSA_SHA1 default (documented: lines 55/57)
grep -n "RSA_SHA1\|DigestMethod\.SHA1\|signatureMethod\s*=" saml-core/src/main/java/org/keycloak/saml/processing/api/saml/v2/sig/SAML2Signature.java

# Domain 52: DockerComposeCertsDirectory — RSA 2048 hardcoding (documented: lines 29-30)
grep -n "getKeyPairGen\|initialize(2048\|KeyType\.RSA" services/src/main/java/org/keycloak/protocol/docker/installation/compose/DockerComposeCertsDirectory.java

# Domain 50: JWTClientCredentialsProvider — no AKP case in switch (documented: lines 76-96)
grep -n "KeyType\.RSA\|KeyType\.EC\|KeyType\.OKP\|Invalid KeyPair algorithm\|switch.*getKeyType\|getKeyType" core/src/main/java/org/keycloak/protocol/oidc/client/authentication/JWTClientCredentialsProvider.java

# Domain 27 / GAP-18: JwtCNonceHandler.selectSigningKey — ES256/RS256 (documented: lines 281/287)
# (covered by GAP-18 grep above — no need to re-run)

# OID4VP row 62: ES256 hardcodings (documented: lines 79, 187-188)
grep -n "ACCEPTED_ALGORITHMS\|Algorithm\.ES256\|getActiveKey.*ES256\|ES256.*getActiveKey" services/src/main/java/org/keycloak/broker/oid4vp/OID4VPIdentityProvider.java

# OID4VP row 62: ECDH-ES/secp256r1 hardcoding (documented: EphemeralKey lines ~35/41, ResponseEncryption line ~38)
grep -n "secp256r1\|CURVE_SEC\|ECDH_ES\|KEY_MANAGEMENT_ALG" services/src/main/java/org/keycloak/broker/oid4vp/EphemeralKey.java services/src/main/java/org/keycloak/broker/oid4vp/ResponseEncryption.java
```

---

### B3 — Algorithm / PQC State Claims to Verify

These are the claims most likely to change as Keycloak progresses on PQC. Run these checks:

```bash
cd /Users/mariedaly/projects/keycloak

# Has any ML-DSA SignatureProviderFactory been added?
find . -name "*.java" ! -path "*/test/*" | xargs grep -l "ML.DSA\|ML.KEM\|AKP" 2>/dev/null \
  | grep -v "Algorithm.java\|KeyType.java\|AKPPublicJWK\|AKPUtils\|JWKBuilder\|JWKParser\|JWKSUtils\|oid4vp" \
  | sort

# Has any CekManagementProviderFactory for ML-KEM been added?
find . -name "*MLKEM*CekManagement*.java" -o -name "*ML.KEM*CekManagement*.java" 2>/dev/null | grep -v test

# Has GeneratedAKPKeyProviderFactory been added?
find . -name "GeneratedAKP*.java" ! -path "*/test/*" 2>/dev/null

# Has a new ML-DSA SignatureProvider been wired in JavaAlgorithm?
grep -n "ML.DSA\|AKP\|mldsa" core/src/main/java/org/keycloak/crypto/JavaAlgorithm.java

# Has bcprov version changed (re-check Quarkus BOM version in pom.xml)?
grep -n "quarkus.version\|quarkus.platform" pom.xml | head -5

# Has bc-fips version changed?
grep -n "bouncycastle.bcfips.version" pom.xml

# Has DefaultKeyProviders been updated to bootstrap ML-DSA keys?
grep -n "AKP\|ML.DSA\|ML.KEM\|akp" server-spi-private/src/main/java/org/keycloak/models/utils/DefaultKeyProviders.java

# Has JWKSServerUtils.toJwk() been updated to handle AKP?
grep -n "AKP\|ML.DSA" services/src/main/java/org/keycloak/protocol/oidc/utils/JWKSServerUtils.java

# Has JWKSUtils.JWK_THUMBPRINT_REQUIRED_MEMBERS been updated for AKP?
grep -n "AKP\|ML.DSA\|REQUIRED_MEMBERS" core/src/main/java/org/keycloak/util/JWKSUtils.java | head -15

# Has FapiConstant.ALLOWED_ALGORITHMS been extended?
grep -n "ALLOWED_ALGORITHMS\|ML.DSA\|AKP" services/src/main/java/org/keycloak/services/clientpolicy/executor/FapiConstant.java

# Has OID4VP signing been made configurable (ES256 hardcoding fixed)?
grep -n "ES256\|ACCEPTED_ALGORITHMS\|Algorithm\." services/src/main/java/org/keycloak/broker/oid4vp/OID4VPIdentityProvider.java | head -10

# Has ClientAsymmetricSignatureVerifierContext RSA guard been fixed?
grep -n "KeyType.RSA\|not RSA\|AKP" services/src/main/java/org/keycloak/crypto/ClientAsymmetricSignatureVerifierContext.java

# Has the SsfSignatureAlgorithms ALLOWED set been extended?
grep -n "ALLOWED\|RS256\|ML.DSA" ssf/transmitter/src/main/java/org/keycloak/ssf/transmitter/event/SsfSignatureAlgorithms.java

# Has OIDCWellKnownProvider static constant been made dynamic?
grep -n "DEFAULT_CLIENT_AUTH_SIGNING\|RS256" services/src/main/java/org/keycloak/protocol/oidc/OIDCWellKnownProvider.java | head -5
```

---

### B4 — New Domain / File Discovery Sweep

Check whether any new crypto-relevant domains have been added to Keycloak since the
last review:

```bash
cd /Users/mariedaly/projects/keycloak

# Any new files in oid4vp that weren't in last review?
find . -path "*/oid4vp/*.java" ! -path "*/test/*" 2>/dev/null | sort

# Any new SignatureProviderFactory implementations?
find . -name "*SignatureProviderFactory.java" ! -path "*/test/*" 2>/dev/null | sort

# Any new CekManagementProviderFactory implementations?
find . -name "*CekManagementProviderFactory.java" ! -path "*/test/*" 2>/dev/null | sort

# Any new key provider factories?
find . -name "Generated*KeyProviderFactory.java" -o -name "Imported*KeyProviderFactory.java" ! -path "*/test/*" 2>/dev/null | grep -v test | sort

# Any new broker identity providers that involve signing/verification?
find . -name "*IdentityProvider.java" ! -path "*/test/*" 2>/dev/null \
  | xargs grep -l "SignatureProvider\|Algorithm\.\|RS256\|ES256\|AKP" 2>/dev/null \
  | grep -v "^Binary" | sort

# Any new files in sdjwt or oid4vc that use signing?
find . \( -path "*/sdjwt/*.java" -o -path "*/oid4vc/*.java" \) ! -path "*/test/*" 2>/dev/null \
  | xargs grep -l "SignatureProvider\|SignatureSignerContext\|algorithm\|RS256\|ES256" 2>/dev/null \
  | grep -v "^Binary" | sort
```

Compare results against the "Files Evaluated — Not Added as New Rows" section of
`pqc_overview.html`. Any file that appears in these results but is NOT covered by an
existing domain row or the "Files Evaluated" section must be investigated and documented
as a potential new domain or gap.

---

## Section C — Reporting

### C1 — Structure of the Output File

Your output file (`pqc_overview_review_findings_YYYY-MM-DD.md`) must follow this structure:

```
# PQC Overview — Complete Accuracy Review
**Date of review:** YYYY-MM-DD
**Keycloak branch:** main (HEAD: <commit hash from pre-flight check>)
**Scope:** Every claim in every row, every GAP, and "Files Evaluated" section

## Part 1 — Errors and Inaccuracies
<one subsection per error found, or "No errors found" if clean>

## Part 2 — Domain-by-Domain Verification Table
<table with columns: Row | Domain | Files exist? | Algorithm claims | PQC state | Notes>
<all 62+ rows>

## Part 3 — GAP Reference Verification
<table with columns: GAP | Title | Files Correct? | Line Numbers | Notes>
<all 25+ GAPs>
<Apply the ±3-line tolerance rule: a shift of ≤3 lines with unchanged crypto code is ✅ not an error>

## Part 4 — "Files Evaluated" Section Verification
<confirm each file in the exclusion list still exists and reason is still valid>

## Part 5 — Status Distribution Table Verification
<recount PENDING/PARTIAL/BLOCKED/SAFE/EXTERNAL and compare to HTML counts>

## Part 6 — New Content Not in Current HTML
<new domains, new files, or resolved gaps that should update the PQC state>

## Part 7 — Summary of Required Changes
<corrections to existing content>
<additions of new content>
<resolved items whose PQC state should be upgraded>
```

### C2 — Severity Labels

Use these labels consistently:

- **ERROR** — factual claim is substantively wrong: wrong file path, crypto code changed or removed, algorithm claim incorrect
- **LINE SHIFTED** — line number has moved by more than ±3 from the documented value, but the crypto code itself is unchanged; note the new line number for a future HTML edit pass (do NOT block the review for this)
- **STALE** — was correct when written, no longer true at current HEAD (e.g. a hardcoded RS256 fallback has been removed or made configurable)
- **NEW** — new file or domain not yet in the HTML
- **RESOLVED** — a gap that has been fixed (PQC state should improve)
- **CONFIRMED** — verified accurate (including line shifts of ≤3 where crypto code is unchanged)

---

## Section D — README Update

After the findings file is written and before reporting back to the user, update
`README.md` in `/Users/mariedaly/projects/pqc_analysis/`. Add or update the
following section (insert after the GitHub Issues section):

```markdown
## PQC Readiness Accuracy Reviews

The following accuracy reviews have been conducted against the live Keycloak source:

| Date | Findings File | Keycloak HEAD |
|------|--------------|---------------|
| YYYY-MM-DD | [YYYY-MM-DD Review](pqc_overview_review_findings_YYYY-MM-DD.md) | <commit> |
```

If the section already exists, append a new row to the table rather than replacing it.

**Do NOT modify `pqc_overview.html` or any domain/gap markdown files.** All changes
to those files require human review and approval first.

---

## Section E — Final Checklist Before Reporting

Before reporting back to the user, confirm:

- [ ] Pre-flight checks completed (repo location confirmed, HEAD commit noted)
- [ ] All Section A facts recorded as "confirmed stable" or flagged if changed
- [ ] All Section B1 file-existence checks completed
- [ ] All Section B2 line-number checks completed
- [ ] All Section B3 algorithm/PQC-state checks completed
- [ ] All Section B4 new-domain discovery checks completed
- [ ] Output file `pqc_overview_review_findings_YYYY-MM-DD.md` written
- [ ] `README.md` updated with date and link to findings file
- [ ] No changes made to `pqc_overview.html` or domain/gap markdown files
- [ ] Summary presented to user with count of: errors found, stale items, new domains, resolved gaps
