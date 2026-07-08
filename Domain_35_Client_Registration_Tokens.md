# Domain 35 — Dynamic Client Registration Tokens

## What is this?

Registration and initial access tokens used in OAuth2 Dynamic Client Registration use symmetric HS512 signing.

## Gap

**RS256 special-case logic (GAP-11, GAP-12).**

RS256 special-case logic in `DescriptionConverter.java` (line 416) omits `id_token_signed_response_alg` only for RS256; no equivalent treatment for ML-DSA. Migration risk for `DEFAULT_SIGNATURE_ALGORITHM` fallback.

## Current PQC State

**SAFE** (registration tokens use HS512 — quantum-safe)

However, GAP-11 and GAP-12 relate to special-case handling that needs updating.

## Required Changes

Update RS256 special-case logic in `DescriptionConverter.java` to handle ML-DSA as a valid realm default. Document migration risk for `DEFAULT_SIGNATURE_ALGORITHM` fallback.

## GitHub Issue Status

**GAP-11** and **GAP-12** — document in [#48823](https://github.com/keycloak/keycloak/issues/48823) (operator guidance).

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
