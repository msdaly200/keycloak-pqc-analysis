# Domain 47 — Client Public Key Loader (JWKS URL & Stored Cert)

## What is this?

Loads client public keys from JWKS URL, inline JWKS string, or stored X.509 certificate.

## Gap

None — JWKS path already handles AKP via `JWKSUtils.getKeyWrappersForUse()`. X.509 path is algorithm-agnostic via `CertPathBuilder`.

## Current PQC State

**PARTIAL**

JWKS URL / JWKS string path calls `JWKSUtils.getKeyWrappersForUse()`, which already handles AKP (ML-DSA) key types. The stored-certificate path uses X.509 cert parsing and is algorithm-agnostic via `CertPathBuilder`. No explicit ML-DSA block here, but end-to-end verification still depends on ML-DSA `SignatureProviderFactory` existing.

## Required Changes

None — will work automatically once providers exist.

## GitHub Issue Status

Covered by [#48824](https://github.com/keycloak/keycloak/issues/48824).

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
