# [REQ-AUTH-100] Subscriber password authentication

## Metadata

- **ID**: REQ-AUTH-100
- **Title**: Subscriber password authentication
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-01
- **Priority**: Critical
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-2.3, FR-2.6, FR-2.8; `SECURITY.md` SEC-AUTHN-3, SEC-AUTHN-4, SEC-AUTHN-2

## Requirement

- **Statement**: The system MUST authenticate a `subscriber` by verifying a submitted password against the stored credential, MUST issue a session only after any enabled second factor has also been satisfied, and MUST refuse password authentication outright for accounts holding the `admin` or `consultant` role.
- **Rationale**: FR-2.3 requires password authentication and forbids revealing which factor was wrong. SEC-AUTHN-4 requires that a first-factor-only context grant access to nothing, so this issue must not issue a session and then upgrade it. FR-2.8 and SEC-AUTHN-2 make the role check part of authentication rather than a later authorization concern: the credential path is selected by the account's persisted role, so a privileged account cannot be authenticated by choosing a different form.
- **Assumptions**: Credential verification is provided by REQ-AUTH-070; the MFA challenge itself is REQ-AUTH-120; the session record is created by REQ-SESSION-030 and transported by REQ-SESSION-050.
- **Out of Scope**: Credential hashing (REQ-AUTH-070); the MFA challenge mechanics and recovery-code redemption (REQ-AUTH-120); passkey authentication (REQ-AUTH-020); the uniform failure response contract (REQ-AUTH-040, which this issue consumes); throttling (REQ-AUTH-060); password reset (REQ-AUTH-130); subscription entitlement, blocked by `REQUIREMENTS.md` OQ-1.
- **Design Traceability**: `DESIGN.md` — Components → Inputs, Buttons (a busy state that blocks repeat submission), and Form feedback and errors. The failure message is deliberately generic, which is the documented resolution of the tension between naming the specific problem and SEC-AUTHN-3.
- **Architecture Traceability**: `ARCHITECTURE.md` — Identity and Session Handling ("credential verification, MFA challenge for subscribers who enable it, session establishment"); trust boundary 2; data flow 1.
- **Security Traceability**: SEC-AUTHN-3 (non-disclosing failures), SEC-AUTHN-4 (no partial authentication), SEC-AUTHN-2 (no password path for privileged roles), SEC-AUTHN-5 (via REQ-AUTH-070), SEC-LOG-4 (authentication events logged).

## Scope

- **Applies To**: Server-Side Application, Web Client, API
- **Components**: Identity and Session Handling; Browser Client (the login view)
- **Interfaces / Operations**: Password authentication; the transition into an MFA challenge when one is enabled
- **Actors**: `subscriber`; an attacker performing credential stuffing; an attacker holding a privileged account's password
- **Preconditions**: An account exists with a stored password credential
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Personal Data — credentials and authentication events
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED (`SECURITY.md` SQ-1)

## Security Context

- **Security Objectives**: Authenticity, Authorization, Confidentiality
- **Control Layers**: Authentication, Session Management
- **Threat References**: `SECURITY.md` TM-S-1 (credential stuffing and spraying), TM-I-4 (account enumeration by content or timing), TM-S-5 (session theft); CWE-287 (improper authentication), CWE-204 (observable response discrepancy), CWE-307 (excessive authentication attempts)
- **Abuse / Misuse Case**: An attacker with a privileged account's password authenticates through the subscriber login form because the endpoint never checked the account's role; or infers which addresses are registered from a faster failure when no credential exists to verify; or obtains a usable session after the first factor while the MFA step is still pending.
- **Trust Boundary**: Boundary 2 — unauthenticated to authenticated.
- **Untrusted Inputs or Assertions**: The submitted identifier and password, and any client indication of which credential path to use. The role that selects the path comes from persisted state.
- **Authoritative Enforcement Point**: Identity and Session Handling; the role is resolved before the credential type is accepted.
- **Independent Verification**: The second factor requirement is read from the account's persisted MFA enrolment, not from a client flag or a prior request.
- **Zero Trust Relevance**: NIST SP 800-207 — authentication strength is a function of the subject, and privileged subjects require stronger authentication. Exact tenet: TO BE DECIDED.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: TO BE DECIDED — blocked by `SECURITY.md` SQ-1.
- **Other**: `REF-AUTH`, `REF-63B`, `REF-ASVS-5`, as named by SEC-AUTHN-3 and SEC-AUTHN-4.
- **Mapping Basis**: The cited rules name these references; the CWE identifiers name the improper-authentication, response-discrepancy, and attempt-restriction classes.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a `subscriber` with a correct password and no MFA enabled, when authentication is submitted, then a session is created (REQ-SESSION-030), the cookie is set (REQ-SESSION-050), and the success is logged.
2. **AC-02 — Boundary or failure behavior**: Given an incorrect password, an unknown identifier, or a correct password for an account that does not permit password authentication, when authentication is submitted, then the response is identical in content, status, and timing across all three cases, and no session is created (SEC-AUTHN-3).
3. **AC-03 — Prohibited behavior**: Given an `admin` or `consultant` account, when a correct password is submitted on any authentication endpoint, then no session is issued and no indication is given that the password was correct or that the account is privileged (FR-2.8, SEC-AUTHN-2).
4. **AC-04 — Additional criterion**: Given a `subscriber` with MFA enabled and a correct password, when the first factor succeeds, then no session usable on any protected route exists until the second factor is satisfied, and any intermediate state cannot be used as a credential elsewhere (SEC-AUTHN-4).
5. **AC-05 — Additional criterion**: Given repeated failures, when they accumulate, then throttling applies per REQ-AUTH-060, and a legitimate holder recovers after decay without administrative action.
6. **AC-06 — Additional criterion**: Given any authentication attempt, when it resolves, then success and failure are both logged with account reference, cause class, and correlation identifier, and never with the password (SEC-LOG-4, SEC-LOG-3).

## Failure Behavior

- **On Invalid Input**: A malformed submission returns the same uniform failure as an incorrect credential; format-level detail MUST NOT distinguish a missing field from a wrong password in a way that reveals account state.
- **On Authentication Failure**: Deny with the uniform response from REQ-AUTH-040; no session, no partial state, no indication of which factor failed.
- **On Authorization Failure**: N/A — authorization follows a successful session.
- **On Security-Decision Failure**: If the role cannot be resolved, or MFA enrolment state cannot be read, deny. There is no default that assumes no MFA.
- **On External Dependency Failure**: If the credential store is unreadable, deny; MUST NOT fall back to any alternative credential source or cached verdict.
- **On System Error**: Generic error with a correlation identifier (SEC-ERR-1).
- **Logging / Audit**: Per AC-06. A privileged password attempt is a notable event and MUST be logged with its cause class, since it indicates either misconfiguration or an attack.
- **Alerting**: TO BE DECIDED — blocked by `SECURITY.md` SQ-3.

## Test Strategy

- **Unit Tests**: Credential path selector returns password-not-permitted for `admin` and `consultant`; MFA requirement read from persisted enrolment; unresolvable role yields denial.
- **Integration Tests**: Successful authentication with and without MFA enabled (AC-01, AC-04); session usable only after the second factor.
- **Security Tests**: Three-way differential response and timing comparison across wrong password, unknown account, and privileged account (AC-02, AC-03); attempt to use any intermediate first-factor state against protected routes (AC-04); route inventory asserting no alternate session-issuing endpoint accepts a password for a privileged role; burst test for AC-05; log assertion that no password appears (AC-06).
- **Compliance Tests / Evidence**: The privileged-password-denial transcript across every authentication endpoint, as evidence for FR-2.8 and SEC-AUTHN-2.
- **Acceptance-Criteria Traceability**: AC-01 — happy path; AC-02 — differential suite; AC-03 — privileged denial suite plus route inventory; AC-04 — partial-state suite; AC-05 — burst test; AC-06 — log assertions.
- **Coverage Target**: Every role × every credential outcome, plus MFA enabled and disabled, positive and negative.
- **Required Test Environment**: PostgreSQL with drizzle-kit migrations applied; a `subscriber` without MFA, a `subscriber` with MFA, an `admin`, a `consultant`, and a known-unregistered identifier; latency measurement for AC-02; Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-AUTH-070, REQ-AUTH-010, REQ-AUTH-040, REQ-AUTH-060, REQ-SESSION-030, REQ-SESSION-050, REQ-AUDIT-010
- **Downstream Requirements**: REQ-AUTH-120 (the MFA challenge this hands off to); every subscriber-facing issue in the plan, all of which presuppose an authenticated session
- **External Dependencies**: None
- **Dependency Assumptions**: REQ-AUTH-070 reports only match or no-match, so no timing or content difference leaks from the credential layer into this one.
- **Failure Impact**: This is the front door for the only role that uses passwords. Without it, no subscriber can reach any of the plan's subscriber-facing functionality in a running system.

## Implementation Notes

- **Constraints**: Node.js runtime with Fastify; Vue single-page application (`CLAUDE.md`). Subscription entitlement is blocked by `REQUIREMENTS.md` OQ-1, so a successful authentication grants an authenticated session with no entitlement evaluation; that gap must be stated when the issue is closed.
- **Prohibited Approaches**: A shared login endpoint that branches on a client-supplied credential-type flag rather than on the persisted role. Issuing any session, even a limited one, before the second factor completes (SEC-AUTHN-4). Returning different messages, status codes, or response times for unknown account, wrong password, and privileged account. Short-circuiting the credential check when no account exists, which creates a timing oracle — perform equivalent work in both cases. Logging the submitted password anywhere, including in a validation error.
- **Implementation Guidance**: Resolve the account and its role first, then select the permitted credential path, then verify — this ordering is what makes AC-03 hold across every endpoint rather than only the one that remembered to check. For AC-02's timing requirement, coordinate carefully with REQ-AUTH-060's early short-circuit, which is the most likely source of an accidental oracle.
- **AI Development Guidance**: `REF-AUTH`, `REF-63B`, `REF-PROMPT-JWT`, `REF-PROMPT-NODE`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of every session-issuing path, not only this one, confirming none accepts a password for a privileged role.
- **Open Decisions**: None specific to this issue. Entitlement evaluation is blocked by `REQUIREMENTS.md` OQ-1 and belongs to a future issue that will extend this path.

**Estimated effort**: 1.5–2 engineer-days. **Estimated changed lines**: 350–750.
**Recommended model**: Claude Opus (`claude-opus-5`) — response uniformity across three failure modes and an absolute no-password rule for privileged roles are both invisible to functional testing and easy to break.
