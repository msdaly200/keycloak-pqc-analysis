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

# 0a. Count domain rows — count <td class="num"> cells in the main audit table.
#     This is the authoritative domain count regardless of whether a domain .md
#     file exists for the row. Do NOT count .md file links (some rows may not have
#     a .md file yet and would be undercounted).
grep -c 'class="num"' pqc_overview.html

# 0b. List all GAP identifiers referenced
grep -o 'GAP-[0-9]*' pqc_overview.html | sort -u

# 0c. Extract every .java filename cited in the HTML (Key Files column + prose)
grep -o '[A-Z][A-Za-z0-9]*\.java' pqc_overview.html | sort -u
```

Record the outputs as:
- **DOMAIN_COUNT** — the number from 0a; use this wherever the prompt says "all N domains".
  The source of truth is the number of rows in the "Keycloak Asymmetric Crypto Audit" table,
  not the number of domain `.md` files (some rows may not yet have a backing `.md` file).
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
| `DefaultSamlArtifactResolver.java` handles artifact storage/routing only — the SOAP back-channel fetch is in `SAMLIdentityProvider.resolveArtifact()` (`SAMLEndpoint.java` only calls it and validates the response). | `grep -n "SOAP\|fetch\|resolve" services/src/main/java/org/keycloak/protocol/saml/DefaultSamlArtifactResolver.java \| head -10; grep -n "Soap\.createMessage\|resolveArtifact" services/src/main/java/org/keycloak/broker/saml/SAMLIdentityProvider.java services/src/main/java/org/keycloak/broker/saml/SAMLEndpoint.java` |
| Root `pom.xml` only defines the Maven Enforcer **version properties** (`version.enforcer.plugin`, `extra.enforcer.rules.version`); the plugin is not declared and there is **no minimum-version rule** for `bcprov-jdk18on`. | `grep -n "bcprov-jdk18on\|enforcer" pom.xml \| head -10` |

---

## Section B — Items to Verify Actively on Every Run

For each item below, grep or read the referenced file and confirm the stated fact still
holds. Mark ✅ (confirmed), ❌ (incorrect — state what changed), or ⚠️ (partially correct
— describe).

### B1 — All Domain Rows: File Existence Check

The file list is derived from **FILE_LIST** established in Pre-Flight Step 0 — do not
use a hardcoded list. For each `.java` filename in FILE_LIST, confirm it still exists
in the Keycloak source (non-test paths only).

**Step (a) — compare FILE_LIST against the tracked Keycloak sources:**

Use `git ls-files` rather than `find .`. A bare `find .` over a full Keycloak checkout
walks `node_modules` and build output and can exceed the tool timeout; `git ls-files`
returns only tracked files in well under a second. Compare basenames, so the check stays
in sync with the HTML automatically however many files it references.

```bash
# Run from the pqc_analysis workspace root: save FILE_LIST (Step 0c output) to a file
grep -o '[A-Z][A-Za-z0-9]*\.java' pqc_overview.html | sort -u > /tmp/pqc_file_list.txt

cd "$KEYCLOAK_DIR"

# Basenames of every tracked, non-test .java file
git ls-files '*.java' | grep -v '/test/' | sed 's|.*/||' | sort -u > /tmp/pqc_kc_files.txt

# FILE_LIST entries with NO matching tracked file (candidates for MISSING)
comm -23 /tmp/pqc_file_list.txt /tmp/pqc_kc_files.txt

# To find the path(s) of a FOUND file (needed for the MOVED check):
git ls-files '*.java' | grep -v '/test/' | grep '/DefaultTokenManager\.java$'
```

Untracked local files are deliberately ignored; the review is against committed source.

**Step (b) — report results:**

For each file in FILE_LIST:
- ✅ **FOUND** — at least one non-test path returned
- ❌ **MISSING** — no path returned; investigate whether it was renamed or merged and
  note whether the row's PQC assessment still holds
- ⚠️ **MOVED** — found but at a different path than the one cited in the HTML; record
  the new path so the HTML can be updated

**Expected false MISSING entries — resolve these before reporting ❌:**

The Step 0c regex extracts bare tokens, so two kinds of entry can never match a filename
exactly:

1. **Glob patterns.** A cited pattern such as `crypto/EcdhEsA*KwCekManagementProviderFactory.java`
   is extracted as `KwCekManagementProviderFactory.java`. Expand the pattern against the
   tracked file list (e.g. `git ls-files | grep 'EcdhEsA.*KwCekManagementProviderFactory\.java'`)
   and report ✅ if it matches at least one file.
2. **Annotated renames.** A filename the HTML itself marks as "Previously cited as …;
   renamed/refactored upstream" (currently `AttestationX509CertificateValidator.java`,
   now `AttestationProofValidator.java`) is intentionally absent. Confirm the replacement
   file exists and report ✅ (RENAMED, annotated), not ❌. Search the HTML for
   "Previously cited as" to find any others.

Only an entry that is neither a glob nor an annotated rename is a genuine ❌ MISSING.

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

# GAP-9: SAMLIdentityProvider.java — two RS256 hardcodings (documented: lines 416, 507)
grep -n "Algorithm\.RS256" services/src/main/java/org/keycloak/broker/saml/SAMLIdentityProvider.java

# GAP-9: SamlProtocol.java — RS256 hardcoding (documented: line 544)
grep -n "Algorithm\.RS256" services/src/main/java/org/keycloak/protocol/saml/SamlProtocol.java

# GAP-9: SamlService.java — RS256 hardcoding (documented: line 1012)
grep -n "Algorithm\.RS256" services/src/main/java/org/keycloak/protocol/saml/SamlService.java

# GAP-10: AbstractOAuth2IdentityProvider.java — HS256 fallback (documented: line 969) and RS256 fallback (documented: line 977)
grep -nE "Algorithm\.(HS256|RS256)" services/src/main/java/org/keycloak/broker/oidc/AbstractOAuth2IdentityProvider.java

# GAP-11: DescriptionConverter.java — RS256 special-case (documented: line 416)
grep -n "Algorithm\.RS256\|RS256.*equals\|equals.*RS256" services/src/main/java/org/keycloak/services/clientregistration/oidc/DescriptionConverter.java

# GAP-12: DefaultTokenManager.java — DEFAULT_SIGNATURE_ALGORITHM fallback (documented: line 240)
grep -n "DEFAULT_SIGNATURE_ALGORITHM\|Constants\.DEFAULT" services/src/main/java/org/keycloak/jose/jws/DefaultTokenManager.java

# GAP-13: AuthUtil.java — rsa256 hardcoding (documented: line 213)
# (use -E: in basic regex "\(" is an unbalanced group and grep errors out)
grep -nE "\.rsa256\(" integration/client-cli/admin-cli/src/main/java/org/keycloak/client/cli/util/AuthUtil.java

# GAP-18: JwtCNonceHandler.java — ES256 then RS256 fallback (documented: lines 281/287)
grep -n "Algorithm\.ES256\|Algorithm\.RS256\|getActiveKey" services/src/main/java/org/keycloak/protocol/oid4vc/issuance/keybinding/JwtCNonceHandler.java

# GAP-20: DefaultTokenManager.decodeClientJWT() — KeyUse.ENC key selection (documented: method at line 122, ENC lookups at lines 144, 150)
grep -n "decodeClientJWT\|KeyUse\.ENC" services/src/main/java/org/keycloak/jose/jws/DefaultTokenManager.java

# GAP-21: UserInfoEndpoint.java — per-client signing / CEK management lookups (documented: lines 348, 386-387)
grep -n "signatureAlgorithm(TokenCategory\.USERINFO\|cekManagementAlgorithm(TokenCategory\.USERINFO\|encryptAlgorithm(TokenCategory\.USERINFO" services/src/main/java/org/keycloak/protocol/oidc/endpoints/UserInfoEndpoint.java

# GAP-22: AttestationBasedClientAuthenticator.java — [TODO] algorithm enforcement
# (documented: lines 418-419 in the HTML/gaps doc; found at lines 423-424 on 2026-10-29 — pending doc update.
#  Judge against whichever value the docs currently carry.)
grep -n "\[TODO\].*alg\|alg.*\[TODO\]" services/src/main/java/org/keycloak/authentication/authenticators/client/AttestationBasedClientAuthenticator.java

# GAP-23: OIDCWellKnownProvider.java — hardcoded RS256 constant (documented: line 85)
grep -n "DEFAULT_CLIENT_AUTH_SIGNING_ALG_VALUES_SUPPORTED" services/src/main/java/org/keycloak/protocol/oidc/OIDCWellKnownProvider.java

# GAP-26: JWKSServerUtils.toJwk() — no AKP branch (documented: lines 59-65)
grep -n "KeyType\.\|toJwk\b\|AKP" services/src/main/java/org/keycloak/protocol/oidc/utils/JWKSServerUtils.java

# GAP-30: ClientAsymmetricSignatureVerifierContext — RSA guard (documented: lines 36-37)
grep -n "not RSA\|Key Type is not RSA\|KeyType\.RSA\|KeyType\.AKP" services/src/main/java/org/keycloak/crypto/ClientAsymmetricSignatureVerifierContext.java

# GAP-24 / OID4VP row 62: ACCEPTED_ALGORITHMS and ES256 hardcodings
# (documented: ACCEPTED_ALGORITHMS line 85, ES256 key lookups lines 210-211, TODO line 81)
grep -n "ACCEPTED_ALGORITHMS\|Algorithm\.ES256\|getActiveKey.*ES256\|ES256.*getActiveKey" services/src/main/java/org/keycloak/broker/oid4vp/OID4VPIdentityProvider.java | head -10

# GAP-25 / OID4VP row 62: ECDH-ES/secp256r1 response encryption (documented: EphemeralKey line 41, ResponseEncryption line 38)
grep -n "secp256r1\|CURVE_SEC\|ECDH_ES\|KEY_MANAGEMENT_ALG" services/src/main/java/org/keycloak/broker/oid4vp/EphemeralKey.java services/src/main/java/org/keycloak/broker/oid4vp/ResponseEncryption.java

# GAP-27 / Domain 49: ClientAttributeCertificateResource — generateKeyPairCertificate RSA hardcoding
# (documented: lines 126, 293 in the HTML/gaps doc; found at 126 and 288 on 2026-10-29 — pending doc update.
#  The RSA keygen itself is in KeycloakModelUtils.generateKeyPairCertificate -> KeyUtils.generateRsaKeyPair.)
grep -n "generateKeyPairCertificate\|ML.DSA\|AKP" services/src/main/java/org/keycloak/services/resources/admin/ClientAttributeCertificateResource.java
grep -n "generateRsaKeyPair" server-spi-private/src/main/java/org/keycloak/models/utils/KeycloakModelUtils.java

# GAP-28: JWTClientCredentialsProvider — switch missing AKP case (documented: lines 77-96)
grep -n "KeyType\.RSA\|KeyType\.EC\|KeyType\.OKP\|KeyType\.AKP\|Invalid KeyPair algorithm" core/src/main/java/org/keycloak/protocol/oidc/client/authentication/JWTClientCredentialsProvider.java

# GAP-29: DPoPGenerator — no ML-DSA convenience method (documented: lines 49-50)
grep -n "generateRsaSignedDPoPProof\|TODO\|ML.DSA\|AKP" core/src/main/java/org/keycloak/util/DPoPGenerator.java | head -10

# GAP-32: Kerberos/SPNEGO — delegated to JDK; no Keycloak enctype policy
# (documented: SPNEGOAuthenticator.establishContext() ~line 160; KerberosJdkProvider Krb5LoginModule ~line 120)
grep -n "GSSManager\|acceptSecContext\|KRB5_OID\|SPNEGO_OID" federation/kerberos/src/main/java/org/keycloak/federation/kerberos/impl/SPNEGOAuthenticator.java
grep -n "Krb5LoginModule" common/src/main/java/org/keycloak/common/util/KerberosJdkProvider.java
# Claim to confirm: NO enctype handling anywhere in Keycloak's Kerberos code (expect zero matches).
# If this grep ever returns matches, mark GAP-32 ❌ CODE CHANGED — Keycloak has started controlling
# enctypes and the gap description and severity MUST be re-assessed.
grep -rInE -i "enctype|rc4|aes128|aes256|des3|permitted_enctypes|default_tkt_enctypes" \
  federation/kerberos services/src/main/java/org/keycloak/authentication/authenticators/browser/SpnegoAuthenticator.java \
  common/src/main/java/org/keycloak/common/util/KerberosJdkProvider.java

# Domain 61: SAML2Signature — RSA_SHA1 default (documented: lines 55/57)
grep -n "RSA_SHA1\|DigestMethod\.SHA1\|signatureMethod\s*=" saml-core/src/main/java/org/keycloak/saml/processing/api/saml/v2/sig/SAML2Signature.java

# Domain 52: DockerComposeCertsDirectory — RSA 2048 hardcoding (documented: lines 29-30)
grep -n "getKeyPairGen\|initialize(2048\|KeyType\.RSA" services/src/main/java/org/keycloak/protocol/docker/installation/compose/DockerComposeCertsDirectory.java

# Domain 50 (JWTClientCredentialsProvider), Domain 27 (JwtCNonceHandler) and OID4VP row 62
# are covered by the GAP-28, GAP-18 and GAP-24/25 greps above — no need to re-run.
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

> **Full gap descriptions** are maintained in
> `gaps/Github_issue_requirements_for_gaps.md`. The checks below verify each
> gap's current state against the Keycloak source. When a gap is RESOLVED,
> update both the HTML GAP summary table and the gaps doc.

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

# Has bcprov version changed? bcprov-jdk18on is not pinned in Keycloak's pom.xml — it comes
# from the Quarkus platform BOM, so a change shows up as a change to the Quarkus pin...
grep -n "quarkus.version\|quarkus.build.version" pom.xml | head -5
# ...and the actual bcprov version must be read from that BOM in the local Maven repository
# (substitute the quarkus.version value from the line above):
grep -n -A2 "<artifactId>bcprov-jdk18on</artifactId>" ~/.m2/repository/io/quarkus/platform/quarkus-bom/<quarkus.version>/quarkus-bom-<quarkus.version>.pom
# If the BOM is not in ~/.m2, run `./mvnw -q dependency:resolve` (or a normal build) first, or
# say so in the report and mark the bcprov version "not independently verified".
# Optionally confirm ML-DSA/ML-KEM content in the resolved JAR:
#   unzip -l ~/.m2/repository/org/bouncycastle/bcprov-jdk18on/<version>/bcprov-jdk18on-<version>.jar | grep -ci "mldsa"

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

# GAP-32: Has Keycloak added Kerberos enctype configuration or PKINIT support?
# (If matches appear, GAP-32 severity must be re-assessed — it is currently LOW/external.)
grep -rInE -i "enctype|pkinit" federation/kerberos common/src/main/java/org/keycloak/common/util/KerberosJdkProvider.java
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
# (git ls-files avoids an operator-precedence bug in the old `find -o ! -path` form)
git ls-files | grep -E "/(Generated|Imported)[A-Za-z]*KeyProviderFactory\.java$" | grep -v "/test/" | sort

# Any new broker identity providers that involve signing/verification?
find . -name "*IdentityProvider.java" ! -path "*/test/*" 2>/dev/null \
  | xargs grep -l "SignatureProvider\|Algorithm\.\|RS256\|ES256\|AKP" 2>/dev/null \
  | grep -v "^Binary" | sort

# Any new files in sdjwt or oid4vc that use signing?
find . \( -path "*/sdjwt/*.java" -o -path "*/oid4vc/*.java" \) ! -path "*/test/*" 2>/dev/null \
  | xargs grep -l "SignatureProvider\|SignatureSignerContext\|algorithm\|RS256\|ES256" 2>/dev/null \
  | grep -v "^Binary" | sort

# GAP-32 sweep: any new Kerberos files not yet covered in "Files Evaluated"?
# Compare against the Kerberos entry in the "Files Evaluated — Not Added as New Rows" section.
find . -path "*/kerberos/*.java" ! -path "*/test/*" 2>/dev/null | sort
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
<all rows — DOMAIN_COUNT from Pre-Flight Step 0a (63 at the 2026-10-30 update)>

## Part 3 — GAP Reference Verification
<table with columns: GAP | Title | Files Correct? | Line Numbers | Notes>
<all GAPs in GAP_LIST from Pre-Flight Step 0b (32 at the 2026-10-30 update: GAP-1 … GAP-32)>
<Apply the ±3-line tolerance rule: a shift of ≤3 lines with unchanged crypto code is ✅ not an error>

<!-- EXTERNAL GAP NOTE: Some GAPs are classified as External Dependencies / Spec-Gated.
     These have no numbered domain row and "Domains: —" in the GAP reference table.
     This is intentional — do NOT flag a missing domain cross-reference as an error for them.
     Currently: GAP-6 (SSH/SFTP), GAP-17 (SAML/RFC spec-gated), GAP-25 (OID4VP ECDH-ES),
     GAP-32 (Kerberos/SPNEGO JDK-delegated, LOW/external).
     GAP-32 in particular: it is a documentation-only gap with no Keycloak code fix.
     Severity LOW. Verify only the two line-number claims below (B2) and the enctype sweep (B2/B3). -->

For each GAP, cross-reference the full description in
`gaps/Github_issue_requirements_for_gaps.md#gap-N`. If the gaps doc
description no longer matches the source, flag it as STALE in both
Part 3 and Part 7.

## Part 4 — "Files Evaluated" Section Verification
<confirm each file in the exclusion list still exists and reason is still valid>

## Part 5 — Status Distribution Table Verification

Recount the per-status totals by counting **primary badges from domain rows only**:

1. For each `<tr>` in the main domain table that contains `class="num"`, find the **first** `class="pqc pqc-*"` badge in that row — that is its primary status. Ignore any secondary badges on the same row (e.g. a row showing `BLOCKED (RSA path) / PENDING PROVIDERS (ECDSA/EdDSA paths)` counts only as BLOCKED).
2. Exclude all badge occurrences in: the legend bar, the Status Distribution summary table itself, and the GAP reference table.
3. Tally: BLOCKED · PENDING PROVIDERS · PARTIAL · SAFE · EXTERNAL DEPENDENCY.
4. **Verify the five counts sum exactly to DOMAIN_COUNT** (from Pre-Flight Step 0a). If they do not, the discrepancy is an error and must be listed in Part 1.
5. Compare your recount against the current counts in the Status Distribution table in `pqc_overview.html`. Record each discrepancy as a required correction in Part 7A.

The expected recount output format:

```
Recount from domain rows (primary badge only):
  BLOCKED:              N
  PENDING PROVIDERS:    N
  PARTIAL:              N
  SAFE:                 N
  EXTERNAL DEPENDENCY:  N
  ──────────────────────
  TOTAL:                N  ← must equal DOMAIN_COUNT

Current HTML Status Distribution table:
  BLOCKED:              N  (correct / WRONG — should be N)
  PENDING PROVIDERS:    N  (correct / WRONG — should be N)
  ...
```

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
- [ ] All Section B2 line-number checks completed (including GAP-32 Kerberos claims and enctype sweep)
- [ ] All Section B3 algorithm/PQC-state checks completed (including GAP-32 enctype/PKINIT sweep)
- [ ] All Section B4 new-domain discovery checks completed (including Kerberos file sweep)
- [ ] External gaps (GAP-6, GAP-17, GAP-25, GAP-32) verified: "Domains: —" is intentional — not an error
- [ ] Part 5 status distribution recount completed; five counts sum exactly to DOMAIN_COUNT
- [ ] Output file `findings/pqc_overview_review_findings_YYYY-MM-DD.md` written
- [ ] `README.md` updated with date and link to findings file
- [ ] No changes made to `pqc_overview.html` or domain/gap markdown files
- [ ] Summary presented to user with count of: errors found, stale items, new domains, resolved gaps
