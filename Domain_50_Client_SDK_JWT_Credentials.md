# Domain 50 — Client SDK — JWT Client Credentials Provider

## What is this?

Client-side `private_key_jwt` assertion signing from the adapter/SDK.

## Gap

**No AKP case in setupKeyPair() switch.**

`setupKeyPair()` defaults to `Algorithm.RS256`. The algorithm-aware overload routes RSA, EC, and OKP (EdDSA) key types to the correct signer context. However the `switch` on `JavaAlgorithm.getKeyType()` has no `AKP` case — passing an ML-DSA key pair will throw `RuntimeException("Invalid KeyPair algorithm")`.

**Location:** `core/.../client/authentication/JWTClientCredentialsProvider.java`

## Current PQC State

**PARTIAL**

## Required Changes

Add an `AKP` case to `setupKeyPair()` wiring an `AsymmetricSignatureSignerContext` for ML-DSA, once ML-DSA providers exist.

## GitHub Issue Status

New issue needed (MEDIUM priority).

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
