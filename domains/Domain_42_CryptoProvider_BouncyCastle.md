# Domain 42 — CryptoProvider SPI — BouncyCastle Default Backend

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

Default JCE cryptography provider using BouncyCastle (`bcprov-jdk18on`). Provides low-level crypto primitives (key generation, signature algorithms, certificate handling) for all non-FIPS Keycloak deployments.

## Gap

**No Maven enforcer minimum-version constraint (GAP-19).**

**Current version:** Keycloak uses `bcprov-jdk18on` **1.85.2** (inherited from Quarkus platform BOM 3.40.0.CR1 in `pom.xml`).

**PQC support:**
- ML-DSA and ML-KEM support **first appeared in BC 1.78** (August 2024)
- Version 1.85.2 confirmed to contain full ML-DSA and ML-KEM implementations

**The problem:**

No Maven Enforcer minimum-version rule enforces `bcprov-jdk18on ≥ 1.78`. A downstream build using an older BOM (e.g., 1.70) could silently lose ML-DSA support **without a build-time failure**.

**File:** `crypto/default/src/main/java/org/keycloak/crypto/def/DefaultCryptoProvider.java`

## Current PQC State

**PARTIAL**

## Required Changes

Add Maven Enforcer rule to root `pom.xml`:

```xml
<requireUpperBoundDeps>
  <excludes>
    <exclude>org.bouncycastle:bcprov-jdk18on</exclude>
  </excludes>
</requireUpperBoundDeps>
<bannedDependencies>
  <rules>
    <rule>
      <groupId>org.bouncycastle</groupId>
      <artifactId>bcprov-jdk18on</artifactId>
      <version>[,1.78)</version>
      <message>bcprov-jdk18on must be ≥ 1.78 for ML-DSA/ML-KEM support</message>
    </rule>
  </rules>
</bannedDependencies>
```

## Dependencies

**BouncyCastle ≥ 1.78** for ML-DSA/ML-KEM support (already met)

## GitHub Issue Status

**GAP-19** — needs GitHub issue under [#43690](https://github.com/keycloak/keycloak/issues/43690)

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
