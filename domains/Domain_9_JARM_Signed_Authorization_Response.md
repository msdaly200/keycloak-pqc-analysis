# Domain 9 — JARM Signed Authorization Response

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

JARM (JWT Secured Authorization Response Mode) is a mechanism defined in the [FAPI JARM specification](https://openid.net/specs/openid-financial-api-jarm-ID1.html) that wraps the OAuth 2.0 / OIDC authorization response (normally plain query parameters such as `code` and `state`) inside a signed JWT. Instead of receiving `?code=abc&state=xyz`, the client receives a single `?response=<JWT>`. The JWT is signed by the authorization server, allowing the client (or any party) to verify its authenticity and integrity.

Keycloak supports three JARM response modes: `query.jwt`, `fragment.jwt`, and `form_post.jwt`. The per-client `authorization_signed_response_alg` parameter controls which algorithm is used to sign the JARM JWT. When no algorithm is configured on the client, Keycloak falls back to the realm-level default signature algorithm.

---

## Gap

**There is no code gap.** The JARM signing path is fully algorithm-agnostic:

| File | Lines | Current behaviour |
|------|-------|-------------------|
| [`jose/jws/DefaultTokenManager.java`](services/src/main/java/org/keycloak/jose/jws/DefaultTokenManager.java) | 198–212 | `signatureAlgorithm(TokenCategory.AUTHORIZATION_RESPONSE)` → reads client attribute `authorization.signed.response.alg`; falls back to realm default or `Constants.DEFAULT_SIGNATURE_ALGORITHM` |
| [`protocol/oidc/utils/OIDCRedirectUriBuilder.java`](services/src/main/java/org/keycloak/protocol/oidc/utils/OIDCRedirectUriBuilder.java) | 243–274 | Calls `session.tokens().encodeAndEncrypt(responseJWT)` — delegates fully to the `TokenManager` SPI |
| [`representations/AuthorizationResponseToken.java`](core/src/main/java/org/keycloak/representations/AuthorizationResponseToken.java) | 8–10 | Returns `TokenCategory.AUTHORIZATION_RESPONSE`; no algorithm hardcoded |

The signing pipeline for JARM is identical to the one used for access tokens and ID tokens. Once an ML-DSA (or FN-DSA / SLH-DSA) `SignatureProvider` is registered, a client may set `authorization_signed_response_alg = ML-DSA-65` and JARM responses will be signed with that algorithm without any further code change.

---

## Required Changes

**None.** No code change required. Automatically resolved when Domain 1 is complete.

---

## What does NOT need changing

- `JWTRedirectUriBuilder` — algorithm selection fully delegated via `TokenManager`
- `DefaultTokenManager.signatureAlgorithm()` — `AUTHORIZATION_RESPONSE` case already reads the correct client attribute
- `OIDCAdvancedConfigWrapper.getAuthorizationSignedResponseAlg()` — stores/retrieves a free-text algorithm name; no allowlist to update
- `AuthorizationResponseToken` — no algorithm reference

---

## Dependencies

| Dependency | Domain | Tracked by |
|-----------|--------|-----------|
| ML-DSA / FN-DSA / SLH-DSA `SignatureProvider` implementations | Domain 1 | [#48824](https://github.com/keycloak/keycloak/issues/48824) / [#43692](https://github.com/keycloak/keycloak/issues/43692) |
| Realm-level default signature algorithm UI / defaults | Domain 1 | [#48824](https://github.com/keycloak/keycloak/issues/48824) |

---

## PQC State

**PENDING PROVIDERS** — No independent code change required. JARM signing will automatically support PQC algorithms once Domain 1 (ML-DSA signature providers) is complete.

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690)
