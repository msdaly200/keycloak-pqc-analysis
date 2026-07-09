# Keycloak Asymmetric Cryptography Summary

**Issue:** [#46336 - Cryptographic Inventory for Keycloak](https://github.com/keycloak/keycloak/issues/46336)  
**Created:** 2026-07-09  
**Scope:** All asymmetric cryptography domains from PQC readiness analysis

---

## Legend

### Type Categories
- **SIGN** - Digital signature operations (signing/verification)
- **ENC** - Encryption/decryption operations (asymmetric key encapsulation)
- **SIGN+ENC** - Both signing and encryption
- **CERT** - Certificate validation/generation (X.509)
- **SIGN+CERT** - Signing plus certificate operations
- **N/A** - Not applicable / no asymmetric crypto

### Strict/Configurable
- **Strict** - Hardcoded algorithm selection; no user configuration
- **Configurable** - Algorithm selection can be configured by users (realm settings, client attributes, policy settings)
- **SPI-driven** - Algorithm-agnostic; automatically supports new algorithms via SPI providers
- **Hybrid** - Partially configurable with some hardcoded defaults or constraints

---

## Asymmetric Algorithm Use Summary Table

| Category | Domain | Type | Asymmetric Algorithm(s) in Use | Strict/Configurable |
|----------|--------|------|-------------------------------|---------------------|
| **Communication** | JGroups ASYM_ENCRYPT (non-default / test config) | ENC | RSA-2048 (test config) or mTLS | Hybrid (mTLS in prod, RSA test config) |
| **Infrastructure** | CryptoProvider SPI — BouncyCastle Default Backend | SIGN+CERT | All asymmetric algorithms | SPI-driven (BC library support) |
| **Infrastructure** | CryptoProvider SPI — FIPS 140-2/3 Backend (BC-FIPS) | SIGN+CERT | RSA, ECDSA, EdDSA (no ML-DSA/ML-KEM) | SPI-driven (limited by BC-FIPS) |
| **Infrastructure** | CryptoProvider SPI — WildFly Elytron Backend | SIGN+CERT | Limited support | SPI-driven (external dependency) |
| **Infrastructure** | kcadm.sh / kcreg.sh — private_key_jwt Auth | SIGN | RS256 ONLY | Strict (hardcoded RS256) |
| **Infrastructure** | Admin API — Client Certificate & Keypair Generation | CERT | RSA 2048 | Strict (hardcoded RSA-2048) |
| **Infrastructure** | Client SDK — JWT Client Credentials Provider | SIGN | RS256 (default), RS*, PS*, ES*, EdDSA | Configurable (RS256 default) |
| **Infrastructure** | Client SDK — DPoP Proof Generation | SIGN | RS*, PS*, ES*, EdDSA | Configurable |
| **Infrastructure** | Docker Registry — Self-Signed Certificate Generation | CERT | RSA-2048 (test/dev) | Strict (hardcoded RSA for test path) |
| **Key Management** | Generated RSA Signing Key Provider | SIGN | RSA 2048/3072/4096 | Configurable (per-provider key size) |
| **Key Management** | Generated RSA Encryption Key Provider | ENC | RSA 2048/3072/4096 (RSA-OAEP) | Configurable (per-provider key size) |
| **Key Management** | Generated ECDSA Signing Key Provider | SIGN | ECDSA P-256/P-384/P-521 | Configurable (per-provider curve) |
| **Key Management** | Generated EdDSA Signing Key Provider | SIGN | EdDSA (Ed25519) | Strict (Ed25519 only) |
| **Key Management** | Generated ECDH Encryption Key Provider | ENC | ECDH P-256/P-384/P-521 (ECDH-ES variants) | Configurable (per-provider curve) |
| **Key Management** | Imported RSA Signing Key Provider | SIGN | RSA (RS*, PS*) | Configurable (import-based) |
| **Key Management** | Java Keystore Key Provider (PKCS12 / BCFKS) | SIGN+ENC | RSA, ECDSA, EdDSA, ~~ML-DSA~~ | Configurable (keystore-based) |
| **Key Management** | JWK Serialisation & Thumbprint | SIGN | RSA, EC, OKP, ~~AKP thumbprint missing~~ | SPI-driven (thumbprint blocked for AKP) |
| **Key Management** | Default Realm Key Providers (Bootstrap) | SIGN+ENC | RSA-2048 SIG, RSA-2048 ENC ONLY | Strict (hardcoded RSA bootstrap) |
| **Key Management** | Client Public Key Loader (JWKS URL & Stored Cert) | SIGN | RSA, ECDSA, EdDSA | SPI-driven |
| **Protocol - CAEP** | Security Event Token (SET) Signing | SIGN | RS*, PS*, ES*, EdDSA | SPI-driven (spec-gated on CAEP) |
| **Protocol - IdP Broker** | OIDC IdP — Token Signature Verification & JWE Decryption | SIGN+ENC | RS*, PS*, ES*, EdDSA (SIGN); RSA-OAEP, ECDH-ES (ENC) | Hybrid (hardcoded RS256 fallback for private_key_jwt) |
| **Protocol - IdP Broker** | Kubernetes Identity Provider | SIGN | RS*, PS*, ES*, EdDSA | SPI-driven |
| **Protocol - IdP Broker** | SPIFFE / SVID Identity Provider | SIGN | RS*, PS*, ES*, EdDSA | SPI-driven |
| **Protocol - IdP Broker** | Default Trust Identity Provider (Trust Broker) | SIGN | RS*, PS*, ES*, EdDSA | SPI-driven |
| **Protocol - OIDC** | Access Token / ID Token Signing | SIGN | RS256/384/512, PS256/384/512, ES256/384/512, EdDSA | Configurable (realm default signature algorithm) |
| **Protocol - OIDC** | ID Token / JARM Encryption (Outbound CEK) | ENC | RSA-OAEP, RSA-OAEP-256, RSA1_5, ECDH-ES variants | Configurable (per-client encryption attributes) |
| **Protocol - OIDC** | Backchannel Logout Token Signing | SIGN | RS*, PS*, ES*, EdDSA | Configurable (inherits realm default) |
| **Protocol - OIDC** | UserInfo Endpoint — Signed & Encrypted Response | SIGN+ENC | Signing: RS*, PS*, ES*, EdDSA<br/>Encryption: RSA-OAEP, ECDH-ES | Hybrid (per-client attribute, does NOT inherit realm default) |
| **Protocol - OIDC** | Introspection — Embedded JWT Response | SIGN | RS*, PS*, ES*, EdDSA | Configurable (inherits realm default) |
| **Protocol - OIDC** | Token Verification (Identity & Session Tokens) | SIGN | RS*, PS*, ES*, EdDSA | SPI-driven (automatic support for new algorithms) |
| **Protocol - OIDC** | JAR — Signed Request Object Verification (Inbound) | SIGN | RS*, PS*, ES*, EdDSA | SPI-driven (client-specified) |
| **Protocol - OIDC** | JAR — Encrypted Request Object Decryption (Inbound) | ENC | RSA-OAEP, RSA-OAEP-256, RSA1_5, ECDH-ES variants | SPI-driven (based on realm ENC key) |
| **Protocol - OIDC** | JARM — Signed Authorization Response | SIGN | RS*, PS*, ES*, EdDSA | Configurable (inherits realm default) |
| **Protocol - OIDC** | private_key_jwt Client Authentication | SIGN | RS*, PS*, ES*, EdDSA | SPI-driven (client-specified from JWKS or cert) |
| **Protocol - OIDC** | X.509 mTLS Client Authentication | CERT | X.509 cert chain validation | Configurable (CA cert validation, policy settings) |
| **Protocol - OIDC** | Attestation-Based Client Authentication | SIGN | RS*, PS*, ES*, EdDSA (dual verification: attester + PoP) | SPI-driven (client-specified) |
| **Protocol - OIDC** | X.509 Browser Authentication Flow | CERT | X.509 cert chain validation | Configurable (CA cert validation) |
| **Protocol - OIDC** | WebAuthn / Passkeys (FIDO2) | SIGN | RS256/384/512, ES256/384/512, EdDSA (COSE) | Configurable (WebAuthn policy settings) |
| **Protocol - OIDC** | DPoP (Demonstrating Proof of Possession) | SIGN | RS*, PS*, ES*, EdDSA | SPI-driven (client-specified in DPoP JWT) |
| **Protocol - OIDC** | CIBA Signed Backchannel Auth Request | SIGN | RS*, PS*, ES*, EdDSA | SPI-driven (client-specified) |
| **Protocol - OIDC** | JWT Authorization Grant — Assertion Verification | SIGN | RS*, PS*, ES*, EdDSA | SPI-driven (client-specified) |
| **Protocol - OIDC** | Dynamic Client Registration Tokens | SIGN | HS512 (symmetric) or RS* | Hybrid (symmetric internal, RSA fallback) |
| **Protocol - OIDC** | OIDC Well-Known Discovery — Algorithm Advertisement | SIGN | Advertises supported algorithms | SPI-driven (auto-populated from providers) |
| **Protocol - OIDC** | FAPI Algorithm Allowlist Enforcement | SIGN | PS256/384/512, ES256/384/512 ONLY (allowlist) | Strict (FAPI allowlist, blocks ML-DSA) |
| **Protocol - OIDC** | Realm JWKS Endpoint (/certs) | SIGN | RSA, EC, OKP (AKP returns null) | SPI-driven (AKP blocked) |
| **Protocol - OIDC** | Client Asymmetric Signature Verifier Context & Providers | SIGN | RSA, ECDSA, EdDSA (RSA-only guard blocks AKP) | SPI-driven (blocked by RSA guard) |
| **Protocol - OIDC** | Federated JWT Client Authentication | SIGN | RS*, PS*, ES*, EdDSA | SPI-driven |
| **Protocol - OIDC** | PAR (Pushed Authorization Request) — JAR Verification | SIGN | RS*, PS*, ES*, EdDSA | SPI-driven |
| **Protocol - OIDC** | Token Exchange — Subject/Actor Token Verification & Exchange | SIGN | RS*, PS*, ES*, EdDSA | SPI-driven |
| **Protocol - OIDC** | Device Authorization Grant — Token Signing | SIGN | RS*, PS*, ES*, EdDSA | Configurable (inherits realm default) |
| **Protocol - OID4VC** | JWT-VC / SD-JWT Credential Signing | SIGN | RS*, PS*, ES*, EdDSA | SPI-driven |
| **Protocol - OID4VC** | LD-Proof Credential Signing (Linked Data) | SIGN | Ed25519 ONLY | Strict (hardcoded EdDSA) |
| **Protocol - OID4VC** | OID4VC Key Binding — JWT Proof Validation | SIGN | RS*, PS*, ES*, EdDSA | SPI-driven |
| **Protocol - OID4VC** | OID4VC c_nonce JWT Signing | SIGN | ES256, RS256 | Strict (hardcoded ES256/RS256 selection) |
| **Protocol - OID4VC** | SD-JWT Issuer Signing & Key Binding Verification | SIGN | RS*, PS*, ES*, EdDSA | SPI-driven |
| **Protocol - Organizations** | Organisation Invitation Token Verification | SIGN | RS*, PS*, ES*, EdDSA | Configurable (inherits realm default) |
| **Protocol - SAML** | SAML Assertion & Document Signing | SIGN | RSA_SHA1, RSA_SHA256/384/512, RSA-PSS, DSA_SHA1 | Configurable (realm + client settings) with **4 hardcoded RS256 sites** |
| **Protocol - SAML** | SAML Assertion Encryption | ENC | RSA-OAEP, RSA1_5 (key transport) | Configurable (per-client) |
| **Protocol - SAML** | SAML IdP Broker — SP Metadata & Federation Signing | SIGN | RSA_SHA256/384/512 | Hybrid (configurable with hardcoded RS256 fallback) |
| **Protocol - SAML** | SAML Metadata Public Key Loader | SIGN | RSA, ECDSA (from metadata) | SPI-driven |
| **Protocol - SAML** | SAML Artifact Resolution | SIGN | RSA_SHA256 | Strict (hardcoded RS256) |
| **Protocol - SAML** | SAML2Signature — Hardcoded RSA-SHA1 Default | SIGN | RSA_SHA1 (deprecated default) | Strict (hardcoded RSA-SHA1) |

---

**Document Metadata**

**Created:** 2026-07-09  
**Verified:** 2026-07-09  
**Source:** `pqc_overview.html` (61 domains analyzed)  
**Total Domains:** 61  
**Asymmetric Only:** Yes  
**Related Issue:** [#46336](https://github.com/keycloak/keycloak/issues/46336)