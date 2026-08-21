# Domain 61 — SAML2Signature — Hardcoded RSA-SHA1 Default

[← Back to PQC Overview](../pqc_overview.html)

## What is this?

Low-level SAML XML signature creation and validation utility. Wraps `XMLSignatureUtil` with default signature method.

## Gap

**Hardcoded RSA-SHA1 default (GAP-2).**

**File:** `saml-core/src/main/java/org/keycloak/saml/processing/api/saml/v2/sig/SAML2Signature.java`

**Current default:**

```java
signatureMethod = SignatureMethod.RSA_SHA1;
digestMethod = DigestMethod.SHA1;
```

**The problem:**

Default signature method is **RSA-SHA1** (deprecated, weak). Related to GAP-2 (no ML-DSA XML Signature URIs). Even after ML-DSA URIs exist, this default needs updating.

**Impact:** When no signature method is explicitly configured, falls back to weak RSA-SHA1.

## Current PQC State

**BLOCKED**

## Required Changes

**Two-step fix:**

### Step 1: Update to stronger classical default (immediate)

```java
// Change from RSA-SHA1 to RSA-SHA256:
signatureMethod = SignatureMethod.RSA_SHA256;
digestMethod = DigestMethod.SHA256;
```

### Step 2: Add ML-DSA option (after GAP-2)

```java
// After ML-DSA XML URIs are defined:
signatureMethod = SignatureMethod.ML_DSA_65;  // when ML-DSA is configured
```

## Dependencies

**GAP-2** — ML-DSA XML Signature URIs ([#50292](https://github.com/keycloak/keycloak/issues/50292))

## GitHub Issue Status

Covered by [#50292](https://github.com/keycloak/keycloak/issues/50292)

## Related Domains

- **Domains 18-20** — SAML signing/verification (same GAP-2 blocker)

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness
