# Domain 61 — SAML2Signature — Hardcoded RSA-SHA1 Default

## What is this?

Default SAML signature algorithm when none is configured.

## Gap

**Hardcoded RSA-SHA1 default (GAP-2).**

Related to GAP-2 (no ML-DSA XML Signature URIs). Even after ML-DSA URIs exist, the default algorithm fallback needs updating.

## Current PQC State

**BLOCKED**

## Required Changes

Update default SAML signature algorithm after ML-DSA XML Signature URIs are added (GAP-2).

## GitHub Issue Status

Covered by [#50292](https://github.com/keycloak/keycloak/issues/50292).

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
