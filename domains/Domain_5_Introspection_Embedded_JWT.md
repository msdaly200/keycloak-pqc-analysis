# Domain 5 — Introspection: Embedded JWT Response

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

The OAuth2 token introspection endpoint (`/protocol/openid-connect/token/introspect`) allows resource servers to validate a token. When the client is configured for a signed introspection response, Keycloak returns the introspection result as a **signed JWT** rather than plain JSON. This signed JWT is produced by calling `session.tokens().encode(transformedToken)` in [`AccessTokenIntrospectionProvider`](services/src/main/java/org/keycloak/protocol/oidc/AccessTokenIntrospectionProvider.java#L129), which routes through `DefaultTokenManager` — the same pipeline as access token signing.

## Gap

**No independent gap.** [`AccessTokenIntrospectionProvider`](services/src/main/java/org/keycloak/protocol/oidc/AccessTokenIntrospectionProvider.java#L129) calls `session.tokens().encode(transformedToken)`, which goes through [`DefaultTokenManager.encode()`](services/src/main/java/org/keycloak/jose/jws/DefaultTokenManager.java#L80) using `TokenCategory.ACCESS`. This resolves the algorithm via `OIDCConfigAttributes.ACCESS_TOKEN_SIGNED_RESPONSE_ALG`, then the realm default. Once Domain 1's ML-DSA `SignatureProviderFactory` and key provider exist, the introspection JWT response will use ML-DSA automatically.

## Required Changes

None needed. Automatically resolved when Domain 1 is complete.

## GitHub Issue

None needed. Automatically resolved when Domain 1 is complete.

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690)
