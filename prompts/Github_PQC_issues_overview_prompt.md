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
- All direct sub-issues (10 total) and their nested sub-issues are shown in a **single table**
- Table columns: Issue | Title | Status | Type | Progress | Sub-Issues
- Sub-issues are indented using `↳` arrows
- Third-level sub-issues are indented with `&nbsp;&nbsp;&nbsp;&nbsp;↳` (4 non-breaking spaces)
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
   - Keep the exact table format with proper indentation (`↳` for level 2, `&nbsp;&nbsp;&nbsp;&nbsp;↳` for level 3)
   - Keep all sub-issue counts in the Sub-Issues column (**bold** for parents, (parentheses) for nested)
   - Keep all issue titles, links, and hierarchy unchanged
   - Only update status badges, progress counts, sub-issue counts, and the Last Updated date
   - When adding new issues, match the formatting and indentation of existing entries

6. **Status badge format**:
   - OPEN (in-progress): `<span style="color: #d97706; font-weight: bold;">🔵 OPEN</span>`
   - OPEN (ready/available): `<span style="color: #059669; font-weight: bold;">🟢 OPEN</span>`
   - CLOSED: `<span style="color: #7c3aed; font-weight: bold;">🟣 CLOSED</span>`

7. **Output the updated file** using the Edit tool to update only the changed status/progress values and add any new issues

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

**⚠️ IMPORTANT:** The number of direct sub-issues under #43690 may change over time. You MUST:
1. First read the current `gh_issues/Github_PQC_issues_overview.md` file to see the current list of sub-issues
2. Check GitHub to see if any new sub-issues have been added or linked to #43690
3. Update the document to reflect the current actual count and list of sub-issues
4. Update the "Direct Sub-Issues: X" count in the document header for #43690
5. If new sub-issues are found, add them to the table in the appropriate location

### Direct Sub-Issues (Current known list - verify against GitHub)
- #43691 - Hybrid key exchange in TLS 1.3 (0 sub-issues)
- #45168 - Review what is needed for PQC readiness (has 12 sub-issues)
- #46333 - Audit and Upgrade Cryptographic Defaults (has 1 sub-issue)
- #48821 - PQC support for OAuth 2.0 and OpenID Connect (has 4 sub-issues)
- #49865 - Milestone for cookies to be PQC ready (has 2 sub-issues)
- #50084 - PQC support for WebAuthn/Passkeys (has 2 sub-issues)
- #50292 - PQC support for SAML 2.0 (has 2 sub-issues)
- #50680 - Add test coverage for truststore loading with PQC certificates (0 sub-issues)
- #50679 - Support loading ML-DSA keys from Java keystores (0 sub-issues)
- #50678 - Add ML-DSA JCE algorithm mapping to JavaAlgorithm (0 sub-issues)

### Under #45168 (12 sub-issues)
- #48819, #48820, #48822, #48823, #48824, #48825, #48826, #48827, #48828, #48829, #48830, #49851

### Under #46333 (1 sub-issue)
- #46336

### Under #48821 (4 sub-issues)
- #43693, #43692 (has 5 nested sub-issues), #50299, #50304

### Under #43692 (5 nested sub-issues)
- #44141, #44142, #43684, #44143, #44144

### Under #49865 (2 sub-issues)
- #49858, #49860

### Under #50084 (2 sub-issues)
- #50085, #50086

### Under #50292 (2 sub-issues)
- #50294, #50295

### Additional Issues (not in main hierarchy)
- #50674, #50675, #50676, #50677, #50678, #50679, #50680, #49968, #50355, #48415

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
2. Have current OPEN/CLOSED status for all issues (read from GitHub)
3. Have current progress counts where applicable
4. Include any newly discovered sub-issues in the appropriate table location
5. Have updated sub-issue counts if new issues were added (e.g., "Direct Sub-Issues: 11" instead of "10")
6. Maintain all other content unchanged

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