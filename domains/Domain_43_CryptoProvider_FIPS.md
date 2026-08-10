# Domain 43 — CryptoProvider SPI — FIPS 140-2/3 Backend (BC-FIPS)

[← Back to PQC Overview](pqc_overview.html)

## What is this?

FIPS-mode cryptography provider using BouncyCastle FIPS (`bc-fips`). Used when Keycloak must operate in FIPS 140-2/140-3 validated mode for regulated environments (government, healthcare, finance).

## Gap

**BC-FIPS 2.1.2 lacks ML-DSA/ML-KEM (GAP-6).**

**Current version:** `bc-fips` **2.1.2** (pinned in root `pom.xml` line 78)

**PQC support status:**

Confirmed by JAR inspection: `bc-fips-2.1.2.jar` contains **only LMS** under `org.bouncycastle.crypto.internal.pqc/`. 

**No ML-DSA classes.**  
**No ML-KEM classes.**

**What's needed:**

BC-FIPS PQC support (FIPS 204/203) requires a forthcoming BC-FIPS 2.x release that achieves **NIST FIPS 140-3 validation** for ML-DSA and ML-KEM algorithms.

**Impact:** HARD BLOCKER for regulated deployments requiring FIPS mode. No workaround until BC-FIPS releases PQC-validated version.

**File:** `crypto/fips1402/src/main/java/org/keycloak/crypto/fips/FIPS1402Provider.java`

## Current PQC State

**BLOCKED** (external dependency)

## Required Changes

**Wait for BC-FIPS release, then upgrade:**

```xml
<!-- Root pom.xml, line 78 -->
<bouncycastle.bcfips.version>2.X.X</bouncycastle.bcfips.version>
```

Where `2.X.X` is the first BC-FIPS version with FIPS 140-3 validated ML-DSA/ML-KEM support.

**Keycloak has no code changes needed** — just version bump once BC-FIPS library is ready.

## Dependencies

**BC-FIPS library with FIPS 140-3 validated ML-DSA/ML-KEM** (external, not under Keycloak control)

## GitHub Issue Status

**GAP-6** — document in [#48823](https://github.com/keycloak/keycloak/issues/48823) (operator guidance)

Track BC-FIPS release status and update when available.

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
