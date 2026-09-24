# core-authn Team: Issue Ownership & Open Issue Gap Analysis

**Repository:** [keycloak/keycloak](https://github.com/keycloak/keycloak)
**Team label used:** `team/core-authn`
**Analysis window (Step 1):** issues labeled `team/core-authn` with any activity (created/updated/commented/labeled) in the last 8 weeks — i.e. `updated:>=2026-06-26` as of the analysis date 2026-08-21.
**Method:** Read-only GitHub CLI/API queries (`gh search issues`, `gh api search/issues`, `gh api graphql` for PR file diffs). No issues were modified.

---

## Data Collection Summary

| Metric | Count |
|---|---|
| Total `team/core-authn` issues/PRs updated in last 8 weeks | 514 |
| — of which are Issues (non-PR) | 379 (84 open / 295 closed) |
| — of which are Pull Requests | 135 |
| Distinct files changed across the 135 linked PRs | 958 |

Labels most frequently co-occurring with `team/core-authn` on this issue set (excluding other team labels, which reflect multi-team collaboration rather than functional ownership):

`area/authentication` (68), `area/authentication/webauthn` (17), `area/oidc` (32), `area/core` (31), `area/docs` (23), `area/login/ui` (22), `area/admin/ui` (18), `area/admin/fine-grained-permissions` (10), `area/saml` (10), `area/infinispan` (10), `area/dist/quarkus` (13), `kind/cve` (44), `kind/weakness` (23), `kind/bug` (191), `flaky-test` (15).

---

## Section 1: core-authn Functional Ownership Map

Based on the code paths touched by the 135 linked pull requests and the recurring functional themes in the 379 issues, `core-authn`'s ownership spans **authentication flows, credentials, sessions, and the login/registration experience** — i.e. the "front door" of Keycloak, as distinct from protocol-endpoint plumbing (OIDC/SAML wire format — largely `team/core-protocols`) or admin/identity-management (`team/core-iam`).

### 1.1 Authentication Engine & Flows
- `services/src/main/java/org/keycloak/authentication/AuthenticationProcessor.java`, `AuthenticationSelectionResolver.java`, `AuthenticatorUtil.java`, `DefaultAuthenticationFlow.java`
- `services/src/main/java/org/keycloak/authentication/authenticators/browser/**` — `WebAuthnAuthenticator`, `SpnegoAuthenticator`, `AbstractUsernameFormAuthenticator`, `IdentityProviderAuthenticator`, `WebAuthnConditionalUIAuthenticator`, `OrganizationAuthenticator`
- `services/src/main/java/org/keycloak/authentication/authenticators/x509/**` — `AbstractX509ClientCertificateAuthenticator`, `CertificateValidator`
- `services/src/main/java/org/keycloak/authentication/authenticators/client/**` — `ClientIdAndSecretAuthenticator`, `JWTClientSecretAuthenticator`, `AttestationBasedClientAuthenticator`
- `services/src/main/java/org/keycloak/authentication/authenticators/directgrant/**`, `resetcred/**`, `broker/**`
- `services/src/main/java/org/keycloak/authentication/authenticators/sessionlimits/**` — `UserSessionLimitsAuthenticator(Factory)`
- `services/src/main/java/org/keycloak/authentication/requiredactions/**` — e.g. `WebAuthnRegister`
- `services/src/main/java/org/keycloak/authentication/actiontoken/**` — including `execactions/ExecuteActionsActionTokenHandler`

### 1.2 Credentials, Passwords & Brute-Force Protection
- `org/keycloak/credential/OTPCredentialProvider.java` and credential SPI usage in `server-spi` / `server-spi-private`
- Password policy providers (referenced across multiple `team/core-authn` issues: `Pbkdf2PasswordHashProvider`, `HistoryPasswordPolicyProvider`, `DefaultPasswordPolicyManagerProvider`)
- Brute-force / lockout logic (`BruteForceUsersResource.java` and related detection paths)
- `org/keycloak/utils/TotpUtils.java` (OTP/TOTP)

### 1.3 WebAuthn / Passkeys
- `tests/webauthn/**`, and production code under `authenticators/browser/WebAuthn*`
- WebAuthn/passkey-related theme templates (`login-passkeys-conditional-authenticate.ftl`, `passkeys.ftl`, `webauthn-authenticate.ftl`)

### 1.4 X.509 / SPNEGO / Kerberos
- `authenticators/x509/**` (cert validation, config model)
- `federation/kerberos/src/main/java/org/keycloak/federation/kerberos/KerberosPrincipal.java`

### 1.5 Sessions
- `model/infinispan/src/main/java/org/keycloak/models/sessions/infinispan/**` — `PersistentUserSessionProvider`, `InfinispanUserSessionProviderFactory`
- `server-spi/src/main/java/org/keycloak/models/UserManager.java`
- Session-limit enforcement (see 1.1) and related admin resources (`RealmAdminResource.java`, `UserResource.java`)

### 1.6 Login/Registration/Logout UI & Theming
- `themes/src/main/resources/theme/keycloak.v2/login/**` (largest single themed area touched — templates, resources, `theme.properties`)
- `themes/src/main/resources/theme/base/login/**` — `template.ftl`, `info.ftl`, `error.ftl`, `frontchannel-logout.ftl`, `register.ftl`, recovery-code templates
- `js/apps/account-ui/src/**`, `js/apps/admin-ui/src/authentication/**` (flow details, policies), `identity-providers/**`, `realm-settings/LoginTab.tsx`
- `web-crypto-shim.js` / `js/libs` crypto shim used by login/registration JS flows

### 1.7 Identity Broker (login-side) & Delegation/Scopes touching login
- `services/src/main/java/org/keycloak/broker/oidc/AbstractOAuth2IdentityProvider.java`, `broker/oid4vp/**`
- Parameterized scope types and token-exchange delegation logic that gate authentication decisions (`scope/DelegationScopeType.java`, `tokenexchange/TokenExchangeDelegationProvider.java`) — shared boundary with `team/core-protocols`

### 1.8 Testing & Quality Infrastructure for the Above
- `testsuite/integration-arquillian/**`, `tests/base/src/test/java/org/keycloak/tests/**` — largest test surface touched (89 files)
- Recurring flaky-test issues concentrated in `MultipleTabsLoginTest`, `WebAuthn*Test`, `ClientAuthPostMethodTest`, `RecoveryAuthnCodesAuthenticatorTest`

---

## Section 2: Open Issues Recommended for core-authn Review

Screening method: pulled all currently **open** issues NOT labeled `team/core-authn`, pre-filtered to labels overlapping the ownership map above (`area/authentication`, `area/authentication/webauthn`, `area/login/ui`, `area/saml`, `area/admin/fine-grained-permissions`, `area/oidc`, `area/core`, `kind/cve`, `kind/weakness`), then matched title/content against core-authn's functional keywords (authenticators, credentials, WebAuthn, X.509/SPNEGO, brute force, password policy, session limits, required actions/recovery codes, login flows). 116 keyword matches found; the table below lists the highest-confidence subset.

| # | Title | URL | Overlap Reason | Current Assignee / Team Label |
|---|---|---|---|---|
| 45667 | Keycloak User Session Count Limiter - Realm-Wide Counting Issue | [#45667](https://github.com/keycloak/keycloak/issues/45667) | Directly targets `UserSessionLimitsAuthenticator`, a core-authn-owned authenticator (§1.1/1.5) | `area/authentication`, `team/core-clients` — unassigned |
| 44474 | Keycloak Maximum Concurrent Session Limit incorrectly triggers Brute Force protection | [#44474](https://github.com/keycloak/keycloak/issues/44474) | Cross-cuts two core-authn-owned subsystems: session limits (§1.5) and brute-force detection (§1.2) | `area/authentication`, `priority/important`, `team/core-clients` — unassigned |
| 30077 | Brute force detection for client credentials (confidential clients) | [#30077](https://github.com/keycloak/keycloak/issues/30077) | Feature request directly extending brute-force protection, a core-authn-owned area (§1.2) | `area/authentication`, `priority/important`, `team/core-clients` — unassigned |
| 37461 | Option to delete user sessions after permanent lockout | [#37461](https://github.com/keycloak/keycloak/issues/37461) | Brute-force lockout + session lifecycle, both core-authn areas (§1.2, §1.5) | `area/authentication`, `team/core-clients` — unassigned |
| 39249 / 39247 | Recovery codes: prompt to regenerate when few remain / more config options | [#39249](https://github.com/keycloak/keycloak/issues/39249), [#39247](https://github.com/keycloak/keycloak/issues/39247) | Recovery-authn-code flow is a core-authn-owned required-action/credential area (§1.1/1.2); matches `RecoveryAuthnCodesAuthenticatorTest` seen in core-authn's own test history | `area/authentication`, `team/core-clients` — unassigned |
| 38902 | Re-authenticating user with no registered passwordless WebAuthn who logged in via IdP fails | [#38902](https://github.com/keycloak/keycloak/issues/38902) | WebAuthn/passkey authenticator bug (§1.3), directly in core-authn's owned `WebAuthn*Authenticator` code | `area/authentication`, `area/authentication/webauthn`, `team/core-clients` — unassigned |
| 17636 | WebAuthn should support U2F migration via AppId extension | [#17636](https://github.com/keycloak/keycloak/issues/17636) | WebAuthn feature request, same owned code area as core-authn's recent WebAuthn PR activity (§1.3) | `area/authentication/webauthn`, `team/core-clients` — unassigned |
| 36317 | WebAuthn tests for extra origins | [#36317](https://github.com/keycloak/keycloak/issues/36317) | Test-coverage gap in WebAuthn, matching core-authn's own `tests/webauthn` ownership (§1.3, §1.8) | `area/authentication/webauthn`, `team/core-clients` — unassigned |
| 10996 / 10368 / 10070 | WebAuthn test coverage/flakiness (AttestationConveyanceRegister, Firefox coverage, AuthAttachmentRegisterTest) | [#10996](https://github.com/keycloak/keycloak/issues/10996), [#10368](https://github.com/keycloak/keycloak/issues/10368), [#10070](https://github.com/keycloak/keycloak/issues/10070) | Same WebAuthn test classes core-authn is actively fixing flakiness in this window (e.g. closed core-authn issue #51512/#51256/#51175 on `WebAuthnOtherSettingsTest`) | `area/testsuite`, `area/authentication/webauthn` — unassigned, no team label |
| 43243 | Subflows not working for client authentication flows | [#43243](https://github.com/keycloak/keycloak/issues/43243) | Authentication-flow engine bug, directly in `AuthenticationProcessor`/flow execution (§1.1) | `area/authentication`, `team/core-clients` — unassigned |
| 44025 | Allow and Deny access does not end entire Authentication flow | [#44025](https://github.com/keycloak/keycloak/issues/44025) | Core authentication-flow execution semantics bug (§1.1) | `area/authentication`, `team/core-clients` — unassigned |
| 38026 | Authenticators shouldn't be internal anymore? | [#38026](https://github.com/keycloak/keycloak/issues/38026) | SPI extensibility request for the authenticator framework core-authn owns (§1.1) | `area/authentication`, `team/core-clients` — unassigned |
| 27732 | Authentication policies | [#27732](https://github.com/keycloak/keycloak/issues/27732) | Feature proposal for the authentication-flow subsystem (§1.1); overlaps `admin-ui/src/authentication/policies` files core-authn PRs touched | `area/authentication`, `team/core-clients` — unassigned |
| 31934 | Allow users to select MFA configuration options from alternatives on first login | [#31934](https://github.com/keycloak/keycloak/issues/31934) | MFA/credential selection during authentication flow (§1.1/1.2) | `area/authentication`, `team/core-clients` — unassigned |
| 23080 | Doublecheck clearing of authenticationSession when switching flows | [#23080](https://github.com/keycloak/keycloak/issues/23080) | Authentication-session lifecycle bug within core-authn's flow engine (§1.1/1.5) | `area/authentication`, `team/core-clients` — unassigned |
| 31616 | Differentiate error messages for different authentication failures | [#31616](https://github.com/keycloak/keycloak/issues/31616) | UX/messaging within the login/authenticator flow core-authn owns (§1.1/1.6) | `area/authentication`, `team/core-clients` — unassigned |
| 14150 | Password policy change not forcing users to update password | [#14150](https://github.com/keycloak/keycloak/issues/14150) | Password-policy enforcement gap, directly in core-authn's owned password-policy providers (§1.2) | `area/authentication`, `team/core-clients` — unassigned |
| 27699 | Improve docs for X.509 | [#27699](https://github.com/keycloak/keycloak/issues/27699) | Documentation for X.509 authenticator, core-authn-owned code area (§1.4) | `area/authentication`, `priority/important`, `team/core-clients` — unassigned |
| 15663 | External IdP Azure error: XML `Format` attribute must be a URI | [#15663](https://github.com/keycloak/keycloak/issues/15663) | SAML broker + `area/authentication` overlap touching login-side broker code (§1.7) | `area/authentication` — unassigned, no team label |
| 38575 | Login[v2]: Improvements | [#38575](https://github.com/keycloak/keycloak/issues/38575) | Directly targets `keycloak.v2/login` theme, the single largest themed area in core-authn's recent PRs (§1.6) | `area/login/ui`, `team/core-clients` — unassigned |
| 31445 | Revise HTML structure of keycloak.v2 login theme | [#31445](https://github.com/keycloak/keycloak/issues/31445) | Same `keycloak.v2/login` theme ownership (§1.6) | `area/login/ui` — assigned: `edewit`, no team label |
| 25782 | Forgot password - login form visible | [#25782](https://github.com/keycloak/keycloak/issues/25782) | Login/reset-password theme UX bug within core-authn's owned login templates (§1.2/1.6) | `area/login/ui` — unassigned, no team label |
| 23573 | [Login UI] Password Strength Indicator | [#23573](https://github.com/keycloak/keycloak/issues/23573) | Login-theme feature tied to password-policy UX (§1.2/1.6) | `area/login/ui`, `team/core-shared` — assigned: `andreas-blaettlinger` |
| 14340 | Add ability to set default credential in account console | [#14340](https://github.com/keycloak/keycloak/issues/14340) | Credential management surfaced via account UI, overlapping core-authn's credential-provider ownership (§1.2) | `area/authentication`, `area/account/ui`, `team/core-shared` — unassigned |
| 16442 | Removing OTP device without reauthentication | [#16442](https://github.com/keycloak/keycloak/issues/16442) | OTP credential-management security gap (§1.2) | `area/authentication`, `area/account/api`, `team/core-clients` — unassigned |
| 44964 | Cannot update user WebAuthn Passwordless from Java admin client | [#44964](https://github.com/keycloak/keycloak/issues/44964) | WebAuthn credential management, core-authn-owned code area (§1.3) | `area/authentication`, `area/admin/api`, `team/core-clients` — unassigned |
| 44354 | Enabling/disabling Kerberos in LDAP federation affects all auth flows without notice | [#44354](https://github.com/keycloak/keycloak/issues/44354) | Kerberos/SPNEGO authenticator config bug, core-authn-owned (§1.4) | `area/authentication`, `area/ldap`, `team/core-iam` — unassigned |
| 13861 | Registration form displayed twice when redirecting from browser flow | [#13861](https://github.com/keycloak/keycloak/issues/13861) | Browser authentication flow / registration flow bug (§1.1/1.6) | `area/authentication` — unassigned, no team label |
| 9048 | Registration while already logged in | [#9048](https://github.com/keycloak/keycloak/issues/9048) | Registration/authentication-flow edge case core-authn's flow engine handles (§1.1) | `area/authentication` — unassigned, no team label |

**Note on labeling:** most of the strongest matches already carry `team/core-clients` rather than being unassigned — meaning they are formally routed to a different current team, but their functional content (authenticators, WebAuthn, brute-force, session limits, login theming, password policy) matches `core-authn`'s established code ownership more closely than `core-clients`'. These are flagged as candidates for `core-authn` to review/co-own or for a label correction, not as unowned issues. A handful (WebAuthn test-flakiness issues #10996/#10368/#10070, and login-UI issues #15663/#13861/#9048/#25782) carry **no team label at all** and are the clearest "pure gap" candidates.

---

## Caveats

- GitHub does not support assigning issues directly to a team; "core-authn issues" here means issues carrying the `team/core-authn` label, per the user-confirmed method.
- The functional ownership map is derived from actual file paths in the 135 merged/open PRs linked to core-authn-labeled issues in the 8-week window, plus recurring keyword/label patterns in the 379 issues — not from a static team charter, since none was available via the GitHub API.
- Overlap in Section 2 is based on label + keyword/content matching against Section 1's map; it is a recommendation heuristic, not a definitive ownership ruling, and should be reviewed by the `core-authn` team before any relabeling.
- This analysis is read-only: no labels, assignees, or issue content were modified as part of this investigation.
