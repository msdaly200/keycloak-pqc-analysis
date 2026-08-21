# Domain 37 — Security Event Token (SET) Signing

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

SSF/CAEP (Shared Signals Framework) Security Event Token signing for RISC events.

## Gap

**CAEP Interoperability Profile 1.0 §2.6 pins transmitters to RS256.**

**File:** `ssf/transmitter/src/main/java/org/keycloak/ssf/transmitter/event/SsfSignatureAlgorithms.java:31`

```java
public static final Set<String> ALLOWED = Set.of(Algorithm.RS256);
```

**From the code comments (lines 13-19):**

> "The CAEP interoperability profile 1.0 §2.6 pins transmitters to RS256... The plumbing around it (per-receiver override on SsfTransmitterConfigurationRepresentation.signatureAlgorithm) is in place, but everything ends up signed with RS256 regardless of override..."

The code explicitly notes this is **spec-gated** and will be relaxed when the CAEP working group broadens the interoperability profile to include PQC algorithms.

**No Keycloak action until CAEP spec evolves.**

## Current PQC State

**EXTERNAL DEPENDENCY**

## Required Changes

**One-line fix when CAEP spec evolves:**

```java
// Before:
public static final Set<String> ALLOWED = Set.of(Algorithm.RS256);

// After (example):
public static final Set<String> ALLOWED = Set.of(
    Algorithm.RS256,
    Algorithm.ML_DSA_65,  // when CAEP adds PQC
    Algorithm.ML_DSA_87
);
```

## Dependencies

**CAEP Interoperability Profile specification** — must add PQC algorithms before Keycloak can support them

## GitHub Issue Status

No dedicated issue needed. Document in [#48823](https://github.com/keycloak/keycloak/issues/48823) (operator guidance) to track CAEP spec status.

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
