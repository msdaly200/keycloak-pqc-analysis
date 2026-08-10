# Domain 45 — JWK Serialisation & Thumbprint

[← Back to PQC Overview](pqc_overview.html)

## What is this?

JWK encoding/decoding and RFC 7638 thumbprint computation for all key types. Used by DPoP, attestation-based auth, and client JWKS handling.

## Gap

**AKP thumbprint throws UnsupportedOperationException (GAP-1).**

**File:** `core/src/main/java/org/keycloak/util/JWKSUtils.java:52-57`

**Current state:**

```java
private static final Map<String, String[]> JWK_THUMBPRINT_REQUIRED_MEMBERS = new HashMap<>();
static {
    JWK_THUMBPRINT_REQUIRED_MEMBERS.put(KeyType.RSA, new String[] { RSAPublicJWK.MODULUS, RSAPublicJWK.PUBLIC_EXPONENT });
    JWK_THUMBPRINT_REQUIRED_MEMBERS.put(KeyType.EC, new String[] { ECPublicJWK.CRV, ECPublicJWK.X, ECPublicJWK.Y });
    JWK_THUMBPRINT_REQUIRED_MEMBERS.put(KeyType.OKP, new String[] { OKPPublicJWK.CRV, OKPPublicJWK.X });
    // NO AKP ENTRY
}
```

**What works:**
- ✅ ML-DSA JWK **encoding** (`JWKBuilder.akp()`)
- ✅ ML-DSA JWK **decoding** (`JWKParser` AKP parsing)
- ✅ `AKPUtils` helper class exists

**What's missing:**
- ❌ AKP in `JWK_THUMBPRINT_REQUIRED_MEMBERS` map
- ❌ `computeThumbprint()` for AKP keys → throws `UnsupportedOperationException`

**Impact:** **CRITICAL** — directly breaks:
- **Domain 15** — DPoP (requires JWK thumbprint for `jkt` claim)
- **Domain 12** — Attestation-based auth (requires JWK thumbprint)

## Current PQC State

**PARTIAL**

## Required Changes

**Add AKP support to thumbprint computation:**

```java
// In static initializer:
JWK_THUMBPRINT_REQUIRED_MEMBERS.put(KeyType.AKP, new String[] { 
    AKPPublicJWK.CRV,  // algorithm name (ML-DSA-44/65/87)
    AKPPublicJWK.X     // public key value
});
```

Per RFC 7638 and draft-ietf-cose-dilithium, AKP thumbprint uses `{"crv", "kty", "x"}` in canonical JSON order.

## Dependencies

**None** — standalone fix, no external dependencies

## GitHub Issue Status

**GAP-1** — needs GitHub issue under [#43690](https://github.com/keycloak/keycloak/issues/43690) with **CRITICAL** priority

## Related Domains

- **Domain 15** — DPoP (blocked on this fix)
- **Domain 12** — Attestation-based auth (blocked on this fix)

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
