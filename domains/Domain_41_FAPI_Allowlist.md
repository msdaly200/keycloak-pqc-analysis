# Domain 41 — FAPI Algorithm Allowlist Enforcement

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

FAPI client policy enforcement that actively blocks non-approved signing algorithms.

## Gap

**FapiConstant.ALLOWED_ALGORITHMS hardcoded to classical algorithms (GAP-8, GAP-17).**

**File:** `services/src/main/java/org/keycloak/services/clientpolicy/executor/FapiConstant.java:30`

```java
public static final Set<String> ALLOWED_ALGORITHMS = new LinkedHashSet<>(Arrays.asList(
    Algorithm.PS256, Algorithm.PS384, Algorithm.PS512,
    Algorithm.ES256, Algorithm.ES384, Algorithm.ES512
));
```

**The problem:**

Multiple FAPI executors call `ALLOWED_ALGORITHMS.contains(sigAlg)` to enforce algorithm restrictions:
- `SecureSigningAlgorithmExecutor` (general signing)
- `SecureSigningAlgorithmForSignedJwtExecutor` (client assertions, JAR)
- `SecureCibaAuthenticationRequestSigningAlgorithmExecutor` (CIBA)

ML-DSA will be **actively rejected** even after providers exist.

**Spec-gated:** FAPI 1.0 Advanced and FAPI 2.0 Security Profile currently specify only PS* and ES* algorithms. ML-DSA cannot be added until FAPI specs evolve.

## Current PQC State

**BLOCKED** (spec-gated)

## Required Changes

**One-line fix when FAPI spec evolves:**

```java
public static final Set<String> ALLOWED_ALGORITHMS = new LinkedHashSet<>(Arrays.asList(
    Algorithm.PS256, Algorithm.PS384, Algorithm.PS512,
    Algorithm.ES256, Algorithm.ES384, Algorithm.ES512,
    Algorithm.ML_DSA_65, Algorithm.ML_DSA_87  // add when FAPI spec includes PQC
));
```

Also update executor factories' admin UI dropdown lists to surface ML-DSA options.

## Dependencies

**FAPI 2.0 Security Profile specification** — must add PQC algorithms before Keycloak can support them

## GitHub Issue Status

**GAP-8** — needs GitHub issue under [#43690](https://github.com/keycloak/keycloak/issues/43690) to track FAPI spec status

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
