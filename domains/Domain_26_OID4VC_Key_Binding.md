# Domain 26 — OID4VC Key Binding — JWT Proof Validation

[← Back to PQC Overview](pqc_overview.html)

## What is this?

In OID4VC (OpenID for Verifiable Credentials), **key binding** proves that a wallet possesses the private key corresponding to a public key that will be embedded in a credential. This prevents an attacker from requesting a credential and then transferring it to another wallet.

**How it works:**

1. A wallet requests a verifiable credential from Keycloak
2. Keycloak issues a `c_nonce` (cryptographic nonce) to prevent replay attacks
3. The wallet generates a **key-binding proof JWT** signed with its private key, containing:
   - The wallet's public key (in `jwk` header or `cnf` claim)
   - The `c_nonce` value
   - The audience (Keycloak's credential issuer URL)
4. Keycloak validates the proof JWT signature to verify the wallet possesses the private key
5. Keycloak issues the credential with the wallet's public key embedded in it
6. Only that wallet can present the credential (because only it has the private key)

**File:** `JwtProofValidator.java` (line 278)

```java
if (!CryptoUtils.getSupportedAsymmetricSignatureAlgorithms(keycloakSession).contains(alg)) {
    throw new VCIssuerException(ErrorType.INVALID_PROOF, "Proof signature algorithm not supported: " + alg);
}
```

The proof validation is SPI-driven via `getSupportedAsymmetricSignatureAlgorithms()`, which dynamically lists all registered asymmetric `SignatureProvider` implementations.

## Gap

**No gap — fully SPI-driven.**

Key-binding proof validation uses the same `SignatureProvider` SPI pattern as all other signature verification domains.

## Current PQC State

**PENDING PROVIDERS**

Proof JWT signature verification is entirely SPI-driven and will support ML-DSA automatically once ML-DSA `SignatureProvider` exists. No independent code gap.

**Note:** The overview mentions "x5c/attestation proof path not fully analysed" — this refers to an alternative proof mechanism using X.509 certificates (`AttestationValidatorUtil.java`). This path likely delegates to certificate validation (similar to Domains 11/13) and should be quantum-safe once PQC certificates are supported.

## Required Changes

**None.** Automatically resolved when ML-DSA providers exist.

## Dependencies

1. **ML-DSA `SignatureProvider`** (tracked under [#48821](https://github.com/keycloak/keycloak/issues/48821) / [#48824](https://github.com/keycloak/keycloak/issues/48824))
2. **Wallet support** — mobile wallets and credential holders must be able to sign key-binding proofs with ML-DSA

## GitHub Issue Status

**Already resolved.**

[#48415](https://github.com/keycloak/keycloak/issues/48415) — "[OID4VCI] Review JwtProofValidator.JWK_PRIVATE_KEY_CLAIMS" — **CLOSED**

This issue reviewed whether the JWK private key claims check was correct. No PQC-specific issue is needed; key-binding proof validation is covered by the core ML-DSA provider work under [#48824](https://github.com/keycloak/keycloak/issues/48824).

## What this means for operators

**When migrating to PQC:**

1. **Wallets generate ML-DSA key pairs** for credential binding
2. **Wallets sign key-binding proofs** with ML-DSA when requesting credentials
3. **Keycloak automatically validates** ML-DSA-signed proofs once ML-DSA providers exist
4. **No Keycloak configuration changes needed** — the algorithm is read from the proof JWT's `alg` header
5. **Hybrid deployments supported:** Some wallets can use ML-DSA while others continue with RSA/ECDSA

**Timeline dependency:** This depends on mobile wallet apps and credential holder software supporting ML-DSA key generation and signing.

## Related Domains

- **Domain 10** — private_key_jwt Client Authentication (similar proof-of-possession pattern)
- **Domain 12** — Attestation-Based Client Authentication (similar key-binding with `cnf.jwk`)
- **Domain 24** — JWT-VC / SD-JWT Credential Signing (the credentials that get bound to these keys)
- **Domain 27** — OID4VC c_nonce JWT Signing (the nonce that prevents replay attacks)

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
