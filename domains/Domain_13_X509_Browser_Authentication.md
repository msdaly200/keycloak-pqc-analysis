# Domain 13 — X.509 Browser Authentication Flow

[← Back to PQC Overview](pqc_overview.html)

## What is this?

X.509 Browser Authentication allows end-users to authenticate to Keycloak using an X.509 client certificate presented by their web browser during the TLS handshake (mutual TLS). This is a browser-based authentication flow, not client authentication.

When a user navigates to Keycloak with a client certificate:
1. The browser presents the user's X.509 certificate during the TLS handshake
2. Keycloak validates the certificate chain against configured trust anchors
3. Keycloak checks certificate revocation status via CRL or OCSP (if enabled)
4. Keycloak checks key usage, extended key usage, and certificate policy extensions
5. Keycloak matches the certificate subject or other attributes to a user account
6. The user is authenticated without requiring a password

This is commonly used in high-security environments (government, enterprise) where users have personal certificates issued by an organizational PKI.

## Gap

**No gap at the application layer.**

### File: `X509ClientCertificateAuthenticator.java` (lines 91-98)

```java
CertificateValidator.CertificateValidatorBuilder builder = certificateValidationParameters(context.getSession(), config);
CertificateValidator validator = builder.build(certs);
validator.validateTrust()
         .validateTimestamps()
         .validateKeyUsage()
         .validateExtendedKeyUsage()
         .validatePolicy()
         .checkRevocationStatus();
```

This uses **exactly the same `CertificateValidator` class** as Domain 11 (X.509 mTLS Client Authentication). All validation logic delegates to the JVM's `CertPathBuilder` via `CryptoProvider` (line 675 of `CertificateValidator.java`).

### Certificate Chain Validation

**File:** `CertificateValidator.java` (line 675)

```java
CertPathBuilder builder = CryptoIntegration.getProvider().getCertPathBuilder();
PKIXCertPathBuilderResult result = (PKIXCertPathBuilderResult) builder.build(pkixParams);
```

Certificate chain validation is **algorithm-agnostic at the application layer**. The JVM's certificate path builder can handle ML-DSA, FN-DSA, or SLH-DSA signed certificates if the underlying crypto provider supports them.

### OCSP Response Verification

**File:** `CertificateValidator.java` (lines 209, 226)

```java
OCSPProvider ocspProvider = CryptoIntegration.getProvider().getOCSPProver(OCSPProvider.class);
ocspRevocationStatus = ocspProvider.check(cert, issuerCertificate, responderUri, responderCert);
```

OCSP response verification delegates to BouncyCastle's `OCSPProvider`. BouncyCastle can parse and verify OCSP responses signed with PQC algorithms in recent versions.

### CRL Verification

CRL verification uses `CertificateFactory.getInstance("X.509")` and the JVM's `CertStore` — algorithm-agnostic.

### Key Usage / Extended Key Usage / Certificate Policy

These are OID-based checks on certificate extensions — no cryptographic operations involved.

## Current PQC State

**SAFE** — with a FIPS deployment caveat.

This domain shares the **exact same conclusion** as Domain 11 (X.509 mTLS Client Authentication):

- **Non-FIPS mode**: BouncyCastle (default crypto provider) can parse and validate ML-DSA/FN-DSA/SLH-DSA certificates today (BC 1.79+)
- **FIPS mode**: BouncyCastle FIPS (BC-FIPS) version 2.1.2 does **not yet support PQC certificates** — this is a hard blocker for FIPS deployments

This FIPS blocker is tracked as **GAP-6** across all certificate-based authentication domains.

## Required Changes

**None for non-FIPS deployments.**

For FIPS deployments: **no Keycloak code changes required** — this is a dependency-based fix waiting for BC-FIPS 2.x with ML-DSA/FN-DSA/SLH-DSA support.

### What Keycloak code does NOT need changing

| Component | Why it's already PQC-ready |
|-----------|---------------------------|
| Certificate chain validation (`CertificateValidator.java` line 675) | Delegates to `CertPathBuilder` via `CryptoProvider` SPI — algorithm-agnostic |
| OCSP response verification (lines 209, 226) | Delegates to BouncyCastle's `OCSPProvider`, which supports PQC in non-FIPS mode |
| CRL verification (`CRLLoaderImpl` implementations) | Uses JVM's `CertificateFactory` and `CertStore` — algorithm-agnostic |
| Key usage enforcement (line 468-499) | OID-based bit checks — no cryptographic operation |
| Extended key usage enforcement (line 526-546) | OID string matching — no cryptographic operation |
| Certificate policy enforcement (line 548-579) | OID string matching — no cryptographic operation |
| Subject DN / SAN matching | String comparison — no cryptographic operation |

## Dependencies

### For non-FIPS deployments
- **BouncyCastle 1.79+** (already included) — supports ML-DSA, FN-DSA, SLH-DSA certificate parsing and verification
- **Issuing CA must use a PQC signature algorithm** — Keycloak validates whatever the CA issues; the CA is responsible for migrating to ML-DSA/FN-DSA/SLH-DSA

### For FIPS deployments
- **BouncyCastle FIPS 2.x** with ML-DSA/FN-DSA/SLH-DSA support (not yet released as of this analysis)
- Tracked as **GAP-6**: "BC-FIPS does not yet support PQC certificates"

## What this means for operators

**When migrating to PQC:**

1. **Browser authentication with PQC certificates works immediately** in non-FIPS mode — no code change, no Keycloak configuration change required.

2. **The issuing CA migration is the critical path** — users must be issued certificates signed with ML-DSA, FN-DSA, or SLH-DSA by their organizational PKI. This is an infrastructure change outside Keycloak's control.

3. **Keycloak's truststore** (configured in the realm's X.509 authentication settings) must include the CA's root certificate — whether that CA uses RSA, ECDSA, or ML-DSA is irrelevant to the trust anchor lookup.

4. **OCSP and CRL endpoints** must also support PQC-signed responses if revocation checking is enabled. The OCSP responder's certificate and CRL signing certificate may be PQC-signed in a post-quantum PKI.

5. **FIPS deployments**: cannot use PQC user certificates until BC-FIPS releases support. Monitor [BouncyCastle's FIPS releases](https://www.bouncycastle.org/fips-java/).

**No Keycloak realm configuration changes needed** — the user certificate's algorithm is transparent to Keycloak's validation logic.

## Differences from Domain 11 (X.509 mTLS Client Authentication)

The **only difference** is the context:
- **Domain 11**: Client authentication (OAuth2/OIDC clients authenticating to the token endpoint)
- **Domain 13**: End-user authentication (human users authenticating via browser)

Both domains:
- Use the **same `CertificateValidator` class**
- Delegate to the **same `CertPathBuilder` / `CryptoProvider` SPI**
- Have the **same SAFE PQC state**
- Share the **same GAP-6 FIPS blocker**

## GitHub Issue Status

No new issue needed.

- **GAP-6** (BC-FIPS blocker) is already identified in the overview table and should be tracked under the broader FIPS crypto provider work (likely a sub-issue under [#43690](https://github.com/keycloak/keycloak/issues/43690) or [#48820](https://github.com/keycloak/keycloak/issues/48820)).
- Operator migration guidance (informing operators that the PKI migration is the critical path, not Keycloak code changes) should be documented in [#48823](https://github.com/keycloak/keycloak/issues/48823) (Operator migration guidance).

## Related Domains

- **Domain 11** — X.509 mTLS Client Authentication: **identical** PQC state, uses the same `CertificateValidator`
- **Domain 42** — CryptoProvider — BouncyCastle Default (supports PQC)
- **Domain 43** — CryptoProvider — FIPS 140-2 (BC-FIPS) (GAP-6 blocker)

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness