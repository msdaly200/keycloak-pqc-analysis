# REASSESS_PQC_READINESS_PROMPT

## Purpose

This prompt instructs an AI agent to perform a complete accuracy re-assessment of
`pqc_overview.html` against the live Keycloak source code. It replicates the methodology
used in the original review (`findings/pqc_overview_review_findings.md`) and produces an updated
findings report.

**Scope:** Every domain row, every GAP entry, and the "Files Evaluated — Not Added as
New Rows" section of `pqc_overview.html`. The exact counts are derived at runtime from
the HTML itself (see Pre-Flight Step 0) — do not assume any hardcoded number.

**Output:** A new findings file (see naming convention below). Do NOT modify
`pqc_overview.html` or any other file until the human reviewer has read and approved
the findings report.

---

## Instructions for the AI Agent

> **Setup required before running this prompt**
> This prompt assumes the `keycloak` repository has been added as a folder to the same
> workspace as this `pqc_analysis` project. If you have not done this yet, open the
> workspace in your IDE and add your local clone of the Keycloak repository as a second
> workspace folder. All Keycloak source paths below are resolved relative to that folder.

You are a Senior Keycloak engineer with deep knowledge of cryptographic algorithms and
the Keycloak codebase.

Your task is to verify that every factual claim in `pqc_overview.html` is accurate
against the Keycloak source in the `keycloak` workspace folder.

Work through each section in order. For each item, verify it against the actual source.
Record your findings in a new file named:

```
findings/pqc_overview_review_findings_YYYY-MM-DD.md
```

where `YYYY-MM-DD` is today's date.

Use the structure in `findings/pqc_overview_review_findings.md` as your template. Do NOT copy
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

### Step 0 — Derive ground truth from `pqc_overview.html` (run this first)

Before any source checks, read the HTML to establish the current domain count, GAP
count, and the exact set of `.java` filenames the HTML claims to cover. These three
values drive every subsequent step and must not be assumed from memory.

```bash
# --- Run from the pqc_analysis workspace root ---

# 0a. Count domain rows (unique domain .md links = one per domain row)
grep -o 'domains/Domain_[0-9]*_[^"]*\.md' pqc_overview.html | sort -u | wc -l

# 0b. List all GAP identifiers referenced
grep -o 'GAP-[0-9]*' pqc_overview.html | sort -u

# 0c. Extract every .java filename cited in the HTML (Key Files column + prose)
grep -o '[A-Z][A-Za-z0-9]*\.java' pqc_overview.html | sort -u
```

Record the outputs as:
- **DOMAIN_COUNT** — the number from 0a; use this wherever the prompt says "all N domains"
- **GAP_LIST** — the sorted list from 0b; use this as the complete GAP checklist in Part 3
- **FILE_LIST** — the sorted list from 0c; use this as the complete file checklist in B1

### Step 1 — Resolve paths

```bash
# Resolve the keycloak workspace folder path (works for any user / machine)
KEYCLOAK_DIR=$(find "$(dirname "$(pwd)")" -maxdepth 2 -type d -name "keycloak" | head -1)
# If the auto-detection above is empty, set KEYCLOAK_DIR manually, e.g.:
# KEYCLOAK_DIR="/path/to/your/keycloak"

# Confirm Keycloak repo location and HEAD commit
cd "$KEYCLOAK_DIR" && git log --oneline -3

# Confirm pqc_analysis workspace
ls pqc_overview.html
```

---

## Section A — Structural Facts (light grep only — these change rarely)

These are facts about class shape, enum values, and method signatures — things that are
stable across minor Keycloak versions and change only when a feature is meaningfully
redesigned. Run the indicated grep for each; if the output matches, record as
"confirmed stable". If the output has changed, flag as ❌ and describe the change.

> **What does NOT belong here:** Library versions, "feature X not yet implemented",
> algorithm allow-lists, and any claim that a PQC gap still exists. Those are state
> claims — they are the very things this review exists to track — and they live in
> **Section B3** below.

| Fact | Verification grep |
|------|-------------------|
| `Algorithm.java` has `ML_DSA_44`, `ML_DSA_65`, `ML_DSA_87` constants; `KeyType.AKP = "AKP"` exists. | `grep -n "ML_DSA\|AKP" core/src/main/java/org/keycloak/crypto/Algorithm.java core/src/main/java/org/keycloak/crypto/KeyType.java` |
| `AKPPublicJWK.java`, `AKPUtils.java`, `JWKBuilder.akp()`, and `JWKParser` AKP parsing all exist in the core JWK layer. | `find . -name "AKPPublicJWK.java" -o -name "AKPUtils.java" \| grep -v test` |
| `TokenCategory.LOGOUT` and `TokenCategory.ID` resolve to the same algorithm via `DefaultTokenManager.signatureAlgorithm()` — both use `ID_TOKEN_SIGNED_RESPONSE_ALG`. | `grep -n "ID_TOKEN_SIGNED_RESPONSE_ALG\|TokenCategory\.LOGOUT\|TokenCategory\.ID" services/src/main/java/org/keycloak/jose/jws/DefaultTokenManager.java` |
| `Constants.DEFAULT_SIGNATURE_ALGORITHM = Algorithm.RS256` and `Constants.INTERNAL_SIGNATURE_ALGORITHM = Algorithm.HS512`. | `grep -n "DEFAULT_SIGNATURE_ALGORITHM\|INTERNAL_SIGNATURE_ALGORITHM" server-spi-private/src/main/java/org/keycloak/models/Constants.java` |
| `SAML2Signature.signatureMethod` defaults to `SignatureMethod.RSA_SHA1`; `digestMethod` defaults to `DigestMethod.SHA1`. | `grep -n "RSA_SHA1\|DigestMethod\.SHA1\|signatureMethod\s*=" saml-core/src/main/java/org/keycloak/saml/processing/api/saml/v2/sig/SAML2Signature.java` |
| `SignatureAlgorithm.java` (SAML) contains only RSA variants. No PQC URIs. | `grep -n "RSA_SHA\|ML_DSA\|MLDSA" saml-core/src/main/java/org/keycloak/saml/SignatureAlgorithm.java` |
| `XMLEncryptionUtil` supports only RSA-OAEP (RSA_OAEP_11 as default), RSA_OAEP, and RSA_v1dot5 key transport. | `grep -n "RSA_OAEP\|RSA_v1dot5\|ML.KEM" saml-core/src/main/java/org/keycloak/saml/processing/core/util/XMLEncryptionUtil.java` |
| `SAMLEncryptionAlgorithms` enum: `RSA_OAEP` and `RSA1_5` variants only. | `grep -n "RSA_OAEP\|RSA1_5\|ML.KEM" services/src/main/java/org/keycloak/protocol/saml/SAMLEncryptionAlgorithms.java` |
| `DefaultSamlArtifactResolver.java` handles artifact storage/routing only — the SOAP fetch path is in `SAMLEndpoint.java`. | `grep -n "SOAP\|fetch\|resolve" services/src/main/java/org/keycloak/protocol/saml/DefaultSamlArtifactResolver.java \| head -10` |
| `DIDUtils.java` is P-256 only by design. | `grep -n "P-256\|ES256\|secp256\|ML_DSA" core/src/main/java/org/keycloak/util/DIDUtils.java` |
| Maven Enforcer plugin is present in `pom.xml` but there is **no minimum-version rule** for `bcprov-jdk18on`. | `grep -n "bcprov-jdk18on\|enforcer" pom.xml \| head -10` |

---

## Section B — Items to Verify Actively on Every Run

For each item below, grep or read the referenced file and confirm the stated fact still
holds. Mark ✅ (confirmed), ❌ (incorrect — state what changed), or ⚠️ (partially correct
— describe).

### B1 — All Domain Rows: File Existence Check

The file list is derived from **FILE_LIST** established in Pre-Flight Step 0 — do not
use a hardcoded list. For each `.java` filename in FILE_LIST, confirm it still exists
in the Keycloak source (non-test paths only).

**Step (a) — build the find expression from FILE_LIST:**

Take every filename from FILE_LIST and construct a single `find` command. The pattern
is the same regardless of how many files the HTML references — it always stays in sync
with the HTML automatically.

```bash
cd "$KEYCLOAK_DIR"

# Build the -name expression dynamically from FILE_LIST.
# Example (replace the names with the actual FILE_LIST output from Step 0):
#
#   find . \( \
#     -name "DefaultTokenManager.java" \
#     -o -name "AsymmetricSignatureProvider.java" \
#     ... one -o -name entry per file in FILE_LIST ...
#   \) ! -path "*/test/*" 2>/dev/null | sort
#
# Then compare the returned paths against FILE_LIST.
# Any filename in FILE_LIST with NO matching path is MISSING.
```

**Step (b) — report results:**

For each file in FILE_LIST:
- ✅ **FOUND** — at least one non-test path returned
- ❌ **MISSING** — no path returned; investigate whether it was renamed or merged and
  note whether the row's PQC assessment still holds
- ⚠️ **MOVED** — found but at a different path than the one cited in the HTML; record
  the new path so the HTML can be updated

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
cd "$KEYCLOAK_DIR"

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

> **These are the PQC progress indicators.** Every item here was previously recorded as
> a gap or limitation. Each one must be re-verified on every run — never assume they are
> still true. If any check shows a gap has been closed, mark it **RESOLVED** and flag it
> for a PQC state upgrade in the findings report.
>
> The following items were formerly in Section A as "stable facts" but belong here
> because they are state claims that will change as PQC work progresses:
> library versions (`bc-fips`, `bcprov`/Quarkus BOM), missing factories
> (`GeneratedAKPKeyProviderFactory`, ML-DSA `SignatureProviderFactory`, ML-KEM
> `CekManagementProviderFactory`), and missing algorithm support in allow-lists
> (`FapiConstant`, `JWKSServerUtils`, `JWKSUtils`, `OIDCWellKnownProvider`,
> `DefaultKeyProviders`, `ClientAsymmetricSignatureVerifierContext`).

These are the claims most likely to change as Keycloak progresses on PQC. Run these checks:

```bash
cd "$KEYCLOAK_DIR"

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
cd "$KEYCLOAK_DIR"

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

Your output file (`findings/pqc_overview_review_findings_YYYY-MM-DD.md`) must follow this structure:

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
`README.md` in the `pqc_analysis` workspace folder. Add or update the
following section (insert after the GitHub Issues section):

```markdown
## PQC Readiness Accuracy Reviews

The following accuracy reviews have been conducted against the live Keycloak source:

| Date | PQC Changes |
|------|-------------|
| YYYY-MM-DD | [pqc_overview_review_findings_YYYY-MM-DD.md](findings/pqc_overview_review_findings_YYYY-MM-DD.md) |
```

If the section already exists, **prepend** a new row at the top of the table (below the header row) so the most recent review always appears first.

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
- [ ] Output file `findings/pqc_overview_review_findings_YYYY-MM-DD.md` written
- [ ] `README.md` updated with date and link to findings file
- [ ] No changes made to `pqc_overview.html` or domain/gap markdown files
- [ ] Summary presented to user with count of: errors found, stale items, new domains, resolved gaps
