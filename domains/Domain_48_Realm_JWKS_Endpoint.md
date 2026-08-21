# Domain 48 — Realm JWKS Endpoint — Key Serialisation

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

Public JWKS endpoint (`/protocol/openid-connect/certs`) that exposes realm public keys. Clients fetch this to verify tokens and encrypt to the realm.

## Gap

**AKP keys return null and are silently omitted from JWKS.**

**File:** `services/src/main/java/org/keycloak/protocol/oidc/utils/JWKSServerUtils.java:59-65`

**Current code:**

```java
if (key.getType().equals(KeyType.RSA)) {
    return JWKBuilder.create().rs256(key.getPublicKey());
} else if (key.getType().equals(KeyType.EC)) {
    return JWKBuilder.create().ec(key.getPublicKey());
} else if (key.getType().equals(KeyType.OKP)) {
    return JWKBuilder.create().okp(key.getPublicKey());
}
// NO AKP CASE → returns null
```

**The problem:**

AKP (ML-DSA) keys fall through all branches and return `null`. These `null` values are filtered out before serialization, meaning **ML-DSA realm keys are silently omitted** from the public JWKS endpoint.

**Impact:** Clients will never discover the realm's ML-DSA keys, breaking token verification and encrypted request objects.

## Current PQC State

**BLOCKED**

## Required Changes

**Add AKP branch (one-line fix):**

```java
if (key.getType().equals(KeyType.RSA)) {
    return JWKBuilder.create().rs256(key.getPublicKey());
} else if (key.getType().equals(KeyType.EC)) {
    return JWKBuilder.create().ec(key.getPublicKey());
} else if (key.getType().equals(KeyType.OKP)) {
    return JWKBuilder.create().okp(key.getPublicKey());
} else if (key.getType().equals(KeyType.AKP)) {
    return JWKBuilder.create().akp(key.getPublicKey());  // ADD THIS
}
```

`JWKBuilder.akp()` already exists, so this is a straightforward addition.

## Dependencies

**JWKBuilder.akp()** — already exists (no blocker)

## GitHub Issue Status

Needs GitHub issue under [#43690](https://github.com/keycloak/keycloak/issues/43690) with **HIGH** priority

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
