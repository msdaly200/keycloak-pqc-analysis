# Domain 4 — UserInfo Endpoint: Signed & Encrypted Response

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

The UserInfo endpoint returns claims about the authenticated user. Clients can request the response as:
- A plain JSON object
- A **signed** JWT (using the realm's signature key)
- A **signed-then-encrypted** JWT (signing key + client's public encryption key)
- An **encrypted-only** JWT

The signing and encryption algorithms are configured **per-client** as independent attributes, not inherited from the realm default. This makes Domain 4 a distinct gap from Domain 1 (token signing) and Domain 2 (ID token encryption) — even after those are fixed, UserInfo requires explicit per-client migration.

## Gap

Two independent gaps:

### Gap A — Encryption (BLOCKED via GAP-4)
[`UserInfoEndpoint.jweFromContent()`](services/src/main/java/org/keycloak/protocol/oidc/endpoints/UserInfoEndpoint.java#L376) calls `session.getProvider(CekManagementProvider.class, algAlgorithm)`. No ML-KEM `CekManagementProvider` exists (same root as Domain 2 / GAP-4). Once Domain 2 is resolved, the encryption path here works automatically — no additional code change needed in `UserInfoEndpoint`.

### Gap B — Signing (migration/operational gap — GAP-21)
[`DefaultTokenManager.signatureAlgorithm(TokenCategory.USERINFO)`](services/src/main/java/org/keycloak/jose/jws/DefaultTokenManager.java#L209) resolves the **per-client** attribute `userinfo.response.signature.alg`, then falls back to the realm default. Unlike `TokenCategory.ID`, this per-client attribute is **not automatically updated** when the realm default changes. Every client that has `userinfo.response.signature.alg` explicitly set must be individually updated during migration.

There is no code to fix here — it is a migration procedure gap. The risk is that after a realm switches to ML-DSA, clients with a stale `userinfo.response.signature.alg = RS256` will continue to receive classically-signed UserInfo responses silently.

## Required Changes

| # | What | Where | Action |
|---|------|--------|--------|
| 1 | Encryption | `UserInfoEndpoint.jweFromContent()` | No change — automatically fixed when Domain 2 (ML-KEM `CekManagementProvider`) is resolved |
| 2 | Signing | Per-client `USER_INFO_RESPONSE_SIGNATURE_ALG` attribute | Operator migration: must explicitly update per-client attribute for every client that has it set |
| 3 | Documentation | Migration guide | Document the per-client update requirement in operator migration guidance (#48823) |

## What Does NOT Need Changing

- `UserInfoEndpoint` signing path — already SPI-driven via `SignatureProvider`; picks up ML-DSA once Domain 1 providers exist
- `UserInfoEndpoint.jweFromContent()` — already SPI-driven via `CekManagementProvider`; picks up ML-KEM once Domain 2 is resolved
- `DefaultTokenManager.signatureAlgorithm()` — the USERINFO case correctly reads the per-client attribute; the behaviour is correct, the gap is operational

## GitHub Issue

No dedicated issue. Document the per-client migration requirement in [#48823](https://github.com/keycloak/keycloak/issues/48823) (operator migration guidance).

## Dependencies

- Domain 1 (ML-DSA `SignatureProviderFactory`) — prerequisite for ML-DSA signed UserInfo
- Domain 2 (ML-KEM `CekManagementProvider`) — prerequisite for ML-KEM encrypted UserInfo

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690)
