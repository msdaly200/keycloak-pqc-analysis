# Domain 53 — Client Asymmetric Signature Verifier Context & Providers

## What is this?

Full client-side signature verifier stack for `private_key_jwt` client authentication.

## Gap

**RSA path hardcoded KeyType.RSA guard; no AKP verifier context/provider.**

`ClientAsymmetricSignatureVerifierContext.getKey()` at line 36 explicitly throws `VerificationException("Key Type is not RSA")` for any non-RSA key — ECDSA, EdDSA, and ML-DSA (AKP) client keys are all unconditionally rejected through the RSA path. The ECDSA and EdDSA paths route through their own contexts and are SPI-driven — they will support ML-DSA automatically once providers exist. ML-DSA (AKP) client assertions currently have no routed path at all.

**Location:** `services/crypto/ClientAsymmetricSignatureVerifierContext.java`

## Current PQC State

**BLOCKED** (RSA path) / **PENDING PROVIDERS** (ECDSA/EdDSA paths)

## Required Changes

Remove or generalise the `KeyType.RSA` guard in `ClientAsymmetricSignatureVerifierContext` to accept any asymmetric key type, or add a dedicated AKP client verifier context + provider factory following the same pattern as the ECDSA/EdDSA paths.

## GitHub Issue Status

New issue needed (HIGH priority).

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
