# Domain 6 — Token Verification (Identity & Session Tokens)

[← Back to PQC Overview](pqc_overview.html)

## What is this?

Keycloak verifies signed JWTs in several internal flows — for example, validating identity tokens during authentication (`AuthenticationManager`), verifying action tokens in login flows (`LoginActionsService`), and re-validating session tokens. All of these call into the `SignatureProvider` SPI to verify the token's signature using the public key identified by the `kid` header.

## Gap

**No independent gap.** The verification path is fully SPI-driven. [`DefaultTokenManager.decode()`](services/src/main/java/org/keycloak/jose/jws/DefaultTokenManager.java#L92) calls `session.getProvider(SignatureProvider.class, signatureAlgorithm)` where `signatureAlgorithm` is read from the JWT header. Once Domain 1 registers an ML-DSA `SignatureProviderFactory`, the verifier for ML-DSA-signed tokens resolves automatically. No hardcoded algorithm checks exist in the verification path.

## Required Changes

None. No code change required.

## GitHub Issue

None needed. Automatically resolved when Domain 1 is complete.

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690)
