# Domain 17 — JWT Authorization Grant — Assertion Verification

## What is this?

JWT Authorization Grant (also known as JWT Bearer Token Grant) is defined in [RFC 7523](https://www.rfc-editor.org/rfc/rfc7523.html). It allows a client to exchange a signed JWT assertion for an OAuth2 access token.

**Use case:** Service-to-service authentication where a trusted issuer (e.g., an upstream identity provider) issues a JWT assertion that the client presents to Keycloak to obtain access tokens for downstream APIs.

**How it works:**

1. The client obtains a signed JWT assertion from a trusted issuer (or creates one itself if it's a trusted client)
2. The client sends the JWT to Keycloak's token endpoint with `grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer`
3. Keycloak verifies the JWT signature using the issuer's public key
4. Keycloak validates claims (`iss`, `sub`, `aud`, `exp`, etc.)
5. Keycloak issues an access token to the client

## Gap

**No gap — fully SPI-driven.**

### File: `AbstractBaseJWTValidator.java` (line 161)

```java
ClientSignatureVerifierProvider signatureProvider = session.getProvider(ClientSignatureVerifierProvider.class, algorithmName);
```

The JWT assertion signature verification is fully delegated to the `ClientSignatureVerifierProvider` SPI, which is the same SPI used for `private_key_jwt` client authentication (Domain 10).

**File:** `DefaultJWTAuthorizationGrantValidator.java` (line 38)

```java
public class DefaultJWTAuthorizationGrantValidator extends AbstractBaseJWTValidator implements JWTAuthorizationGrantValidator
```

The validator inherits all signature verification logic from `AbstractBaseJWTValidator`, which uses `ClientSignatureVerifierProvider` to verify the assertion signature.

## Current PQC State

**PENDING PROVIDERS**

Signature verification is **entirely SPI-driven** and will support ML-DSA automatically once ML-DSA `ClientSignatureVerifierProvider` exists. No independent code gap.

## Required Changes

**None.**

JWT authorization grant assertion verification will support ML-DSA automatically when:
1. ML-DSA `SignatureProvider` exists (tracked under #48821 / #48824)
2. The `ClientSignatureVerifierProvider` SPI routes to it (happens automatically)

## What does NOT need changing

| Component | Why it's already PQC-ready |
|-----------|---------------------------|
| Signature verification (line 161 of `AbstractBaseJWTValidator`) | Uses `ClientSignatureVerifierProvider` SPI — same as Domain 10 (private_key_jwt) |
| Algorithm validation | Delegates to the SPI — any registered asymmetric algorithm is supported |
| JWT parsing | Algorithm-agnostic — reads `alg` from header and looks up provider |
| Claim validation | Pure business logic — no cryptographic operations |

## Dependencies

1. **ML-DSA `SignatureProvider`** (tracked under #48821 / #48824) — required before JWT assertions can use ML-DSA signatures

## GitHub Issue Status

No new issue needed.

JWT authorization grant verification is covered by the core ML-DSA provider work under **#48824**. Once providers exist, this grant type will automatically support ML-DSA with zero code changes.

## What this means for operators

**When migrating to PQC:**

1. **Upstream IdPs** that issue JWT assertions can migrate to ML-DSA signatures
2. Keycloak will **automatically verify** ML-DSA-signed assertions once ML-DSA providers exist
3. **No Keycloak configuration changes needed** — the algorithm is read from the JWT's `alg` header
4. **Hybrid deployments supported:** Some issuers can use ML-DSA while others continue with RSA/ECDSA

## Related Domains

- **Domain 10** — private_key_jwt Client Authentication (uses the same `ClientSignatureVerifierProvider` SPI and `AbstractBaseJWTValidator` base class)
- **Domain 57** — Token Exchange Grant (also mentions verification via `SignatureProvider` SPI)

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness