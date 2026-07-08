# Domain 37 — Security Event Token (SET) Signing

## What is this?

SSF/CAEP (Shared Signals Framework) Security Event Token signing for RISC events.

## Gap

**CAEP Interoperability Profile 1.0 §2.6 pins transmitters to RS256.**

Code comment explicitly notes this is intended to be relaxed as the working group broadens the profile. No Keycloak action until CAEP spec includes PQC algorithms.

**Location:**
- `ssf/transmitter/.../event/SsfSignatureAlgorithms.java` — `ALLOWED = Set.of(Algorithm.RS256)`

## Current PQC State

**EXTERNAL DEPENDENCY**

## Required Changes

Update `ALLOWED` set in `SsfSignatureAlgorithms.java` once CAEP spec includes PQC algorithms. One-line fix when spec moves.

## GitHub Issue Status

No action needed until CAEP spec evolves. Document in [#48823](https://github.com/keycloak/keycloak/issues/48823) (operator guidance).

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
