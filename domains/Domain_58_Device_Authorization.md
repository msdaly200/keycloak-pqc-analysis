# Domain 58 — Device Authorization Grant — Token Signing

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

Device Authorization Grant (RFC 8628, OAuth 2.0 Device Flow) — signs access/refresh tokens after device code polling completes.

## Gap

**None** — inherits SPI-driven signing path.

**File:** `services/src/main/java/org/keycloak/protocol/oidc/grants/device/DeviceGrantType.java`

## Current PQC State

**PENDING PROVIDERS**

Uses `TokenManager` (**Domain 1** path) for token signing. Algorithm-agnostic, will support ML-DSA automatically.

## Required Changes

**None.**

## Dependencies

**Domain 1** — Token routing and signing  
**GAP-15** — ML-DSA SignatureProvider ([#48824](https://github.com/keycloak/keycloak/issues/48824))

## GitHub Issue Status

No dedicated issue needed. Covered by GAP-15.

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
