# Domain 25 — LD-Proof Credential Signing (Linked Data)

## What is this?

Linked Data Proofs use cryptographic suites (like Ed25519Signature2018) to sign verifiable credentials in JSON-LD format.

## Gap

**Hardcoded to Ed255192018Suite (GAP-7).**

**File:** `LDCredentialSigner.java`
LD-Proof signing is hardcoded to `Ed255192018Suite`. A new LD cryptographic suite for ML-DSA would need to be implemented separately — it does not inherit from the `SignatureProvider` SPI path.

## Current PQC State

**BLOCKED**

## Required Changes

Implement a new ML-DSA LD cryptographic suite if LD-Proofs with PQC are required.

## GitHub Issue Status

**GAP-7** needs a new issue (low priority).

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
