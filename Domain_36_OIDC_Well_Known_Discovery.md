# Domain 36 — OIDC Well-Known Discovery — Algorithm Advertisement

## What is this?

OIDC `.well-known/openid-configuration` endpoint advertises supported algorithms to clients.

## Gap

**Hardcoded DEFAULT_CLIENT_AUTH_SIGNING_ALG_VALUES_SUPPORTED (GAP-23).**

Main token signing algorithm lists (e.g. `id_token_signing_alg_values_supported`) are dynamically populated from registered providers — will auto-include ML-DSA once providers exist. However, `DEFAULT_CLIENT_AUTH_SIGNING_ALG_VALUES_SUPPORTED` is a static constant hardcoded to RS256. CIBA signing algorithm support will work in practice but not be advertised to conformant clients.

**Location:** `protocol/oidc/OIDCWellKnownProvider.java`

## Current PQC State

**PARTIAL**

## Required Changes

Replace static `DEFAULT_CLIENT_AUTH_SIGNING_ALG_VALUES_SUPPORTED` constant with a dynamic lookup merging registered asymmetric `SignatureProvider` algorithm names. Same pattern as `id_token_signing_alg_values_supported`.

## GitHub Issue Status

**GAP-23** needs new issue.

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
