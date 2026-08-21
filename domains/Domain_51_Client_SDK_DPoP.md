# Domain 51 — Client SDK — DPoP Proof Generation

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

Client-side DPoP proof JWT generation from the Keycloak adapter/SDK. Used by clients (not the server) to bind tokens to specific keys.

## Gap

**Convenience method hardcoded to RSA.**

**File:** `core/src/main/java/org/keycloak/util/DPoPGenerator.java`

**Current state:**

```java
// Generic method (algorithm-agnostic):
public static String generateSignedDPoPProof(..., KeyWrapper keyWrapper, ...) {
    // ✅ Works for any algorithm via KeyWrapper
}

// Convenience method (RSA only):
public static String generateRsaSignedDPoPProof(..., PrivateKey rsaPrivateKey, ...) {
    // ❌ Hardcoded to RSA
    // TODO: Add EC, EdDSA equivalents
}
```

**The problem:**

- Generic `KeyWrapper` path is algorithm-agnostic and **will work for ML-DSA**
- Convenience method is RSA-only
- Code includes `TODO` noting EC and EdDSA are missing too

**Impact:** LOW priority — if client SDK callers use the generic `KeyWrapper` path, ML-DSA will work automatically.

## Current PQC State

**PARTIAL**

## Required Changes

**Option 1:** Add ML-DSA convenience method

```java
public static String generateMLDsaSignedDPoPProof(..., PrivateKey mldsaPrivateKey, ...) {
    // Similar to generateRsaSignedDPoPProof
}
```

**Option 2 (recommended):** Update callers to use generic `KeyWrapper` path, which already supports all algorithms.

**Priority:** LOW — only needed if SDK convenience API is important for developer experience.

## Dependencies

**None** — generic path already works

## GitHub Issue Status

Needs GitHub issue under [#43690](https://github.com/keycloak/keycloak/issues/43690) with LOW priority (or close TODO if generic path is sufficient)

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
