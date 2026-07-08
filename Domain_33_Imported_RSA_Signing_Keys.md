# Domain 33 — Imported RSA Signing Key Provider

## What is this?

Allows operators to import externally-generated RSA signing keys into Keycloak realms.

## Gap

**No ML-DSA key import path (GAP-5, GAP-16).**

Implicit in keystore import gap.

## Current PQC State

**BLOCKED**

## Required Changes

Extend import logic to support ML-DSA keys. Related to Domain 34 (Java Keystore import).

## GitHub Issue Status

**GAP-5** and **GAP-16** need new issue.

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
