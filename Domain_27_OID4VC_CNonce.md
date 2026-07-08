# Domain 27 — OID4VC c_nonce JWT Signing

## What is this?

The `c_nonce` (challenge nonce) is a JWT signed by Keycloak to prevent replay attacks in OID4VC credential issuance.

## Gap

**Hardcoded algorithm selection (GAP-18).**

**File:** `JwtCNonceHandler.selectSigningKey()`
Hardcodes ES256 then RS256. Independent of `Constants.DEFAULT_SIGNATURE_ALGORITHM`. c_nonce JWTs will not use an active ML-DSA key even after providers exist.

## Current PQC State

**BLOCKED**

## Required Changes

Replace hardcoded algorithm selection with a configurable or SPI-driven lookup so ML-DSA keys can be used for c_nonce signing.

## GitHub Issue Status

**GAP-18** needs a new issue (low priority).

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
