# [REQ-AUTH-110] MFA enrolment, recovery codes, and disablement

## Metadata

- **ID**: REQ-AUTH-110
- **Title**: MFA enrolment, recovery codes, and disablement
- **Version**: 1.1.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: High
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-2.5, FR-2.13, FR-2.14; `SECURITY.md` SEC-AUTHN-7, SEC-AUTHN-10, SEC-AUTHN-11, SEC-AUTHN-12

## Requirement

- **Statement**: The system MUST allow a `subscriber` to enable and disable MFA on their own account using TOTP or a passkey, MUST issue single-use recovery codes at enrolment and present them exactly once, MUST state before enrolment that losing every factor and code makes the account permanently unrecoverable, and MUST require fresh re-authentication for each of these changes.
- **Rationale**: FR-2.5 makes MFA optional and user-controlled. FR-2.13 supplies the only sanctioned route around a lost factor, because SEC-AUTHN-10 forbids any other bypass — no admin reset, no email fallback — which would otherwise reduce MFA to the security of whatever the bypass depends on (threat TM-S-2). FR-2.14 requires the consequence be stated up front, since the alternative to a bypass is permanent loss of access to one's own health data — a trade `REQUIREMENTS.md` OQ-17 resolves as standing, softened only by the FR-9.11 out-of-band deletion channel (REQ-PRIVACY-110), which destroys data without restoring access. These three behaviors are one user flow: codes are generated at enrolment and the warning is what makes enrolment informed.
- **Assumptions**: Recovery codes are user-typed secrets and so are stored with the Argon2id function from REQ-AUTH-070, as amended SEC-AUTHN-11 requires — the HMAC-SHA-256 path it defines is for machine-held high-entropy tokens only and does not apply here. Disabling MFA is itself a credential change and so revokes sessions via REQ-SESSION-040.
- **Out of Scope**: The MFA challenge during authentication and recovery-code redemption (REQ-AUTH-120); password authentication (REQ-AUTH-100); passkey authentication for privileged roles (REQ-AUTH-020); throttling (REQ-AUTH-060); SMS and email codes, excluded by `REQUIREMENTS.md` OQ-9.
- **Design Traceability**: `DESIGN.md` — Product Patterns → Credentials, account security, and administration (MFA enrolment and recovery codes: the FR-2.14 interstitial with a primary **I understand — enable two-factor** action and no pre-checked control; the ten recovery codes shown exactly once in the data face with copy and download affordances, left only through an explicit **I saved my recovery codes** action; disablement and regeneration require re-authentication and end other sessions); Core Components → Actions (destructive actions use `error` and require explicit confirmation, which applies to disabling MFA), Forms and validation; Accessibility (the one-time code display must be selectable, readable, and announced, not an image).
- **Architecture Traceability**: `ARCHITECTURE.md` — Identity and Session Handling owns MFA enrolment state; Relational Persistence stores enrolment and recovery-code entities; Browser Client presents enrolment; trust boundary 2.
- **Security Traceability**: SEC-AUTHN-7 (re-authentication and audit for security-relevant changes), SEC-AUTHN-10 (no other bypass exists), SEC-AUTHN-11 (recovery secret handling), SEC-AUTHN-12 (change revokes sessions), SEC-SECRET-4 (generation).

## Scope

- **Applies To**: Server-Side Application, Web Client, API
- **Components**: Identity and Session Handling; Browser Client; Relational Persistence
- **Interfaces / Operations**: MFA enrolment, MFA disablement, recovery-code issuance and regeneration
- **Actors**: `subscriber`; an attacker holding a hijacked session attempting to disable MFA or harvest codes
- **Preconditions**: An authenticated session for the account whose MFA is being changed
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Personal Data — MFA enrolment state, TOTP secret, recovery codes
- **Jurisdiction / Regulatory Scope**: Global service with GDPR as the design ceiling (`SECURITY.md` SQ-1 RESOLVED): GDPR/UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable

## Security Context

- **Security Objectives**: Authenticity, Confidentiality, Accountability
- **Control Layers**: Authentication, Session Management, Data Protection
- **Threat References**: `SECURITY.md` TM-S-2 (recovery used to bypass MFA), TM-S-5 (session theft), TM-I-9 (residue on shared devices); CWE-308 (use of single-factor authentication), CWE-522 (insufficiently protected credentials), CWE-256 (plaintext storage of a secret)
- **Abuse / Misuse Case**: An attacker with a hijacked session silently disables MFA, or regenerates recovery codes and reads them, converting a temporary session compromise into durable account ownership. Re-authentication is what breaks that chain.
- **Trust Boundary**: Boundary 2 — and specifically the boundary between an ordinary authenticated session and a security-relevant account change, which SEC-AUTHN-7 treats as requiring more.
- **Untrusted Inputs or Assertions**: The submitted TOTP secret confirmation, the account identifier, and any client-supplied enrolment state. The subject of the change is always the authenticated account, never a value from the request.
- **Authoritative Enforcement Point**: Identity and Session Handling; re-authentication is verified server-side and is not a client-side prompt.
- **Independent Verification**: Enrolment completes only after the user proves possession of the new factor — a TOTP code the server verifies, or a completed passkey registration — so a mis-scanned secret cannot lock the user out.
- **Zero Trust Relevance**: NIST SP 800-207 — a sensitive operation is re-authorized independently of the existing session. Exact tenet: TO BE DECIDED.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — mapping verified only at the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — mapping verified only at the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: GDPR/UK GDPR (EU/UK data subjects); CCPA/CPRA, Washington My Health My Data, FTC Health Breach Notification Rule (US users); HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED). Statute-section precision: TO BE DECIDED pending the SQ-1 pre-launch counsel review.
- **Other**: RFC 6238 (TOTP), named by `REQUIREMENTS.md` OQ-9; `REF-63B`, `REF-AUTH`, `REF-PASSKEY`, `REF-WEBAUTHN`.
- **Mapping Basis**: OQ-9 selects TOTP by RFC 6238 and passkeys; SEC-AUTHN-7 and SEC-AUTHN-11 name the OWASP and NIST references; the CWE identifiers name the single-factor and secret-protection classes.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated `subscriber` who re-authenticates, when they enrol a TOTP factor and prove possession with a valid code, then MFA becomes enabled, recovery codes are issued and displayed exactly once, and subsequent authentication requires the second factor.
2. **AC-02 — Boundary or failure behavior**: Given an enrolment attempt where the possession proof is wrong or expired, when it is submitted, then MFA is not enabled, no recovery codes are issued, and the prior state is unchanged.
3. **AC-03 — Prohibited behavior**: Given any of enrolment, disablement, or code regeneration, when it is attempted without fresh re-authentication, then it MUST be refused; and no `admin` or support operation may enable, disable, or reset another account's MFA (SEC-AUTHN-10).
4. **AC-04 — Additional criterion**: Given recovery codes, when they are stored, then only Argon2id digests are persisted, each code is single-use, and the plaintext is never retrievable after the single display (SEC-AUTHN-11).
5. **AC-05 — Additional criterion**: Given regeneration of recovery codes, when it completes, then all previously issued codes for that account are invalidated.
6. **AC-06 — Additional criterion**: Given enrolment, when the flow is presented, then the FR-2.14 statement — that losing every factor and every code makes the account permanently unrecoverable — is shown before the user commits, paired with text and icon rather than color alone.
7. **AC-07 — Additional criterion**: Given enrolment, disablement, or regeneration, when it completes, then all existing sessions for the account are revoked (SEC-AUTHN-12) and an audit entry is written (SEC-AUTHN-7).

## Failure Behavior

- **On Invalid Input**: A malformed or incorrect possession proof is refused with a specific format-level message; nothing about account state is revealed, since the actor is already authenticated as the subject.
- **On Authentication Failure**: A stale or absent re-authentication refuses the change and does not partially apply it.
- **On Authorization Failure**: An attempt to change another account's MFA is refused by owner scoping (REQ-AUTHZ-020); the subject is always the authenticated account.
- **On Security-Decision Failure**: If recovery codes cannot be generated or persisted, enrolment MUST fail and MUST NOT enable MFA — enabling a factor without issuing its recovery path would create the unrecoverable state FR-2.14 warns about, without the user having accepted it.
- **On External Dependency Failure**: N/A — TOTP verification is local; passkey registration is handled by REQ-AUTH-030's mechanism.
- **On System Error**: Generic error with a correlation identifier (SEC-ERR-1); the TOTP secret and recovery codes MUST NOT appear in the response after the single display.
- **Logging / Audit**: Audit each change with account, action, and time (SEC-AUTHN-7, SEC-LOG-4). The TOTP secret, recovery codes, and their digests MUST NOT be logged (SEC-LOG-3, SEC-SECRET-1).
- **Alerting**: Threshold alerts route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-3, SQ-11 RESOLVED); SEC-LOG-4 security-relevant account-change events are among those inputs.

## Test Strategy

- **Unit Tests**: TOTP verification accepts a valid code and rejects a stale one within the permitted drift; recovery-code generator uses the secure generator; regeneration invalidates predecessors; a code-persistence failure aborts enrolment.
- **Integration Tests**: Full enrolment, disablement, and regeneration flows (AC-01, AC-05); failed possession proof leaving prior state intact (AC-02); session revocation on each change (AC-07).
- **Security Tests**: Each operation attempted without re-authentication (AC-03); an `admin` attempting to alter a subscriber's MFA, asserted to be refused (AC-03, SEC-AUTHN-10); storage inspection confirming no plaintext code (AC-04); log assertion that no secret or code is emitted; cross-account attempt.
- **Compliance Tests / Evidence**: The no-admin-MFA-reset transcript, as evidence for SEC-AUTHN-10 and part of the TM-S-2 closure.
- **Acceptance-Criteria Traceability**: AC-01 — enrolment suite; AC-02 — failed-proof test; AC-03 — re-authentication and admin-denial suites; AC-04 — storage inspection; AC-05 — regeneration test; AC-06 — client presentation and accessibility assertion; AC-07 — revocation and audit assertions.
- **Coverage Target**: Every operation × re-authenticated and not, plus both factor types.
- **Required Test Environment**: PostgreSQL with drizzle-kit migrations applied; a `subscriber` with and without MFA; an `admin` for the denial test; a controllable clock for TOTP drift; a WebAuthn authenticator simulator for the passkey factor; Playwright with axe-core for AC-06; Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-AUTH-100, REQ-AUTH-070, REQ-AUTH-060, REQ-SESSION-040, REQ-AUTHZ-020, REQ-AUDIT-010, REQ-PLATFORM-030
- **Downstream Requirements**: REQ-AUTH-120 (the challenge and redemption path consumes this enrolment state)
- **External Dependencies**: A vetted TOTP library and, for the passkey factor, the WebAuthn library from REQ-AUTH-020; both subject to DEP-1 … DEP-8.
- **Dependency Assumptions**: The TOTP library implements RFC 6238 with a configurable drift window and does not log secrets. Confirm during dependency review rather than assume.
- **Failure Impact**: Without recovery codes, an optional security feature becomes a trap: a subscriber who enables MFA and loses their device loses their health data permanently with no warning and no route back.

## Implementation Notes

- **Constraints**: Node.js runtime with Fastify; PostgreSQL with Drizzle ORM; Vue single-page application (`CLAUDE.md`). The recovery-code count is fixed at 10 (SEC-AUTHN-11; `SECURITY.md` SQ-3 RESOLVED) and the re-authentication freshness window at 5 minutes (SEC-AUTHN-7); both MUST be named constants. The TOTP drift window is fixed by no spec document and MUST be a named constant with a documented basis. SMS and email codes are excluded by `REQUIREMENTS.md` OQ-9 and MUST NOT be added.
- **Prohibited Approaches**: Any administrative or support path that enables, disables, or resets a subscriber's MFA (SEC-AUTHN-10). Storing recovery codes or the TOTP secret in recoverable form. Displaying recovery codes more than once or making them re-retrievable. Enabling MFA before possession is proven, which locks users out of their own accounts through a mis-scanned code. Enabling MFA without successfully issuing recovery codes. Treating the FR-2.14 warning as optional copy — it is a normative requirement with its own acceptance criterion.
- **Implementation Guidance**: Order enrolment as re-authenticate, generate secret, prove possession, generate and persist codes, then enable — so any failure leaves MFA off rather than half-on. Because disabling MFA weakens the account, `DESIGN.md` treats it as destructive: confirm explicitly and use the `error` color with text.
- **AI Development Guidance**: `REF-63B`, `REF-AUTH`, `REF-PASSKEY`, `REF-WEBAUTHN`, `REF-PROMPT-QUALITY`, `REF-PROMPT-VUE`; `CLAUDE.md`.
- **Required Human Review**: Security review confirming no path bypasses re-authentication and no administrative route touches subscriber MFA; accessibility review of the one-time code display.
- **Open Decisions**: The TOTP drift window — an implementation constant no spec document fixes. The recovery-code count is resolved at 10 (`SECURITY.md` SQ-3, SEC-AUTHN-11). Whether a subscriber may enrol several second factors simultaneously is implied by `REQUIREMENTS.md` OQ-9 offering both TOTP and passkey, but the multi-factor case is not explicitly specified; implement one factor per account unless a decision records otherwise, and note the position as provisional.

**Estimated effort**: 2 engineer-days. **Estimated changed lines**: 500–900.
**Recommended model**: Claude Opus (`claude-opus-5`) — several irreversible operations whose partial-failure states silently strand users, plus an absolute prohibition on administrative bypass.
