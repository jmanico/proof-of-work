# [REQ-AUTH-120] MFA challenge and recovery-code redemption

## Metadata

- **ID**: REQ-AUTH-120
- **Title**: MFA challenge and recovery-code redemption
- **Version**: 1.1.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-2.6, FR-2.13; `SECURITY.md` SEC-AUTHN-4, SEC-AUTHN-10, SEC-AUTHN-11

## Requirement

- **Statement**: When MFA is enabled for an account, the system MUST require a successful second factor — a TOTP code, a passkey assertion, or a single-use recovery code — before issuing any session, and MUST invalidate a recovery code on use.
- **Rationale**: FR-2.6 requires the second factor before an authenticated session exists, and SEC-AUTHN-4 forbids a partially authenticated state granting access to anything. Recovery-code redemption belongs here rather than in enrolment because it is a challenge outcome: it is the one sanctioned substitute for the factor, and SEC-AUTHN-10 forbids every other. Putting redemption anywhere else would create a second session-issuing path, which is exactly what the threat model warns about.
- **Assumptions**: The first factor has already succeeded (REQ-AUTH-100). Enrolment state and stored recovery codes come from REQ-AUTH-110.
- **Out of Scope**: Enrolment, disablement, and code issuance (REQ-AUTH-110); password authentication (REQ-AUTH-100); passkey authentication as a primary factor for privileged roles (REQ-AUTH-020); throttling (REQ-AUTH-060); session creation mechanics (REQ-SESSION-030). The recovery-code count is fixed at 10 by SEC-AUTHN-11 (`SECURITY.md` SQ-3 RESOLVED) and is REQ-AUTH-110's concern.
- **Design Traceability**: `DESIGN.md` — Product Patterns → Credentials, account security, and administration (Sign-in and second factor: the second step requests the TOTP code or passkey, a quiet "Use a recovery code instead" action swaps the input and states that each code works once, and failure copy never names the failing factor); Core Components → Forms and validation, Actions (busy state preventing repeat submission); Accessibility and Design Verification (keyboard-only completion of the second-factor and recovery-code steps).
- **Architecture Traceability**: `ARCHITECTURE.md` — Identity and Session Handling ("MFA challenge for subscribers who enable it"); trust boundary 2; data flow 1.
- **Security Traceability**: SEC-AUTHN-4 (no session before the second factor), SEC-AUTHN-10 (no other bypass), SEC-AUTHN-11 (single-use, hashed recovery codes), SEC-AUTHN-3 (non-disclosing failure), SEC-LOG-4.

## Scope

- **Applies To**: Server-Side Application, Web Client, API
- **Components**: Identity and Session Handling; Browser Client (the challenge view)
- **Interfaces / Operations**: MFA challenge; recovery-code redemption
- **Actors**: `subscriber` with MFA enabled; an attacker holding a valid password but not the second factor
- **Preconditions**: The first factor has succeeded for an account with MFA enabled
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Personal Data — second-factor material and recovery codes
- **Jurisdiction / Regulatory Scope**: Global service with GDPR as the design ceiling (`SECURITY.md` SQ-1 RESOLVED): GDPR/UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable

## Security Context

- **Security Objectives**: Authenticity, Authorization
- **Control Layers**: Authentication, Session Management
- **Threat References**: `SECURITY.md` TM-S-2 (recovery used to bypass MFA), TM-S-1 (credential attacks reaching the second factor), TM-S-5 (session theft); CWE-308 (use of single-factor authentication), CWE-287 (improper authentication), CWE-307 (excessive attempts)
- **Abuse / Misuse Case**: An attacker who has the password brute-forces a six-digit TOTP code, or replays a code already used, or replays a recovery code that was not invalidated on first use — any of which converts knowledge of one factor into a session.
- **Trust Boundary**: Boundary 2 — the challenge is the boundary itself, and nothing before it may confer access.
- **Untrusted Inputs or Assertions**: The submitted code or assertion, and any client-supplied indication that the first factor succeeded. That fact MUST come from server-held state tied to the in-progress attempt, never from the request.
- **Authoritative Enforcement Point**: Identity and Session Handling; the session is created only on the success path of this challenge.
- **Independent Verification**: TOTP codes are verified against the stored secret; recovery codes against stored Argon2id digests; passkey assertions against the registered credential.
- **Zero Trust Relevance**: NIST SP 800-207 — access is granted only after the full authentication requirement is met, not incrementally. Exact tenet: TO BE DECIDED.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — mapping verified only at the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — mapping verified only at the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: GDPR/UK GDPR (EU/UK data subjects); CCPA/CPRA, Washington My Health My Data, FTC Health Breach Notification Rule (US users); HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED). Statute-section precision: TO BE DECIDED pending the SQ-1 pre-launch counsel review.
- **Other**: RFC 6238 (TOTP); `REF-63B`, `REF-AUTH`, `REF-WEBAUTHN`, `REF-PASSKEY`.
- **Mapping Basis**: `REQUIREMENTS.md` OQ-9 selects RFC 6238 TOTP and passkeys; SEC-AUTHN-4 and SEC-AUTHN-11 name the OWASP and NIST references; the CWE identifiers name the single-factor and attempt-restriction classes.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a successful first factor on an MFA-enabled account, when a valid second factor is presented, then a session is created and the success is logged.
2. **AC-02 — Boundary or failure behavior**: Given a wrong, expired, or replayed second factor, when it is presented, then no session is created, the in-progress attempt is not advanced, and the response does not reveal which factor or value was wrong (SEC-AUTHN-3).
3. **AC-03 — Prohibited behavior**: Given an in-progress authentication that has passed only the first factor, when any protected route is called with whatever state that attempt produced, then it MUST be refused — no partial credential may function as a session (SEC-AUTHN-4).
4. **AC-04 — Additional criterion**: Given a valid recovery code, when it is redeemed, then a session is created, that code is invalidated, and presenting it again is refused (SEC-AUTHN-11).
5. **AC-05 — Additional criterion**: Given a TOTP code already used within its window, when it is presented again, then it is refused, so a captured code cannot be replayed inside its validity period.
6. **AC-06 — Additional criterion**: Given repeated failed challenges, when they accumulate, then throttling applies per REQ-AUTH-060 — a six-digit code is otherwise brute-forceable — and no sequence of third-party failures permanently disables the account (TM-D-2).
7. **AC-07 — Additional criterion**: Given a recovery-code redemption, when it succeeds, then an audit entry records that recovery was used, distinctly from an ordinary factor success, since it is a signal the user may have lost a device (SEC-LOG-4).

## Failure Behavior

- **On Invalid Input**: A malformed code is refused with the same uniform response as an incorrect one.
- **On Authentication Failure**: Deny; no session; the in-progress attempt MUST expire rather than allow unlimited retries against one first-factor success.
- **On Authorization Failure**: N/A
- **On Security-Decision Failure**: If enrolment state or stored codes cannot be read, deny. There is no default that treats the account as having no second factor — that would silently disable MFA for every account whenever the store misbehaved.
- **On External Dependency Failure**: N/A — verification is local.
- **On System Error**: Generic error with a correlation identifier (SEC-ERR-1); no code or secret in the response.
- **Logging / Audit**: Log challenge success and failure with account reference, cause class, and correlation identifier; audit recovery-code use distinctly per AC-07. The submitted code, the stored secret, and the code digests MUST NOT be logged (SEC-LOG-3).
- **Alerting**: Threshold alerts route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-3, SQ-11 RESOLVED); repeated failed second factors and recovery-code use are both SEC-LOG-4 signals feeding that channel.

## Test Strategy

- **Unit Tests**: TOTP verifier accepts within drift and rejects outside it; a used code within its window is refused (AC-05); recovery-code verifier matches against digests and marks single-use; an unreadable enrolment state yields denial.
- **Integration Tests**: Full challenge with TOTP, with a passkey, and with a recovery code (AC-01, AC-04); expiry of an in-progress attempt.
- **Security Tests**: Attempt to use partial first-factor state against protected routes (AC-03); recovery-code replay (AC-04); TOTP replay (AC-05); brute-force burst asserting throttling and asserting the account is not permanently disabled (AC-06); differential response comparison (AC-02).
- **Compliance Tests / Evidence**: The partial-authentication denial transcript, as evidence for SEC-AUTHN-4.
- **Acceptance-Criteria Traceability**: AC-01 — challenge suite; AC-02 — differential suite; AC-03 — partial-state suite; AC-04 — recovery redemption and replay; AC-05 — TOTP replay; AC-06 — burst and lockout-prohibition test; AC-07 — audit assertion.
- **Coverage Target**: Every second-factor type × valid, invalid, expired, and replayed, plus partial-state probes against a representative protected route.
- **Required Test Environment**: PostgreSQL with drizzle-kit migrations applied; a `subscriber` with TOTP enrolled, one with a passkey factor, and issued recovery codes; a controllable clock for drift and replay windows; a WebAuthn authenticator simulator; Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-AUTH-100, REQ-AUTH-110, REQ-AUTH-070, REQ-AUTH-060, REQ-SESSION-030, REQ-AUDIT-010
- **Downstream Requirements**: Every subscriber-facing issue, since an MFA-enabled subscriber reaches a session only through this path
- **External Dependencies**: The TOTP and WebAuthn libraries introduced by REQ-AUTH-110 and REQ-AUTH-020; no new dependency.
- **Dependency Assumptions**: The TOTP library exposes the drift window explicitly so replay prevention can be implemented on top of it, rather than accepting any code within a fixed tolerance without record.
- **Failure Impact**: A flaw here means an attacker holding only a password reaches a full session, which nullifies MFA for every subscriber who enabled it and is the specific outcome FR-2.6 exists to prevent.

## Implementation Notes

- **Constraints**: Node.js runtime with Fastify; Vue single-page application (`CLAUDE.md`). The TOTP drift window and in-progress attempt expiry are fixed by no spec document (`SECURITY.md` SQ-3 resolved without naming them) and MUST be named constants with a documented basis; the SEC-AUTHN-6 backoff parameters that AC-06 relies on are fixed (3 consecutive failures, 1 s doubling to a 15-minute cap, 24-hour reset).
- **Prohibited Approaches**: Issuing any session, cookie, or token before the second factor succeeds (SEC-AUTHN-4). Carrying the "first factor passed" fact in a client-held value rather than server state. Accepting a TOTP code more than once within its window. Failing to invalidate a redeemed recovery code. Any additional bypass — support override, email code, admin reset — all forbidden by SEC-AUTHN-10. Unlimited retries against a single first-factor success.
- **Implementation Guidance**: Hold the in-progress attempt as short-lived server state keyed to an opaque identifier, and make the session-creation call reachable only from this challenge's success branch, so there is exactly one place in the codebase where an MFA-enabled account can obtain a session. Record used TOTP codes for the duration of their window to satisfy AC-05, since drift tolerance alone permits replay.
- **AI Development Guidance**: `REF-63B`, `REF-AUTH`, `REF-WEBAUTHN`, `REF-PROMPT-JWT`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of every path that can create a session, confirming that an MFA-enabled account has exactly one, and that no partial state is usable.
- **Open Decisions**: TOTP drift window and in-progress attempt expiry — implementation constants no spec document fixes. The recovery-code count is resolved at 10 (`SECURITY.md` SQ-3, SEC-AUTHN-11). Whether a user with several enrolled factors may choose which to present depends on the multi-factor question left provisional in REQ-AUTH-110.

**Estimated effort**: 1.5–2 engineer-days. **Estimated changed lines**: 350–700.
**Recommended model**: Claude Opus (`claude-opus-5`) — the single most consequential branch in the authentication system, where a partial state leaking session authority silently defeats MFA entirely.
