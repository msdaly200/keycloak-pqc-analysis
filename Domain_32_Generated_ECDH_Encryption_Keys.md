# Domain 32 — Generated ECDH Encryption Key Provider

## What is this?

Realm key providers generate ECDH key pairs for encrypting tokens (JWE with ECDH-ES key agreement).

## Gap

**No ML-KEM equivalent key generation provider.**

GAP-15 only addresses ML-DSA SIG side. ML-KEM ENC key generation is untracked.

## Current PQC State

**BLOCKED**

## Required Changes

Implement ML-KEM ENC key generation provider. Needs explicit tracking (same as Domain 29).

## GitHub Issue Status

New issue needed for ML-KEM key generation (separate from GAP-15).

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
