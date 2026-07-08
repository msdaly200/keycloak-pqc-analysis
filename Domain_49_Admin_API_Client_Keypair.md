# Domain 49 — Admin API — Client Certificate & Keypair Generation

## What is this?

Admin API endpoints for generating client keypairs and self-signed certificates.

## Gap

**Hardcoded RSA key generation.**

Both `generate()` and `generateAndGetKeystore()` call `KeycloakModelUtils.generateKeyPairCertificate()`, which hardcodes RSA key generation (`KeyUtils.generateRsaKeyPair(keysize)`). No algorithm selection is exposed in the API or UI. Admin-generated client keypairs will always be RSA regardless of realm PQC configuration.

**Location:** `services/resources/admin/ClientAttributeCertificateResource.java`

## Current PQC State

**BLOCKED**

## Required Changes

Expose an algorithm parameter in the `generateAndGetKeystore` endpoint and update `KeycloakModelUtils.generateKeyPairCertificate()` to support ML-DSA key generation once providers exist.

## GitHub Issue Status

New issue needed (MEDIUM priority).

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
