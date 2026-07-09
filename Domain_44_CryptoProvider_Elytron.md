# Domain 44 — CryptoProvider SPI — WildFly Elytron Backend

[← Back to PQC Overview](pqc_overview.html)

## What is this?

WildFly Elytron security subsystem cryptography provider. Used when Keycloak runs on WildFly application server (legacy distribution).

## Gap

**None** — external dependency on WildFly Elytron roadmap.

## Current PQC State

**EXTERNAL DEPENDENCY**

WildFly Elytron currently supports only RSA and EC (ECDSA) algorithms. No PQC support.

**File:** `crypto/elytron/src/main/java/org/keycloak/crypto/elytron/WildFlyElytronProvider.java`

## Required Changes

None for Keycloak code. Wait for WildFly Elytron PQC support.

## Dependencies

**WildFly Elytron PQC support** (external, not under Keycloak control)

## GitHub Issue Status

No Keycloak issue needed. Track WildFly Elytron roadmap externally.

## What this means for operators

**WildFly users:** Cannot use PQC until WildFly Elytron adds ML-DSA/ML-KEM support.

**Workaround:** Migrate to Keycloak Quarkus distribution, which uses BouncyCastle (Domain 42) and already has PQC support.

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
