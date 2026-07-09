# Keycloak Post-Quantum Cryptography (PQC) Readiness Analysis

With the publication of NIST FIPS 203 (ML-KEM) and FIPS 204 (ML-DSA) in August 2024, widely-used asymmetric algorithms — RSA, ECDSA, ECDH, and EdDSA — are now considered quantum-vulnerable. A sufficiently powerful quantum computer running Shor's algorithm could break the mathematical hardness assumptions underpinning all of these schemes. This analysis was conducted to determine how exposed the Keycloak codebase is to that risk and what work is required to make it PQC-ready.

## What Was Analysed

The entire Keycloak codebase (main branch) was searched for every location that uses, configures, or depends on an asymmetric cryptographic algorithm. The search covered:

- **Signing operations** — JWT/JWS signing and verification (OIDC, SAML, SD-JWT, OID4VC, DPoP, CIBA, client assertions)
- **Encryption operations** — JWE key encapsulation (CEK management), SAML assertion encryption, JAR request object encryption
- **Key management** — realm key generation, import, and bootstrap providers; keystore loading; JWKS serialisation and endpoints
- **Certificate handling** — X.509 mTLS, browser auth, admin-generated client certificates
- **Crypto backend providers** — BouncyCastle (default), BC-FIPS (FIPS 140-2/3 mode), WildFly Elytron
- **Client-side tooling** — SDK JWT credential providers, DPoP proof generation, admin CLI (`kcadm.sh` / `kcreg.sh`)
- **Protocol-specific enforcement** — FAPI algorithm allowlists, CAEP/SET signing, WebAuthn/FIDO2

Note: searches were conducted up to 3rd of July 2026. Any changes/improvements from that date will not be automatically updated in these files.

## How It Was Conducted

AI agents were used to perform a systematic grep-based sweep of the Keycloak source tree targeting asymmetric algorithm identifiers (`RSA`, `ECDSA`, `ECDH`, `EdDSA`, `EC`, `PS256`, `ES256`, etc.) and their associated factory, provider, and manager classes. Each matched file was inspected to determine:

1. Which asymmetric algorithm(s) it uses or configures
2. Whether the usage is hardcoded or SPI-driven (and therefore automatically extensible)
3. What specific code change, if any, is needed for ML-DSA or ML-KEM support
4. Any external standards dependency (FAPI, CAEP, FIDO2) that gates Keycloak's own changes

Each distinct use-case was classified as an independent **domain**. 61 domains were identified in total. Each domain was assigned a PQC state and, where applicable, a concrete implementation plan.

To verify each Domain, you can run the following prompt to ensure the information is still accurate. So for the first row in the table, run...
 ```bash
/verify-pqc-domain 1
 ```

## Quick Start

  📊 **[View Main Analysis](pqc_overview.html)** - Complete table of all 61 domains with gaps and implementation plans

  To view the HTML file locally:
  ```bash
  open pqc_overview.html
  ```

  All links in the html file are markdown files.  It is recommended to include the markdown viewer extension in your browser to view the associated files. 

  ## Status Distribution

  - **PENDING PROVIDERS:** 20 domains (33%)
  - **PARTIAL:** 10 domains (16%)
  - **BLOCKED:** 24 domains (39%)
  - **SAFE:** 4 domains (7%)
  - **EXTERNAL DEPENDENCY:** 3 domains (5%)


## Other Considerations for PQC Readiness

### Storage Constraints and Database Schema

**Realm Attribute Value Length (`REALM_ATTRIBUTE.VALUE`, `NVARCHAR(255)`)**
The 255-character column limit is sufficient for current algorithm policy strings but may be tight if multiple PQC algorithm identifiers are concatenated alongside attestation or user-verification parameters. **Recommendation:** Add a Liquibase migration expanding to `NVARCHAR(1024)` or `TEXT/CLOB`. **Priority: Medium** — current configurations are unlikely to overflow, but this future-proofs the schema.

**Credential Storage (Already PQC-Ready)**
The `CREDENTIAL` table uses `CLOB` columns (`SECRET_DATA`, `CREDENTIAL_DATA`) with no size limit. ML-DSA-87 public keys (2,592 bytes) and signatures (4,595 bytes) fit comfortably, as do large Base64-encoded WebAuthn credentials. **No schema change required.**


