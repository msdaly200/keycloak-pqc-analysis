# Domain 46 — Default Realm Key Providers (Bootstrap)

## What is this?

Automatic realm key provider creation when a new realm is created.

## Gap

**Hardcoded RSA-only bootstrap (no ML-DSA or ML-KEM).**

`createProviders()` hardcodes `rsa-generated` (SIG) and `rsa-enc-generated` (ENC, `Algorithm.RSA_OAEP`) as the only default providers. New realms will never get an ML-DSA or ML-KEM key by default until this bootstrap logic is extended. Even after ML-DSA `GeneratedAKPKeyProviderFactory` exists (GAP-15), new realms will not use it unless `DefaultKeyProviders` is updated.

**Location:** `server-spi-private/.../utils/DefaultKeyProviders.java`

## Current PQC State

**BLOCKED**

## Required Changes

Add conditional bootstrap of an ML-DSA SIG provider and an ML-KEM ENC provider in `createProviders()`, guarded by the same `hasProvider()` check pattern. Depends on GAP-15 / ML-DSA key provider work being completed first.

## GitHub Issue Status

Covered by [#48824](https://github.com/keycloak/keycloak/issues/48824) (depends on ML-DSA key generation).

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
