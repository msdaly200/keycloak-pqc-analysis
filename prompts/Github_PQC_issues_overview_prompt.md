# Prompt: Update GitHub PQC Issues Overview

## ⚠️ CRITICAL: READ-ONLY GITHUB ACCESS

**DO NOT EDIT, MODIFY, UPDATE, OR WRITE TO ANY GITHUB ISSUES**

This task is **READ-ONLY** access to GitHub. You will:
- ✅ **ONLY READ** issue status from GitHub
- ✅ **ONLY UPDATE** the local markdown file
- ❌ **NEVER WRITE** to any GitHub issues
- ❌ **NEVER MODIFY** any GitHub issues
- ❌ **NEVER CREATE** any GitHub issues
- ❌ **NEVER COMMENT** on any GitHub issues

All updates are made **ONLY** to the local file: `gh_issues/Github_PQC_issues_overview.md`

**Note:** Simple expansion of this task that only involves reading GitHub issues and updating the local file does **NOT require user approval**. You may proceed directly with reading GitHub data and updating the markdown file.

---

## Task
Update the file `gh_issues/Github_PQC_issues_overview.md` with the latest status from GitHub for all Keycloak PQC-related issues.

## Document Structure

The document uses a **table-based hierarchy** format:
- Top-level issue #43690 is shown with heading and metadata (Status, Type, Progress)
- All direct sub-issues and their nested sub-issues are shown in a **single table**
- Table columns: Issue | Title | Status | Type | Progress | Sub-Issues
- Sub-issues are indented using `↳` arrows
- Level-2 sub-issues use `&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳` (6 non-breaking spaces)
- Level-3 sub-issues use `&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳` (12 non-breaking spaces)
- Sub-issue counts shown in **bold** (e.g., **12**) for parent issues with children
- Nested sub-issue counts shown in parentheses (e.g., (2), (5)) for third-level nesting

## Instructions

1. **Read the current file** to understand the existing table structure and issue list

2. **Check for new sub-issues** under parent issues:
   - Compare the current document's issue list against GitHub
   - Look for any newly linked sub-issues under #43690 and other parent issues
   - If new issues are found, add them to the table in the appropriate hierarchical position
   - Update sub-issue counts (e.g., "10" → "11" if a new issue was added under #43690)

3. **Fetch latest GitHub status** for all issues mentioned in the document:
   - Use `gh issue view <issue-number> --repo keycloak/keycloak --json number,title,state,closedAt` for each issue
   - Get progress counts for issues with sub-issues by checking the GitHub UI tracking
   - Note: GitHub's tracking data may require GraphQL API or parsing the issue body for task checkboxes
   - For new issues discovered, fetch their full details including title and type

4. **Update the following fields IN THE TABLE**:
   - **Status** badges (🔵 OPEN, 🟢 OPEN, 🟣 CLOSED) with colored spans for each issue
   - **Progress** counts (e.g., "4 of 12", "0 of 7") in the Progress column where applicable
   - **Last Updated** date at the top of the document (use current date in format YYYY-MM-DD)
   - **#43690 Progress** - Update based on completed direct sub-issues count (e.g., "2 of 11" if 2 closed out of 11 total)
   - **Direct Sub-Issues count** in the header if new issues were added

5. **Preserve the table structure**:
   - Keep the exact table format with proper indentation (`&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳` for level 2, `&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↳` for level 3)
   - Keep all sub-issue counts in the Sub-Issues column (**bold** for parents, (parentheses) for nested)
   - Keep all issue titles, links, and hierarchy unchanged
   - Only update status badges, progress counts, sub-issue counts, and the Last Updated date
   - When adding new issues, match the formatting and indentation of existing entries

6. **Status badge format**:
   - OPEN (in-progress): `<span style="color: #d97706; font-weight: bold;">🔵 OPEN</span>`
   - OPEN (ready/available): `<span style="color: #059669; font-weight: bold;">🟢 OPEN</span>`
   - CLOSED: `<span style="color: #7c3aed; font-weight: bold;">🟣 CLOSED</span>`

7. **Update the Additional Issues section**:
   - For every issue listed in the "Additional Issues" tables in the document, fetch its current state from GitHub
   - Update the **Status** column value (e.g. `OPEN` → `CLOSED`) for any issue whose state has changed
   - Use plain text `OPEN` / `CLOSED` in that table (no coloured spans — the Additional Issues table uses plain text status, not badge format)
   - **Remove any row from the Additional Issues tables whose issue number already appears anywhere in the main hierarchy table** (direct sub-issues or nested sub-issues). An issue must not appear in both sections. If a row has *(now in main hierarchy)* or similar text, or if its issue number is present in the main table, delete that row entirely.
   - Apart from the removals above, do not add or remove other rows — only update the Status column values

8. **Recalculate the Status Dashboard summary table**:
   - Count all issues across the main hierarchy table and Additional Issues tables
   - Update **Tracked Issues** = total count of all distinct issue numbers in the document
   - Update **🟢 Open** = count of issues currently with an OPEN status
   - Update **🟣 Closed** = count of issues currently with a CLOSED status
   - Update **Domains Analysed** by reading `pqc_overview.html`:
     - Search for the heading matching the pattern `Analysis Complete — All <N> Domains` — the number `<N>` is the current domain count
   - Update **🔴 New Issues Needed** by reading `pqc_overview.html`:
     - Count the number of `<tr id="gap-N">` rows in the PQC Gap Reference table — each row is one identified gap that requires a new GitHub issue

9. **Output the updated file** using the Edit tool to update only the changed status/progress values and add any new issues

10. **Sync the dashboard** — after the markdown file has been fully updated, run `prompts/UPDATE_PQC_DASHBOARD_PROMPT.md` to sync `pqc_dashboard.html` to match.

## Example Commands

```bash
# Get basic issue status
gh issue view 43690 --repo keycloak/keycloak --json number,title,state,closedAt

# Get multiple issues in batch
for issue in 43690 45168 46333 48821 49865 50084 50292; do
  gh issue view $issue --repo keycloak/keycloak --json number,title,state,closedAt --jq '{number, title, state, closedAt}'
done

# For sub-issue counts, you may need to use GraphQL:
gh api graphql -f query='
query {
  repository(owner: "keycloak", name: "keycloak") {
    issue(number: 45168) {
      number
      title
      state
      trackedIssues(first: 100) {
        totalCount
        nodes {
          number
          state
        }
      }
    }
  }
}'
```

## Key Issues to Check

### Top-Level Parent Issue
- #43690 - Post-Quantum Cryptography (PQC) readiness

**⚠️ IMPORTANT:** Do NOT rely on any hardcoded list of sub-issues. The hierarchy MUST be
discovered fresh from GitHub every time this prompt runs. Use the GraphQL queries below to
fetch the live list of direct and nested sub-issues under #43690, then reconcile against the
current document.

### Step A — Discover all direct sub-issues of #43690 from GitHub

```bash
gh api graphql -f query='
query {
  repository(owner: "keycloak", name: "keycloak") {
    issue(number: 43690) {
      trackedIssues(first: 100) {
        totalCount
        nodes {
          number
          title
          state
        }
      }
    }
  }
}'
```

Use `totalCount` to update the **Direct Sub-Issues: N** header in the document.
Use the `nodes` list as the authoritative set of direct sub-issues — do not assume any
specific numbers or count. Any issue in the document but not in this response has been
de-linked; any issue in this response but not in the document is newly added.

### Step B — Discover nested sub-issues for any direct sub-issue that has children

For each direct sub-issue returned in Step A, check whether it has its own tracked issues:

```bash
gh api graphql -f query='
query {
  repository(owner: "keycloak", name: "keycloak") {
    issue(number: ISSUE_NUMBER) {
      number
      title
      state
      trackedIssues(first: 100) {
        totalCount
        nodes {
          number
          title
          state
          trackedIssues(first: 100) {
            totalCount
            nodes {
              number
              title
              state
            }
          }
        }
      }
    }
  }
}'
```

Replace `ISSUE_NUMBER` with each direct sub-issue number. This gives you the full three-level
hierarchy in one query per direct sub-issue (or combine into a single query using aliases).

### Additional Issues (not in main hierarchy)

Fetch the current document to identify any issues listed in the "Additional Issues" section,
then check their status with:

```bash
gh issue view ISSUE_NUMBER --repo keycloak/keycloak --json number,title,state,closedAt
```

Do not maintain a hardcoded list of additional issues in this prompt — the document itself
is the source of truth for which additional issues exist.

## Update Format

The document uses a **table format** for the issue hierarchy. Update values within table cells:

**Table Cell Status Update Example:**

**Before:**
```markdown
| [#43691](https://github.com/keycloak/keycloak/issues/43691) | Hybrid key exchange in TLS 1.3 | <span style="color: #059669; font-weight: bold;">🟢 OPEN</span> | milestone | N/A | 0 |
```

**After (if status changed to CLOSED):**
```markdown
| [#43691](https://github.com/keycloak/keycloak/issues/43691) | Hybrid key exchange in TLS 1.3 | <span style="color: #7c3aed; font-weight: bold;">🟣 CLOSED</span> | milestone | N/A | 0 |
```

**Progress Update Example:**

**Before:**
```markdown
| [#45168](https://github.com/keycloak/keycloak/issues/45168) | Review what is needed for PQC readiness in Keycloak | <span style="color: #d97706; font-weight: bold;">🔵 OPEN</span> | spike | 4 of 12 | **12** |
```

**After (if progress changed):**
```markdown
| [#45168](https://github.com/keycloak/keycloak/issues/45168) | Review what is needed for PQC readiness in Keycloak | <span style="color: #d97706; font-weight: bold;">🔵 OPEN</span> | spike | 6 of 12 | **12** |
```

**Top-level Header Update:**

**Before:**
```markdown
**Status:** <span style="color: #d97706; font-weight: bold;">🔵 OPEN</span> | **Type:** feature | **Progress:** 0 of 10
```

**After:**
```markdown
**Status:** <span style="color: #d97706; font-weight: bold;">🔵 OPEN</span> | **Type:** feature | **Progress:** 3 of 10
```

**Last Updated Date:**

**Before:**
```markdown
**Last Updated:** 2026-07-08
```

**After:**
```markdown
**Last Updated:** 2026-07-09
```

## Notes

- ⚠️ **READ-ONLY GitHub access** - **ABSOLUTELY NO MODIFICATIONS TO ANY GITHUB ISSUES**
- You are **ONLY** reading issue status to update the **LOCAL** markdown file
- **DO NOT** use any GitHub API endpoints that modify, create, update, or comment on issues
- Use only `gh issue view` (read-only) and `gh api` with read-only GraphQL queries
- If GitHub rate limits are hit, pause and retry
- If progress counts cannot be retrieved via API, note in the output which ones need manual verification
- Maintain exact formatting and structure of the original document
- Only update status-related fields and the date in the **LOCAL FILE**

## Completion Criteria

The updated **LOCAL FILE** should:
1. Have today's date in the "Last Updated" field
2. Have current OPEN/CLOSED status for all issues in the main hierarchy table (read from GitHub)
3. Have current OPEN/CLOSED status for all issues in the Additional Issues tables (read from GitHub)
4. Have current progress counts where applicable
5. Include any newly discovered sub-issues in the appropriate table location
6. Have updated sub-issue counts if new issues were added (e.g., "Direct Sub-Issues: 11" instead of "10")
7. Have recalculated Tracked Issues / Open / Closed counts in the Status Dashboard table
8. Have no issue appearing in both the main hierarchy table and the Additional Issues tables — any such duplicate rows must have been removed from the Additional Issues section
9. Maintain all other content unchanged

## Final Reminder

🚫 **NO GITHUB ISSUES SHOULD BE MODIFIED IN ANY WAY** 🚫

This is a **READ-ONLY** operation. You are only:
- Reading status from GitHub
- Updating the local markdown file

You are **NEVER**:
- Writing to GitHub
- Modifying GitHub issues
- Creating GitHub issues
- Commenting on GitHub issues