# [REQ-AUTH-020] Passkey authentication for admin and consultant accounts

## Metadata

- **ID**: REQ-AUTH-020
- **Title**: Passkey authentication for admin and consultant accounts
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-2.8; `SECURITY.md` SEC-AUTHN-2

## Requirement

- **Statement**: Accounts with the `admin` or `consultant` role MUST authenticate with a passkey (WebAuthn), and MUST NOT be able to complete authentication with a password alone or through any password-only fallback path.
- **Rationale**: FR-2.8 requires passkey authentication for privileged roles and forbids password-alone authentication for them; SEC-AUTHN-2 restates it and extends the prohibition to recovery paths. Both roles reach subscriber health data, and the threat model treats compromised admin and consultant accounts as primary adversaries.
- **Assumptions**: Role is resolved from persisted state before the credential path is selected (REQ-AUTH-010).
- **Out of Scope**: Passkey registration and replacement (REQ-AUTH-030); how the *first* passkey for a privileged account is enrolled, which `SECURITY.md` SQ-12 and threat TM-S-4 leave open; subscriber password authentication and MFA, blocked by `REQUIREMENTS.md` OQ-8 and OQ-9; account recovery flows, blocked by OQ-8.
- **Design Traceability**: `DESIGN.md` — Components (Buttons, Inputs, Focus states, Form feedback and errors) govern the authentication view's presentation; `DESIGN.md` OQ-7 (distinct treatment for privileged roles) is open and not required here.
- **Architecture Traceability**: `ARCHITECTURE.md` — Identity and Session Handling ("passkey registration and verification for admin and consultant accounts"); trust boundary 2; data flow 1.
- **Security Traceability**: SEC-AUTHN-2; supports SEC-AUTHN-3, SEC-AUTHN-4, SEC-LOG-4.

## Scope

- **Applies To**: Server-Side Application, Web Client, API
- **Components**: Identity and Session Handling; Browser Client (authentication view)
- **Interfaces / Operations**: Authentication for `admin` and `consultant` accounts
- **Actors**: `admin`, `consultant`; anonymous attacker attempting a privileged login
- **Preconditions**: The account exists, carries the `admin` or `consultant` role, and has at least one registered passkey (REQ-AUTH-030)
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Personal Data — credential and passkey registration material
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED (`SECURITY.md` SQ-1)

## Security Context

- **Security Objectives**: Authenticity, Authorization, Confidentiality
- **Control Layers**: Authentication, Session Management
- **Threat References**: `SECURITY.md` TM-S-2 (recovery used to bypass passkey), TM-S-4 (privileged bootstrap), TM-S-1 (credential stuffing); CWE-287 (improper authentication), CWE-640 (weak password recovery mechanism)
- **Abuse / Misuse Case**: An attacker who has a privileged account's password authenticates anyway — through a password endpoint that does not check role, a legacy path, an error branch that falls back to password, or a recovery flow that issues a session without a passkey.
- **Trust Boundary**: Boundary 2 — unauthenticated → authenticated.
- **Untrusted Inputs or Assertions**: The submitted credential, the claimed account identifier, and the WebAuthn assertion until it is verified.
- **Authoritative Enforcement Point**: Identity and Session Handling — the role-to-credential-path rule is applied server-side after resolving the account's role.
- **Independent Verification**: The WebAuthn assertion is verified against the registered credential; the role that triggers the requirement comes from persisted state, not from the client's choice of login form.
- **Zero Trust Relevance**: NIST SP 800-207 — stronger authentication is required for subjects with broader resource access. Exact tenet: TO BE DECIDED (not verified in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: TO BE DECIDED — blocked by `SECURITY.md` SQ-1.
- **Other**: `REF-WEBAUTHN` (W3C Web Authentication) and `REF-PASSKEY` (FIDO Alliance Passkeys), included in `SECURITY.md` specifically because passkeys are selected for these roles; `REF-63B` and `REF-AUTH` as cited by SEC-AUTHN-2.
- **Mapping Basis**: SEC-AUTHN-2 names `REF-PASSKEY`, `REF-WEBAUTHN`, and `REF-ASVS-5`; the CWE identifiers name the improper-authentication and recovery-bypass classes the rule prevents.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an `admin` or `consultant` account with a registered passkey, when the actor completes a WebAuthn assertion that verifies against that registration, then a session is issued and the authentication success is logged.
2. **AC-02 — Boundary or failure behavior**: Given an `admin` or `consultant` account, when authentication is attempted with a correct password and no passkey assertion, then authentication fails, no session is issued, and the response does not reveal that the password was correct or that the account is privileged (SEC-AUTHN-3).
3. **AC-03 — Prohibited behavior**: Given any authentication or recovery path, when the account's role is `admin` or `consultant`, then no path — including error handling, recovery, and any legacy or alternate endpoint — may issue a session without a verified passkey assertion.
4. **AC-04 — Additional criterion**: Given a WebAuthn assertion that fails verification, is replayed, or is bound to a different account or origin, when it is presented, then it is rejected and the failure is logged with cause class but without credential material.

## Failure Behavior

- **On Invalid Input**: A malformed assertion is rejected as an authentication failure with a uniform response.
- **On Authentication Failure**: Deny; uniform response that does not disclose which factor failed or whether the account exists (FR-2.3, SEC-AUTHN-3); no session issued.
- **On Authorization Failure**: N/A — this precedes authorization.
- **On Security-Decision Failure**: If role cannot be resolved, or if assertion verification errors, deny. There is no fallback path (AC-03).
- **On External Dependency Failure**: If stored passkey registration material cannot be read, deny; MUST NOT fall back to password.
- **On System Error**: Generic error with a correlation identifier; no session issued; no credential material in the response.
- **Logging / Audit**: Log authentication success and failure with account identifier, role, cause class, and correlation identifier (SEC-LOG-4). MUST NOT log the assertion, challenge, private key material, or password (SEC-LOG-3, SEC-SECRET-1).
- **Alerting**: TO BE DECIDED — thresholds for repeated privileged authentication failures are blocked by `SECURITY.md` SQ-3.

## Test Strategy

- **Unit Tests**: Credential-path selector returns passkey-required for `admin` and `consultant` and rejects a password submission for those roles; assertion verifier rejects a bad signature, a replayed challenge, and a mismatched origin or account.
- **Integration Tests**: Full WebAuthn authentication for each privileged role issuing a session; password submission for each privileged role denied.
- **Security Tests**: Attempt password-only authentication for both privileged roles on every authentication and recovery endpoint, asserting rejection (SEC-AUTHN-2 verification); challenge-replay test; cross-account assertion test; a route inventory assertion that no alternate session-issuing endpoint exists.
- **Compliance Tests / Evidence**: Record of the passkey-only enforcement test across all authentication paths, as evidence for FR-2.8.
- **Acceptance-Criteria Traceability**: AC-01 — passkey success suite; AC-02 — password-denial suite; AC-03 — all-paths enforcement suite plus route inventory; AC-04 — assertion negative suite.
- **Coverage Target**: Both privileged roles × every session-issuing path, positive and negative.
- **Required Test Environment**: A WebAuthn authenticator simulator, one `admin` and one `consultant` identity with registered passkeys, and one `subscriber` for contrast. Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-AUTH-010, REQ-SESSION-010, REQ-SESSION-020
- **Downstream Requirements**: REQ-AUTHZ-030, REQ-CONSULT-010, REQ-AUTH-030, REQ-AUDIT-030
- **External Dependencies**: A WebAuthn server library, subject to DEP-1…DEP-8; DEP-1 forbids replacing vetted protocol implementations with custom code. The authenticator itself is user-held hardware or platform software, not a system integration.
- **Dependency Assumptions**: The library verifies signature, challenge freshness, origin, and relying-party identifier; it does not accept an assertion whose challenge the server did not issue.
- **Failure Impact**: A single password-accepting path for a privileged role reduces the strongest authentication control in the specification to a password.

## Implementation Notes

- **Constraints**: Node.js runtime with Fastify (`CLAUDE.md`). Passkeys are mandatory only for `admin` and `consultant`; subscriber authentication is a separate, currently blocked path and MUST NOT be conflated with this one.
- **Prohibited Approaches**: A shared login endpoint that branches on a client-supplied flag; a "temporary" password path for privileged accounts; recovery that issues a session on email possession alone (SEC-AUTHN-2 covers recovery explicitly); logging assertions or challenges.
- **Implementation Guidance**: Resolve the account's role before deciding which credential the endpoint will accept, so the rule cannot be bypassed by choosing a different form. Keep challenge generation on the cryptographically secure generator required by SEC-SECRET-4.
- **AI Development Guidance**: `REF-PROMPT-JWT` (session issuance), `REF-PROMPT-NODE`, `REF-PROMPT-QUALITY`; `REF-WEBAUTHN`, `REF-PASSKEY`; `CLAUDE.md`.
- **Required Human Review**: Security review of every session-issuing path, not only the passkey path.
- **Open Decisions**: `SECURITY.md` SQ-12 and threat TM-S-4 — how the first passkey for a privileged account is enrolled — remain open and are handled as blocked scope. This issue assumes a registered passkey already exists and does not create one. Account recovery for privileged accounts is blocked by `REQUIREMENTS.md` OQ-8.

**Estimated effort**: 1.5–2 engineer-days. **Estimated changed lines**: 400–900.
**Recommended model**: Claude Opus (`claude-opus-5`) — cryptographic protocol integration and an absolute no-fallback rule that must hold across every path.
