# Domain 51 — Client SDK — DPoP Proof Generation

## What is this?

Client-side DPoP proof JWT generation from the adapter/SDK.

## Gap

**Convenience method hardcoded to RSA.**

The generic `generateSignedDPoPProof(..., KeyWrapper, ...)` method is algorithm-agnostic and will support ML-DSA once providers exist. However, the convenience method `generateRsaSignedDPoPProof()` is hardcoded to RSA and includes an explicit `TODO` noting EC and EdDSA equivalents are missing. No ML-DSA convenience path exists.

**Location:** `core/.../util/DPoPGenerator.java`

## Current PQC State

**PARTIAL**

## Required Changes

Add ML-DSA convenience method or update callers to use the generic `KeyWrapper` path. Low priority if callers already use the generic path.

## GitHub Issue Status

New issue needed (LOW priority).

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
