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

All updates are made **ONLY** to the local file: `/Users/mariedaly/projects/pqc_analysis/Github_PQC_issues_overview.md`

---

## Task
Update the file `/Users/mariedaly/projects/pqc_analysis/Github_PQC_issues_overview.md` with the latest status from GitHub for all Keycloak PQC-related issues.

## Instructions

1. **Read the current file** to understand the existing structure and issue list

2. **Fetch latest GitHub status** for all issues mentioned in the document:
   - Use `gh issue view <issue-number> --repo keycloak/keycloak --json number,title,state,closedAt` for each issue
   - Get progress counts for issues with sub-issues by checking the GitHub UI tracking
   - Note: GitHub's tracking data may require GraphQL API or parsing the issue body for task checkboxes

3. **Update the following fields**:
   - **Status** (OPEN/CLOSED) for each issue
   - **Progress** counts (e.g., "4 of 12", "0 of 7") where applicable
   - **Last Updated** date at the top of the document (use current date in format YYYY-MM-DD)

4. **Preserve everything else**:
   - Keep the exact same structure and hierarchy
   - Keep all descriptions, domains, GAPs, and analysis sections unchanged
   - Only update status and date fields

5. **Output the updated file** by rewriting it with the Edit tool

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

### Top-Level (under #43690)
- #43691 - Hybrid key exchange in TLS 1.3
- #45168 - Review what is needed for PQC readiness (has 12 sub-issues)
- #46333 - Audit and Upgrade Cryptographic Defaults (has 1 sub-issue)
- #48821 - PQC support for OAuth 2.0 and OpenID Connect (has 4 sub-issues)
- #49865 - Milestone for cookies to be PQC ready (has 2 sub-issues)
- #50084 - PQC support for WebAuthn/Passkeys (has 2 sub-issues)
- #50292 - PQC support for SAML 2.0 (has 2 sub-issues)

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

For each issue, update the status line following this pattern:

**Before:**
```markdown
**Status:** OPEN | **Type:** milestone | **Progress:** 0 of 7
```

**After (if status changed to CLOSED):**
```markdown
**Status:** CLOSED | **Type:** milestone | **Progress:** 7 of 7
```

And update the top-level date:

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
4. Maintain all other content unchanged

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