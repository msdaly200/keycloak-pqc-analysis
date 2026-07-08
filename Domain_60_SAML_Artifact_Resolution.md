# Domain 60 — SAML Artifact Resolution

## What is this?

SAML artifact binding resolution and response signing.

## Gap

**Same as Domains 18-20 (GAP-2, GAP-9).**

## Current PQC State

**BLOCKED**

## Required Changes

None for artifact resolution itself. Blocked on GAP-2 (XML Signature URIs) and GAP-9 (hardcoded RS256 key selection in `SamlProtocol.java:544`) from Domains 18-20.

## GitHub Issue Status

Covered by [#50292](https://github.com/keycloak/keycloak/issues/50292) and [#50294](https://github.com/keycloak/keycloak/issues/50294).

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
