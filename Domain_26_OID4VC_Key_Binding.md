# Domain 26 — OID4VC Key Binding — JWT Proof Validation

## What is this?

OID4VC key binding validates that a wallet possesses the private key corresponding to a public key in a credential by verifying a signed JWT proof.

## Current PQC State

**PENDING PROVIDERS**

`JwtProofValidator` uses `getSupportedAsymmetricSignatureAlgorithms()` — SPI-driven. Will support ML-DSA once providers exist.

## Required Changes

**None.** Automatically resolved when ML-DSA providers exist.

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
