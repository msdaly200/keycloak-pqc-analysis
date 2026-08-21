# Domain 52 — Docker Registry — Self-Signed Certificate Generation

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

RSA-2048 key pair generation for Docker registry TLS certificate. Used by Docker compose test/development installations.

## Gap

**Hardcoded RSA-2048 key generation.**

**File:** `services/src/main/java/org/keycloak/protocol/docker/installation/compose/DockerComposeCertsDirectory.java`

**Current implementation:**

```java
CryptoIntegration.getProvider().getKeyPairGen(KeyType.RSA);
keyGen.initialize(2048);
```

**The problem:**

- Hardcoded to `KeyType.RSA` and `2048` bit size
- No algorithm parameter or configuration option
- Docker registry TLS certificates will always be RSA-2048

**Impact:** LOW — Docker compose is a **developer/test installation path**, not production.

## Current PQC State

**BLOCKED**

## Required Changes

**Add algorithm parameter (low priority):**

```java
public void generateCerts(String algorithm, int keySize) {
    KeyType keyType = KeyType.valueOf(algorithm);
    CryptoIntegration.getProvider().getKeyPairGen(keyType);
    keyGen.initialize(keySize);
}
```

**Priority:** LOW — affects only test/dev installations.

## Dependencies

**ML-DSA certificate generation support**

## GitHub Issue Status

Needs GitHub issue under [#43690](https://github.com/keycloak/keycloak/issues/43690) with LOW priority

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
