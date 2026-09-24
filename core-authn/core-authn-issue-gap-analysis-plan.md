# Plan: Core-Authn Team Issue Ownership & Gap Analysis (keycloak/keycloak)

## Top-Level Overview

**Goal:** Determine the functional ownership map of the `core-authn` team in `keycloak/keycloak` based on the last 8 weeks of issue activity on issues labeled with a `core-authn`-related label (e.g. `team/core-authn`, or any label containing "authn"), then cross-reference all currently open issues NOT carrying that label to find candidates that should likely be reviewed/owned by `core-authn`.

**Scope:**
- Read-only investigation. No GitHub issues, labels, or assignees will be modified.
- Data source: GitHub REST/GraphQL API or `gh` CLI (read-only commands only) against `keycloak/keycloak`.
- "Last 8 weeks" = any issue with activity (created, commented, labeled, updated) within the past 8 weeks — not just creation date.
- Both open and closed issues are included in the Step 1 (team history) pull.
- Team identification method: search issues by label matching `core-authn` (e.g. `label:team/core-authn`, or any label containing "authn"), per user's confirmed approach — no reliance on GitHub "assignee" semantics for team, since GitHub does not support assigning issues to a team directly, only to individual users.
- Final output: a single markdown report saved to the workspace (e.g. `gh_issues/core-authn_gap_analysis.md`), containing:
  - Section 1: functional/code ownership map derived from Step 1 data
  - Section 2: prioritized table of open issues recommended for `core-authn` review, with justification and current assignee/label

**Non-goals:**
- No code changes to this repo's PQC analysis content.
- No issue edits, comments, label changes, or assignments on GitHub — read-only.
- No time estimates.

## Sub-Task 1: Retrieve core-authn Team Issue History (Last 8 Weeks)

**Intent:** Establish the raw dataset of `core-authn`-labeled issues (open + closed) with activity in the last 8 weeks, which will be mined for functional/code ownership signals.

**Expected Outcomes:**
- A collected list of issue numbers, titles, states, labels, URLs, descriptions, and any linked PR references for all issues matching a `core-authn`-style label with activity in the last 8 weeks.
- Linked PRs (if any) identified for follow-up code-path extraction.

**Todo List:**
1. Use `gh` CLI (read-only) or GitHub search API against `keycloak/keycloak` to search issues by label — try candidate label patterns: `team/core-authn`, `core-authn`, and any label containing `authn` (e.g. via `gh label list` first to discover the exact label name(s) in use, then search issues with that label).
2. Filter results to issues updated within the last 8 weeks (`updated:>=<date>` search qualifier), including both `is:open` and `is:closed`.
3. For each matching issue, fetch: number, title, state, labels, body/description, and any cross-referenced/linked PRs or commits.
4. For each linked PR, fetch changed file paths (read-only `gh pr view --json files` or API `files` list) to extract concrete code paths.
5. Compile the raw findings into an intermediate list (can be kept in-memory/notes, or written to a scratch section of the report) — do not create the final report yet.

**Relevant Context:**
- Use the `github-cli` skill for the correct read-only `gh` commands (`gh issue list`, `gh api search/issues`, `gh pr view --json files`, `gh label list`).
- No files in this repo need to be read/modified for this sub-task — it is pure external data retrieval.

**Status:** [x] done — report saved to `gh_issues/core-authn_gap_analysis.md`

## Sub-Task 2: Build Functional Ownership Map (Section 1 of Report)

**Intent:** Convert the raw issue/PR data from Sub-Task 1 into a structured, de-duplicated summary of functional areas and code paths owned by `core-authn`.

**Expected Outcomes:**
- A structured list grouping: (a) functional/feature areas (e.g. "authentication flows", "credential validation", "brute force protection") and (b) concrete code files/modules/packages referenced, each with the issue(s) that surfaced them.

**Todo List:**
1. Aggregate all file paths, module names, and package references pulled from issue bodies, comments, and linked PR diffs in Sub-Task 1.
2. Group these into functional areas based on directory/module naming conventions and issue titles/labels (e.g. group by top-level Keycloak module: `services/src/main/java/org/keycloak/authentication/...`, `services/.../credential/...`, etc.).
3. Note any labels other than the core-authn label that co-occur on these issues (e.g. `area/authentication`), since these are useful overlap signals for Sub-Task 3.
4. Draft Section 1 content: structured list of functional areas → representative code paths → source issue(s).

**Relevant Context:**
- Depends on Sub-Task 1 output.
- This is analysis/organization only — no repository code needs to be cloned or read; file paths come from GitHub PR diff metadata, not local inspection.

**Status:** [x] done — report saved to `gh_issues/core-authn_gap_analysis.md`

## Sub-Task 3: Retrieve and Screen Currently Open Non-core-authn Issues

**Intent:** Pull all currently open issues that do NOT carry the core-authn label, and screen each against the ownership map from Sub-Task 2 to find candidates that overlap with `core-authn`'s domain.

**Expected Outcomes:**
- Full list of currently open issues excluding those with the core-authn label.
- Each screened against the Sub-Task 2 ownership map, with an overlap determination (yes/no + reason) recorded for issues showing overlap.

**Todo List:**
1. Use `gh`/API to list all currently open issues in `keycloak/keycloak` excluding the core-authn label (`is:open -label:<core-authn-label>`).
2. For each candidate issue, inspect title, body, labels, and any linked PR file paths for overlap with the Sub-Task 2 functional/code map (matching on module paths, functional keywords, or co-occurring labels identified in Sub-Task 2).
3. Record for each overlapping issue: issue number, title, URL, current assignee (if any), current labels/team tags (if any), and the specific reason/evidence for the overlap (e.g. "touches `AuthenticationProcessor.java`, matches core-authn's credential-validation area from issue #XXXX").
4. Discard issues with no discernible overlap; keep a running prioritized list of overlapping ones.

**Relevant Context:**
- Depends on Sub-Task 2's ownership map.
- Given repo scale, this may require paging through open issues — use search qualifiers to keep result sets manageable (e.g. filter by relevant `area/*` labels first as a pre-filter, then verify against the ownership map, to avoid manually screening every open issue in the repo).

**Status:** [x] done — report saved to `gh_issues/core-authn_gap_analysis.md`

## Sub-Task 4: Prioritize Gaps and Write Final Report

**Intent:** Convert the Sub-Task 3 overlap findings into a prioritized recommendation list, and assemble the complete two-section markdown report.

**Expected Outcomes:**
- A saved markdown file in the workspace containing:
  - Section 1: functional ownership map (from Sub-Task 2)
  - Section 2: prioritized table/list of open issues recommended for `core-authn` review — issue number, title, URL, overlap reason, current assignee/label
- No GitHub issues modified.

**Todo List:**
1. Rank the Sub-Task 3 overlap list by strength of evidence (e.g. direct code-path match > functional keyword match > label proximity match).
2. Assemble Section 2 as a table with columns: Issue #, Title, URL, Overlap Reason, Current Assignee/Label.
3. Assemble Section 1 from Sub-Task 2 draft content.
4. Write the combined report to a new markdown file in the workspace (suggested path: `gh_issues/core-authn_gap_analysis.md`, consistent with the existing `gh_issues/` convention in this repo).
5. Present a summary of key findings and the report location to the user.

**Relevant Context:**
- Follows existing repo convention of storing GitHub issue analysis under `gh_issues/` (see [`gh_issues/Github_PQC_issues_overview.md`](gh_issues/Github_PQC_issues_overview.md:1) for formatting style reference — tables with issue links, status badges).
- Depends on Sub-Tasks 1–3 being complete.

**Status:** [x] done — report saved to `gh_issues/core-authn_gap_analysis.md`
