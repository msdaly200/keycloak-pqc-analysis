# Domain 7 — JAR: Signed Request Object Verification

[← Back to PQC Overview](pqc_overview.html)

## What is this?

OAuth 2.0 JAR (JWT-Secured Authorization Requests, RFC 9101) lets a client send its
authorization request as a signed JWT — either directly in the `request` query parameter,
or fetched from a URL in the `request_uri` parameter. Keycloak validates the signature of
that JWT before trusting any of the request parameters inside it.

The same path also handles **PAR** (Pushed Authorization Requests): when a client pushes a
signed request object to the PAR endpoint (`/protocol/openid-connect/ext/par/request`), the
same signature-verification code runs via `ParEndpointRequestObjectParser`, which simply
extends `AuthzEndpointRequestObjectParser`.

## Gap

**No independent gap.** The verification path is fully SPI-driven.

### Exact code path

1. [`AuthorizationEndpointRequestParserProcessor.parseRequest()`](services/src/main/java/org/keycloak/protocol/oidc/endpoints/request/AuthorizationEndpointRequestParserProcessor.java#L88)
   creates an `AuthzEndpointRequestObjectParser` with the raw JWT string and the `ClientModel`.

2. [`AuthzEndpointRequestObjectParser` constructor](services/src/main/java/org/keycloak/protocol/oidc/endpoints/request/AuthzEndpointRequestObjectParser.java#L44)
   calls `session.tokens().decodeClientJWT(requestObject, client, validator, JsonNode.class, true)`.

3. [`DefaultTokenManager.decodeClientJWT()`](services/src/main/java/org/keycloak/jose/jws/DefaultTokenManager.java#L123)
   delegates to [`verifyJWS()`](services/src/main/java/org/keycloak/jose/jws/DefaultTokenManager.java#L177),
   which looks up:
   ```java
   ClientSignatureVerifierProvider signatureProvider =
       session.getProvider(ClientSignatureVerifierProvider.class, signatureAlgorithm);
   ```
   where `signatureAlgorithm` is read from the JWS header. The provider is resolved purely from
   the SPI registry — there are no hardcoded algorithm checks.

4. For each currently-registered RSA/ECDSA algorithm, a dedicated factory (e.g.
   [`RS256ClientSignatureVerifierProviderFactory`](services/src/main/java/org/keycloak/crypto/RS256ClientSignatureVerifierProviderFactory.java))
   creates an [`AsymmetricClientSignatureVerifierProvider`](services/src/main/java/org/keycloak/crypto/AsymmetricClientSignatureVerifierProvider.java),
   which in turn calls `ClientAsymmetricSignatureVerifierContext` to look up the client's
   public key from its JWKS URI or registered key material.

5. The per-client algorithm constraint — `request_object_signing_alg` — is enforced inside
   the [`createRequestObjectValidator()`](services/src/main/java/org/keycloak/protocol/oidc/endpoints/request/AuthzEndpointRequestObjectParser.java#L84)
   lambda. It reads:
   ```java
   OIDCAdvancedConfigWrapper.fromClientModel(client).getRequestObjectSignatureAlg()
   ```
   and rejects mismatches — but this check is algorithm-name driven, not restricted to any
   fixed list.

6. The discovery document field `request_object_signing_alg_values_supported` is populated by
   [`OIDCWellKnownProvider`](services/src/main/java/org/keycloak/protocol/oidc/OIDCWellKnownProvider.java#L147):
   ```java
   config.setRequestObjectSigningAlgValuesSupported(getSupportedClientSigningAlgorithms(true));
   ```
   which enumerates all registered `ClientSignatureVerifierProvider` factories — so it will
   automatically include ML-DSA/FN-DSA/SLH-DSA once those factories are registered.

### The only narrow concern — FAPI `SecureSigningAlgorithmExecutor`

[`FapiConstant.ALLOWED_ALGORITHMS`](services/src/main/java/org/keycloak/services/clientpolicy/executor/FapiConstant.java#L30)
is a hardcoded set: `{ PS256, PS384, PS512, ES256, ES384, ES512 }`. This set is used by
[`SecureSigningAlgorithmExecutor`](services/src/main/java/org/keycloak/services/clientpolicy/executor/SecureSigningAlgorithmExecutor.java#L165)
to enforce that `request_object_signing_alg` is a "secure" algorithm. This is a **Domain 41
gap** (FAPI Algorithm Allowlist Enforcement), not a Domain 7 gap — Domain 7's signature
verification itself is unaffected.

## Required Changes

None. No code change required for Domain 7 itself.

The JAR signature verification path is fully SPI-driven. When `ClientSignatureVerifierProviderFactory`
implementations for ML-DSA, FN-DSA, and SLH-DSA are registered (the same pattern as the
existing `AsymmetricClientSignatureVerifierProvider` factories), Keycloak will:

- Accept ML-DSA/FN-DSA/SLH-DSA signed JAR request objects automatically
- Advertise those algorithms in `request_object_signing_alg_values_supported` automatically
- Allow per-client pinning via `request_object_signing_alg` automatically

## What does NOT need changing

| Item | Why no change needed |
|------|---------------------|
| `AuthzEndpointRequestObjectParser` | Delegates to SPI; no algorithm hardcoding |
| `AuthorizationEndpointRequestParserProcessor` | Entry point only; no algorithm checks |
| `DefaultTokenManager.verifyJWS()` | SPI lookup on algorithm name from JWT header |
| `ParEndpointRequestObjectParser` | Extends `AuthzEndpointRequestObjectParser`; inherits the same SPI path |
| `OIDCWellKnownProvider` | Already enumerates all registered `ClientSignatureVerifierProvider` factories |
| `createRequestObjectValidator` lambda | Algorithm-name comparison; works with any algorithm string |

## Dependencies

| Dependency | Blocking? |
|------------|-----------|
| `ClientSignatureVerifierProviderFactory` for ML-DSA/FN-DSA/SLH-DSA | Yes — same factories being added in Domain 1 (#48824) resolve this automatically |
| Domain 41 (FAPI allowlist) | Only affects clients enrolled in a FAPI policy; does not block the general verification path |

## GitHub Issue

No new issue needed. Domain 7 is automatically resolved when Domain 1 (#48824) registers
ML-DSA (and FN-DSA / SLH-DSA) `ClientSignatureVerifierProviderFactory` implementations.
The `SecureSigningAlgorithmExecutor` concern is already tracked under Domain 41.

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690)
