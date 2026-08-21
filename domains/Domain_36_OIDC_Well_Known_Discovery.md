# Domain 36 — OIDC Well-Known Discovery — Algorithm Advertisement

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

OIDC `.well-known/openid-configuration` endpoint advertises supported algorithms to clients.

## Gap

**Hardcoded DEFAULT_CLIENT_AUTH_SIGNING_ALG_VALUES_SUPPORTED (GAP-23).**

**File:** `services/src/main/java/org/keycloak/protocol/oidc/OIDCWellKnownProvider.java:85`

```java
public static final List<String> DEFAULT_CLIENT_AUTH_SIGNING_ALG_VALUES_SUPPORTED = list(Algorithm.RS256.toString());
```

**The problem:**

Main token signing algorithm lists (e.g. `id_token_signing_alg_values_supported`, `userinfo_signing_alg_values_supported`) are **dynamically populated** from registered `SignatureProvider` implementations — they will auto-include ML-DSA once providers exist.

However, **four client authentication-related discovery fields** use the hardcoded `DEFAULT_CLIENT_AUTH_SIGNING_ALG_VALUES_SUPPORTED` constant:
- `request_object_signing_alg_values_supported`
- `backchannel_authentication_request_signing_alg_values_supported`  
- `authorization_signing_alg_values_supported`
- `authorization_encryption_alg_values_supported`

**Impact:** ML-DSA will work in practice (runtime uses SPI), but conformant clients won't discover it's supported.

## Current PQC State

**PARTIAL**

## Required Changes

Replace static `DEFAULT_CLIENT_AUTH_SIGNING_ALG_VALUES_SUPPORTED` with dynamic lookup:

```java
// Before (line 85):
public static final List<String> DEFAULT_CLIENT_AUTH_SIGNING_ALG_VALUES_SUPPORTED = list(Algorithm.RS256.toString());

// After:
private List<String> getClientAuthSigningAlgs() {
    return session.getAllProviders(SignatureProvider.class).stream()
        .filter(p -> p.isAsymmetricAlgorithm())
        .map(p -> p.getAlgorithm())
        .collect(Collectors.toList());
}
```

Same pattern already used for `id_token_signing_alg_values_supported` — apply to client auth fields.

## Dependencies

**SignatureProvider SPI** — dynamic lookup depends on registered providers

## GitHub Issue Status

**GAP-23** — needs new GitHub issue under [#43690](https://github.com/keycloak/keycloak/issues/43690)

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
