# Domain 23 — SPIFFE / SVID Identity Provider

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

Keycloak can federate with SPIFFE (Secure Production Identity Framework For Everyone) workload identity providers by verifying SPIFFE JWT-SVIDs (SPIFFE Verifiable Identity Documents). SPIFFE provides a framework for workload-to-workload authentication using cryptographically verifiable identities.

**How it works:**

1. A workload (e.g., a service) obtains a JWT-SVID from its SPIFFE runtime
2. The workload presents the JWT-SVID to Keycloak for authentication
3. Keycloak verifies the JWT signature using public keys from the SPIFFE bundle endpoint
4. Keycloak validates the SPIFFE ID (subject claim) matches the expected trust domain

**File:** `SpiffeIdentityProvider.java` (line 82)

```java
SignatureProvider signatureProvider = session.getProvider(SignatureProvider.class, alg);
// ... later ...
return signatureProvider.verifier(publicKey).verify(jws.getEncodedSignatureInput().getBytes(StandardCharsets.UTF_8), jws.getSignature());
```

The verification path is fully SPI-driven via `SignatureProvider`, reading the algorithm from the JWT header.

## Gap

**No gap — fully SPI-driven.**

SPIFFE JWT-SVID verification uses the same `SignatureProvider` SPI pattern as Domains 3, 5, 6, 9, 10, 17, 21, and 22 (inbound verification).

## Current PQC State

**PENDING PROVIDERS**

Signature verification is entirely SPI-driven and will support ML-DSA automatically once ML-DSA `SignatureProvider` exists. No independent code gap.

## Required Changes

**None.** Automatically resolved when ML-DSA providers exist.

## Dependencies

1. **ML-DSA `SignatureProvider`** (tracked under [#48821](https://github.com/keycloak/keycloak/issues/48821) / [#48824](https://github.com/keycloak/keycloak/issues/48824))
2. **SPIFFE runtime** must support ML-DSA-signed JWT-SVIDs (external dependency)
3. **SPIFFE bundle endpoint** must include ML-DSA public keys in JWKS format

## GitHub Issue Status

No new issue needed.

SPIFFE IdP verification is covered by the core ML-DSA provider work under [#48824](https://github.com/keycloak/keycloak/issues/48824). Once providers exist, SPIFFE JWT-SVIDs will automatically support ML-DSA with zero code changes.

## What this means for operators

**When migrating to PQC:**

1. **SPIFFE runtime** (e.g., SPIRE server) must be updated to sign JWT-SVIDs with ML-DSA
2. **SPIFFE bundle endpoint** must publish ML-DSA public keys
3. **Keycloak will automatically verify** ML-DSA-signed JWT-SVIDs once ML-DSA providers exist
4. **No Keycloak configuration changes needed** — the algorithm is read from the JWT's `alg` header
5. **Hybrid deployments supported:** Some workloads can use ML-DSA while others continue with RSA/ECDSA

**Timeline dependency:** This depends on SPIFFE/SPIRE upstream supporting PQC signatures for JWT-SVIDs.

**SPIFFE specification reference:**
- [draft-schwenkschuster-oauth-spiffe-client-auth](https://datatracker.ietf.org/doc/draft-schwenkschuster-oauth-spiffe-client-auth/) — SPIFFE JWT-SVID client authentication

## Related Domains

- **Domain 10** — private_key_jwt Client Authentication (uses the same `SignatureProvider` SPI pattern)
- **Domain 22** — Kubernetes Identity Provider (similar workload identity verification; nearly identical code)
- **Domain 21** — OIDC IdP Broker (similar identity provider verification path)

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
