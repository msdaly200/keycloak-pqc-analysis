# Domain 38 — kcadm.sh / kcreg.sh — private_key_jwt Auth

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

Admin CLI tools (`kcadm.sh` and `kcreg.sh`) authenticate to Keycloak using `private_key_jwt` client authentication.

## Gap

**Hardcoded RS256 (GAP-13).**

**File:** `integration/client-cli/admin-cli/src/main/java/org/keycloak/client/cli/util/AuthUtil.java:211-213`

```java
String signedRequestToken = new JWSBuilder()
    .jsonContent(sigContent)
    .rsa256(keypair.getPrivate());
```

**The problem:**

`AuthUtil.getSignedRequestToken()` **always** produces an RS256-signed JWT, regardless of:
- Realm configuration
- What algorithms the realm supports
- What algorithm the client requested

**No `--sigalg` or `--algorithm` CLI parameter exists.**

**Impact:** If a realm requires ML-DSA for `private_key_jwt` client assertions, both `kcadm.sh` and `kcreg.sh` will fail to authenticate via the keystore flow.

## Current PQC State

**BLOCKED**

## Required Changes

### Change 1: Add algorithm parameter

**File:** `AuthUtil.java`

```java
// Add algorithm parameter:
private static String getSignedRequestToken(String keystore, String storePass, 
    String keyPass, String alias, String algorithm) {
    
    // Replace hardcoded .rsa256() with dynamic selection:
    SignatureProvider sigProvider = SignatureProviderFactory.create(algorithm);
    String signedRequestToken = new JWSBuilder()
        .jsonContent(sigContent)
        .sign(sigProvider, keypair.getPrivate());
}
```

### Change 2: Add CLI option

Add `--sigalg <algorithm>` option to both `kcadm.sh` and `kcreg.sh` credential configuration commands. Default to RS256 for backward compatibility.

## Dependencies

**BouncyCastle ML-DSA support** for keystore loading

## GitHub Issue Status

**GAP-13** — needs GitHub issue under [#43690](https://github.com/keycloak/keycloak/issues/43690)

Potentially track as sub-issue under [#48824](https://github.com/keycloak/keycloak/issues/48824) (client authentication) or create dedicated CLI issue.

## What this means for operators

**Today:**

```bash
# This always produces RS256, regardless of realm config:
kcadm.sh config credentials --server ... --realm ... \
  --keystore mykeystore.jks --storepass ... --alias ...
```

**When GAP-13 is fixed:**

```bash
# Add --sigalg option:
kcadm.sh config credentials --server ... --realm ... \
  --keystore mykeystore.bcfks --storepass ... --alias ml-dsa-key \
  --sigalg ML-DSA-65
```

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
