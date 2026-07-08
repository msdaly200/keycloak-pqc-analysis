# Domain 48 — Realm JWKS Endpoint — Key Serialisation

## What is this?

Public JWKS endpoint (`/protocol/openid-connect/certs`) that exposes realm public keys.

## Gap

**AKP keys return null and are silently omitted from JWKS.**

`JWKSServerUtils.toJwk()` has explicit `if/else if` branches for `KeyType.RSA`, `KeyType.EC`, and `KeyType.OKP` only. An AKP (ML-DSA) key falls through all branches and returns `null`, meaning ML-DSA realm keys will be silently omitted from the public JWKS endpoint. Clients will never discover the realm's ML-DSA keys.

**Location:** `protocol/oidc/utils/JWKSServerUtils.java`

## Current PQC State

**BLOCKED**

## Required Changes

Add an `AKP` branch in `toJwk()` calling `JWKBuilder.akp()`. Straightforward one-branch addition; `JWKBuilder.akp()` already exists.

## GitHub Issue Status

New issue needed (HIGH priority).

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
