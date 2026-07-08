# Domain 43 — CryptoProvider SPI — FIPS 140-2/3 Backend (BC-FIPS)

## What is this?

FIPS-mode cryptography provider using BouncyCastle FIPS (bc-fips).

## Gap

**BC-FIPS 2.1.2 lacks ML-DSA/ML-KEM (GAP-6).**

BC-FIPS 2.1.2 (pinned in root `pom.xml`) confirmed by JAR inspection to contain only LMS under `org.bouncycastle.crypto.internal.pqc`. No ML-DSA and no ML-KEM classes are present. BC-FIPS PQC support (FIPS 204/203) requires a forthcoming BC-FIPS 2.x release achieving NIST FIPS 140-3 validation for the new algorithms. Confirmed hard blocker for regulated deployments.

**Location:** `crypto/fips1402/.../fips/FIPS1402Provider.java`

## Current PQC State

**BLOCKED** (external dependency)

## Required Changes

Upgrade `bc-fips` to a version that includes FIPS 140-3 validated ML-DSA/ML-KEM support once available. Dependency on BC-FIPS release schedule.

## GitHub Issue Status

**GAP-6** — external dependency (BC-FIPS release). Document in [#48823](https://github.com/keycloak/keycloak/issues/48823).

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
