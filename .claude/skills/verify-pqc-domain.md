---
description: Verify or generate a PQC domain analysis for 100% accuracy against Keycloak source code
---

# Verify (or Generate) PQC Domain

You are verifying or generating the Post-Quantum Cryptography (PQC) readiness
analysis for a specific Keycloak domain.

## Context

- Analysis repo: `/Users/mariedaly/projects/pqc_analysis` (current directory)
- Keycloak source: `~/projects/keycloak`
- Overview file: `pqc_overview.html` (domain count is derived at runtime — see Step 0)
- Domain files: `domains/Domain_<n>_<name>.md`
- GitHub issues: `gh_issues/Github_PQC_issues_overview.md`

## Modes

This skill operates in two modes depending on whether the domain file already
exists:

- **VERIFY mode** — the `Domain_<n>_*.md` file exists. Verify all claims against
  live source and report findings.
- **GENERATE mode** — no `Domain_<n>_*.md` file exists yet (e.g. newly discovered
  domain). Generate the file from source, then run VERIFY mode on it before
  committing it to disk.

Determine the mode automatically:

```bash
ls domains/Domain_<n>_*.md 2>/dev/null | head -1
```

If the file is found → VERIFY mode. If not → GENERATE mode.

---

## Step 0 — Establish current domain count (always run first)

```bash
grep -c 'class="num"' pqc_overview.html
```

Use this as the authoritative total domain count in all output. Do not use any
hardcoded number.

---

## VERIFY Mode

The domain number to verify will be provided as an argument. You must:

1. **Read the current analysis** from `pqc_overview.html` and the corresponding
   `domains/Domain_<n>_*.md` file.
2. **Verify against Keycloak source code** at `~/projects/keycloak`:
   - Check that all listed files exist (non-test paths only)
   - Verify the asymmetric algorithms in use at the documented line numbers
   - Apply the ±3-line tolerance rule: a line shift of ≤3 with unchanged crypto
     code is ✅ CONFIRMED, not an error
   - Confirm the PQC state assessment is accurate
   - Validate the fix description is technically correct
3. **Cross-reference GitHub issues** in `gh_issues/Github_PQC_issues_overview.md`:
   - Find existing issues that cover this domain
   - Determine if a new issue is needed
4. **Check domain file completeness**:
   - Verify the backlink `[← Back to PQC Overview](../pqc_overview.html)` exists
   - Verify a `detail-link` in the HTML row points to this `.md` file
5. **Report findings** using the format below.

### VERIFY Output Format

## Domain <N> Verification Results

### ✅ Verified Accurate
[List confirmed correct items with the grep/find command used]

### ⚠️ Needs Correction
[List errors found with specific corrections — file path, line number, proposed fix]

### 📋 GitHub Issue Status
- **Best fit issue**: [Most specific issue covering this domain]
- **Additional issues**: [Related issues]
- **New issue needed**: [Yes/No with justification]

### 🔗 Proposed Changes
[Specific changes to approve — list every file that would be modified]

---

## GENERATE Mode

When no `Domain_<n>_*.md` file exists, follow these steps in order. Do not
write any file until Step G4 is complete.

### G1 — Determine the domain number and slug

If the domain number was provided as an argument, use it. Otherwise derive the
next available number:

```bash
grep -o 'class="num">[0-9]*' pqc_overview.html | grep -o '[0-9]*' | sort -n | tail -1
```

New domain number = that value + 1. Derive the slug from the domain title
(e.g., `OID4VP_Response_Encryption`, `Client_SDK_DPoP`).

### G2 — Read the HTML row

Read the `<tr>` for this domain number in `pqc_overview.html` to extract:
- Title (column 2)
- Key files (column 3)
- Algorithm description (column 4)
- Category badge (column 5)
- Gap summary (column 6)
- PQC state and description (column 7)
- Action / fix description (column 8)
- Any existing GAP-tag links

### G3 — Verify every claim against Keycloak source

For each file cited in the HTML row:

```bash
cd ~/projects/keycloak
find . -name "<File.java>" ! -path "*/test/*" 2>/dev/null
```

Read the relevant lines. Verify:
- The file exists at the cited path
- The algorithm or pattern described is present at (or within ±3 lines of) the
  documented line number
- The gap description accurately reflects what the code does

If any claim cannot be verified, note it explicitly — do not fabricate source
evidence.

### G4 — Draft the domain markdown

Using the verified source evidence, draft the full `.md` file content following
this template. Every section must be filled with verified content — no
placeholders:

```markdown
# Domain N — <Title>

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

<One or two sentences describing the component and its role in Keycloak.>

## Gap

**<One-line gap summary>.**

**File:** `<module/path/to/File.java>:<line>`

**Current code:**

```java
<Exact code snippet copied from the live source>
```

**The problem:**

<Explanation of why this code blocks or limits PQC readiness.>

## Current PQC State

**<BLOCKED | PARTIAL | PENDING PROVIDERS | SAFE | EXTERNAL DEPENDENCY>**

## Required Changes

<Step-by-step description of the code change needed. Include a code snippet
for the proposed fix where possible.>

## Dependencies

<List any GAP tags or GitHub issues that must be resolved first, or "None".>

## GitHub Issue Status

<State whether an existing issue covers this domain and its number, or
"Needs GitHub issue under [#43690](https://github.com/keycloak/keycloak/issues/43690)
with <SEVERITY> priority".>

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
```

### G5 — Present draft for approval

Present the full draft content to the user. Wait for explicit approval before
writing the file. List any unverified claims or open questions clearly.

### G6 — Write the file and add the detail-link

After approval:

1. Write `domains/Domain_N_<slug>.md`.
2. Check whether the HTML row for domain N already has a `detail-link`:
   ```bash
   grep -A 20 'class="num">N<' pqc_overview.html | grep "detail-link"
   ```
3. If no `detail-link` is present, propose adding the following to the action
   cell (column 8) of the domain row, immediately before the closing `</td>`:
   ```html
   <br />
   <a href="domains/Domain_N_<slug>.md" class="detail-link">→ View implementation plan</a>
   ```
   Wait for approval before editing `pqc_overview.html`.

---

## Critical Constraints

⚠️ **GitHub READ-ONLY**: Never edit, modify, create, or comment on ANY GitHub
issues. Only read issue status.

⚠️ **100% Accuracy Required**: The Keycloak team relies on this analysis. Verify
every claim against source code before writing it. Never assert a line number
without running `grep -n` to confirm it.

⚠️ **Propose Before Editing**: Outline all changes for user approval. Wait for
explicit approval before updating any file.

⚠️ **No Recaps**: User reviews diffs directly. Don't summarize changes after
making them.

⚠️ **Three artifacts rule**: A new domain is only complete when all three exist —
the HTML `<tr>` row, the `domains/Domain_N_*.md` file, and the `detail-link` in
the action cell. Flag any domain that is missing any of the three.
