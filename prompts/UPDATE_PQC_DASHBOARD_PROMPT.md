# Update pqc_dashboard.html

Update `pqc_dashboard.html` to match the current data in `gh_issues/Github_PQC_issues_overview.md` and `pqc_overview.html`.

## Scope

Only update the local file `pqc_dashboard.html`.

Do not modify:
- `gh_issues/Github_PQC_issues_overview.md`
- `pqc_overview.html`
- `README.md`
- any files under `domains/`
- any files under `findings/`

## Required inputs

1. Read `pqc_dashboard.html` fully.
2. Read `gh_issues/Github_PQC_issues_overview.md` fully.
3. Read the parts of `pqc_overview.html` needed to verify the current domain PQC readiness counts if those counts are shown in the dashboard.

## Update rules

Use `gh_issues/Github_PQC_issues_overview.md` as the source of truth for:
- Last updated date
- tracked issue count
- open issue count
- closed issue count
- new issues needed count
- hierarchy status counts
- “New Issues Needed” priority breakdown
- links to issue hierarchy markdown

Use `pqc_overview.html` as the source of truth only for:
- total domains analysed
- domain PQC readiness badge counts, if those are displayed in the dashboard

## What to update in pqc_dashboard.html

Update all affected dashboard content so it is internally consistent:
- page title
- subtitle / “Last updated” text
- summary stat cards
- donut chart numbers, labels, legends, and SVG dash values
- “New Issues Needed” heading total
- summary tables and counts
- any links to `pqc_overview.html` or `gh_issues/Github_PQC_issues_overview.md` if needed

## Consistency requirements

Make sure:
- the visible totals match the markdown sources exactly
- donut legend counts add up to the displayed totals
- SVG donut segments match the displayed counts
- issue status totals equal tracked issues
- priority bucket totals equal “New Issues Needed”
- domain readiness totals equal “Domains Analysed”

## Constraints

- Preserve the existing structure and styling of `pqc_dashboard.html` unless a small change is required to keep the numbers accurate.
- Make the minimal necessary edits.
- Do not add new sections, features, JavaScript, or redesign the page.
- Keep all existing navigation links working.

## Validation

Before finishing:
1. Re-read the updated `pqc_dashboard.html`.
2. Verify every displayed number against the markdown sources.
3. Verify all totals and subtotals reconcile.
4. Confirm no other files were changed.

## Deliverable

Return a short summary listing:
- which values changed
- which source file each change came from
- confirmation that only `pqc_dashboard.html` was modified
