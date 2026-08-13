# PQC Gap Reference — Canonical Detail

> **This is the single source of truth for PQC gap descriptions.** The summary
> table in [`pqc_overview.html`](../pqc_overview.html) links here for full
> detail. Gap numbering matches the HTML's `GAP-N` tags exactly.

**Total gaps:** 30 (GAP-1 through GAP-30)  
**Keycloak HEAD at verification:** `f03a2104ec`  
**Last reconciled:** 2026-08-11

---

## GAP-1 — JWK Thumbprint — AKP (ML-DSA) Keys {#gap-1}

| Field | Value |
|-------|-------|
| **Severity** | HIGH |
| **Domains Affected** | 12, 15, 45 |
| **Affected File(s)** | `core/src/main/java/org/keycloak/util/JWKSUtils.java` (lines 53–58) |
| **GitHub Issue** | Check [#44141](https://github.com/keycloak/keycloak/issues/44141) (CLOSED) or [#43692](https://github.com/keycloak/keycloak/issues/43692) |

### Description & Impact

`AKP` is absent from `JWK_THUMBPRINT_REQUIRED_MEMBERS`. `JWKSUtils.java` lines 56–58 populate the map for `KeyType.RSA`, `KeyType.EC`, and `KeyType.OKP` only. Calling `computeThumbprint()` on an ML-DSA JWK throws `UnsupportedOperationException`. Directly breaks DPoP `dpop_jkt` binding (domain 15) and attestation-based client auth (domain 12) for ML-DSA keys.

### Resolution Criteria

An `AKP` key type entry appears in `JWK_THUMBPRINT_REQUIRED_MEMBERS` with the correct required member set for AKP JWKs, and `computeThumbprint()` succeeds for ML-DSA keys.

---

## GAP-2 — SAML Signing — No PQC Algorithm Entries {#gap-2}

| Field | Value |
|-------|-------|
| **Severity** | HIGH |
| **Domains Affected** | 18, 20, 59, 60, 61 |
| **Affected File(s)** | `saml-core/src/main/java/org/keycloak/saml/SignatureAlgorithm.java`; `saml-core/src/main/java/org/keycloak/saml/processing/core/util/XMLSignatureUtil.java` |
| **GitHub Issue** | [#50294](https://github.com/keycloak/keycloak/issues/50294) |

### Description & Impact

`SignatureAlgorithm` enum contains only RSA and DSA variants (lines 29–33); no PQC XML Digital Signature URIs are registered. `XMLSignatureUtil` uses `javax.xml.crypto.dsig` with standard algorithms only. SAML assertion signing cannot use ML-DSA at the algorithm layer. Tracked under [#50292](https://github.com/keycloak/keycloak/issues/50292) / [#50294](https://github.com/keycloak/keycloak/issues/50294) (partial).

### Resolution Criteria

Any ML-DSA URI constant appears in `SignatureAlgorithm`.

---

## GAP-3 — SAML Encryption — No ML-KEM Key Wrapping {#gap-3}

| Field | Value |
|-------|-------|
| **Severity** | HIGH |
| **Domains Affected** | 19 |
| **Affected File(s)** | `saml-core/src/main/java/org/keycloak/saml/processing/core/util/XMLEncryptionUtil.java` (lines 77, 186, 193, 196, 431) |
| **GitHub Issue** | [#50295](https://github.com/keycloak/keycloak/issues/50295) |

### Description & Impact

`XMLEncryptionUtil` supports only RSA-OAEP and RSA1_5 for key transport (confirmed RSA-only at lines 77, 186, 193, 196, 431). No ML-KEM key-encapsulation path exists. SAML assertion encryption cannot use quantum-safe key wrapping. Tracked under [#50292](https://github.com/keycloak/keycloak/issues/50292) / [#50295](https://github.com/keycloak/keycloak/issues/50295) (partial).

### Resolution Criteria

Any ML-KEM path appears in `XMLEncryptionUtil`.

---

## GAP-4 — JWE Key Management — No ML-KEM CEK Provider {#gap-4}

| Field | Value |
|-------|-------|
| **Severity** | HIGH |
| **Domains Affected** | 2, 4, 8, 63 |
| **Affected File(s)** | `crypto/…/RsaCekManagementProvider.java`; `crypto/…/EcdhEsCekManagementProvider.java`; `services/src/main/java/org/keycloak/jose/jws/DefaultTokenManager.java` |
| **GitHub Issue** | [#48821](https://github.com/keycloak/keycloak/issues/48821) (create sub-issue) |

### Description & Impact

All `CekManagementProviderFactory` implementations use RSA-OAEP or ECDH-ES. No ML-KEM provider exists (confirmed: no `*MLKEM*CekManagement*.java` found). Clients requesting encrypted ID tokens, encrypted UserInfo responses, or encrypted authorisation responses have no quantum-safe key encapsulation option. Blocks domains 2, 4, 8, 63. Must be implemented before any outbound token encryption can be quantum-safe. Related: [#50299](https://github.com/keycloak/keycloak/issues/50299) (HPKE alternative approach).

### Resolution Criteria

At least one `CekManagementProviderFactory` file supporting ML-KEM is found outside test paths.

---

## GAP-5 — JavaKeystoreKeyProvider — ML-DSA Key Import {#gap-5}

| Field | Value |
|-------|-------|
| **Severity** | MEDIUM |
| **Domains Affected** | 33, 34 |
| **Affected File(s)** | `services/src/main/java/org/keycloak/keys/JavaKeystoreKeyProvider.java`; `services/src/main/java/org/keycloak/keys/JavaKeystoreKeyProviderFactory.java` |
| **GitHub Issue** | [#50679](https://github.com/keycloak/keycloak/issues/50679) |

### Description & Impact

`JavaKeystoreKeyProvider` import logic handles RSA (line 223), EC (line 214), and OKP (line 205) branches only — no AKP branch present. BCFKS and PKCS12 keystores can physically store ML-DSA keys, but the provider cannot import them. Operators cannot import ML-DSA keys from a keystore file. See also GAP-16.

### Resolution Criteria

An `AKP` case appears in `JavaKeystoreKeyProvider`.

---

## GAP-6 — FIPS 140-2 Backend — ML-DSA Availability Blocked {#gap-6}

| Field | Value |
|-------|-------|
| **Severity** | HIGH |
| **Domains Affected** | 11, 13, 43 |
| **Affected File(s)** | `crypto/fips1402/…/FIPS1402Provider.java`; `crypto/fips1402/…/BCFIPSCertificateUtilsProvider.java`; `pom.xml` (line 78: `bouncycastle.bcfips.version=2.1.2`) |
| **GitHub Issue** | *External Dependency* |

### Description & Impact

BC-FIPS 2.1.2 (pinned at `pom.xml` line 78) contains only LMS under `org.bouncycastle.crypto.internal.pqc`. No ML-DSA and no ML-KEM classes are present. FIPS-mode Keycloak deployments cannot use ML-DSA or ML-KEM. Confirmed hard blocker for regulated environments targeting FIPS 140-2/3 compliance. Unblocked only when BC-FIPS 2.x achieves NIST FIPS 140-3 validation for ML-DSA/ML-KEM. Affects X.509 cert validation with PQC certificates in FIPS mode.

### Resolution Criteria

BC-FIPS ≥ 2.x with FIPS 140-3 validated ML-DSA/ML-KEM support is adopted and confirmed present.

---

## GAP-7 — OID4VC Linked Data Proof Suite — Hardcoded Ed25519 {#gap-7}

| Field | Value |
|-------|-------|
| **Severity** | MEDIUM |
| **Domains Affected** | 25 |
| **Affected File(s)** | `services/src/main/java/org/keycloak/protocol/oid4vc/issuance/signing/LDCredentialSigner.java`; `services/src/main/java/org/keycloak/protocol/oid4vc/issuance/signing/vcdm/Ed255192018Suite.java` (line 66) |
| **GitHub Issue** | *None — needs new issue* |

### Description & Impact

`Ed255192018Suite.PROOF_TYPE = "Ed25519Signature2018"` (line 66). LD-Proof credential signing is hardcoded to `Ed25519Signature2018`. It does not use the `SignatureProvider` SPI path and will not inherit ML-DSA support automatically. A new ML-DSA LD cryptographic suite would need to be implemented separately if LD-Proofs are required with PQC.

### Resolution Criteria

An ML-DSA LD suite appears in the codebase.

---

## GAP-8 — FAPI / Client Policy Allowlist — ML-DSA Actively Blocked {#gap-8}

| Field | Value |
|-------|-------|
| **Severity** | HIGH |
| **Domains Affected** | 16, 41 |
| **Affected File(s)** | `services/src/main/java/org/keycloak/services/clientpolicy/executor/FapiConstant.java` (line 30); `SecureSigningAlgorithmExecutor.java`; `SecureCibaAuthenticationRequestSigningAlgorithmExecutor.java` |
| **GitHub Issue** | *Spec-Gated (FAPI 2.0)* |

### Description & Impact

`FapiConstant.ALLOWED_ALGORITHMS` (line 30) is hardcoded to `{PS256, PS384, PS512, ES256, ES384, ES512}`. `SecureSigningAlgorithmExecutor.isSecureAlgorithm()` and the equivalent CIBA executor call `ALLOWED_ALGORITHMS.contains(sigAlg)` — ML-DSA will be *actively rejected* even after providers exist. When a FAPI client policy is applied, PQC algorithms are blocked at runtime and in the admin UI. Blocked on FAPI 2.0 / FAPI profiles including PQC algorithms.

### Resolution Criteria

Any ML-DSA algorithm name appears in `FapiConstant.ALLOWED_ALGORITHMS` (requires GAP-8 resolved at the spec level first).

---

## GAP-9 — SAML Protocol — Hardcoded RS256 Key Lookup {#gap-9}

| Field | Value |
|-------|-------|
| **Severity** | HIGH |
| **Domains Affected** | 18, 20, 59, 60, 61 |
| **Affected File(s)** | `services/src/main/java/org/keycloak/broker/saml/SAMLIdentityProvider.java` (lines 416, 507); `services/src/main/java/org/keycloak/protocol/saml/SamlProtocol.java` (line 544); `services/src/main/java/org/keycloak/protocol/saml/SamlService.java` (line 965) |
| **GitHub Issue** | [#50292](https://github.com/keycloak/keycloak/issues/50292) (create sub-issue) |

### Description & Impact

Four hardcoded `Algorithm.RS256` key-selection call sites in SAML code: SP metadata signing key export (line 416), SP metadata document signing (line 507), artifact resolve response signing (line 544), and IDP metadata descriptor redirect-binding export (line 965). Even if GAP-2 is fixed at the algorithm layer, ML-DSA support can never be reached because these call sites will never select an AKP key. Both layers (GAP-2 and GAP-9) must be fixed. Requires a third sub-issue under [#50292](https://github.com/keycloak/keycloak/issues/50292).

### Resolution Criteria

All four hardcoded `Algorithm.RS256` references are removed or made configurable.

---

## GAP-10 — IdP Broker — Hardcoded RS256 / HS256 Fallback {#gap-10}

| Field | Value |
|-------|-------|
| **Severity** | MEDIUM |
| **Domains Affected** | 21 |
| **Affected File(s)** | `services/src/main/java/org/keycloak/broker/oidc/AbstractOAuth2IdentityProvider.java` (lines 737, 745) |
| **GitHub Issue** | [#48821](https://github.com/keycloak/keycloak/issues/48821) (create sub-issue) |

### Description & Impact

Line 737 hardcodes `Algorithm.HS256` as the fallback when `client_secret_jwt` has no configured algorithm. Line 745 hardcodes `Algorithm.RS256` as the fallback for all other client assertion signing (e.g., `private_key_jwt`). An IdP broker configured for `private_key_jwt` will always fall back to RS256 unless the admin explicitly sets `clientAssertionSigningAlg`. No path to select ML-DSA as the outbound default even after providers exist.

### Resolution Criteria

The RS256 fallback at line 745 is removed or made configurable.

---

## GAP-11 — Dynamic Client Registration — RS256 Special-Case Logic {#gap-11}

| Field | Value |
|-------|-------|
| **Severity** | LOW |
| **Domains Affected** | 35 |
| **Affected File(s)** | `services/src/main/java/org/keycloak/services/clientregistration/oidc/DescriptionConverter.java` (line 416) |
| **GitHub Issue** | [#48821](https://github.com/keycloak/keycloak/issues/48821) or [#48823](https://github.com/keycloak/keycloak/issues/48823) |

### Description & Impact

`if (Algorithm.RS256.equals(defaultSignatureAlgorithm) || StringUtil.isBlank(defaultSignatureAlgorithm))` (line 416) — RS256 is explicitly special-cased so that `id_token_signed_response_alg` is omitted from the DCR response when the realm default is RS256. No equivalent treatment is defined for ML-DSA. Minor inconsistency; will need updating when ML-DSA becomes a valid realm default.

### Resolution Criteria

The RS256 special-case is removed or extended to handle ML-DSA.

---

## GAP-12 — Constants.DEFAULT_SIGNATURE_ALGORITHM — RS256 Fallback {#gap-12}

| Field | Value |
|-------|-------|
| **Severity** | MEDIUM |
| **Domains Affected** | 35 |
| **Affected File(s)** | `server-spi-private/src/main/java/org/keycloak/models/Constants.java` (line 68); `services/src/main/java/org/keycloak/jose/jws/DefaultTokenManager.java` (line 233) |
| **GitHub Issue** | [#48823](https://github.com/keycloak/keycloak/issues/48823) (document) |

### Description & Impact

`DEFAULT_SIGNATURE_ALGORITHM = Algorithm.RS256` (line 68) is the ultimate fallback for all token signing when neither client nor realm has configured an algorithm. `DefaultTokenManager` returns this constant at line 233. Any incomplete PQC migration (realm default updated but client attributes not set) will silently revert to classical RS256. Operators and documentation must explicitly flag this transition risk.

### Resolution Criteria

The fallback is removed or ML-DSA becomes the default; migration documentation covers this risk.

---

## GAP-13 — Admin CLI & Authz Client — Hardcoded RS256 {#gap-13}

| Field | Value |
|-------|-------|
| **Severity** | MEDIUM |
| **Domains Affected** | 38 |
| **Affected File(s)** | `integration/client-cli/admin-cli/src/main/java/org/keycloak/client/cli/util/AuthUtil.java` (line 213); `core/src/main/java/org/keycloak/jose/jws/JWSBuilder.java`; `core/src/main/java/org/keycloak/protocol/oidc/client/authentication/JWTClientCredentialsProvider.java` (lines 62, 96) |
| **GitHub Issue** | [#48821](https://github.com/keycloak/keycloak/issues/48821) or [#48823](https://github.com/keycloak/keycloak/issues/48823) (create sub-issue) |

### Description & Impact

**Admin CLI (`kcadm.sh` / `kcreg.sh`):** `getSignedRequestToken()` calls `.rsa256(keypair.getPrivate())` at line 213, hardcoding RS256. No `--sigalg` parameter exists in the CLI. If a realm requires ML-DSA for client assertions (e.g., via a FAPI policy), both CLI tools will be unable to authenticate via the keystore-based `private_key_jwt` flow.

**Authz Client (`keycloak-authz-client` library):** `JWTClientCredentialsProvider.setupKeyPair(KeyPair)` (line 62) hardcodes `Algorithm.RS256`. The `switch` on key type (lines 77–96) has no `AKP` case — passing an ML-DSA key pair throws `RuntimeException("Invalid KeyPair algorithm")`. This is the same gap as GAP-28 (Client SDK) manifested in the separate `keycloak-authz-client` library.

### Resolution Criteria

The `.rsa256()` call at line 213 is replaced with a configurable signing method; a `--sigalg` option is plumbed through the CLI.

---

## GAP-14 — JGroups ASYM_ENCRYPT — RSA Cluster Key Exchange {#gap-14}

| Field | Value |
|-------|-------|
| **Severity** | LOW |
| **Domains Affected** | 39 |
| **Affected File(s)** | `quarkus/tests/integration/src/test/resources/cache-ispn-asym-enc.xml` (test-only) |
| **GitHub Issue** | [#48823](https://github.com/keycloak/keycloak/issues/48823) (document) |

### Description & Impact

JGroups' `ASYM_ENCRYPT` protocol uses RSA-2048 for AES group-key distribution to joining cluster nodes — quantum-vulnerable under a Harvest Now, Decrypt Later attack. The file exists only in test resources; production Keycloak uses mTLS (`JGroupsCertificateProvider`) which will inherit quantum-safety from PQC-capable TLS ([#48820](https://github.com/keycloak/keycloak/issues/48820)). Risk is confined to operators who manually copy the example config.

### Resolution Criteria

The example config uses a PQC-safe key exchange or deployment documentation advises against using `ASYM_ENCRYPT`; no code change required.

---

## GAP-15 — Realm Key Management — No ML-DSA Key Generation {#gap-15}

| Field | Value |
|-------|-------|
| **Severity** | HIGH |
| **Domains Affected** | 1, 28, 29, 30, 31, 46 |
| **Affected File(s)** | `services/src/main/java/org/keycloak/keys/GeneratedRsaKeyProviderFactory.java`; `services/src/main/java/org/keycloak/keys/GeneratedEcdsaKeyProviderFactory.java`; `services/src/main/java/org/keycloak/keys/GeneratedEddsaKeyProviderFactory.java`; `services/src/main/java/org/keycloak/keys/DefaultKeyManager.java`; `core/src/main/java/org/keycloak/common/crypto/CryptoProvider.java` |
| **GitHub Issue** | [#44142](https://github.com/keycloak/keycloak/issues/44142) / [#48824](https://github.com/keycloak/keycloak/issues/48824) |

### Description & Impact

No `GeneratedAKPKeyProviderFactory` (or equivalent) exists for ML-DSA key generation (confirmed: no `GeneratedAKP*.java` or `GeneratedMLDsa*.java` found outside test paths). `DefaultKeyManager.createFallbackKeys()` does not know about ML-DSA. `CryptoProvider.getKeyPairGen(String algorithm)` would need to support ML-DSA algorithm names. This is the foundational gap — until ML-DSA key providers exist, no other ML-DSA signing work can be completed end-to-end.

### Resolution Criteria

At least one `GeneratedAKP*.java` or `GeneratedMLDsa*.java` file exists outside test paths.

---

## GAP-16 — JavaKeystoreKeyProviderFactory — ML-DSA Excluded from Admin UI {#gap-16}

| Field | Value |
|-------|-------|
| **Severity** | MEDIUM |
| **Domains Affected** | 33, 34 |
| **Affected File(s)** | `services/src/main/java/org/keycloak/keys/JavaKeystoreKeyProviderFactory.java` (lines 189–202) |
| **GitHub Issue** | [#50679](https://github.com/keycloak/keycloak/issues/50679) |

### Description & Impact

`mergedAlgorithmProperties()` (lines 189–202) builds the selectable algorithm list shown in the admin UI from RSA, EC, HS, and ECDH variants only — no ML-DSA names present. Even if GAP-5 (import logic) is fixed, the admin cannot select an ML-DSA algorithm for a keystore-based key provider in the UI. Both the provider logic (GAP-5) and the configuration surface (GAP-16) must be fixed together.

### Resolution Criteria

ML-DSA names appear in `mergedAlgorithmProperties()`.

---

## GAP-17 — FAPI / CIBA Executor Factories — Admin UI Algorithm Lists {#gap-17}

| Field | Value |
|-------|-------|
| **Severity** | MEDIUM |
| **Domains Affected** | 16, 41 |
| **Affected File(s)** | `services/src/main/java/org/keycloak/services/clientpolicy/executor/SecureSigningAlgorithmExecutorFactory.java` (line 42); `services/src/main/java/org/keycloak/protocol/oidc/grants/ciba/clientpolicy/executor/SecureCibaAuthenticationRequestSigningAlgorithmExecutorFactory.java` (line 45) |
| **GitHub Issue** | *Depends on GAP-8* |

### Description & Impact

Both executor factories build their selectable option lists directly from `FapiConstant.ALLOWED_ALGORITHMS`. ML-DSA is blocked both at runtime (GAP-8) and in the admin configuration UI. The UI-level block means an administrator cannot even configure a FAPI realm to accept ML-DSA before the algorithm constant is updated. Depends on GAP-8 being resolved first.

### Resolution Criteria

ML-DSA appears in the executor factory option lists (requires GAP-8 resolved first).

---

## GAP-18 — JwtCNonceHandler — c_nonce Signing Hardcoded ES256/RS256 {#gap-18}

| Field | Value |
|-------|-------|
| **Severity** | MEDIUM |
| **Domains Affected** | 27 |
| **Affected File(s)** | `services/src/main/java/org/keycloak/protocol/oid4vc/issuance/keybinding/JwtCNonceHandler.java` (lines 281, 287) |
| **GitHub Issue** | *None — needs new OID4VC issue* |

### Description & Impact

`selectSigningKey()` first attempts `Algorithm.ES256` (line 281), then falls back to `Algorithm.RS256` (line 287). This is independent of `Constants.DEFAULT_SIGNATURE_ALGORITHM` and completely bypasses the realm default algorithm. c_nonce JWTs will not use an active ML-DSA key even after ML-DSA providers and realm keys exist.

### Resolution Criteria

The hardcoded lookups are replaced with a configurable or SPI-driven selection.

---

## GAP-19 — No Minimum-Version Constraint on bcprov-jdk18on {#gap-19}

| Field | Value |
|-------|-------|
| **Severity** | MEDIUM |
| **Domains Affected** | 42 |
| **Affected File(s)** | `pom.xml` (root); `crypto/default/pom.xml` (version inherited from Quarkus BOM) |
| **GitHub Issue** | [#46333](https://github.com/keycloak/keycloak/issues/46333) or [#46336](https://github.com/keycloak/keycloak/issues/46336) (create sub-issue) |

### Description & Impact

`bcprov-jdk18on` is inherited via the Quarkus BOM with no explicit version or Maven Enforcer minimum-version rule in Keycloak's own `pom.xml`. ML-DSA classes (`org.bouncycastle.pqc.crypto.mldsa.*`) first appeared in version 1.78. A downstream build with an older BOM version could silently drop ML-DSA support without a build-time failure.

### Resolution Criteria

An Enforcer rule pinning `bcprov-jdk18on ≥ 1.78` is present in `pom.xml`.

---

## GAP-20 — JAR Encrypted Request Object — No ML-KEM Inbound Path {#gap-20}

| Field | Value |
|-------|-------|
| **Severity** | MEDIUM |
| **Domains Affected** | 8 |
| **Affected File(s)** | `services/src/main/java/org/keycloak/jose/jws/DefaultTokenManager.java` (line 123); `services/src/main/java/org/keycloak/protocol/oidc/endpoints/request/AuthzEndpointRequestObjectParser.java`; `services/src/main/java/org/keycloak/protocol/oidc/par/endpoints/request/ParEndpointRequestObjectParser.java` |
| **GitHub Issue** | [#48821](https://github.com/keycloak/keycloak/issues/48821) (create sub-issue) |

### Description & Impact

`decodeClientJWT()` (line 123) selects a realm private key by `KeyUse.ENC` (lines 139, 144) and passes it directly to the JWE engine. No ML-KEM `CekManagementProviderFactory` exists and the realm cannot hold an ML-KEM ENC key. A client wishing to send an encrypted JAR using ML-KEM key encapsulation cannot do so. Distinct from GAP-4 (outbound) — this is *inbound* decryption.

### Resolution Criteria

An ML-KEM decapsulation path is present in `decodeClientJWT()` (contingent on GAP-4).

---

## GAP-21 — UserInfo Endpoint — Per-Client Signing & Encryption Gap {#gap-21}

| Field | Value |
|-------|-------|
| **Severity** | MEDIUM |
| **Domains Affected** | 4 |
| **Affected File(s)** | `services/src/main/java/org/keycloak/protocol/oidc/endpoints/UserInfoEndpoint.java` (lines 341, 379–380) |
| **GitHub Issue** | [#48821](https://github.com/keycloak/keycloak/issues/48821) (create sub-issue); document in [#48823](https://github.com/keycloak/keycloak/issues/48823) |

### Description & Impact

Signing: `signatureAlgorithm(TokenCategory.USERINFO)` (line 341) resolves only the per-client `userinfo.response.signature.alg` attribute, then falls back to the realm default (`RS256`). Unlike ID tokens, there is no automatic inheritance — each client's explicit attribute must be updated independently during migration. Encryption: `jweFromContent()` uses `CekManagementProvider` SPI (lines 379–380, no ML-KEM provider, same as GAP-4) but in a separate class and code path. UserInfo encryption is a confirmed hard gap requiring explicit testing once GAP-4 is resolved.

### Resolution Criteria

The per-client attribute inherits from the realm default; ML-KEM encryption path verified once GAP-4 is resolved.

---

## GAP-22 — Attestation-Based Client Auth — Algorithm Enforcement {#gap-22}

| Field | Value |
|-------|-------|
| **Severity** | MEDIUM |
| **Domains Affected** | 12 |
| **Affected File(s)** | `services/src/main/java/org/keycloak/authentication/authenticators/client/AttestationBasedClientAuthenticator.java` (lines 418–419) |
| **GitHub Issue** | [#43684](https://github.com/keycloak/keycloak/issues/43684) or combine with GAP-30 |

### Description & Impact

Two `[TODO]` comments at lines 418–419 note that algorithm-type enforcement (must be a registered asymmetric algorithm) is not yet implemented — a security hardening gap independent of PQC. Additionally, if the `cnf.jwk` claim contains an ML-DSA (AKP) key, `JWKSUtils.computeThumbprint()` will throw `UnsupportedOperationException` (dependent on GAP-1).

### Resolution Criteria

The `[TODO]` at line 418 is removed and algorithm enforcement is implemented.

---

## GAP-23 — OIDC Discovery — Hardcoded RS256 CIBA Constant {#gap-23}

| Field | Value |
|-------|-------|
| **Severity** | LOW |
| **Domains Affected** | 36 |
| **Affected File(s)** | `services/src/main/java/org/keycloak/protocol/oidc/OIDCWellKnownProvider.java` (line 85) |
| **GitHub Issue** | [#48821](https://github.com/keycloak/keycloak/issues/48821) (create sub-issue) |

### Description & Impact

`DEFAULT_CLIENT_AUTH_SIGNING_ALG_VALUES_SUPPORTED = list(Algorithm.RS256.toString())` (line 85) is a static constant used to advertise supported CIBA backchannel authentication request signing algorithms. Unlike other discovery fields (which are dynamically populated from registered `SignatureProvider` instances), this one remains hardcoded to RS256 even after ML-DSA providers are registered. A conformant CIBA client relying on this field will never attempt ML-DSA-signed requests.

### Resolution Criteria

The static constant is replaced with a dynamic lookup merging registered asymmetric `SignatureProvider` algorithm names.

---

## GAP-24 — OID4VP — Hardcoded ES256 Signing / ACCEPTED_ALGORITHMS {#gap-24}

| Field | Value |
|-------|-------|
| **Severity** | HIGH |
| **Domains Affected** | 62 |
| **Affected File(s)** | `services/src/main/java/org/keycloak/broker/oid4vp/OID4VPIdentityProvider.java` (lines 83, 196–197) |
| **GitHub Issue** | *None — needs new issue under [#45168](https://github.com/keycloak/keycloak/issues/45168)* |

### Description & Impact

`ACCEPTED_ALGORITHMS = List.of(Algorithm.ES256)` (line 83) is used by `OID4VPIdentityProviderEndpoint.requireAcceptedAlgorithm()` to actively reject all non-ES256 VP token signatures at runtime, including ML-DSA. `signingKey()` (lines 196–197) hardcodes ES256 for both active-key lookup (`getKeyIncludingDisabled`) and kid-based lookup (`getActiveKey`), preventing ML-DSA key selection. Even after ML-DSA providers exist, OID4VP will silently fail at the algorithm enforcement layer.

### Resolution Criteria

`ACCEPTED_ALGORITHMS` is configurable or includes ML-DSA; the hardcoded ES256 lookups at lines 196–197 are replaced with configurable/SPI-driven selection.

---

## GAP-25 — OID4VP — ECDH-ES/secp256r1 Response Encryption (HAIP spec-gated) {#gap-25}

| Field | Value |
|-------|-------|
| **Severity** | LOW (EXTERNAL) |
| **Domains Affected** | 62 |
| **Affected File(s)** | `services/src/main/java/org/keycloak/broker/oid4vp/EphemeralKey.java` (line 41); `services/src/main/java/org/keycloak/broker/oid4vp/ResponseEncryption.java` (line 38) |
| **GitHub Issue** | *External — track HAIP spec evolution* |

### Description & Impact

`EphemeralKey.generate()` hardcodes `CURVE_SEC = "secp256r1"` (line 41); `ResponseEncryption.KEY_MANAGEMENT_ALG = ECDH_ES` (line 38). Both are spec-gated: the HAIP (High Assurance Interoperability Profile) currently mandates ECDH-ES/P-256 for `direct_post.jwt` response encryption. No Keycloak-side fix is appropriate until HAIP specifies ML-KEM support. No new GitHub issue required at this time — existing HAIP tracker is the dependency.

### Resolution Criteria

HAIP spec updated to specify ML-KEM; implementation follows accordingly.

---

## GAP-26 — JWKS Endpoint — AKP Branch in JWKSServerUtils.toJwk() {#gap-26}

| Field | Value |
|-------|-------|
| **Severity** | HIGH |
| **Domains Affected** | 48 |
| **Affected File(s)** | `services/src/main/java/org/keycloak/protocol/oidc/utils/JWKSServerUtils.java` (lines 59–65) |
| **GitHub Issue** | Check [#44141](https://github.com/keycloak/keycloak/issues/44141) (CLOSED) or [#43692](https://github.com/keycloak/keycloak/issues/43692) |

### Description & Impact

`JWKSServerUtils.toJwk()` has explicit `if/else if` branches for `KeyType.RSA` (line 59), `KeyType.EC` (line 61), and `KeyType.OKP` (line 63) only. An AKP (ML-DSA) key falls through all branches and returns `null`, meaning ML-DSA realm keys will be silently omitted from the public JWKS endpoint (`/protocol/openid-connect/certs`). Clients will never discover the realm's ML-DSA keys. `JWKBuilder.akp()` already exists — this is a straightforward one-branch addition.

### Resolution Criteria

An `AKP` branch appears in `toJwk()` calling `JWKBuilder.akp()`.

---

## GAP-27 — Admin API — Client Keypair Generation Hardcodes RSA {#gap-27}

| Field | Value |
|-------|-------|
| **Severity** | MEDIUM |
| **Domains Affected** | 49 |
| **Affected File(s)** | `services/src/main/java/org/keycloak/services/resources/admin/ClientAttributeCertificateResource.java` (lines 119, 258) |
| **GitHub Issue** | [#48821](https://github.com/keycloak/keycloak/issues/48821) (create sub-issue) |

### Description & Impact

Both `generate()` (line 119) and `generateAndGetKeystore()` (line 258) call `KeycloakModelUtils.generateKeyPairCertificate()`, which hardcodes RSA key generation (`KeyUtils.generateRsaKeyPair(keysize)`). No algorithm selection is exposed in the API or UI. Admin-generated client keypairs will always be RSA regardless of realm PQC configuration.

### Resolution Criteria

An algorithm parameter is accepted by `generateAndGetKeystore` and ML-DSA key generation is possible.

---

## GAP-28 — Client SDK — JWTClientCredentialsProvider Missing AKP {#gap-28}

| Field | Value |
|-------|-------|
| **Severity** | MEDIUM |
| **Domains Affected** | 50 |
| **Affected File(s)** | `core/src/main/java/org/keycloak/protocol/oidc/client/authentication/JWTClientCredentialsProvider.java` (lines 77–96) |
| **GitHub Issue** | [#48821](https://github.com/keycloak/keycloak/issues/48821) (create sub-issue; could combine with GAP-29) |

### Description & Impact

`setupKeyPair()` defaults to `Algorithm.RS256`. The `switch` on key type (lines 77–96) handles `KeyType.RSA` (line 77), `KeyType.EC` (line 83), and `KeyType.OKP` (line 89) only — no `AKP` case. Passing an ML-DSA key pair throws `RuntimeException("Invalid KeyPair algorithm")` (line 96). Distinct from the server-side verifier (GAP-30) and the admin CLI path (GAP-13).

### Resolution Criteria

An `AKP` case is present in the switch block.

---

## GAP-29 — Client SDK — DPoPGenerator No ML-DSA Convenience Path {#gap-29}

| Field | Value |
|-------|-------|
| **Severity** | MEDIUM |
| **Domains Affected** | 51 |
| **Affected File(s)** | `core/src/main/java/org/keycloak/util/DPoPGenerator.java` (lines 49–50) |
| **GitHub Issue** | [#48821](https://github.com/keycloak/keycloak/issues/48821) (create sub-issue; could combine with GAP-28) |

### Description & Impact

The generic `generateSignedDPoPProof(…, KeyWrapper, …)` method is algorithm-agnostic and will support ML-DSA once providers exist. However, `generateRsaSignedDPoPProof()` (line 50) is hardcoded to RSA and includes an explicit `TODO` at line 49 noting EC and EdDSA equivalents are missing. No ML-DSA convenience path exists. Distinct from server-side DPoP verification (domain 15).

### Resolution Criteria

An ML-DSA convenience method exists or the TODO is addressed for all key types.

---

## GAP-30 — Client Signature Verifier — RSA Guard Blocks AKP Keys {#gap-30}

| Field | Value |
|-------|-------|
| **Severity** | HIGH |
| **Domains Affected** | 53 |
| **Affected File(s)** | `services/src/main/java/org/keycloak/crypto/ClientAsymmetricSignatureVerifierContext.java` (lines 36–37); `services/src/main/java/org/keycloak/crypto/AsymmetricClientSignatureVerifierProvider.java`; `services/src/main/java/org/keycloak/crypto/ClientECDSASignatureVerifierContext.java`; `services/src/main/java/org/keycloak/crypto/ClientEdDSASignatureVerifierContext.java` |
| **GitHub Issue** | [#43684](https://github.com/keycloak/keycloak/issues/43684) |

### Description & Impact

`ClientAsymmetricSignatureVerifierContext.getKey()` at line 36 explicitly throws `VerificationException("Key Type is not RSA: " + key.getType())` at line 37 for any non-RSA key — ECDSA, EdDSA, and ML-DSA (AKP) client keys are all unconditionally rejected through the RSA path. The ECDSA and EdDSA paths route through their own contexts and are SPI-driven — they will support ML-DSA automatically once providers exist. ML-DSA (AKP) client assertions currently have no routed path at all. Covered by [#43684](https://github.com/keycloak/keycloak/issues/43684).

### Resolution Criteria

The `KeyType.RSA` guard is removed or an AKP path is added.

---

## Gap Summary by Severity

### HIGH Priority (11 gaps)
- **GAP-1** — JWK Thumbprint — AKP (ML-DSA) Keys
- **GAP-2** — SAML Signing — No PQC Algorithm Entries
- **GAP-3** — SAML Encryption — No ML-KEM Key Wrapping
- **GAP-4** — JWE Key Management — No ML-KEM CEK Provider
- **GAP-6** — FIPS 140-2 Backend — ML-DSA Availability Blocked
- **GAP-8** — FAPI / Client Policy Allowlist — ML-DSA Actively Blocked
- **GAP-9** — SAML Protocol — Hardcoded RS256 Key Lookup
- **GAP-15** — Realm Key Management — No ML-DSA Key Generation
- **GAP-24** — OID4VP — Hardcoded ES256 Signing / ACCEPTED_ALGORITHMS
- **GAP-26** — JWKS Endpoint — AKP Branch in JWKSServerUtils.toJwk()
- **GAP-30** — Client Signature Verifier — RSA Guard Blocks AKP Keys

### MEDIUM Priority (15 gaps)
- **GAP-5** — JavaKeystoreKeyProvider — ML-DSA Key Import
- **GAP-7** — OID4VC Linked Data Proof Suite — Hardcoded Ed25519
- **GAP-10** — IdP Broker — Hardcoded RS256 / HS256 Fallback
- **GAP-12** — Constants.DEFAULT_SIGNATURE_ALGORITHM — RS256 Fallback
- **GAP-13** — Admin CLI & Authz Client — Hardcoded RS256
- **GAP-16** — JavaKeystoreKeyProviderFactory — ML-DSA Excluded from Admin UI
- **GAP-17** — FAPI / CIBA Executor Factories — Admin UI Algorithm Lists
- **GAP-18** — JwtCNonceHandler — c_nonce Signing Hardcoded ES256/RS256
- **GAP-19** — No Minimum-Version Constraint on bcprov-jdk18on
- **GAP-20** — JAR Encrypted Request Object — No ML-KEM Inbound Path
- **GAP-21** — UserInfo Endpoint — Per-Client Signing & Encryption Gap
- **GAP-22** — Attestation-Based Client Auth — Algorithm Enforcement
- **GAP-27** — Admin API — Client Keypair Generation Hardcodes RSA
- **GAP-28** — Client SDK — JWTClientCredentialsProvider Missing AKP
- **GAP-29** — Client SDK — DPoPGenerator No ML-DSA Convenience Path

### LOW Priority (4 gaps)
- **GAP-11** — Dynamic Client Registration — RS256 Special-Case Logic
- **GAP-14** — JGroups ASYM_ENCRYPT — RSA Cluster Key Exchange
- **GAP-23** — OIDC Discovery — Hardcoded RS256 CIBA Constant
- **GAP-25** — OID4VP — ECDH-ES/secp256r1 Response Encryption (HAIP spec-gated)

**Verified counts:** HIGH=11, MEDIUM=15, LOW=4. Total = 30.

---

## Gaps by GitHub Issue Status

### With Existing GitHub Issues
- **GAP-2** → [#50294](https://github.com/keycloak/keycloak/issues/50294) (SAML signing URIs)
- **GAP-3** → [#50295](https://github.com/keycloak/keycloak/issues/50295) (SAML encryption)
- **GAP-5** → [#50679](https://github.com/keycloak/keycloak/issues/50679) (Keystore import)
- **GAP-15** → [#44142](https://github.com/keycloak/keycloak/issues/44142) / [#48824](https://github.com/keycloak/keycloak/issues/48824) (ML-DSA key generation)
- **GAP-16** → [#50679](https://github.com/keycloak/keycloak/issues/50679) (same as GAP-5)
- **GAP-30** → [#43684](https://github.com/keycloak/keycloak/issues/43684) (ClientSignatureVerifierProviderFactory)

### Need Verification
- **GAP-1** → Check if [#44141](https://github.com/keycloak/keycloak/issues/44141) (CLOSED) covered JWK thumbprint; if not, create sub-issue under [#43692](https://github.com/keycloak/keycloak/issues/43692)
- **GAP-26** → Check if [#44141](https://github.com/keycloak/keycloak/issues/44141) (CLOSED) covered JWKS endpoint AKP branch; if not, create sub-issue under [#43692](https://github.com/keycloak/keycloak/issues/43692)

### Need New Issues — Under [#48821](https://github.com/keycloak/keycloak/issues/48821) OAuth/OIDC
- **GAP-4** — ML-KEM CEK Provider (HIGH)
- **GAP-10** — IdP Broker Fallback (MEDIUM)
- **GAP-20** — JAR Encrypted Request Decryption (MEDIUM)
- **GAP-21** — UserInfo Per-Client Signing (MEDIUM) *also document in #48823*
- **GAP-23** — OIDC Discovery Constant (LOW)
- **GAP-27** — Admin API Client Keypair (MEDIUM)
- **GAP-28** — Client SDK JWT Credentials (MEDIUM) *could combine with GAP-29*
- **GAP-29** — Client SDK DPoP (MEDIUM) *could combine with GAP-28*

### Need New Issues — Under [#50292](https://github.com/keycloak/keycloak/issues/50292) SAML
- **GAP-9** — SAML Hardcoded RS256 Key Selection (HIGH)

### Need New Issues — Under [#46333](https://github.com/keycloak/keycloak/issues/46333) Crypto Defaults
- **GAP-19** — bcprov Minimum Version (MEDIUM)

### Need New Issues — Standalone or Under [#43684](https://github.com/keycloak/keycloak/issues/43684)
- **GAP-22** — Attestation Algorithm Enforcement (MEDIUM) *could combine with GAP-30*

### Need New Issues — OID4VC
- **GAP-18** — OID4VC c_nonce Hardcoded Algorithm (MEDIUM)
- **GAP-24** — OID4VP Hardcoded ES256 / ACCEPTED_ALGORITHMS (HIGH)

### Need New Issues — Documentation Only
- **GAP-11** — DCR RS256 Special-Case (LOW) *could document in #48823*
- **GAP-13** — Admin CLI (MEDIUM) *or under #48823*

### External Dependencies / Documentation Only
- **GAP-6** — BC-FIPS external dependency (document only; blocked on BC-FIPS release)
- **GAP-7** — LD-Proof spec dependency (document only)
- **GAP-8** — FAPI 2.0 spec dependency (document in #48823)
- **GAP-12** — Documentation/operator guidance (document in #48823)
- **GAP-14** — Operator guidance only (document in #48823)
- **GAP-17** — Resolves when GAP-8 is addressed (no separate issue needed)
- **GAP-25** — HAIP spec dependency (track HAIP evolution)

---

## Domain Coverage

| Domain # | Gap ID(s) | Notes |
|----------|-----------|-------|
| 1 | GAP-15 | ML-DSA key generation |
| 2 | GAP-4 | ML-KEM CEK provider |
| 4 | GAP-4, GAP-21 | ML-KEM CEK; UserInfo per-client signing |
| 8 | GAP-4, GAP-20 | ML-KEM CEK; JAR encrypted request decryption |
| 11 | GAP-6 | BC-FIPS external |
| 12 | GAP-1, GAP-22 | JWK thumbprint; attestation algorithm enforcement |
| 13 | GAP-6 | BC-FIPS external |
| 15 | GAP-1 | JWK thumbprint (DPoP dpop_jkt) |
| 16 | GAP-8, GAP-17 | FAPI allowlist; executor factory UI |
| 18 | GAP-2, GAP-9 | SAML signing URIs; SAML hardcoded RS256 |
| 19 | GAP-3 | SAML encryption ML-KEM |
| 20 | GAP-2, GAP-9 | SAML signing URIs; SAML hardcoded RS256 |
| 21 | GAP-10 | IdP broker fallback |
| 25 | GAP-7 | LD-Proof suite |
| 27 | GAP-18 | c_nonce signing hardcoded |
| 28 | GAP-15 | ML-DSA key generation |
| 29 | GAP-15 | ML-DSA key generation |
| 30 | GAP-15 | ML-DSA key generation |
| 31 | GAP-15 | ML-DSA key generation |
| 33 | GAP-5, GAP-16 | Keystore import; admin UI |
| 34 | GAP-5, GAP-16 | Keystore import; admin UI |
| 35 | GAP-11, GAP-12 | DCR RS256 special-case; default signature algorithm |
| 36 | GAP-23 | OIDC discovery hardcoded CIBA constant |
| 38 | GAP-13 | Admin CLI hardcoded RS256 |
| 39 | GAP-14 | JGroups ASYM_ENCRYPT |
| 41 | GAP-8, GAP-17 | FAPI allowlist; executor factory UI |
| 42 | GAP-19 | bcprov minimum version |
| 43 | GAP-6 | BC-FIPS external |
| 45 | GAP-1 | JWK thumbprint (attestation) |
| 46 | GAP-15 | ML-DSA key generation |
| 48 | GAP-26 | JWKS endpoint AKP branch |
| 49 | GAP-27 | Admin API client keypair generation |
| 50 | GAP-28 | Client SDK JWT credentials provider |
| 51 | GAP-29 | Client SDK DPoP generator |
| 53 | GAP-30 | Client signature verifier RSA guard |
| 59 | GAP-2, GAP-9 | SAML signing URIs; SAML hardcoded RS256 |
| 60 | GAP-2, GAP-9 | SAML signing URIs; SAML hardcoded RS256 |
| 61 | GAP-2, GAP-9 | SAML signing URIs; SAML hardcoded RS256 |
| 62 | GAP-24, GAP-25 | OID4VP ES256 hardcoded; OID4VP HAIP encryption |
| 63 | GAP-4 | ML-KEM CEK provider |

---

**Total Gaps:** 30  
**With Existing GitHub Issues:** 6  
**Need Verification:** 2  
**Need New Issues:** 16  
**External/Documentation:** 7
