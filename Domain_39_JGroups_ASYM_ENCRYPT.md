# Domain 39 — JGroups ASYM_ENCRYPT (non-default / test config)

## What is this?

JGroups cluster encryption using RSA-2048 key exchange for distributing shared AES group key.

## Gap

**RSA-2048 asymmetric key exchange in test/operator config (GAP-14).**

Production Keycloak Quarkus does not enable `ASYM_ENCRYPT` by default. Production stack uses mTLS via `JGroupsCertificateProvider` (TLSv1.3), which will inherit quantum-safety when PQC-capable TLS is deployed. Risk is confined to operators who manually copy the example ASYM_ENCRYPT config.

**Location:**
- `quarkus/tests/integration/.../cache-ispn-asym-enc.xml`

## Current PQC State

**SAFE** (production)

## Required Changes

No code change needed. Document in operator migration guidance: operators using `ASYM_ENCRYPT` should switch to `SSL_KEY_EXCHANGE`.

## GitHub Issue Status

**GAP-14** — document in [#48823](https://github.com/keycloak/keycloak/issues/48823) (operator guidance).

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
