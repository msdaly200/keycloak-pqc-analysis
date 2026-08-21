# Domain 54 — Federated JWT Client Authentication

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

JWT-based client authentication in federated/brokered scenarios (SPIFFE, external IdPs). Verifies client assertions issued by external identity providers.

## Gap

**None** — inherits SPI-driven verification path.

**Files:**
- `services/src/main/java/org/keycloak/authentication/authenticators/client/FederatedJWTClientAuthenticator.java`
- `services/src/main/java/org/keycloak/authentication/authenticators/client/DefaultClientAssertionStrategy.java`
- `services/src/main/java/org/keycloak/broker/spiffe/SpiffeClientAssertionStrategy.java`

## Current PQC State

**PENDING PROVIDERS**

Algorithm-agnostic verification via `ClientSignatureVerifierProvider` SPI. Will support ML-DSA automatically once providers exist.

## Required Changes

**None.**

## Dependencies

**GAP-15** — ML-DSA SignatureProvider ([#48824](https://github.com/keycloak/keycloak/issues/48824))

## GitHub Issue Status

No dedicated issue needed. Covered by GAP-15.

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
