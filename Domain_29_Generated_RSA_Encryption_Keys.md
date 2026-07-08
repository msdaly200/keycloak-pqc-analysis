# Domain 29 — Generated RSA Encryption Key Provider

## What is this?

Realm key providers generate RSA key pairs for encrypting tokens (JWE encryption).

## Gap

**No ML-KEM key generation provider exists.**

No ML-KEM equivalent key generation provider exists. GAP-15 focuses on ML-DSA SIG side only; RSA ENC key provider equivalent for ML-KEM is not explicitly tracked.

## Current PQC State

**BLOCKED**

## Required Changes

Implement ML-KEM key generation provider (equivalent of `GeneratedRsaEncKeyProviderFactory` for ML-KEM). Needs to be explicitly tracked alongside GAP-15.

## GitHub Issue Status

New issue needed for ML-KEM key generation (separate from GAP-15 which covers ML-DSA).

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
