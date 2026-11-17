# Domain 55 — Default Trust Identity Provider

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

Generic trust material broker — resolves public keys from JWKS URL or inline JWKS for verifying federated JWT assertions.

## Gap

**GAP-32 (x5c path only)** — the JWKS/`kid` path inherits the SPI-driven verification path.

**Files:**
- `services/src/main/java/org/keycloak/broker/trust/DefaultTrustIdentityProvider.java`
- `services/src/main/java/org/keycloak/broker/trust/DefaultTrustIdentityProviderConfig.java`
- `services/src/main/java/org/keycloak/broker/trust/DefaultTrustIdentityProviderFactory.java`

## Current PQC State

**PARTIAL**

The JWKS/`kid` resolution path uses `JWKSUtils.getKeyWrappersForUse()`, which already handles AKP key types and will work automatically once ML-DSA providers exist. However, the `x5c` path (`TrustMaterialIdentityProvider.validateX509Chain()` → `TrustMaterialResolver` → `X509CertificateChainValidator.toJwk()`) converts the leaf key for RSA and EC only, so ML-DSA leaf certificates are rejected (GAP-32).

## Required Changes

**GAP-32:** Add AKP/OKP handling to `X509CertificateChainValidator.toJwk()` so `validateX509Chain()` accepts ML-DSA leaf certificates.

## Dependencies

**GAP-15** — ML-DSA SignatureProvider ([#48824](https://github.com/keycloak/keycloak/issues/48824))

**GAP-32** — x5c leaf-key conversion (`X509CertificateChainValidator.toJwk()`)

## GitHub Issue Status

JWKS path: covered by GAP-15. x5c path: tracked as GAP-32 (new issue needed).

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
