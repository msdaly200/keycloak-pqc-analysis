# Domain 16 — CIBA Signed Backchannel Auth Request

## What is this?

CIBA (Client-Initiated Backchannel Authentication) is an OAuth 2.0/OIDC flow defined in [OpenID Connect CIBA Core 1.0](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html) that allows clients to initiate authentication flows where the user authenticates on a separate device (e.g., mobile app) rather than in the browser.

**How it works:**

1. The client sends a **signed backchannel authentication request** to Keycloak's backchannel authentication endpoint
2. The request is a signed JWT (similar to JAR — JWT-secured Authorization Request) containing authentication context (e.g., `login_hint`, `binding_message`, `scope`)
3. Keycloak verifies the JWT signature and validates the request
4. Keycloak notifies the user via a separate channel (push notification, etc.)
5. The user authenticates on their authentication device (AD)
6. The client polls the token endpoint to receive tokens once authentication completes

**CIBA for FAPI:** The FAPI (Financial-grade API) profile requires CIBA backchannel authentication requests to be signed with PS256, PS384, PS512, ES256, ES384, or ES512.

## Gap

**One gap prevents full ML-DSA support:**

### FAPI algorithm allowlist does not include ML-DSA (GAP-8)

**File:** `FapiConstant.java` (lines 30-37)

```java
public static final Set<String> ALLOWED_ALGORITHMS = new LinkedHashSet<>(Arrays.asList(
        Algorithm.PS256,
        Algorithm.PS384,
        Algorithm.PS512,
        Algorithm.ES256,
        Algorithm.ES384,
        Algorithm.ES512
));
```

This static allowlist is used by `SecureCibaAuthenticationRequestSigningAlgorithmExecutor` (line 146) to enforce FAPI-compliant signature algorithms. When a FAPI client policy is active, **ML-DSA algorithms will be rejected** even if ML-DSA `SignatureProvider` exists.

**File:** `SecureCibaAuthenticationRequestSigningAlgorithmExecutor.java` (lines 145-147)

```java
private static boolean isSecureAlgorithm(String sigAlg) {
    return FapiConstant.ALLOWED_ALGORITHMS.contains(sigAlg);
}
```

**Impact:**

- **Without FAPI policy:** CIBA signed requests with ML-DSA will work (verification is SPI-driven)
- **With FAPI policy:** CIBA signed requests with ML-DSA will be rejected during client registration/update

**When can ML-DSA be added?**

The FAPI working group must publish an update to the FAPI CIBA profile that includes PQC algorithms. Until then, adding ML-DSA to the allowlist would violate the FAPI 1.0 Advanced profile specification.

## Current PQC State

**PARTIAL**

The **signature verification path** is fully SPI-driven and will support ML-DSA automatically once providers exist. The **FAPI policy enforcement layer** will reject ML-DSA until the FAPI spec is updated.

### What already works (SPI-driven verification)

**File:** `BackchannelAuthenticationEndpointSignedRequestParser.java` (lines 65-76)

```java
SignatureProvider signatureProvider = session.getProvider(SignatureProvider.class, headerAlgorithm.name());
if (signatureProvider == null) {
    throw new RuntimeException("Not found provider for the algorithm " + headerAlgorithm.name());
}
if (!signatureProvider.isAsymmetricAlgorithm()) {
    throw new RuntimeException("Signed algorithm is not allowed");
}
// ...
this.requestParams = session.tokens().decodeClientJWT(signedAuthReq, client, JsonNode.class);
```

The signature verification:
1. Looks up a `SignatureProvider` by algorithm name (line 65) — **SPI-driven**
2. Checks that it's asymmetric (line 69) — will correctly identify ML-DSA as asymmetric
3. Delegates actual verification to `session.tokens().decodeClientJWT()` (line 76) — uses `SignatureProvider` SPI

Once ML-DSA `SignatureProvider` exists, the verification path will work **automatically**.

### What's blocked (FAPI policy layer)

**File:** `SecureCibaAuthenticationRequestSigningAlgorithmExecutor.java` (lines 126-143)

When a FAPI client policy is active, the executor enforces that `backchannel_authentication_request_signing_alg` is in `FapiConstant.ALLOWED_ALGORITHMS`. This happens during:
- Client registration (line 98-104)
- Client update (line 106-114)

If an operator tries to register a client with `backchannel_authentication_request_signing_alg: ML-DSA-65`, the registration will fail with "not allowed signature algorithm" (line 142).

### What's already quantum-safe

**Internal CIBA request serialization** (for communication with authentication devices):

**File:** `CIBAAuthenticationRequest.java` (lines 156-161)

```java
SecretKey aesKey = session.keys().getActiveKey(session.getContext().getRealm(), KeyUse.ENC, Algorithm.AES).getSecretKey();
// Serializes to JWE using HS512 + A256CBC-HS512
```

The internal serialization of CIBA authentication requests uses **symmetric encryption** (AES-CBC with HMAC) — quantum-safe. This is separate from the client-to-Keycloak signed request (which is quantum-vulnerable with classical algorithms).

## Required Changes

### Change: Extend FAPI allowed algorithms (when FAPI 2.0 includes PQC)

**File:** `FapiConstant.java`

**Location:** Lines 30-37

**Current code:**
```java
public static final Set<String> ALLOWED_ALGORITHMS = new LinkedHashSet<>(Arrays.asList(
        Algorithm.PS256,
        Algorithm.PS384,
        Algorithm.PS512,
        Algorithm.ES256,
        Algorithm.ES384,
        Algorithm.ES512
));
```

**Proposed fix (when FAPI spec allows):**
```java
public static final Set<String> ALLOWED_ALGORITHMS = new LinkedHashSet<>(Arrays.asList(
        Algorithm.PS256,
        Algorithm.PS384,
        Algorithm.PS512,
        Algorithm.ES256,
        Algorithm.ES384,
        Algorithm.ES512,
        "ML-DSA-44",  // Add when FAPI 2.0 includes PQC
        "ML-DSA-65",
        "ML-DSA-87"
        // FN-DSA, SLH-DSA similarly
));
```

**Critical:** Do not add ML-DSA to this list until the FAPI working group publishes an updated profile that explicitly permits PQC algorithms. Adding them prematurely would claim FAPI compliance for a non-compliant configuration.

### Change: Update admin UI algorithm options

**File:** `SecureCibaAuthenticationRequestSigningAlgorithmExecutorFactory.java`

The admin UI configuration for the executor exposes algorithm options in a dropdown. This will need to include ML-DSA options once the FAPI spec is updated.

**Note:** The factory file was not read in this analysis, but based on the pattern in other executors, there's likely a `getConfigProperties()` method that returns a `SELECT_ONE` config property with algorithm options.

## What does NOT need changing

| Component | Why it's already PQC-ready |
|-----------|---------------------------|
| Signature verification (line 76) | Fully SPI-driven via `session.tokens().decodeClientJWT()` — will support ML-DSA automatically |
| Asymmetric algorithm check (line 69) | Uses `signatureProvider.isAsymmetricAlgorithm()` — ML-DSA providers will return `true` |
| Algorithm lookup (line 65) | Dynamically queries `SignatureProvider` by name — works for any registered provider |
| Internal CIBA request serialization (line 156-161) | Uses symmetric AES + HMAC — already quantum-safe |
| JWT parsing (line 47-54) | Algorithm-agnostic `JOSEParser` — no hardcoded assumptions |

## Dependencies

### For non-FAPI deployments
1. **ML-DSA `SignatureProvider`** (tracked under #48821 / #48824) — required before CIBA signed requests can use ML-DSA

### For FAPI deployments
2. **FAPI 2.0 specification update** — FAPI working group must publish an updated CIBA profile that includes PQC algorithms
3. **GAP-8 fix** — update `FapiConstant.ALLOWED_ALGORITHMS` to include ML-DSA (only after FAPI spec allows it)
4. **Admin UI updates** — expose ML-DSA algorithm options in the client policy executor configuration

## GitHub Issue Status

**Partially tracked:**

**GAP-8** is identified in the overview table but does not have a dedicated GitHub issue. The overview table correctly notes this is **spec-gated** — cannot be fixed until FAPI 2.0 includes PQC.

**Recommended approach:**

Create a **placeholder issue** under #43690 with the title:
- **"CIBA FAPI policy: add ML-DSA to allowed algorithms (pending FAPI 2.0 spec)"**
- **Description:** 
  > `FapiConstant.ALLOWED_ALGORITHMS` currently rejects ML-DSA algorithms even when ML-DSA `SignatureProvider` exists. This is correct behavior under FAPI 1.0 Advanced profile, which only permits PS256/384/512 and ES256/384/512.
  > 
  > **Action required:** Once the FAPI working group publishes an updated CIBA profile that includes PQC algorithms:
  > 1. Add ML-DSA, FN-DSA, SLH-DSA to `FapiConstant.ALLOWED_ALGORITHMS`
  > 2. Update admin UI options in `SecureCibaAuthenticationRequestSigningAlgorithmExecutorFactory`
  > 
  > **Blocked on:** FAPI 2.0 specification (external dependency)
- **Label:** `spec-gated`, `pqc-readiness`
- **Sub-issue under:** #43690

**GAP-17** is also mentioned in the overview — this may refer to a broader FAPI PQC gap across multiple domains (JAR, CIBA, etc.). Check if GAP-17 should be a parent issue covering all FAPI algorithm allowlists.

## What this means for operators

**Today:**

1. **Non-FAPI deployments:** CIBA signed requests can use ML-DSA once ML-DSA providers exist — no FAPI policy enforcement, so the SPI-driven verification path works automatically

2. **FAPI deployments:** CIBA signed requests **cannot use ML-DSA** even after ML-DSA providers exist — the FAPI client policy executor will reject ML-DSA during client registration

**When FAPI 2.0 includes PQC (future):**

1. Keycloak will update `FapiConstant.ALLOWED_ALGORITHMS` to include ML-DSA
2. FAPI-compliant clients will be able to register with `backchannel_authentication_request_signing_alg: ML-DSA-65`
3. CIBA signed requests will use ML-DSA signatures, eliminating the quantum vulnerability

**Migration path:**

- **Non-FAPI deployments:** Can adopt ML-DSA for CIBA immediately once Keycloak providers exist
- **FAPI deployments:** Must wait for FAPI 2.0 spec + Keycloak update before using ML-DSA
- **Hybrid environments:** Can have some realms with FAPI policies (classical-only) and others without (ML-DSA-enabled) during the transition

## Related Domains

- **Domain 7** — JAR Signed Request Object Verification (similar SPI-driven verification path; likely has same FAPI algorithm allowlist gap)
- **Domain 10** — private_key_jwt Client Authentication (uses same `SignatureProvider` SPI pattern)
- **Domain 41** — FAPI Algorithm Allowlist Enforcement (likely the parent domain for all FAPI algorithm gaps)

## Standards to monitor

- [FAPI Working Group](https://openid.net/wg/fapi/) — for FAPI 2.0 specifications including PQC support
- [OpenID CIBA specification](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html)

---

> Part of [#43690](https://github.com/keycloak/keycloak/issues/43690) — PQC Readiness