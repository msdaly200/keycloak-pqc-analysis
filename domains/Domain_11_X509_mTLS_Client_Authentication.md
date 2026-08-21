# Domain 11 — X.509 mTLS Client Authentication

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

X.509 mTLS (mutual TLS) Client Authentication allows OAuth2/OIDC clients to authenticate using X.509 client certificates instead of client secrets or JWT assertions. This is defined in RFC 8705 (OAuth 2.0 Mutual-TLS Client Authentication and Certificate-Bound Access Tokens).

When a client connects with a client certificate:
1. Keycloak validates the certificate chain against a configured trust anchor
2. Keycloak checks certificate revocation status (via CRL or OCSP if enabled)
3. Keycloak matches the certificate subject DN or other attributes against the client's stored configuration
4. Optionally, Keycloak can bind access tokens to the certificate using a SHA-256 thumbprint (Holder-of-Key binding)

The quantum vulnerability lies **entirely in the certificate's own signing algorithm** — which is determined by the issuing Certificate Authority (CA), not by Keycloak. Keycloak's role is to validate the certificate chain, which it delegates to the JVM's `CertPathBuilder` via the `CryptoProvider` SPI.

## Gap

**No gap at the application layer.**

### File: `CertificateValidator.java` (line 675)
```java
CertPathBuilder builder = CryptoIntegration.getProvider().getCertPathBuilder();
PKIXCertPathBuilderResult result = (PKIXCertPathBuilderResult) builder.build(pkixParams);
```

Certificate chain validation is fully delegated to the JVM's `CertPathBuilder`, obtained through Keycloak's `CryptoProvider` SPI. This is **algorithm-agnostic at the application layer** — the JVM's certificate path validator can already handle ML-DSA, FN-DSA, or SLH-DSA signed certificates if the underlying crypto provider (e.g., BouncyCastle) supports them.

### File: `MtlsHoKTokenUtil.java` (line 38)
```java
private static final String DIGEST_ALG = "SHA-256";
```

The Holder-of-Key (HoK) token binding mechanism uses **SHA-256** to compute the certificate thumbprint (`x5t#S256` claim in the `cnf` field of access tokens). SHA-256 is a **quantum-safe hash function** — Grover's algorithm provides only a quadratic speedup, requiring ~2^128 operations to find a collision, which remains infeasible.

### OCSP Response Verification

OCSP responses are verified using BouncyCastle's `OCSPProvider` (line 209, 226 in `CertificateValidator.java`). BouncyCastle already supports parsing and verifying OCSP responses signed with PQC algorithms in recent versions.

## Current PQC State

**SAFE** — with a FIPS deployment caveat.

Keycloak's X.509 mTLS client authentication code does not contain any hardcoded algorithm assumptions or quantum-vulnerable operations. The certificate's signing algorithm is determined by the issuing CA, not by Keycloak. 

- **Non-FIPS mode**: BouncyCastle (the default crypto provider) can parse and validate ML-DSA/FN-DSA/SLH-DSA certificates today (as of BC 1.79+).
- **FIPS mode**: BouncyCastle FIPS (BC-FIPS) version 2.1.2 (current as of this analysis) does **not yet support PQC certificates**. FIPS deployments are **hard-blocked** until BC-FIPS 2.x ships with ML-DSA/FN-DSA/SLH-DSA certificate support.

This FIPS blocker is tracked as **GAP-6** across all certificate-based authentication domains (rows 11, 13).

## Required Changes

**None for non-FIPS deployments.**

For FIPS deployments: **no Keycloak code changes required** — this is a dependency-based fix.

### What Keycloak code does NOT need changing

| Component | Why it's already PQC-ready |
|-----------|---------------------------|
| Certificate chain validation (`CertificateValidator.java` line 675) | Delegates to `CertPathBuilder` via `CryptoProvider` SPI — algorithm-agnostic |
| Holder-of-Key thumbprinting (`MtlsHoKTokenUtil.java` line 38) | Uses SHA-256, which is quantum-safe |
| OCSP response verification (line 209, 226) | Delegates to BouncyCastle's `OCSPProvider`, which supports PQC in non-FIPS mode |
| CRL verification | Delegated to `CertificateFactory` and JVM's `CertStore` — algorithm-agnostic |
| Subject DN matching | String comparison — no cryptographic operation |
| Key usage / EKU enforcement | OID-based checks — no cryptographic operation |

## Dependencies

### For non-FIPS deployments
- **BouncyCastle 1.79+** (already included) — supports ML-DSA, FN-DSA, SLH-DSA certificate parsing and verification
- **Issuing CA must use a PQC signature algorithm** — Keycloak validates whatever the CA issues; the CA is responsible for migrating to ML-DSA/FN-DSA/SLH-DSA

### For FIPS deployments
- **BouncyCastle FIPS 2.x** with ML-DSA/FN-DSA/SLH-DSA support (not yet released as of this analysis)
- Tracked as **GAP-6**: "BC-FIPS does not yet support PQC certificates"

## What this means for operators

**When migrating to PQC:**
1. Keycloak's X.509 mTLS client authentication will work with PQC-signed client certificates **immediately** in non-FIPS mode — no code change, no configuration change required.
2. The issuing CA must be configured to issue certificates signed with ML-DSA, FN-DSA, or SLH-DSA. This is an infrastructure change outside Keycloak's control.
3. The Keycloak truststore (used by `TruststoreProvider`) must include the CA's root certificate — whether that CA uses RSA, ECDSA, or ML-DSA is irrelevant to the trust anchor lookup.
4. **FIPS deployments**: cannot use PQC client certificates until BC-FIPS releases support. Monitor [BouncyCastle's FIPS releases](https://www.bouncycastle.org/fips-java/).

**No Keycloak realm configuration changes needed** — the client certificate's algorithm is transparent to Keycloak's validation logic.

## GitHub Issue Status

No new issue needed.

- **GAP-6** (BC-FIPS blocker) is already identified in the overview table and should be tracked under the broader FIPS crypto provider work (likely a sub-issue under [#43690](https://github.com/keycloak/keycloak/issues/43690) or [#48820](https://github.com/keycloak/keycloak/issues/48820)).
- Operator migration guidance (informing operators that the CA migration is the critical path, not Keycloak code changes) should be documented in [#48823](https://github.com/keycloak/keycloak/issues/48823) (Operator migration guidance).

## Related Domains

- **Domain 13** — X.509 Browser Authentication Flow: same SAFE state, same GAP-6 FIPS blocker
- **Domain 42** — CryptoProvider — BouncyCastle Default (supports PQC)
- **Domain 43** — CryptoProvider — FIPS 140-2 (BC-FIPS) (GAP-6 blocker)

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness