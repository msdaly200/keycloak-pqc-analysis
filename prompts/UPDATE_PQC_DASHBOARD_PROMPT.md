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

> **Note on `pqc_overview.html`:** The "New GitHub Issues Needed" table inside `pqc_overview.html`
> is **not managed by this prompt**. It is static HTML maintained manually (or via
> `Github_PQC_issues_overview_prompt.md` when new issues are filed against tracked gaps).
> When a gap gets a GitHub issue filed against it, update that table in `pqc_overview.html`
> directly by adding a ✅ Issue column entry — it is not auto-synced by any prompt.

## Required inputs

1. Read `pqc_dashboard.html` fully.
2. Read `gh_issues/Github_PQC_issues_overview.md` fully.
3. Read `pqc_overview.html` to derive domain counts:
   - Extract the authoritative domain total from the heading matching `Analysis Complete — All <N> Domains` — that `<N>` is the domain count.
   - Extract the per-status counts from the Status Distribution table.
   - **Verify** that the five status counts (BLOCKED + PENDING PROVIDERS + PARTIAL + SAFE + EXTERNAL DEPENDENCY) sum exactly to `<N>`.
   - If the sum does not equal `<N>`, do **not** update the dashboard — report the discrepancy and stop.

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
- subtitle / "Last updated" text
- summary stat cards
- donut chart numbers, labels, legends, and SVG dash values
- "New Issues Needed" heading total
- summary tables and counts
- any links to `pqc_overview.html` or `gh_issues/Github_PQC_issues_overview.md` if needed

## SVG Donut Maths

Both donuts use `r="40"`, so the full circumference **C = 2π × 40 = 251.33**.

Each segment is a `<circle>` with `stroke-dasharray="ARC C"` and `stroke-dashoffset="OFFSET"`,
plus `transform="rotate(-90 50 50)"` so 0° is at 12 o'clock.

### How to compute arc lengths

```
arc = round(count / total * 251.33, 2)
```

All arcs for a donut must sum to exactly 251.33 (adjust the largest arc by any rounding
remainder if needed).

### How to compute dashoffsets

The segments are drawn in **smallest-arc-first order** in the HTML (so the largest sits on
top and is most visible). Visually they read left-to-right in the legend order. The
**first legend item** (e.g. green "Open") always starts at the same fixed start angle,
which is preserved across updates by keeping `green_dashoffset = 62.83` constant.

Given legend order **green → amber → purple**:

```
green_dashoffset  = 62.83          # fixed — do not change
amber_dashoffset  = green_dashoffset - green_arc
purple_dashoffset = amber_dashoffset - amber_arc
```

The draw order in the HTML is the **reverse** of the legend: purple first, amber second,
green last (so green is on top).

### Example — GitHub Issue Status donut (current values)

| Segment | count | arc    | dashoffset |
|---------|-------|--------|------------|
| green (Open) | 44 of 62 | 178.36 | 62.83 |
| amber (In Progress) | 3 of 62 | 12.16 | −115.53 |
| purple (Closed) | 15 of 62 | 60.81 | −127.69 |

The centre `<text>` shows the **total** (62). The legend counts show each segment's count.

### Quick Python snippet

```python
import math
C = round(2 * math.pi * 40, 2)  # 251.33
total, green, amber, purple = 62, 44, 3, 15
g = round(green  / total * C, 2)
a = round(amber  / total * C, 2)
p = round(C - g - a, 2)          # use subtraction for last segment to avoid rounding drift
g_off = 62.83                     # fixed
a_off = round(g_off - g, 2)
p_off = round(a_off - a, 2)
print(f"green: arc={g}, offset={g_off}")
print(f"amber: arc={a}, offset={a_off}")
print(f"purple: arc={p}, offset={p_off}")
```

The **Domain PQC Readiness** donut follows the same maths with its own segment counts
(BLOCKED, PENDING, PARTIAL, SAFE, EXTERNAL) and its own fixed start offset for the first
segment. Preserve the existing first segment's dashoffset value when updating that donut.

## Consistency requirements

Make sure:
- the visible totals match the markdown sources exactly
- donut legend counts add up to the displayed totals
- SVG donut segments match the displayed counts
- issue status totals equal tracked issues
- priority bucket totals equal “New Issues Needed”
- domain readiness status counts (BLOCKED + PENDING PROVIDERS + PARTIAL + SAFE + EXTERNAL) sum exactly to the "Domains Analysed" figure, and that figure matches the `Analysis Complete — All <N> Domains` heading in `pqc_overview.html`

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
3a. Confirm that the five domain status counts in the dashboard sum to the "Domains Analysed" stat card value, and that this value matches the `Analysis Complete — All <N> Domains` heading in `pqc_overview.html`. If there is any mismatch, report it as an error — do not silently accept mismatched counts.
4. Confirm no other files were changed.

## Deliverable

Return a short summary listing:
- which values changed
- which source file each change came from
- confirmation that only `pqc_dashboard.html` was modified
