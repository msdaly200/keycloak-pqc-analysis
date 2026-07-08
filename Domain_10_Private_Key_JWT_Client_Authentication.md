# Domain 10 — private_key_jwt Client Authentication

## What is this?

`private_key_jwt` is one of the client authentication methods defined in [OpenID Connect Core](https://openid.net/specs/openid-connect-core-1_0.html#ClientAuthentication) and [RFC 7523](https://www.rfc-editor.org/rfc/rfc7523). Instead of sending a client secret, the client proves its identity by presenting a signed JWT (`client_assertion`) at the token endpoint. Keycloak, acting as the authorization server, verifies that JWT's signature against the client's registered public key.

The algorithm used is whatever the client chooses to sign with — Keycloak accepts any algorithm for which a registered `ClientSignatureVerifierProvider` SPI implementation exists.

---

## Gap

**There is no code gap.** The verification path is fully algorithm-agnostic:

| File | Lines | Current behaviour |
|------|-------|-------------------|
| [`authenticators/client/JWTClientAuthenticator.java`](services/src/main/java/org/keycloak/authentication/authenticators/client/JWTClientAuthenticator.java) | 114–115 | Reads algorithm from `jose.getHeader().getRawAlgorithm()`, looks up `ClientSignatureVerifierProvider` by name — no hardcoded list |
| [`keys/loader/PublicKeyStorageManager.java`](services/src/main/java/org/keycloak/keys/loader/PublicKeyStorageManager.java) | 52–58 | Fetches client public key by `kid` + `alg` from the JWT header — no algorithm filter |
| [`authenticators/client/AbstractJWTClientValidator.java`](services/src/main/java/org/keycloak/authentication/authenticators/client/AbstractJWTClientValidator.java) | 66–72 | Validates assertion type, client, signature, audience, and expiry — all algorithm-independent |

The only thing required for a client to authenticate with an ML-DSA (or FN-DSA / SLH-DSA) signed `client_assertion` is:

1. A `ClientSignatureVerifierProvider` for that algorithm (provided by #48824).
2. The client's ML-DSA public key registered in Keycloak (as a JWKS URL or uploaded JWK).

---

## What does NOT need changing

- `JWTClientAuthenticator.verifySignature()` — algorithm read from header; `ClientSignatureVerifierProvider` looked up dynamically
- `PublicKeyStorageManager.getClientPublicKeyWrapper()` — key loading by `kid`/`alg` is open
- `AbstractJWTClientValidator` — no algorithm assumptions
- `ClientPublicKeyLoader` — delegates to `ClientSignatureVerifierProvider`, which is the SPI extension point

---

## Dependencies

| Dependency | Tracked by |
|-----------|-----------|
| ML-DSA / FN-DSA / SLH-DSA `ClientSignatureVerifierProvider` implementations | [#48824](https://github.com/keycloak/keycloak/issues/48824) |
| Client public key registration (JWKS URL / JWK upload) supporting PQC key types | [#48824](https://github.com/keycloak/keycloak/issues/48824) |

---

## PQC State

**PENDING PROVIDERS** — No independent code change required. `private_key_jwt` client authentication will automatically support PQC algorithms once the core signature provider implementations are available.

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690)
