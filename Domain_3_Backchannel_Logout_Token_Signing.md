# Domain 3 — Backchannel Logout Token Signing

## What is this?

The OpenID Connect Back-Channel Logout specification (OIDC BCLG) allows Keycloak to
notify client applications that a session has ended by sending a signed **logout token**
as an HTTP POST. The logout token is a JWT signed by Keycloak with the realm's active
signing key — the same pipeline used for ID tokens.

## Gap

**No independent gap.** `TokenCategory.LOGOUT` routes through the same
[`DefaultTokenManager.signatureAlgorithm()`](services/src/main/java/org/keycloak/jose/jws/DefaultTokenManager.java)
path as ID tokens, resolving the signing algorithm from the realm's active signing key
provider. Once Domain 1's ML-DSA `SignatureProviderFactory` and key provider are in place,
logout tokens will automatically be signed with ML-DSA.

The only operational concern is coupling: logout token algorithm and ID token algorithm
are controlled by the same realm attribute. There is no independent per-token-category
algorithm control. On migration, downstream clients (relying parties) receiving logout
tokens must therefore support ML-DSA at the same time the realm switches. This is an
operator migration coordination concern, not a code gap.

## Required Changes

None. No code change required.

## GitHub Issue

No new issue needed. Covered by operator migration guidance in
[#48823](https://github.com/keycloak/keycloak/issues/48823). Automatically resolved
when Domain 1 is complete.

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690)
