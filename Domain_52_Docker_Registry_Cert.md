# Domain 52 — Docker Registry — Self-Signed Certificate Generation

## What is this?

RSA-2048 key pair generation for Docker registry TLS certificate.

## Gap

**Hardcoded RSA-2048 key generation.**

Directly calls `CryptoIntegration.getProvider().getKeyPairGen(KeyType.RSA)` with `keyGen.initialize(2048)` hardcoded. No algorithm parameter. Docker registry certificates will always be RSA-2048.

**Location:** `protocol/docker/installation/compose/DockerComposeCertsDirectory.java`

## Current PQC State

**BLOCKED**

## Required Changes

Make key algorithm and size configurable once ML-DSA cert generation is supported. Low urgency — Docker compose is a developer/test installation path.

## GitHub Issue Status

New issue needed (LOW priority).

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
