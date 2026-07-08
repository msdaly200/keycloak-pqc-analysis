# Domain 41 — FAPI Algorithm Allowlist Enforcement

## What is this?

FAPI client policy enforcement that actively blocks non-approved signing algorithms.

## Gap

**FapiConstant.ALLOWED_ALGORITHMS hardcoded to classical algorithms (GAP-8, GAP-17).**

`FapiConstant.ALLOWED_ALGORITHMS` contains only `{PS256, PS384, PS512, ES256, ES384, ES512}`. `SecureSigningAlgorithmExecutor.isSecureAlgorithm()` calls `ALLOWED_ALGORITHMS.contains(sigAlg)` — ML-DSA will be **actively rejected** even after providers exist.

**Locations:**
- `clientpolicy/executor/FapiConstant.java`
- `clientpolicy/executor/SecureSigningAlgorithmExecutor.java`
- `clientpolicy/executor/SecureSigningAlgorithmForSignedJwtExecutor.java`
- `ciba/clientpolicy/executor/SecureCibaAuthenticationRequestSigningAlgorithmExecutor.java`

## Current PQC State

**BLOCKED** (spec-gated)

## Required Changes

Extend `FapiConstant.ALLOWED_ALGORITHMS` with ML-DSA once FAPI 2.0 / FAPI profiles include PQC. Update executor factories' admin UI option lists to surface ML-DSA.

## GitHub Issue Status

**GAP-8** — blocked on FAPI 2.0 spec. New issue needed (spec-gated).

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
