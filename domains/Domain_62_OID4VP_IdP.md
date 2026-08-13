# Domain 62 — OID4VP Identity Provider — Request Signing & Response Encryption

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

Keycloak acting as an OID4VP *verifier*: it signs the request object sent to the wallet (using `OID4VPIdentityProvider.signingKey()`), enforces accepted VP token signature algorithms via `OID4VPIdentityProviderEndpoint.requireAcceptedAlgorithm()`, and uses ECDH-ES / secp256r1 to encrypt the `direct_post.jwt` response.

## Key Files

- `services/src/main/java/org/keycloak/broker/oid4vp/OID4VPIdentityProvider.java`
- `services/src/main/java/org/keycloak/broker/oid4vp/OID4VPIdentityProviderEndpoint.java`
- `services/src/main/java/org/keycloak/broker/oid4vp/OID4VPIdentityProviderConfig.java`
- `services/src/main/java/org/keycloak/broker/oid4vp/OID4VPIdentityProviderFactory.java`
- `services/src/main/java/org/keycloak/broker/oid4vp/RequestObject.java`
- `services/src/main/java/org/keycloak/broker/oid4vp/EphemeralKey.java`
- `services/src/main/java/org/keycloak/broker/oid4vp/ResponseEncryption.java`
- `services/src/main/java/org/keycloak/broker/oid4vp/ClientIdentifier.java`
- `services/src/main/java/org/keycloak/broker/oid4vp/DecryptedResponse.java`
- `services/src/main/java/org/keycloak/broker/oid4vp/ParsedResponse.java`
- `services/src/main/java/org/keycloak/broker/oid4vp/RequestContext.java`
- `services/src/main/java/org/keycloak/broker/oid4vp/ResponseMode.java`

## Gaps

### GAP-24 — Hardcoded ES256 Signing / `ACCEPTED_ALGORITHMS` List (HIGH)

**File:** `broker/oid4vp/OID4VPIdentityProvider.java`

```java
// Line 83
private static final List<String> ACCEPTED_ALGORITHMS = List.of(Algorithm.ES256);

// Lines 196–197 (signingKey())
getKeyIncludingDisabled(realm, kid, KeyUse.SIG, Algorithm.ES256)
getActiveKey(realm, KeyUse.SIG, Algorithm.ES256)
```

`ACCEPTED_ALGORITHMS` is used by `OID4VPIdentityProviderEndpoint.requireAcceptedAlgorithm()` to actively reject all non-ES256 VP token signatures at runtime — including ML-DSA. Even after ML-DSA providers exist, OID4VP will silently fail at the algorithm enforcement layer.

Developer-acknowledged TODO at line 81.

**Fix:** Replace `ACCEPTED_ALGORITHMS` hardcoded list with a configurable/SPI-driven set; update `signingKey()` to select from the configured algorithm. The `SIGNING_KEY_ID` config attribute in `OID4VPIdentityProviderConfig` already provides the configuration surface.

### GAP-25 — Hardcoded ECDH-ES/secp256r1 Response Encryption (LOW — EXTERNAL / spec-gated)

**Files:**
- `broker/oid4vp/EphemeralKey.java` (line 41): `CURVE_SEC = "secp256r1"`
- `broker/oid4vp/ResponseEncryption.java` (line 38): `KEY_MANAGEMENT_ALG = ECDH_ES`

Both are spec-gated: the HAIP (High Assurance Interoperability Profile) currently mandates ECDH-ES/P-256 for `direct_post.jwt` response encryption. No Keycloak-side fix is appropriate until HAIP specifies ML-KEM support.

**Fix:** Track HAIP spec evolution; update once the spec gates lift. No new GitHub issue required at this time — existing HAIP tracker is the dependency.

## Current PQC State

**BLOCKED**

Signing is blocked on both GAP-24 (hardcoded algorithm enforcement) and the upstream `GeneratedAKPKeyProviderFactory` / ML-DSA `SignatureProviderFactory` gaps. Response encryption is blocked on the HAIP specification.

## Required Changes

1. Replace `ACCEPTED_ALGORITHMS = List.of(Algorithm.ES256)` with a configurable/SPI-driven set.
2. Update `signingKey()` to select the key based on the configured algorithm rather than hardcoding `Algorithm.ES256`.
3. Track HAIP spec evolution for ECDH-ES/ML-KEM once the spec gates lift.

## Dependencies

- **GAP-1** — ML-DSA `SignatureProviderFactory` missing
- **GAP-2** — `GeneratedAKPKeyProviderFactory` missing
- **GAP-25** — HAIP spec gate on response encryption

## GitHub Issue Status

No GitHub issue yet — requires a new issue under the PQC tracking parent ([#43690](https://github.com/keycloak/keycloak/issues/43690)).

## Related Domains

- **Domain 63** — OID4VCI Credential Response Encryption (same CEK/encryption gap pattern)
- **Domain 40** — JWEUtils / JWERegistry (GAP-3 / no ML-KEM CEK factory)

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
