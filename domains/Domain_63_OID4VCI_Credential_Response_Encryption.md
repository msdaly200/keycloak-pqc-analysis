# Domain 63 — OID4VCI Credential Response Encryption

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

OID4VCI (OpenID for Verifiable Credential Issuance) credential response encryption. When a credential client requests encrypted responses, `OID4VCIssuerEndpoint` validates the requested encryption parameters against `CredentialResponseEncryptionMetadata` and dispatches to a `CekManagementProviderFactory` SPI to perform the actual key encapsulation.

## Key Files

- `services/src/main/java/org/keycloak/protocol/oid4vc/issuance/OID4VCIssuerEndpoint.java` (lines ~855–866+)
- `core/src/main/java/org/keycloak/representations/oid4vci/CredentialResponseEncryption.java`
- `core/src/main/java/org/keycloak/protocol/oid4vc/model/CredentialResponseEncryptionMetadata.java`

## Gap

**No ML-KEM `CekManagementProviderFactory` (GAP-3)**

The credential response encryption path in `OID4VCIssuerEndpoint` validates the client-requested `enc` / `zip` / `jwk` fields against `CredentialResponseEncryptionMetadata` and dispatches to the `CekManagementProvider` SPI. No ML-KEM `CekManagementProviderFactory` exists in Keycloak, so a credential client requesting ML-KEM key encapsulation will fail.

`CredentialResponseEncryption` holds the `enc`, `zip`, and `jwk` fields from the OIDC4VCI spec. The model is spec-driven and algorithm-agnostic — no hardcoding here — but the SPI implementation gap means no ML-KEM CEK is available at runtime.

**Fix:** Implement an ML-KEM `CekManagementProviderFactory` (tracked under GAP-3). Once GAP-3 is resolved, OID4VCI credential response encryption gains ML-KEM support without further changes to `OID4VCIssuerEndpoint`.

## Current PQC State

**BLOCKED**

Blocked on GAP-3 (no ML-KEM `CekManagementProviderFactory`). The issuance endpoint itself is architecture-correct (SPI-driven); the gap is entirely in the missing provider implementation.

## Required Changes

1. Implement `MlKemCekManagementProviderFactory` (GAP-3 / [#50292](https://github.com/keycloak/keycloak/issues/50292) or related sub-issue).
2. Once GAP-3 is resolved, verify `CredentialResponseEncryptionMetadata` advertises ML-KEM algorithms in OIDC4VCI discovery.

## Dependencies

- **GAP-3** — ML-KEM `CekManagementProviderFactory` missing
- **GAP-4** — `DefaultKeyProviders` / no AKP bootstrap (for encryption key generation)

## GitHub Issue Status

No dedicated GitHub issue yet — should be tracked under the PQC readiness parent ([#43690](https://github.com/keycloak/keycloak/issues/43690)) or the GAP-3 sub-issue.

## Related Domains

- **Domain 40** — JWEUtils / JWERegistry / JWEHeader (same CEK SPI gap, GAP-3)
- **Domain 41** — RsaCekManagement / EcdhEsCekManagement (no ML-KEM CEK factory)
- **Domain 62** — OID4VP Identity Provider (related OID4V* domain)

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
