# Domain 55 — Default Trust Identity Provider

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

Generic trust material broker — resolves public keys from JWKS URL or inline JWKS for verifying federated JWT assertions.

## Gap

**None** — inherits SPI-driven verification path.

**Files:**
- `services/src/main/java/org/keycloak/broker/trust/DefaultTrustIdentityProvider.java`
- `services/src/main/java/org/keycloak/broker/trust/DefaultTrustIdentityProviderConfig.java`
- `services/src/main/java/org/keycloak/broker/trust/DefaultTrustIdentityProviderFactory.java`

## Current PQC State

**PENDING PROVIDERS**

Uses `JWKSUtils.getKeyWrappersForUse()` which already handles AKP key types. Will work automatically once ML-DSA providers exist.

## Required Changes

**None.**

## Dependencies

**GAP-15** — ML-DSA SignatureProvider ([#48824](https://github.com/keycloak/keycloak/issues/48824))

## GitHub Issue Status

No dedicated issue needed. Covered by GAP-15.

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
