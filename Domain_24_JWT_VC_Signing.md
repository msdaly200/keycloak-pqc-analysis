# Domain 24 — JWT-VC / SD-JWT Credential Signing

## What is this?

OID4VC (OpenID for Verifiable Credentials) allows Keycloak to issue verifiable credentials in JWT or SD-JWT format.

## Current PQC State

**PENDING PROVIDERS**

Algorithm is passed via `CredentialBuildConfig.signingAlgorithm` through the `SignatureProvider` SPI. Will automatically pick up ML-DSA once signature providers exist.

## Required Changes

**None.** Automatically resolved when ML-DSA providers exist.

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
