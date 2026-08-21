# Domain 22 — Kubernetes Identity Provider

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

Keycloak can federate with Kubernetes as an identity provider by verifying Kubernetes service account JWTs. When a Kubernetes service account attempts to authenticate, it presents a signed JWT that Keycloak verifies using the Kubernetes API server's public keys.

**File:** `KubernetesIdentityProvider.java` (line 53)

```java
SignatureProvider signatureProvider = session.getProvider(SignatureProvider.class, alg);
// ... later ...
return signatureProvider.verifier(publicKey).verify(jws.getEncodedSignatureInput().getBytes(StandardCharsets.UTF_8), jws.getSignature());
```

The verification path is fully SPI-driven via `SignatureProvider`, reading the algorithm from the JWT header.

## Gap

**No gap — fully SPI-driven.**

Kubernetes service account JWT verification uses the same `SignatureProvider` SPI pattern as Domains 3, 5, 6, 9, 10, 17, and 21 (inbound verification).

## Current PQC State

**PENDING PROVIDERS**

Signature verification is entirely SPI-driven and will support ML-DSA automatically once ML-DSA `SignatureProvider` exists. No independent code gap.

## Required Changes

**None.** Automatically resolved when ML-DSA providers exist.

## Dependencies

1. **ML-DSA `SignatureProvider`** (tracked under [#48821](https://github.com/keycloak/keycloak/issues/48821) / [#48824](https://github.com/keycloak/keycloak/issues/48824))
2. **Kubernetes API server** must support ML-DSA-signed service account tokens (external dependency)

## GitHub Issue Status

No new issue needed.

Kubernetes IdP verification is covered by the core ML-DSA provider work under [#48824](https://github.com/keycloak/keycloak/issues/48824). Once providers exist, Kubernetes service account JWTs will automatically support ML-DSA with zero code changes.

## What this means for operators

**When migrating to PQC:**

1. **Kubernetes API server** must be updated to sign service account tokens with ML-DSA
2. **Keycloak will automatically verify** ML-DSA-signed tokens once ML-DSA providers exist
3. **No Keycloak configuration changes needed** — the algorithm is read from the JWT's `alg` header
4. **Hybrid deployments supported:** Some Kubernetes clusters can use ML-DSA while others continue with RSA/ECDSA

**Timeline dependency:** This depends on Kubernetes upstream supporting PQC signatures for service account tokens.

## Related Domains

- **Domain 10** — private_key_jwt Client Authentication (uses the same `SignatureProvider` SPI pattern)
- **Domain 17** — JWT Authorization Grant (similar inbound token verification)
- **Domain 21** — OIDC IdP Broker (similar identity provider verification path)

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
