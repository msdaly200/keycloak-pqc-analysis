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

Segments are laid out clockwise from 12 o'clock in **legend order**. Because the dash
pattern is `"ARC C"` (period = ARC + C), it never wraps, so every segment's offset must be
**zero or negative** — a positive offset trims the start of the arc and leaves a gap in the ring.

```
offset_1 = 0                                  # first legend item starts at 12 o'clock
offset_n = -(sum of arcs of legend items 1 .. n-1)
```

The draw order in the HTML is the **reverse** of the legend (last legend item first, first
legend item last). With the cumulative offsets above the segments do not overlap, so the
ring closes exactly: the last segment's `arc + |offset|` must equal 251.33.

### Example — GitHub Issue Status donut (68 issues)

| Segment | count | arc    | dashoffset |
|---------|-------|--------|------------|
| green (Open) | 47 of 68 | 173.71 | 0 |
| amber (In Progress) | 4 of 68 | 14.78 | −173.71 |
| purple (Closed) | 17 of 68 | 62.84 | −188.49 |

The centre `<text>` shows the **total** (68). The legend counts show each segment's count.

### Quick Python snippet

```python
import math
C = round(2 * math.pi * 40, 2)  # 251.33
counts = [47, 4, 17]             # legend order; replace with current values
total = sum(counts)
arcs = [round(c / total * C, 2) for c in counts[:-1]]
arcs.append(round(C - sum(arcs), 2))   # last arc by subtraction to avoid rounding drift
offset = 0.0
for c, a in zip(counts, arcs):
    print(f"count={c} arc={a} offset={-round(offset, 2) if offset else 0}")
    offset += a
```

The **Domain PQC Readiness** donut uses the same maths with legend order BLOCKED, PENDING,
PARTIAL, SAFE, EXTERNAL (first segment offset 0). The segment counts must sum to the
domain total.


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
