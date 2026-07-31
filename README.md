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

**Last updated:** 30 July 2026 — Review findings from `pqc_overview_review_findings_2026-07-30.md` applied: GAP-24 and GAP-25 added; domain row 63 (OID4VCI Credential Response Encryption) added; status distribution counts corrected (PARTIAL 14, BLOCKED 29, SAFE 7, EXTERNAL 5, total 63); row 62 `ACCEPTED_ALGORITHMS` line number corrected to 79; three new files added to the Files Evaluated section.

## How It Was Conducted

AI agents were used to perform a systematic grep-based sweep of the Keycloak source tree targeting asymmetric algorithm identifiers (`RSA`, `ECDSA`, `ECDH`, `EdDSA`, `EC`, `PS256`, `ES256`, etc.) and their associated factory, provider, and manager classes. Each matched file was inspected to determine:

1. Which asymmetric algorithm(s) it uses or configures
2. Whether the usage is hardcoded or SPI-driven (and therefore automatically extensible)
3. What specific code change, if any, is needed for ML-DSA or ML-KEM support
4. Any external standards dependency (FAPI, CAEP, FIDO2) that gates Keycloak's own changes

Each distinct use-case was classified as an independent **domain**. 62 domains were identified in total. Each domain was assigned a PQC state and, where applicable, a concrete implementation plan.

## Quick Start

  📊 **[View Main Analysis](pqc_overview.html)** - Complete table of all 62 domains with gaps and implementation plans

  To view the HTML file locally, download all files and run:
  ```bash
  open pqc_overview.html
  ```

  All links in the html file are markdown files.  It is recommended to include the markdown viewer extension in your browser to view the associated files. 

## Update Analysis Files

### Reassess PQC Readiness (`pqc_overview.html`)

Run [`REASSESS_PQC_READINESS_PROMPT.md`](REASSESS_PQC_READINESS_PROMPT.md) whenever the Keycloak `main` branch advances or before acting on any gap. The prompt drives an AI agent through a systematic verification of every domain row, GAP entry, file path, line number, and algorithm claim in `pqc_overview.html` against the live source tree. It covers:

- **Pre-flight checks** — confirms the Keycloak HEAD commit and scope
- **Domain file existence** — verifies every source file referenced in all 62 domain rows still exists at the stated path
- **Line-number verification** — re-checks every hardcoded line number claim within a ±5-line tolerance
- **Algorithm / PQC state claims** — confirms that reported hardcodings, SPI usages, and PQC states still match the code
- **New domain discovery** — sweeps for newly added files or features that introduce fresh asymmetric crypto usage

The agent writes its output to a dated findings file (e.g. `pqc_overview_review_findings_YYYY-MM-DD.md`) and updates the README table in the **PQC Readiness Accuracy Reviews** section below.

### Review Findings Files (`pqc_overview_review_findings_*.md`)

Each run of the reassessment prompt produces a structured findings file with up to seven parts:

| Part | Contents |
|------|----------|
| **Part 1 — Errors & Inaccuracies** | Every confirmed error found: wrong file paths, stale line numbers, inaccurate descriptions. Each entry states what the HTML currently says, what the source actually contains, and the exact correction needed. |
| **Part 2 — Domain-by-Domain Verification** | A row-by-row table (✅ / ⚠️ / ℹ️) for every domain confirming file existence, algorithm claims, and PQC state. |
| **Part 3 — GAP Reference Verification** | Every GAP entry checked for correct files and line numbers. |
| **Part 4 — Files Evaluated Section** | Confirms the "Files Evaluated — Not Added as New Rows" section is still accurate. |
| **Part 5 — Status Distribution** | Recounts PENDING PROVIDERS / PARTIAL / BLOCKED / SAFE / EXTERNAL DEPENDENCY totals. |
| **Part 6 — New Content** | Newly discovered domains, files, or gaps not yet in `pqc_overview.html`. |
| **Part 7 — Summary of Required Changes** | A consolidated checklist of every correction and addition to apply to `pqc_overview.html`. |

> **⚠️ Human intervention required.** The findings file is a read-only report — the agent does **not** automatically edit `pqc_overview.html`. A human must review Part 7 of the findings file and apply each listed correction and addition to `pqc_overview.html` manually (or by running a targeted edit prompt). Once applied, update the **Last updated** line at the top of `README.md` and add a row to the accuracy review table below.

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


## PQC Readiness Accuracy Reviews

Re-running the prompt [`REASSESS_PQC_READINESS_PROMPT.md`](REASSESS_PQC_READINESS_PROMPT.md) verifies that every file path, line number, algorithm claim, domain row, and GAP entry in `pqc_overview.html` is still accurate against the live Keycloak source. Run it whenever the Keycloak `main` branch advances significantly or before acting on any gap.

| Date | Findings File | Keycloak HEAD |
|------|--------------|---------------|
| 2026-07-09 | [2026-07-09 Review](pqc_overview_review_findings.md) | `5f289cf668` |
| 2026-07-30 | [2026-07-30 Review](pqc_overview_review_findings_2026-07-30.md) | `5f289cf668` |
