# Domain 35 — Dynamic Client Registration Tokens

[← Back to PQC Overview](pqc_overview.html)

## What is this?

Registration and initial access tokens used in OAuth2 Dynamic Client Registration are signed using `TokenCategory.INTERNAL`, which routes to symmetric HS512 via **Domain 1** token routing logic.

## Gap

**None.**

Registration tokens themselves use **HS512** (symmetric, quantum-safe) via `TokenCategory.INTERNAL` routing. No PQC gap for this domain.

**Note:** GAP-11 and GAP-12 relate to RS256 special-case logic in client registration **metadata** handling (`DescriptionConverter.java`), not the registration tokens themselves. Those gaps affect how clients declare their preferred algorithms during registration.

## Current PQC State

**SAFE** (registration tokens use HS512 — quantum-safe)

However, GAP-11 and GAP-12 relate to special-case handling that needs updating.

## Required Changes

Update RS256 special-case logic in `DescriptionConverter.java` to handle ML-DSA as a valid realm default. Document migration risk for `DEFAULT_SIGNATURE_ALGORITHM` fallback.

## Dependencies

**Domain 1** — Token routing logic (`TokenCategory.INTERNAL` → HS512)

## GitHub Issue Status

No dedicated issue needed. Registration tokens are quantum-safe.

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
