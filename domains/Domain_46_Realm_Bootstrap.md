# Domain 46 — Default Realm Key Providers (Bootstrap)

[← Back to PQC Overview](pqc_overview.html)

## What is this?

Automatic realm key provider creation when a new realm is created. Every realm gets default signing and encryption keys without operator intervention.

## Gap

**Hardcoded RSA-only bootstrap (no ML-DSA or ML-KEM).**

**File:** `server-spi-private/src/main/java/org/keycloak/models/utils/DefaultKeyProviders.java`

**Current bootstrap logic:**

```java
if (!hasProvider(realm, "rsa-generated")) {
    createRsaKeyProvider("rsa-generated", realm);
}

if (!hasProvider(realm, "rsa-enc-generated")) {
    createRsaEncKeyProvider("rsa-enc-generated", realm);
}
```

**The problem:**

Every new realm gets:
- ✅ RSA signing keys (`rsa-generated`)
- ✅ RSA encryption keys (`rsa-enc-generated`, RSA-OAEP)
- ❌ NO ML-DSA signing keys
- ❌ NO ML-KEM encryption keys

Even after GAP-15 is fixed (ML-DSA key generation provider exists), **new realms will not use ML-DSA by default** unless `DefaultKeyProviders` is updated.

## Current PQC State

**BLOCKED**

## Required Changes

**Add ML-DSA and ML-KEM bootstrap (after GAP-15 is fixed):**

```java
// After RSA providers:
if (!hasProvider(realm, "ml-dsa-generated")) {
    createMLDsaKeyProvider("ml-dsa-generated", realm);
}

if (!hasProvider(realm, "ml-kem-generated")) {
    createMLKemKeyProvider("ml-kem-generated", realm);
}
```

**Policy decision needed:** Should new realms get:
1. **Only PQC keys** (ML-DSA + ML-KEM)?
2. **Both classical and PQC keys** (RSA + ML-DSA + ML-KEM)?
3. **Configurable** via system property?

## Dependencies

**GAP-15** — ML-DSA key generation ([#48824](https://github.com/keycloak/keycloak/issues/48824))  
**Domains 29, 32** — ML-KEM key generation (not yet tracked)

## GitHub Issue Status

Covered by [#48824](https://github.com/keycloak/keycloak/issues/48824)

## What this means for operators

**Today:**

Every new realm automatically gets RSA keys. Operators who want ML-DSA must manually add key providers.

**When fixed:**

New realms will get ML-DSA (and possibly ML-KEM) keys automatically, reducing migration friction.

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
