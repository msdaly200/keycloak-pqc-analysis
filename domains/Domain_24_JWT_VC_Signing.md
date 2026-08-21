# Domain 24 — JWT-VC / SD-JWT Credential Signing

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

OID4VC (OpenID for Verifiable Credentials) allows Keycloak to act as a credential issuer, signing verifiable credentials in two formats:

1. **JWT-VC** — Simple signed JWT containing credential claims
2. **SD-JWT** — Selective Disclosure JWT with privacy-preserving claim disclosure

**How it works:**

1. A credential holder requests a verifiable credential from Keycloak
2. Keycloak builds the credential payload according to the credential type configuration
3. Keycloak signs the credential using the algorithm specified in `CredentialBuildConfig.signingAlgorithm`
4. The signed credential is returned to the holder
5. The holder can present the credential to verifiers, who verify Keycloak's signature

**File:** `AbstractCredentialSigner.java` (lines 59-62)

```java
SignatureProvider signatureProvider = keycloakSession
        .getProvider(SignatureProvider.class, credentialBuildConfig.getSigningAlgorithm());

return signatureProvider.signer(signingKey);
```

The signing path is fully SPI-driven via `SignatureProvider`, reading the algorithm from the credential configuration.

## Gap

**No gap — fully SPI-driven.**

Verifiable credential signing uses the same `SignatureProvider` SPI pattern as all other signing domains.

## Current PQC State

**PENDING PROVIDERS**

Credential signing is entirely SPI-driven and will support ML-DSA automatically once ML-DSA `SignatureProvider` exists. No independent code gap.

## Required Changes

**None.** Automatically resolved when ML-DSA providers exist.

## Dependencies

1. **ML-DSA `SignatureProvider`** (tracked under [#48821](https://github.com/keycloak/keycloak/issues/48821) / [#48824](https://github.com/keycloak/keycloak/issues/48824))
2. **Credential type configuration** — admin must configure which algorithm to use for each credential type
3. **Verifier support** — verifiers (relying parties) must be able to verify ML-DSA signatures

## GitHub Issue Status

No new issue needed.

OID4VC credential signing is covered by the core ML-DSA provider work under [#48824](https://github.com/keycloak/keycloak/issues/48824). Once providers exist, credentials can be signed with ML-DSA with zero code changes.

**Note on OID4VC-specific issues:**

Other OID4VC domains have specific gaps:
- **Domain 25** — Linked Data Proof signing (GAP-7, hardcoded Ed25519)
- **Domain 26** — JWT proof validation ([#48415](https://github.com/keycloak/keycloak/issues/48415), CLOSED)
- **Domain 27** — c_nonce JWT signing (GAP-18, hardcoded ES256/RS256)

Domain 24 (JWT-VC/SD-JWT credential signing) has no such gaps — it's purely SPI-driven.

## What this means for operators

**When migrating to PQC:**

1. **Configure credential types** to use ML-DSA algorithms:
   - Set `signingAlgorithm` to `ML-DSA-65` (or desired variant) in credential type configuration
   - This is typically done via the OID4VC admin UI or realm configuration
2. **Keycloak will sign credentials** with ML-DSA once providers exist
3. **Verifiers must support ML-DSA** — the verifiable credential ecosystem must adopt PQC verification
4. **Hybrid deployments supported:** Different credential types can use different algorithms during migration

**OID4VC ecosystem dependency:**

The broader verifiable credentials ecosystem (wallets, verifiers, credential schemas) must also migrate to PQC. This includes:
- Mobile wallet apps must verify ML-DSA signatures
- Verifier applications must support ML-DSA
- VC standards (W3C VC Data Model, OID4VC) must define ML-DSA usage

## Related Domains

- **Domain 1** — Access Token / ID Token Signing (uses the same `SignatureProvider` SPI for JWT signing)
- **Domain 56** — SD-JWT Issuer Signing (SD-JWT-specific signing, same SPI pattern)
- **Domain 25** — LD-Proof Credential Signing (Linked Data proofs, different signing mechanism)

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
