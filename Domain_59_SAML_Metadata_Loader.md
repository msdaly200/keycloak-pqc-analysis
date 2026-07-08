# Domain 59 — SAML Metadata Public Key Loader

## What is this?

Loads public keys from SAML metadata for signature verification.

## Gap

**Same as Domains 18-20 (GAP-2, GAP-9).**

Loading layer is algorithm-agnostic, but blocked on SAML signing infrastructure (no ML-DSA XML URIs, hardcoded RS256 key selection).

## Current PQC State

**SAFE** (loading layer) / **BLOCKED** (end-to-end)

## Required Changes

None for loader itself. Blocked on GAP-2 (XML Signature URIs) and GAP-9 (hardcoded RS256 key selection) from Domains 18-20.

## GitHub Issue Status

Covered by [#50292](https://github.com/keycloak/keycloak/issues/50292) and [#50294](https://github.com/keycloak/keycloak/issues/50294).

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
