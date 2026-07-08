# Domain 38 — kcadm.sh / kcreg.sh — private_key_jwt Auth

## What is this?

Admin CLI tools (`kcadm.sh` and `kcreg.sh`) authenticate to Keycloak using `private_key_jwt` client authentication.

## Gap

**Hardcoded RS256 (GAP-13).**

`AuthUtil.getSignedRequestToken()` always produces an RS256-signed JWT regardless of realm configuration. No `--sigalg` parameter exists. If a realm requires ML-DSA for client assertions, both `kcadm.sh` and `kcreg.sh` will be unable to authenticate via the keystore-based flow.

**Location:**
- `client-cli/admin-cli/.../util/AuthUtil.java` — `new JWSBuilder().rsa256(keypair.getPrivate())`

## Current PQC State

**BLOCKED**

## Required Changes

Add algorithm parameter to `getSignedRequestToken()`. Plumb a `--sigalg` option through the credential configuration command chain.

## GitHub Issue Status

**GAP-13** — track under [#48824](https://github.com/keycloak/keycloak/issues/48824) or dedicated CLI sub-issue.

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
