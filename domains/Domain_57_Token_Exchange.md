# Domain 57 — Token Exchange — Verification & Signing

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

Token Exchange Grant (RFC 8693) — verifies inbound subject/actor tokens and signs exchanged output tokens.

## Gap

**None** — inherits SPI-driven verification and signing paths.

**Files:**
- `services/src/main/java/org/keycloak/protocol/oidc/grants/TokenExchangeGrantType.java`
- `services/src/main/java/org/keycloak/protocol/oidc/tokenexchange/StandardTokenExchangeProvider.java`
- `services/src/main/java/org/keycloak/protocol/oidc/tokenexchange/AbstractTokenExchangeProvider.java`

## Current PQC State

**PENDING PROVIDERS**

Verification uses `SignatureProvider` SPI, signing uses `TokenManager` (**Domain 1** path). Both are algorithm-agnostic.

## Required Changes

**None.**

## Dependencies

**Domain 1** — Token routing and signing  
**GAP-15** — ML-DSA SignatureProvider ([#48824](https://github.com/keycloak/keycloak/issues/48824))

## GitHub Issue Status

No dedicated issue needed. Covered by GAP-15.

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
