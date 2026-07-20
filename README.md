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

Note: searches were conducted up to 10th of July 2026. Any changes/improvements from that date will not be automatically updated in these files.

## How It Was Conducted

AI agents were used to perform a systematic grep-based sweep of the Keycloak source tree targeting asymmetric algorithm identifiers (`RSA`, `ECDSA`, `ECDH`, `EdDSA`, `EC`, `PS256`, `ES256`, etc.) and their associated factory, provider, and manager classes. Each matched file was inspected to determine:

1. Which asymmetric algorithm(s) it uses or configures
2. Whether the usage is hardcoded or SPI-driven (and therefore automatically extensible)
3. What specific code change, if any, is needed for ML-DSA or ML-KEM support
4. Any external standards dependency (FAPI, CAEP, FIDO2) that gates Keycloak's own changes

Each distinct use-case was classified as an independent **domain**. 61 domains were identified in total. Each domain was assigned a PQC state and, where applicable, a concrete implementation plan.

## Quick Start

  📊 **[View Main Analysis](pqc_overview.html)** - Complete table of all 61 domains with gaps and implementation plans

  To view the HTML file locally, download all files and run:
  ```bash
  open pqc_overview.html
  ```

  All links in the html file are markdown files.  It is recommended to include the markdown viewer extension in your browser to view the associated files. 

## GitHub Issues Tracking

📋 **[View GitHub Issues Overview](Github_PQC_issues_overview.md)** - Complete hierarchy of all PQC-related GitHub issues

This file tracks PQC GitHub issues in the [Keycloak repository](https://github.com/keycloak/keycloak). It provides:

- **Issue Hierarchy** - Visual tree structure showing parent/child relationships between all PQC issues
- **Gap Mapping** - Which gaps are covered by existing issues vs. which need new issues
- **Status Tracking** - Current state (OPEN/CLOSED) and progress for each issue
- **Domain Coverage** - Links between domains, gaps, and GitHub issues

### Keeping the GitHub Issues Overview Updated

To update the `Github_PQC_issues_overview.md` file with the latest GitHub issue status, use the following prompt in a claude terminal:

```bash
Update the file Github_PQC_issues_overview.md with the latest status from GitHub for all Keycloak PQC-related issues
```
This prompt provides detailed instructions for using the GitHub CLI (`gh`) to fetch current issue statuses and update the local file. **Note:** This is READ-ONLY access to GitHub - it will not modify any issues, only read their status to update the local markdown file.

