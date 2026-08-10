# Apply PQC Review Findings to pqc_overview.html

## Purpose

This prompt instructs an AI agent to apply the corrections and additions documented in a PQC accuracy review findings file (`findings/pqc_overview_review_findings_YYYY-MM-DD.md`) to `pqc_overview.html` and `README.md`.

## How to Use

Run this prompt with a specific findings file, e.g.:

```
Apply the findings from findings/pqc_overview_review_findings_2026-07-30.md to pqc_overview.html and README.md following the instructions in APPLY_PQC_REVIEW_FINDINGS_PROMPT.md
```

---

## Instructions for the Agent

You are applying a set of reviewed, verified corrections to `pqc_overview.html`. The findings file is structured in parts; follow the sections below in order.

### Step 1 — Read the Inputs

1. Read the findings file (`findings/pqc_overview_review_findings_YYYY-MM-DD.md`) in full.
2. Read `pqc_overview.html` in full (or in chunks if large — it is typically ~2200 lines).
3. Read `README.md`.

Identify the review date from the findings file header (`**Date of review:**`).

---

### Step 2 — Apply Part 1 Corrections (Errors and Inaccuracies)

Work through every item under **Part 1 — Errors and Inaccuracies**.

For each error:

- **If severity is ERROR:** Apply the correction unconditionally.
- **If severity is ⚠️ LINE SHIFTED (within tolerance):** Update the stated line number to the verified actual line number.
- **If severity is NOTE or CONFIRMED — NO CHANGE:** Skip; no edit required.

#### Common correction types and where to find them in the HTML

| Correction type | Location in HTML |
|-----------------|-----------------|
| Domain count in headings ("All N Domains") | `<h2>Analysis Complete — All N Domains</h2>` and `<h3>Status Distribution (All N Domains)</h3>` |
| Domain count in status paragraph | `<p … background:#f0f9ff …>` near the footer |
| Status distribution table counts | `<tbody>` rows of the first table after "Status Distribution" heading — the `font-weight:600` `<td>` in each row |
| Missing GAP entries (broken `#gap-N` anchors) | The GAP reference `<table>` which contains `<tr id="gap-1">` … `<tr id="gap-N">`. Add new `<tr id="gap-N">` rows after the last existing one |
| Line number claim in a domain row | The PQC state `<td>` for that row number |

---

### Step 3 — Apply Part 7 Additions (New Content)

Work through **Part 7B — Additions of New Content** in order of priority (HIGH first, then MEDIUM, then LOW).

For each addition item:

#### A — Adding a new domain row

1. Identify the correct section header row (e.g. `<tr class="section-row"><td colspan="8">…</td></tr>`).
2. If a new section is needed, insert a new section-row before the new domain row.
3. Insert the new `<tr>` domain row after the section-row, using the next sequential domain number.
4. Follow the existing column structure exactly:
   - Col 1: `<td class="num">N</td>`
   - Col 2: Domain name (`<strong>…</strong>`)
   - Col 3: Key files (`<td class="files"><code>…</code><br/>…</td>`)
   - Col 4: Short summary of crypto usage
   - Col 5: Category badge (`<span class="cat sign|enc|both|cert|mixed">…</span>`)
   - Col 6: Short summary of gap
   - Col 7: PQC state badge + detailed description + GAP tag links
   - Col 8: Action / implementation plan

#### B — Adding a new GAP table entry

Find the last `<tr id="gap-N">` row in the GAP reference table (search for `id="gap-` to locate it). Insert the new entry after it, following this template:

```html
<tr id="gap-N" [style="background:#f7f8fa;" for odd-positioned rows]>
  <td style="padding:6px 9px; border:1px solid #e5e7eb; vertical-align:top; font-weight:700;"><span class="gap-tag">GAP-N</span> <span style="font-size:10px; color:#57606a;">(new)</span></td>
  <td style="padding:6px 9px; border:1px solid #e5e7eb; vertical-align:top;"><strong>TITLE</strong></td>
  <td style="padding:6px 9px; border:1px solid #e5e7eb; vertical-align:top; color:SEVERITY_COLOR; font-weight:600;">SEVERITY</td>
  <td style="padding:6px 9px; border:1px solid #e5e7eb; vertical-align:top;"><code>FILE.java</code> (line N)</td>
  <td style="padding:6px 9px; border:1px solid #e5e7eb; vertical-align:top;">DESCRIPTION</td>
</tr>
```

Severity colours: CRITICAL = `#991b1b`, HIGH = `#d97706`, MEDIUM = `#57606a`, LOW = `#57606a`.

Alternate `style="background:#f7f8fa;"` on even-positioned new rows for visual striping consistency.

#### C — Adding entries to the "Files Evaluated" section

Find the last `<tr>` row in the "Files Evaluated — Not Added as New Rows" table (the table after the `<h2>Files Evaluated…</h2>` heading). Insert new rows at the end of that table's `<tbody>`, before `</tbody>`, following this template:

```html
<tr [style="background:#f7f8fa;" for alternating rows]>
  <td style="padding:6px 9px; border:1px solid #e5e7eb; vertical-align:top;"><code>path/to/File.java</code></td>
  <td style="padding:6px 9px; border:1px solid #e5e7eb; vertical-align:top;">REASON — explain why not a new row, what it covers, and any PQC note.</td>
</tr>
```

---

### Step 4 — Update Status Distribution Table

After applying all domain row additions, **recount** the domain rows by PQC state:

1. Search the HTML for all `<span class="pqc pqc-pending">`, `pqc-partial`, `pqc-blocked`, `pqc-safe`, `pqc-external` occurrences **within `<td>` cells in the main domain table** (rows 1–N).
2. Exclude occurrences in: the legend, the status distribution table, and the GAP table.
3. Update the `font-weight:600` count cells in the status distribution table accordingly.
4. Update the total domain count in:
   - `<h2>Analysis Complete — All N Domains</h2>`
   - `<h3>Status Distribution (All N Domains)</h3>`
   - The status paragraph (`<p … background:#f0f9ff …>All N domains complete…</p>`)

**Important:** The PENDING count includes only domains whose primary PQC state badge is `pqc-pending`. Domains with PARTIAL, BLOCKED, SAFE, or EXTERNAL badges that also contain a PENDING note do not count as PENDING.

---

### Step 5 — Update the Footer and README

#### pqc_overview.html footer

Replace the `<footer>` content with:

```html
<footer>
  <p>Last updated: DD Month YYYY (review findings applied: [brief summary of changes])</p>
</footer>
```

#### README.md

1. Find the existing `**Last updated:**` line (if present) or insert one after the "Note: searches were conducted…" paragraph.
2. Set it to: `**Last updated:** DD Month YYYY — Review findings from \`findings/pqc_overview_review_findings_YYYY-MM-DD.md\` applied: [brief summary matching the footer].`
3. In the **PQC Readiness Accuracy Reviews** table, prepend a new row at the top (below the header) so the most recent review always appears first. The table has two columns — **Date** and **PQC Changes** — with no Keycloak HEAD column:

```markdown
| Date | PQC Changes |
|------|-------------|
| YYYY-MM-DD | [pqc_overview_review_findings_YYYY-MM-DD.md](findings/pqc_overview_review_findings_YYYY-MM-DD.md) |
```

---

### Step 6 — Skip List (Items That Require No Changes)

Do **not** modify anything for items listed under:

- **Part 7C — Items Confirmed Not Changed**
- Any finding whose severity is labelled `CONFIRMED — STABLE`, `CONFIRMED`, or `NOTE`
- GAP-10 / prior review corrections that only affect a **previous** findings file (not the current HTML)

---

### Step 7 — Validation

After all edits, verify:

1. The domain count in all three headings/paragraphs is consistent and equals the highest domain row number present in the table.
2. All `<a class="gap-tag" href="#gap-N">GAP-N</a>` links referenced in domain rows have a corresponding `<tr id="gap-N">` in the GAP table.
3. The status distribution total (PENDING + PARTIAL + BLOCKED + SAFE + EXTERNAL) equals the domain count.
4. No `<tr id="gap-N">` is duplicated.

Report any inconsistencies before finishing.

---

### Notes

- Preserve all existing HTML structure, CSS classes, and inline styles exactly — do not reformat unrelated content.
- Only touch lines that correspond to a finding in the review file.
- The review findings file itself is read-only; do not modify it.
- After completing all edits, confirm which items from Part 7A (Corrections) and Part 7B (Additions) were applied, and which (if any) were skipped and why.
