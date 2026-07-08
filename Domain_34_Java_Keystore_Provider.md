# Domain 34 — Java Keystore Key Provider (PKCS12 / BCFKS)

## What is this?

Allows operators to import keys from Java keystores (PKCS12 or BCFKS format) into Keycloak realms.

## Gap

**JavaKeystoreKeyProviderFactory only supports RSA, EC, OKP (GAP-5, GAP-16).**

BCFKS and PKCS12 can store PQC keys, but `JavaKeystoreKeyProviderFactory` only supports RSA, EC, and OKP/EdDSA import paths. ML-DSA not exposed in admin UI option list built by `mergedAlgorithmProperties()`. Both provider logic and configuration surface need extension.

## Current PQC State

**BLOCKED**

## Required Changes

Extend `JavaKeystoreKeyProviderFactory` import logic and `mergedAlgorithmProperties()` to expose ML-DSA.

## GitHub Issue Status

**GAP-5** and **GAP-16** need new issue.

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
