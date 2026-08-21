# Domain 53 — Client Asymmetric Signature Verifier Context & Providers

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

Full client-side signature verifier stack for `private_key_jwt` client authentication. Multiple provider implementations for different algorithms.

## Gap

**RSA path hardcoded KeyType.RSA guard; no AKP verifier context/provider.**

**File:** `services/src/main/java/org/keycloak/crypto/ClientAsymmetricSignatureVerifierContext.java:36`

**Current providers:**
- ✅ `ClientAsymmetricSignatureVerifierContext` → RSA only (line 36: throws for non-RSA)
- ✅ `ClientECDSASignatureVerifierContext` → ECDSA
- ✅ `ClientEdDSASignatureVerifierContext` → EdDSA
- ❌ NO `ClientMLDSASignatureVerifierContext` → ML-DSA

**The problem:**

`ClientAsymmetricSignatureVerifierContext.getKey()` line 36:

```java
if (!KeyType.RSA.equals(key.getType())) {
    throw new VerificationException("Key Type is not RSA");
}
```

This **actively rejects** any non-RSA key in the RSA path. ECDSA and EdDSA have their own dedicated contexts, but **ML-DSA has no routed path**.

## Current PQC State

**BLOCKED** (RSA path) / **PENDING PROVIDERS** (ECDSA/EdDSA paths)

## Required Changes

**Add ML-DSA verifier context + provider (following ECDSA/EdDSA pattern):**

### New files:
1. `ClientMLDSASignatureVerifierContext.java`
2. `MLDSAClientSignatureVerifierProvider.java`
3. `MLDSAClientSignatureVerifierProviderFactory.java`

### Pattern to follow:

```java
public class ClientMLDSASignatureVerifierContext extends AsymmetricSignatureVerifierContext {
    public ClientMLDSASignatureVerifierContext(KeyWrapper key) throws VerificationException {
        super(getSignatureProvider(key));
    }
    
    private static SignatureProvider getSignatureProvider(KeyWrapper key) {
        if (!KeyType.AKP.equals(key.getType())) {
            throw new VerificationException("Key Type is not AKP");
        }
        // ... ML-DSA provider lookup
    }
}
```

## Dependencies

**GAP-15** — ML-DSA SignatureProvider ([#48824](https://github.com/keycloak/keycloak/issues/48824))

## GitHub Issue Status

Needs GitHub issue under [#43690](https://github.com/keycloak/keycloak/issues/43690) with HIGH priority

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
