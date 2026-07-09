---
description: Verify a PQC domain analysis for 100% accuracy against Keycloak source code
---

# Verify PQC Domain

You are verifying the Post-Quantum Cryptography (PQC) readiness analysis for a specific Keycloak domain.

## Context
- Analysis repo: `/Users/mariedaly/projects/pqc_analysis` (current directory)
- Keycloak source: `~/projects/keycloak`
- Overview file: `pqc_overview.html` (61 domains)
- Domain files: `Domain_<n>_<name>.md`
- GitHub issues: `Github_PQC_issues_overview.md`

## Your Task

The domain number to verify will be provided as an argument. You must:

1. **Read the current analysis** from `pqc_overview.html` and the corresponding `Domain_<n>_*.md` file
2. **Verify against Keycloak source code** at `~/projects/keycloak`:
   - Check that all listed files exist
   - Verify the asymmetric algorithms in use
   - Confirm the PQC state assessment is accurate
   - Validate the fix description is technically correct
3. **Cross-reference GitHub issues** in `Github_PQC_issues_overview.md`:
   - Find existing issues that cover this domain
   - Determine if new issue is needed
4. **Check domain file completeness**:
   - Verify backlink to `pqc_overview.html` exists
5. **Report findings** in this format:

## Domain <N> Verification Results

### ✅ Verified Accurate
[List confirmed correct items]

### ⚠️ Needs Correction  
[List errors found with specific corrections]

### 📋 GitHub Issue Status
- **Best fit issue**: [Most specific issue covering this domain]
- **Additional issues**: [Related issues]
- **New issue needed**: [Yes/No with justification]

### 🔗 Proposed Changes
[Specific changes to approve]

## Critical Constraints

⚠️ **GitHub READ-ONLY**: Never edit, modify, create, or comment on ANY GitHub issues. Only read issue status.

⚠️ **100% Accuracy Required**: The Keycloak team relies on this analysis. Verify every claim against source code.

⚠️ **Propose Before Editing**: Outline all changes for user approval. Wait for approval before updating files.

⚠️ **No Recaps**: User reviews diffs directly. Don't summarize changes after making them.
