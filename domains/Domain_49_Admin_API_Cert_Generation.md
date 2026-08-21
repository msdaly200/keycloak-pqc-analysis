# Domain 49 — Admin API — Client Certificate & Keypair Generation

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

Admin API endpoints for generating client keypairs and self-signed certificates. Used by admins to create client credentials without external tools.

## Gap

**Hardcoded RSA key generation.**

**File:** `services/src/main/java/org/keycloak/services/resources/admin/ClientAttributeCertificateResource.java:119, 258`

**Current implementation:**

```java
// Line 119:
CertificateRepresentation info = KeycloakModelUtils.generateKeyPairCertificate(client.getClientId());

// Line 258:
CertificateRepresentation info = KeycloakModelUtils.generateKeyPairCertificate(client.getClientId(), keySize, calendar);
```

Both call `KeycloakModelUtils.generateKeyPairCertificate()`, which **hardcodes RSA key generation** via `KeyUtils.generateRsaKeyPair(keysize)`.

**The problem:**

- No algorithm parameter in the API
- No algorithm selector in the admin UI
- Admin-generated client keypairs will **always be RSA**, regardless of realm PQC configuration

## Current PQC State

**BLOCKED**

## Required Changes

### Change 1: Add algorithm parameter to API

```java
@Path("generate-and-download")
@POST
public Response generateAndGetKeystore(@QueryParam("algorithm") String algorithm) {
    String alg = algorithm != null ? algorithm : "RS256";  // default RS256
    CertificateRepresentation info = KeycloakModelUtils.generateKeyPairCertificate(
        client.getClientId(), keySize, calendar, alg);
    // ...
}
```

### Change 2: Update KeycloakModelUtils

```java
public static CertificateRepresentation generateKeyPairCertificate(
    String subject, int keySize, Calendar expirationDate, String algorithm) {
    
    KeyPair keyPair;
    switch (algorithm) {
        case "RS256": keyPair = KeyUtils.generateRsaKeyPair(keySize); break;
        case "ML-DSA-65": keyPair = KeyUtils.generateMLDsaKeyPair(); break;
        // ...
    }
    // ...
}
```

### Change 3: Add UI dropdown

Admin console should offer algorithm selection (RS256, ML-DSA-65, etc.).

## Dependencies

**BouncyCastle ML-DSA support** for key generation (already available)

## GitHub Issue Status

Needs GitHub issue under [#43690](https://github.com/keycloak/keycloak/issues/43690) with MEDIUM priority

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
