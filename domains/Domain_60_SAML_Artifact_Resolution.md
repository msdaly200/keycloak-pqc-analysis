# Domain 60 — SAML Artifact Resolution

[← Back to PQC Overview](pqc_overview.html)

## What is this?

SAML artifact resolution — fetches full SAML assertions over a SOAP backchannel (`ArtifactResolve` / `ArtifactResponse`). Resolved assertions are then verified using the same SAML signing chain as Domains 18-20.

## Gap

**Same as Domains 18-20 (GAP-2, GAP-9).**

**Files:**
- `services/src/main/java/org/keycloak/protocol/saml/DefaultSamlArtifactResolver.java`
- `services/src/main/java/org/keycloak/protocol/saml/DefaultSamlArtifactResolverFactory.java`

**Blocked on SAML signing infrastructure:**
- **GAP-2** — No ML-DSA XML Signature URIs ([#50292](https://github.com/keycloak/keycloak/issues/50292))
- **GAP-9** — Hardcoded RS256 key selection in `SamlProtocol.java:544` ([#50294](https://github.com/keycloak/keycloak/issues/50294))

## Current PQC State

**BLOCKED**

## Required Changes

**None for this domain.**

Artifact resolution is algorithm-agnostic. Blocked on **Domains 18-20** fixes (GAP-2, GAP-9).

## Dependencies

**Domains 18-20** — SAML signing infrastructure:
- **GAP-2** — ML-DSA XML Signature URIs ([#50292](https://github.com/keycloak/keycloak/issues/50292))
- **GAP-9** — Hardcoded RS256 key selection ([#50294](https://github.com/keycloak/keycloak/issues/50294))

## GitHub Issue Status

No dedicated issue needed. Covered by [#50292](https://github.com/keycloak/keycloak/issues/50292) and [#50294](https://github.com/keycloak/keycloak/issues/50294).

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
