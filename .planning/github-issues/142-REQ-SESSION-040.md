# [REQ-SESSION-040] Logout and session revocation on credential or authorization change

## Metadata

- **ID**: REQ-SESSION-040
- **Title**: Logout and session revocation on credential or authorization change
- **Version**: 1.1.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-2.4, FR-2.12, FR-2.18; `SECURITY.md` SEC-SESSION-3, SEC-SESSION-4, SEC-AUTHN-12

## Requirement

- **Statement**: The system MUST invalidate a session on logout, and MUST invalidate every session belonging to an account when that account's authentication credentials, factors, or recovery anchor change — password reset or change, MFA enable or disable, recovery-code regeneration, email-address change (FR-2.18), passkey registration or replacement — such that no invalidated session is accepted on any subsequent request.
- **Rationale**: FR-2.4 requires a user to be able to end their session, and SEC-SESSION-3 requires that ending it actually work rather than merely clear a client-side value. SEC-AUTHN-12 extends this to credential changes, so that a recovery performed after a compromise evicts the attacker instead of running alongside them — which is the whole point of resetting a password one suspects is known. SEC-AUTHN-12 (2026-08-03) adds the email-address change (FR-2.18) to the trigger list, because the registered address is the account's recovery anchor. SEC-SESSION-4 requires authorization-state changes to take effect without waiting for expiry; the engagement case is delivered by REQ-CONSULT-020, and this issue supplies the mechanism it uses.
- **Assumptions**: Sessions are persisted records resolved on every request (REQ-SESSION-030), so invalidation is a state change rather than a best-effort denylist.
- **Out of Scope**: Session creation; the session record model and per-request resolution (REQ-SESSION-030); cookie clearing mechanics at the transport layer (REQ-SESSION-050); the consultant-engagement trigger (REQ-CONSULT-020); subscription lapse (`REQUIREMENTS.md` OQ-1 RESOLVED: admin-granted periods, FR-3.5/FR-3.6), which takes effect through per-request entitlement enforcement (SEC-AUTHZ-8, REQ-ENTITLE-010) rather than session invalidation; the individual credential-change operations themselves, which live in REQ-AUTH-110, REQ-AUTH-130, REQ-AUTH-140, and REQ-AUTH-180 and call into this behavior.
- **Design Traceability**: `DESIGN.md` — Components → Buttons (logout is an ordinary action, not a destructive one requiring confirmation) and Form feedback (the post-logout state is announced, not merely navigated to); Product Patterns → Credentials, account security, and administration (added 2026-08-03): the password-reset completion screen states that other sessions end (SEC-AUTHN-12), and the re-authentication prompt covers the sensitive operations whose completion triggers revocation.
- **Architecture Traceability**: `ARCHITECTURE.md` — Identity and Session Handling ("session establishment and termination"); trust boundary 2; data flow 1.
- **Security Traceability**: SEC-SESSION-3, SEC-SESSION-4, SEC-AUTHN-12; supports SEC-AUTHN-7 (security-relevant account changes are audited).

## Scope

- **Applies To**: Server-Side Application, Web Client, API
- **Components**: Identity and Session Handling; Relational Persistence; Browser Client (the logout affordance)
- **Interfaces / Operations**: Logout; the revocation call made by every credential-change operation
- **Actors**: `subscriber`, `consultant`, `admin`; an attacker holding a captured session
- **Preconditions**: An authenticated session exists
- **Data Classification**: Confidential
- **Personal or Regulated Data**: Personal Data — session state
- **Jurisdiction / Regulatory Scope**: Global service, single US primary region with standard lawful cross-border transfer mechanisms (`SECURITY.md` SQ-1 RESOLVED). Regimes: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable.

## Security Context

- **Security Objectives**: Authenticity, Authorization, Accountability
- **Control Layers**: Session Management
- **Threat References**: `SECURITY.md` TM-S-5 (theft and replay of a token), TM-I-9 (health-data residue on shared devices), TM-I-2 (consultant retains access); CWE-613 (insufficient session expiration), CWE-384 (session fixation)
- **Abuse / Misuse Case**: A user logs out on a shared machine and an attacker replays the captured token afterwards; or a subscriber who suspects compromise resets their password, and the attacker's existing session survives the reset because only the credential changed.
- **Trust Boundary**: Boundary 2 — unauthenticated to authenticated.
- **Untrusted Inputs or Assertions**: The request to log out identifies the session only through the resolved session context, never through a client-supplied session or account identifier — otherwise logout becomes a way to terminate someone else's session.
- **Authoritative Enforcement Point**: Identity and Session Handling; invalidation is a persisted state change, and REQ-SESSION-030's per-request resolution is what makes it effective.
- **Independent Verification**: The next request re-reads the session record and finds it invalid; nothing relies on the client having discarded its copy.
- **Zero Trust Relevance**: NIST SP 800-207 — access is re-evaluated per request, so withdrawal takes effect immediately. Exact tenet: TO BE DECIDED.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: GDPR/UK GDPR (EU/UK data subjects); CCPA/CPRA, Washington My Health My Data, FTC HBNR (US users); HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED). Specific article/section mappings: TO BE DECIDED — no source document states one for session revocation.
- **Other**: `REF-SESSION`, `REF-AUTH`, `REF-PROMPT-JWT`, as named by SEC-SESSION-3 and SEC-AUTHN-12.
- **Mapping Basis**: The rules cited name these references; the CWE identifiers name the expiration and fixation classes.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated session, when the user logs out, then the session record is invalidated, the transport credential is cleared (REQ-SESSION-050), and the response confirms the ended session.
2. **AC-02 — Boundary or failure behavior**: Given a session captured before logout, when it is replayed on any protected route afterwards, then it is refused with the uniform unauthenticated response and no handler executes.
3. **AC-03 — Prohibited behavior**: Given a logout request, when it carries a session or account identifier in its body, query, or headers, then that value MUST be ignored — a user MUST NOT be able to terminate another account's session.
4. **AC-04 — Additional criterion**: Given an account with several concurrent sessions, when its password is reset, its MFA is enabled or disabled, its recovery codes are regenerated, its registered email address is changed (FR-2.18), or a passkey is registered or replaced, then every session for that account is invalidated and each previously captured session is refused (SEC-AUTHN-12).
5. **AC-05 — Additional criterion**: Given a logout on an already-invalid session, when it is submitted, then the response is indistinguishable from a successful logout and no error discloses the session's prior state.
6. **AC-06 — Additional criterion**: Given any logout or revocation, when it completes, then an audit entry records the acting account, the trigger class, and the time (SEC-LOG-4, SEC-AUTHN-7), without recording the session identifier (SEC-LOG-3).

## Failure Behavior

- **On Invalid Input**: Extraneous identifiers in a logout request are ignored, not rejected with detail (AC-03).
- **On Authentication Failure**: An unauthenticated logout returns the same result as a successful one (AC-05).
- **On Authorization Failure**: N/A — an actor may only ever end their own session, which is enforced by sourcing the session from the resolved context.
- **On Security-Decision Failure**: If invalidation cannot be persisted, the operation MUST fail loudly rather than report success — a logout that silently did nothing is worse than a visible error, because the user believes they are protected.
- **On External Dependency Failure**: If Relational Persistence is unavailable, logout fails with a generic error and the session remains valid; the client MUST NOT present this as success.
- **On System Error**: Generic error with a correlation identifier (SEC-ERR-1).
- **Logging / Audit**: Audit entry per AC-06. Log revocation counts on credential change for anomaly detection, without session identifiers or credential material (SEC-LOG-3).
- **Alerting**: Threshold alerts on anomalous revocation patterns route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-3, SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: Invalidation marks exactly the intended records; bulk revocation selects all sessions for the account and none for any other; a persistence error propagates as failure rather than success.
- **Integration Tests**: Logout then replay (AC-02); multi-session revocation on each credential-change trigger (AC-04); logout with a foreign identifier asserted to have no effect (AC-03).
- **Security Tests**: Cross-account logout attempt; replay after every SEC-AUTHN-12 trigger — password reset or change, MFA enable/disable, recovery-code regeneration, email-address change, passkey registration or replacement; assertion that a failed invalidation never returns success; audit assertions per AC-06.
- **Compliance Tests / Evidence**: The credential-change revocation matrix, as evidence for SEC-AUTHN-12.
- **Acceptance-Criteria Traceability**: AC-01 — logout suite; AC-02 — replay suite; AC-03 — cross-account suite; AC-04 — trigger matrix; AC-05 — idempotence test; AC-06 — audit assertions.
- **Coverage Target**: Every credential-change trigger × multi-session state, plus positive and negative logout.
- **Required Test Environment**: PostgreSQL with drizzle-kit migrations applied; an account with several concurrent sessions; a second account for cross-account probes; audit capture; Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-SESSION-030, REQ-AUDIT-010, REQ-API-040
- **Downstream Requirements**: REQ-CONSULT-020 (engagement termination uses this mechanism); REQ-AUTH-110, REQ-AUTH-130, REQ-AUTH-140, REQ-AUTH-180 (each credential or recovery-anchor change calls it); REQ-SESSION-050
- **External Dependencies**: None
- **Dependency Assumptions**: Invalidation is visible to the very next request, not eventually — see REQ-SESSION-030's consistency assumption.
- **Failure Impact**: FR-2.4 becomes unsatisfiable and SEC-AUTHN-12 unenforceable; a password reset would leave a compromised session live, which is the specific failure the recovery design was built to prevent.

## Implementation Notes

- **Constraints**: Node.js runtime with Fastify; PostgreSQL with Drizzle ORM (`CLAUDE.md`). Subscription lapse is named in SEC-SESSION-4 and `REQUIREMENTS.md` OQ-1 is RESOLVED (admin-granted periods, FR-3.5/FR-3.6): because the token carries no authorization state and entitlement is re-read from persisted state on every request (SEC-AUTHZ-8, DR-3), lapse takes effect immediately through the entitlement gate (REQ-ENTITLE-010) with no session invalidation required. This issue delivers logout, credential-change revocation, and the mechanism REQ-CONSULT-020 consumes.
- **Prohibited Approaches**: Treating logout as a client-side action that only discards the cookie. Reporting success when invalidation failed. Accepting a client-supplied session or account identifier as the logout target. Leaving credential-change revocation as a convention each credential operation must remember — it MUST be a dependency of those paths, in the spirit of DR-9.
- **Implementation Guidance**: Expose one revocation function taking an account and a trigger class, and have every credential-change path call it, so a new credential operation cannot forget. Bulk revocation on credential change should be a single statement rather than a read-then-write loop, to avoid a window where a session issued mid-operation survives.
- **AI Development Guidance**: `REF-SESSION`, `REF-AUTH`, `REF-PROMPT-JWT`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review that every credential-change path reaches the revocation call, and that no path reports success on a failed invalidation.
- **Open Decisions**: None from the specifications — OQ-1 is RESOLVED and the lapse case is satisfied by per-request entitlement enforcement (REQ-ENTITLE-010). Whether logout offers "this device" versus "all devices" is not specified by any source document and MUST NOT be invented here; this issue implements single-session logout plus account-wide revocation on credential change, and a wider user-facing option would be a new requirement.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 250–500.
**Recommended model**: Claude Opus (`claude-opus-5`) — the correctness bar is that no path silently skips revocation, and the failure mode is invisible to functional testing.
