# Domain 42 — CryptoProvider SPI — BouncyCastle Default Backend

## What is this?

Default JCE cryptography provider using BouncyCastle (bcprov-jdk18on).

## Gap

**No Maven enforcer minimum-version constraint (GAP-19).**

`bcprov-jdk18on` 1.84 (in use) confirmed to contain ML-DSA and ML-KEM support. ML-DSA classes first appeared in version 1.78. However, no Maven Enforcer minimum-version rule exists. A downstream build with an older BOM could silently lose ML-DSA support without a build-time failure.

**Location:** `crypto/default/.../def/DefaultCryptoProvider.java`

## Current PQC State

**PARTIAL**

## Required Changes

Add Maven enforcer minimum-version constraint for `bcprov-jdk18on` (≥ 1.78).

## GitHub Issue Status

**GAP-19** needs new issue.

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
