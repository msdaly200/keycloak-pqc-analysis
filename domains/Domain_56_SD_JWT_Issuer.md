# Domain 56 — SD-JWT Issuer Signing & Key Binding

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

Selective Disclosure JWT (SD-JWT) credential signing and key binding validation. Used for privacy-preserving credential issuance and presentation.

## Gap

**None** — inherits SPI-driven signing/verification paths.

**Files:**
- `core/src/main/java/org/keycloak/sdjwt/SdJwt.java`
- `core/src/main/java/org/keycloak/sdjwt/IssuerSignedJWT.java`
- `core/src/main/java/org/keycloak/sdjwt/SdJwtVerificationContext.java`
- `core/src/main/java/org/keycloak/sdjwt/vp/KeyBindingJWT.java`

## Current PQC State

**PENDING PROVIDERS**

Algorithm-agnostic via `SignatureProvider` SPI. Will support ML-DSA automatically once providers exist.

## Required Changes

**None.**

## Dependencies

**GAP-15** — ML-DSA SignatureProvider ([#48824](https://github.com/keycloak/keycloak/issues/48824))

## GitHub Issue Status

No dedicated issue needed. Covered by GAP-15.

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
