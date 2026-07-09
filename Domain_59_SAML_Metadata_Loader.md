# Domain 59 — SAML Metadata Public Key Loader

[← Back to PQC Overview](pqc_overview.html)

## What is this?

Loads signing and encryption keys from SAML metadata XML (`EntityDescriptor` / `IDPSSODescriptor` / `SPSSODescriptor`). Used to verify inbound SAML assertions from external IdPs.

## Gap

**Same as Domains 18-20 (GAP-2, GAP-9).**

**File:** `services/src/main/java/org/keycloak/protocol/saml/SamlAbstractMetadataPublicKeyLoader.java`

**Loading layer is algorithm-agnostic** (X.509 certificate extraction from metadata), but blocked on SAML signing infrastructure:
- **GAP-2** — No ML-DSA XML Signature URIs ([#50292](https://github.com/keycloak/keycloak/issues/50292))
- **GAP-9** — Hardcoded RS256 key selection ([#50294](https://github.com/keycloak/keycloak/issues/50294))

## Current PQC State

**SAFE** (loading layer) / **BLOCKED** (end-to-end)

## Required Changes

**None for this domain.**

Loader is algorithm-agnostic. Blocked on **Domains 18-20** fixes (GAP-2, GAP-9).

## Dependencies

**Domains 18-20** — SAML signing infrastructure:
- **GAP-2** — ML-DSA XML Signature URIs ([#50292](https://github.com/keycloak/keycloak/issues/50292))
- **GAP-9** — Hardcoded RS256 key selection ([#50294](https://github.com/keycloak/keycloak/issues/50294))

## GitHub Issue Status

No dedicated issue needed. Covered by [#50292](https://github.com/keycloak/keycloak/issues/50292) and [#50294](https://github.com/keycloak/keycloak/issues/50294).

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
