# Prompt: Update GitHub PQC Issues Overview

## Context

This prompt is used to update the `Github_PQC_issues_overview.md` file with the latest status of PQC-related GitHub issues from the Keycloak repository.

## Prerequisites

Before using this prompt:

1. Navigate to the [Keycloak PQC Issues page](https://github.com/keycloak/keycloak/issues?q=is%3Aissue+label%3Apqc)
2. Take screenshots of the complete issue list showing:
   - Issue numbers and titles
   - Status (Open/Closed)
   - Any progress indicators or sub-issue counts
   - The complete hierarchy if visible

You can also check specific parent issues to see their sub-issue structure:
- [#43690 - PQC readiness (root)](https://github.com/keycloak/keycloak/issues/43690)
- [#45168 - Review PQC readiness](https://github.com/keycloak/keycloak/issues/45168)
- [#48821 - OAuth 2.0/OIDC PQC](https://github.com/keycloak/keycloak/issues/48821)
- [#43692 - ML-DSA support](https://github.com/keycloak/keycloak/issues/43692)
- [#50292 - SAML PQC](https://github.com/keycloak/keycloak/issues/50292)

## Prompt

```
I have screenshots of the current PQC-related GitHub issues from the Keycloak repository. 

Please update the `Github_PQC_issues_overview.md` file with the latest information from these screenshots.

Specifically, please update:

1. **Issue Status** - Change any issues from OPEN to CLOSED (or vice versa) based on the current status
2. **Progress Metrics** - Update the "Progress" column (e.g., "1 of 5", "2 of 2") for parent issues with sub-issues
3. **New Issues** - Add any new PQC-related issues that have been created since the last update
4. **Issue Hierarchy** - Update parent-child relationships if they've changed
5. **Last Updated Date** - Update the date at the top of the file to today's date

Please maintain:
- The existing visual tree structure with proper indentation (6 spaces for level 1, 12 spaces for level 2)
- The color-coded status indicators (🔵 OPEN, 🟢 OPEN, 🟣 CLOSED)
- The table structure and formatting
- The "Domains Missing GitHub Issues" section (don't modify this unless I specifically ask)

After updating, please provide a summary of what changed:
- How many issues changed status
- How many new issues were added
- Any significant changes to the hierarchy
```

## Example Usage

**User message:**
```
I have screenshots of the current Keycloak PQC issues. [attach screenshots]

Please update the Github_PQC_issues_overview.md file with the latest information.
```

**Expected Response:**
Claude will:
1. Read the current `Github_PQC_issues_overview.md` file
2. Analyze the screenshots to extract current issue information
3. Update the file with new status, progress, and any new issues
4. Provide a summary of changes made

## Notes

- If you see new issues that aren't in the file, Claude will add them to the appropriate section
- If issues have been moved or re-parented, Claude will update the hierarchy
- The file uses HTML spans for color-coded status, so updates will maintain that format
- Progress metrics like "1 of 5" indicate completed sub-issues out of total sub-issues

## Frequency

Recommended update frequency:
- **Weekly** during active PQC development
- **Monthly** during maintenance phases
- **As needed** when you know significant issue changes have occurred