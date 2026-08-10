# Keycloak Post-Quantum Cryptography (PQC) Readiness Analysis

With the publication of NIST FIPS 203 (ML-KEM) and FIPS 204 (ML-DSA) in August 2024, widely-used asymmetric algorithms — RSA, ECDSA, ECDH, and EdDSA — are now considered quantum-vulnerable. A sufficiently powerful quantum computer running Shor's algorithm could break the mathematical hardness assumptions underpinning all of these schemes. 

Symmetric cryptography in Keycloak was reviewed separately and is not the main PQC risk surface: production uses such as AES, HMAC, Argon2, and PBKDF2 were found not to be quantum-broken, with only some legacy SHA-1–based usages noted as classical hygiene concerns rather than PQC vulnerabilities. 

This analysis was conducted to determine how exposed the Keycloak codebase is to asymmetric quantum risk and what work is required to make it PQC-ready.

## What Was Analysed

The entire Keycloak codebase (main branch) was searched for every location that uses, configures, or depends on an asymmetric cryptographic algorithm. The search covered:

- **Signing operations** — JWT/JWS signing and verification (OIDC, SAML, SD-JWT, OID4VC, DPoP, CIBA, client assertions)
- **Encryption operations** — JWE key encapsulation (CEK management), SAML assertion encryption, JAR request object encryption
- **Key management** — realm key generation, import, and bootstrap providers; keystore loading; JWKS serialisation and endpoints
- **Certificate handling** — X.509 mTLS, browser auth, admin-generated client certificates
- **Crypto backend providers** — BouncyCastle (default), BC-FIPS (FIPS 140-2/3 mode), WildFly Elytron
- **Client-side tooling** — SDK JWT credential providers, DPoP proof generation, admin CLI (`kcadm.sh` / `kcreg.sh`)
- **Protocol-specific enforcement** — FAPI algorithm allowlists, CAEP/SET signing, WebAuthn/FIDO2


## How It Was Conducted

AI agents were used to perform a systematic grep-based sweep of the Keycloak source tree targeting asymmetric algorithm identifiers (`RSA`, `ECDSA`, `ECDH`, `EdDSA`, `EC`, `PS256`, `ES256`, etc.) and their associated factory, provider, and manager classes. Each matched file was inspected to determine:

1. Which asymmetric algorithm(s) it uses or configures
2. Whether the usage is hardcoded or SPI-driven (and therefore automatically extensible)
3. What specific code change, if any, is needed for ML-DSA or ML-KEM support
4. Any external standards dependency (FAPI, CAEP, FIDO2) that gates Keycloak's own changes

Each distinct use-case was classified as an independent **domain**. 62 domains were identified in total. Each domain was assigned a PQC state and, where applicable, a concrete implementation plan.

## Quick Start

  📊 **[View Main Analysis](pqc_overview.html)** - Complete table of all PQC domains with gaps and implementation plans

  To view the HTML file locally, download all files and run:
  ```bash
  open pqc_overview.html
  ```

  All links in the html file are markdown files.  It is recommended to include the markdown viewer extension in your browser to view the associated files. 

## Keeping up to date

To update `pqc_overview.html` with the latest Keycloak PQC information, please read [README_UPDATE.md](README_UPDATE.md).

**Last updated:** 10 August 2026 — Review findings from `findings/pqc_overview_review_findings_2026-08-10.md` applied: 5 new OID4VC keybinding/credentialbuilder files added to Files Evaluated section; no PQC state changes.

## PQC Readiness Accuracy Reviews

The following accuracy reviews have been conducted against the live Keycloak source:

| Date | PQC Changes |
|------|-------------|
| 2026-08-10 | [pqc_overview_review_findings_2026-08-10.md](findings/pqc_overview_review_findings_2026-08-10.md) |
| 2026-07-30 | [pqc_overview_review_findings_2026-07-30.md](findings/pqc_overview_review_findings_2026-07-30.md) |
| 2026-07-09 | [pqc_overview_review_findings.md](findings/pqc_overview_review_findings.md) |