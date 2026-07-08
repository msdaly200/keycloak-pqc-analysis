# Domain 45 — JWK Serialisation & Thumbprint

## What is this?

JWK encoding/decoding and RFC 7638 thumbprint computation for all key types.

## Gap

**AKP thumbprint throws UnsupportedOperationException (GAP-1).**

ML-DSA JWK encode/decode is wired (`AKPUtils`, `JWKBuilder.akp()`, `JWKParser` AKP parsing all exist). JWK thumbprint computation for AKP keys is missing — `JWKSUtils.computeThumbprint()` throws `UnsupportedOperationException` for AKP keys. Directly breaks DPoP and attestation flows for ML-DSA keys.

**Location:** `core/.../util/JWKSUtils.java`

## Current PQC State

**PARTIAL**

## Required Changes

Add AKP to `JWK_THUMBPRINT_REQUIRED_MEMBERS` in `JWKSUtils` and implement `computeThumbprint()` for AKP JWKs. High-priority blocker affecting DPoP (Domain 15) and attestation-based auth (Domain 12).

## GitHub Issue Status

**GAP-1** needs new issue (CRITICAL priority).

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
