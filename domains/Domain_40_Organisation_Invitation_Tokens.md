# Domain 40 — Organisation Invitation Token Verification

[← Back to PQC Overview](pqc_overview.html)

## What is this?

Organisation invitation tokens are signed JWTs verified using the **SignatureProvider SPI**.

**File:** `services/src/main/java/org/keycloak/organization/utils/Organizations.java:196`

```java
SignatureVerifierContext verifierContext = CryptoUtils.getSignatureProvider(
    session, 
    verifier.getHeader().getAlgorithm().name()
).verifier(verifier.getHeader().getKeyId());
```

**How it works:**

1. Invitation token is signed with realm key (any algorithm: RS*, PS*, ES*, EdDSA, or **ML-DSA** once providers exist)
2. Token header includes `alg` and `kid`
3. Verification uses `SignatureProvider` SPI to look up the verifier for that algorithm
4. 100% algorithm-agnostic — no hardcoded algorithm logic

## Gap

**None.**

Already algorithm-agnostic via SPI. Will support ML-DSA automatically once **GAP-15** (key generation) is fixed.

## Current PQC State

**PENDING PROVIDERS**

Will support ML-DSA once providers exist. No independent gap.

## Required Changes

**None specific to this domain.**

Depends only on **GAP-15** (ML-DSA key generation provider).

## Dependencies

**GAP-15** — ML-DSA key generation ([#48824](https://github.com/keycloak/keycloak/issues/48824))

## GitHub Issue Status

No dedicated issue needed. Covered by GAP-15.

## What this means for operators

**Today:**

Invitation tokens can use RS256, PS256, ES256, EdDSA — any algorithm with a realm signing key.

**When GAP-15 is fixed:**

Operators can configure realms to use **ML-DSA** keys. Organisation invitation tokens will automatically use ML-DSA signing with no code changes to Domain 40.

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
