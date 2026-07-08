# Domain 28 — Generated RSA Signing Key Provider

## What is this?

Realm key providers generate RSA key pairs for signing tokens. This is the default key generation mechanism when a realm is created.

## Gap

**No ML-DSA key generation provider exists (GAP-15).**

No `GeneratedAKPKeyProviderFactory` equivalent exists. `DefaultKeyManager.createFallbackKeys()` mechanism does not know about ML-DSA. `CryptoProvider.getKeyPairGen(String algorithm)` will need to support ML-DSA names.

## Current PQC State

**BLOCKED**

## Required Changes

Implement `GeneratedAKPKeyProviderFactory` (ML-DSA key generation). Update `DefaultKeyManager` fallback mechanism.

## GitHub Issue Status

**GAP-15** tracked under [#48824](https://github.com/keycloak/keycloak/issues/48824).

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
