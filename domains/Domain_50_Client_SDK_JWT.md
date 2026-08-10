# Domain 50 — Client SDK — JWT Client Credentials Provider

[← Back to PQC Overview](pqc_overview.html)

## What is this?

Client-side `private_key_jwt` assertion signing from the Keycloak adapter/SDK. Used by clients (not the server) to authenticate to Keycloak.

## Gap

**No AKP case in setupKeyPair() switch.**

**File:** `core/src/main/java/org/keycloak/protocol/oidc/client/authentication/JWTClientCredentialsProvider.java`

**Current code:**

```java
switch (JavaAlgorithm.getKeyType(privateKey)) {
    case RSA:
        // ... create RSA signer
    case EC:
        // ... create ECDSA signer
    case OKP:
        // ... create EdDSA signer
    default:
        throw new RuntimeException("Invalid KeyPair algorithm");
}
```

**The problem:**

No `AKP` case in the switch. Passing an ML-DSA key pair will throw `RuntimeException("Invalid KeyPair algorithm")`.

**Impact:** Client SDKs cannot use ML-DSA keys for `private_key_jwt` authentication.

## Current PQC State

**PARTIAL**

## Required Changes

**Add AKP case to switch:**

```java
switch (JavaAlgorithm.getKeyType(privateKey)) {
    case RSA:
        // ... create RSA signer
    case EC:
        // ... create ECDSA signer
    case OKP:
        // ... create EdDSA signer
    case AKP:
        // ... create ML-DSA signer
        return new AsymmetricSignatureSignerContext(keyWrapper);
    default:
        throw new RuntimeException("Invalid KeyPair algorithm");
}
```

## Dependencies

**ML-DSA SignatureProvider** (GAP-15)

## GitHub Issue Status

Needs GitHub issue under [#43690](https://github.com/keycloak/keycloak/issues/43690) with MEDIUM priority

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
