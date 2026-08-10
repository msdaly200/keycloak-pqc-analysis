# Keeping `pqc_overview.html` Up to Date

This document explains how to reassess and update the PQC readiness analysis when the Keycloak codebase changes. There are two steps/prompts:

---

## Step 1 — Run the reassessment prompt

Open [`prompts/REASSESS_PQC_READINESS_PROMPT.md`](prompts/REASSESS_PQC_READINESS_PROMPT.md) and run it with an AI agent.

The prompt verifies every claim in `pqc_overview.html` against the live Keycloak source — file paths, line numbers, hardcoded algorithm usages, PQC states, GAP entries, and the "Files Evaluated" section. It also sweeps for newly added files or features that may introduce new crypto gaps.

The agent writes its output to a dated findings file:

```
findings/pqc_overview_review_findings_YYYY-MM-DD.md
```

> **The agent does not modify `pqc_overview.html`.** It only produces the findings report.



Read the findings file, paying particular attention to **Part 1** (errors) and **Part 7** (summary of required changes).

Verify that each flagged item is a genuine issue before proceeding. Not every finding will require a change — some may be false positives.

---

## Step 2 — Apply approved changes

Once you are happy with the findings, run [`prompts/APPLY_PQC_REVIEW_FINDINGS_PROMPT.md`](prompts/APPLY_PQC_REVIEW_FINDINGS_PROMPT.md) and specify the findings file to apply from. The prompt will make only the changes you have reviewed.

```
Apply the findings from findings/pqc_overview_review_findings_YYYY-MM-DD.md
following the instructions in prompts/APPLY_PQC_REVIEW_FINDINGS_PROMPT.md
```
