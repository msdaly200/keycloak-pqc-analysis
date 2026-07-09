# Domain 39 — JGroups ASYM_ENCRYPT (non-default / test config)

[← Back to PQC Overview](pqc_overview.html)

## What is this?

JGroups cluster encryption using RSA-2048 key exchange for distributing shared AES group key.

## Gap

**RSA-2048 asymmetric key exchange in test/operator config (GAP-14).**

**File:** `quarkus/tests/integration/src/test/resources/cache-ispn-asym-enc.xml`

```xml
<ASYM_ENCRYPT asym_keylength="2048"
              asym_algorithm="RSA"
              ...>
```

**Production impact: NONE**

Production Keycloak Quarkus **does not enable `ASYM_ENCRYPT` by default**. Production deployments use:
- **mTLS** via `JGroupsCertificateProvider` (TLSv1.3)
- **SSL_KEY_EXCHANGE** protocol for cluster key distribution

Both will inherit quantum-safety when PQC-capable TLS stacks are deployed (TLS 1.3 with ML-KEM/ML-DSA support).

**Risk is confined to:**
- Test configurations (this file)
- Operators who manually enable `ASYM_ENCRYPT` in custom configs

## Current PQC State

**SAFE** (production)

## Required Changes

**No code change needed.**

Document in operator migration guidance:
1. **Recommended:** Use `SSL_KEY_EXCHANGE` with mTLS instead of `ASYM_ENCRYPT`
2. If `ASYM_ENCRYPT` must be used, upgrade JGroups to version with PQC support (when available)

## Dependencies

**JGroups library PQC support** (external dependency, not under Keycloak control)

## GitHub Issue Status

**GAP-14** — document in [#48823](https://github.com/keycloak/keycloak/issues/48823) (operator guidance)

## What this means for operators

**Production deployments:** Already quantum-ready via mTLS path (will inherit TLS 1.3 PQC support).

**Custom deployments using ASYM_ENCRYPT:** Should migrate to `SSL_KEY_EXCHANGE` with mTLS, or wait for JGroups PQC support.

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
