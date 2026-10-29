# Keycloak PQC Configuration Strategy


Keycloak's existing SPI architecture and cascading configuration model (server -> realm -> client -> protocol) supports per-area algorithm selection. PQC algorithms slot into the same configuration points where admins currently choose between RS256, ES256, etc.

Evidence: 16 of 63 audited domains are PENDING PROVIDERS — they will work automatically once ML-DSA/ML-KEM providers are registered, with zero code changes. This confirms the SPI model is sound.

---

## Configuration Cascade (Existing)

```
Server level (CryptoProvider SPI)     — what algorithms are available
    └── Realm level (realm config)    — default algorithm for this realm
         └── Client level (per-client attributes) — override per client
              └── Protocol-specific (SAML, OIDC, OID4VC) — protocol config
```

Clients already have independent algorithm settings for token signing, token encryption, UserInfo signing, JAR, etc. An operator can configure one client to use ML-DSA-65 for token signing while another stays on ES256.

---

## Domain Breakdown by Type

| Crypto Operation  | Domains | PQC Algorithm |
|-------------------|---------|---------------|
| Signing (SIGN)    | 39      | ML-DSA (FIPS 204) |
| Encryption (ENC)  | 8       | ML-KEM (FIPS 203) |
| Both (SIGN+ENC)   | 5       | ML-DSA + ML-KEM |
| Certificate (CERT)| 4       | Determined by issuing CA |
| Mixed (SIGN+CERT) | 3       | ML-DSA + CA-dependent |
| External blocked  | 3       | Waiting on FIDO2/CAEP/Elytron specs |

---

## Three Implementation Tracks

### Track 1: Core Provider Implementations
New SPI providers that plug into the existing framework:
- ML-DSA `SignatureProviderFactory` (GAP-15)
- ML-KEM `CekManagementProviderFactory` (GAP-4)
- `GeneratedAKPKeyProviderFactory` for ML-DSA key generation
- ML-KEM key generation provider (domains 29, 32)

Once registered, 16 PENDING PROVIDERS domains light up automatically.

### Track 2: Hardcoded Algorithm Removal (~12 sites)
Three sub-categories:

**Config exists but hardcoding bypasses it:**
- SAML RS256 key lookup (GAP-9) — 4 call sites ignore realm SAML algorithm config
- IdP broker RS256/HS256 fallback (GAP-10) — broker config model exists but isn't read
- Fix: wire existing config into the code path

**No config surface exists — new config needed:**
- `JwtCNonceHandler` ES256/RS256 (GAP-18) — needs new OID4VC config attribute
- `OID4VPIdentityProvider` ES256 (GAP-24) — needs configurable algorithm set on IdP instance
- Admin CLI RS256 (GAP-13) — needs `--sigalg` CLI parameter
- Fix: add attribute to relevant config model, expose in `getConfigProperties()`, read at use site

**Static allowlists that actively reject PQC:**
- `FapiConstant.ALLOWED_ALGORITHMS` (GAP-8) — must become per-executor-instance config
- CAEP `SsfSignatureAlgorithms.ALLOWED` (domain 37) — spec-gated, one-line fix when spec moves
- Fix: make list configurable rather than static; follows existing executor factory pattern

### Track 3: Missing AKP Branches (small additions)
- JWKS endpoint `toJwk()` needs AKP case (GAP-26)
- JWK thumbprint needs AKP in required-members map (GAP-1)
- Client signature verifier RSA-only guard (GAP-30)
- Client SDK `JWTClientCredentialsProvider` missing AKP case (GAP-28)

---

## Internal vs Client-Facing: Migration Impact

### Internal to Keycloak (operator controls unilaterally)
- Realm key generation (domains 28-32, 46)
- Internal token signing (action tokens, session tokens, org invitations)
- CIBA internal serialisation (already HS512, quantum-safe)
- JGroups cluster encryption (domain 39)
- Realm bootstrap defaults (domain 46)

Operator can switch to PQC on their own schedule. No coordination needed.

### Client-Facing (requires coordinated migration)

**Outbound to clients:**
- Access/ID token signing (domain 1) — every relying party must support ML-DSA verification
- JWKS endpoint (domain 48) — clients must understand AKP JWKs
- OIDC discovery (domain 36) — must advertise ML-DSA for conformant client negotiation
- UserInfo/JARM/introspection responses

**Inbound from clients:**
- `private_key_jwt` assertions (domain 10) — client controls their signing algorithm
- JAR signed request objects (domain 7)
- DPoP proofs (domain 15) — blocked by JWK thumbprint gap (GAP-1)

**Bidirectional / federation:**
- SAML (domains 18-20) — hardest migration; multi-party federations are slow to change
- OIDC IdP brokering (domain 21) — upstream IdP and Keycloak must agree
- OID4VP/OID4VCI (domains 62-63) — wallets and verifiers must agree

### Hybrid Migration Strategy (uses existing per-client config)
1. Realm publishes both classical and ML-DSA keys on JWKS endpoint (multiple active keys already supported)
2. Clients that support ML-DSA opt in via per-client `id_token_signed_response_alg = ML-DSA-65`
3. Clients that don't yet support ML-DSA continue receiving RS256/ES256
4. Realm default stays classical until all clients have migrated, then flips
5. Migration order: first-party apps first, then coordinated partners, then third-party integrations last

---

## Status Summary (63 Domains)

| Status             | Count | Description |
|--------------------|-------|-------------|
| PENDING PROVIDERS  | 16    | Will work automatically once ML-DSA/ML-KEM providers exist |
| PARTIAL            | 12    | Mostly ready; specific gaps to fix |
| BLOCKED            | 27    | Require code changes (hardcoded algorithms, missing providers) |
| SAFE               | 5     | Already quantum-safe (symmetric or CA-controlled) |
| EXTERNAL DEPENDENCY| 3     | Blocked on external specs (FIDO2, CAEP, Elytron) |

## External Blockers
- BC-FIPS 2.1.3 has no ML-DSA/ML-KEM — hard blocker for all FIPS deployments (GAP-6)
- FAPI 2.0 profile has not yet included PQC algorithms (GAP-8)
- FIDO2/WebAuthn has no PQC COSE algorithm identifiers (domain 14)
- CAEP Interoperability Profile pins RS256 (domain 37)

## Reference
- Full domain audit: `pqc_overview.html`
- Individual domain implementation plans: `domains/Domain_N_*.md`
- Gap reference: `gaps/Github_issue_requirements_for_gaps.md`
- Parent tracking issue: [#43690](https://github.com/keycloak/keycloak/issues/43690)