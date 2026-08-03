# [REQ-AUTH-170] Privileged deprovisioning

## Metadata

- **ID**: REQ-AUTH-170
- **Title**: Privileged deprovisioning
- **Version**: 1.0.1
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-2.17 (resolves the deprovisioning half of `SECURITY.md` SQ-12); `SECURITY.md` SEC-AUTHN-13

## Requirement

- **Statement**: The system MUST allow an `admin` to deprovision any `admin` or `consultant` account — disabling the account, terminating all of its sessions immediately, invalidating its registered passkeys, and, for a consultant, ending every active engagement with access revoked — MUST refuse the action when it would leave fewer than two `admin` accounts, MUST produce an audit entry, and MUST NOT allow a deprovisioned account to be re-enabled except by a fresh invitation.
- **Rationale**: A departed or compromised privileged account must lose all access at once — sessions, passkeys, and engagement-scoped subscriber data — rather than at token expiry (SEC-SESSION-4); the two-admin floor (FR-2.15) keeps the system administrable and post-hoc review possible; routing re-enablement through a fresh vetted invitation (FR-2.10, SEC-AUTHN-9) prevents quiet restoration of revoked privilege.
- **Assumptions**: Server-side session records make immediate termination enforceable (SEC-SESSION-3; REQ-SESSION-010). Engagement-ending revocation semantics exist (FR-11.3, SEC-AUTHZ-3; REQ-CONSULT-020). Invitation provisioning and the two-admin/two-passkey minimums exist (REQ-AUTH-140, REQ-AUTH-150).
- **Out of Scope**: Subscriber account deletion and the FR-9.4 privileged-deletion floor interaction (REQ-PRIVACY-090); invitation issuance for a replacement or re-enabled account (REQ-AUTH-140, REQ-AUTH-160); administrative revocation of a single engagement without deprovisioning (FR-11.5; REQ-CONSULT-030); passkey lifecycle outside deprovisioning (REQ-AUTH-150).
- **Design Traceability**: `DESIGN.md` — Credentials, account security, and administration → People (admin): "Deprovisioning uses a dedicated destructive page stating what happens — sessions end immediately, passkeys are invalidated, engagements end — and, when the action would leave fewer than two admins, the refusal explains the two-admin minimum and the remedy of inviting a replacement first (FR-2.17)." Destructive actions use text labels and explicit verbs (Core Components → Actions; Typography and Iconography).
- **Architecture Traceability**: `ARCHITECTURE.md` — Identity and Session Handling (credential material, passkey registrations, session state); REST API Application (consultant engagement state); Relational Persistence; trust boundary 2; traceability row FR-2.1–FR-2.18 ("deprovisioning defined (SQ-12 RESOLVED)").
- **Security Traceability**: SEC-AUTHN-13; SEC-SESSION-3 (session termination), SEC-SESSION-4 (revocation without waiting for expiry), SEC-AUTHZ-3 (engagement access ends), SEC-AUTHN-9 (fresh-invitation re-enable), SEC-LOG-4 (audit).

## Scope

- **Applies To**: Multiple — API, Server-Side Application, Web Client
- **Components**: REST API Application; Identity and Session Handling; Relational Persistence; Browser Client (dedicated destructive page)
- **Interfaces / Operations**: Deprovision operation in the admin workspace (People); session-record invalidation; passkey invalidation; engagement termination; authentication attempts by deprovisioned accounts
- **Actors**: `admin` (actor); `admin` and `consultant` (targets); engaged subscribers (whose data access is revoked); deprovisioned account holder (denied)
- **Preconditions**: An authenticated `admin` session; a target `admin` or `consultant` account
- **Data Classification**: Restricted — consultant engagement revocation protects subscriber health-data access
- **Personal or Regulated Data**: Health Data — indirectly, through the engagement access this action revokes
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED). Section-level mappings TO BE DECIDED.

## Security Context

- **Security Objectives**: Authorization, Confidentiality, Accountability, Availability
- **Control Layers**: Authorization, Session Management, Authentication, Business-Rule Validation, Logging and Monitoring
- **Threat References**: `SECURITY.md` TM-I-2 (consultant retains access via still-valid JWT), TM-E-3 (consultant capability creep), TM-D-2's inverse concern — the floor prevents self-inflicted admin lockout; CWE-613 (insufficient session expiration), CWE-284 (improper access control)
- **Abuse / Misuse Case**: A deprovisioned consultant keeps reading an engaged subscriber's health data on an existing session or re-registers a passkey to return; a hostile admin deprovisions the other admins to seize sole control — refused by the two-admin floor; a deprovisioned account is quietly re-enabled without fresh vetting.
- **Trust Boundary**: Boundary 2 — the action revokes authenticated standing; boundary 1 — the target identifier arrives as client input from the acting admin.
- **Untrusted Inputs or Assertions**: The target account identifier in the request; any subsequent session token or passkey assertion presented by the deprovisioned account.
- **Authoritative Enforcement Point**: REST API Application with Identity and Session Handling — role check, floor check, and the disable/terminate/invalidate/end sequence all execute server-side in one operation.
- **Independent Verification**: The acting admin's role comes from Identity and Session Handling (DR-3); the admin count for the floor check is read from persisted state at execution time, never from the client.
- **Zero Trust Relevance**: TO BE DECIDED — not verified against NIST SP 800-207 in this session.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — per-issue mapping awaits the SQ-10 independent pre-launch assessment.
- **OWASP AISVS 1.0**: N/A — no AI-enabled component is involved.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: The SQ-1 regime set applies to the subscriber health-data access this action revokes; no spec document states a section-level mapping — TO BE DECIDED.
- **Other**: `REF-AUTH`, `REF-SESSION`, `REF-ASVS-5` as cited by SEC-AUTHN-13.
- **Mapping Basis**: SEC-AUTHN-13 names these references; the CWE identifiers describe the retained-session and access-control failure classes this operation closes.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated `admin` and a target `consultant` with active sessions, registered passkeys, and an active engagement, when the admin deprovisions the target, then in one operation the account is disabled, every session record is invalidated, every registered passkey is invalidated, every active engagement is ended with the consultant's access to that subscriber's data revoked (FR-11.3, SEC-AUTHZ-3), and exactly one audit entry records the acting admin, the action, the affected account, and the time.
2. **AC-02 — Immediate effect**: Given a session token captured before deprovisioning, when it is presented on any request afterwards, then the request is denied without waiting for token expiry (SEC-SESSION-3, SEC-SESSION-4), and a previously registered passkey assertion no longer completes authentication.
3. **AC-03 — Boundary or failure behavior**: Given exactly two `admin` accounts, when deprovisioning of either admin is attempted, then the action is refused, the refusal names the two-admin minimum with the remedy of inviting a replacement first (FR-2.15; DESIGN.md), no partial state change occurs, and the refusal is logged.
4. **AC-04 — Prohibited behavior**: Given a deprovisioned account, when any actor attempts to re-enable it directly — by flag change, passkey re-registration, or any path other than a fresh invitation under FR-2.10/SEC-AUTHN-9 — then the attempt MUST fail; and deprovisioning MUST NOT be executable by `subscriber` or `consultant` actors.
5. **AC-05 — Ordering safety**: Given the deprovision operation fails partway (for example, engagement termination errors), when the failure occurs, then the operation resolves to a state that is never access-granting: the account does not remain enabled with sessions alive after any revocation step has been reported complete, and the failure is logged with a correlation identifier.

## Failure Behavior

- **On Invalid Input**: An unknown or non-privileged target identifier is rejected against the schema and policy (SEC-INPUT-1) without disclosing account existence beyond what the admin People listing already shows (FR-10.3 administrative fields).
- **On Authentication Failure**: Denied by the deny-by-default gate (REQ-AUTHZ-010); the acting admin authenticates with a passkey (SEC-AUTHN-2).
- **On Authorization Failure**: Non-`admin` actors are denied through the policy module (deny-overrides, SEC-AUTHZ-7); the denial is logged (SEC-LOG-4).
- **On Security-Decision Failure**: Deny by default — an error in the floor check refuses the action rather than proceeding; an error mid-sequence never leaves revocation silently incomplete while reporting success.
- **On External Dependency Failure**: N/A — the operation touches only in-system state (sessions, passkeys, engagements, audit).
- **On System Error**: Generic error with a correlation identifier (SEC-ERR-1); the operation is retryable and idempotent — re-running a partially applied deprovision completes it without duplicate audit entries.
- **Logging / Audit**: One audit entry per completed deprovision (acting admin, action, affected account, time — SEC-AUTHN-13, SEC-LOG-4); floor-refusals and authorization denials logged with reason class; ended engagements produce their FR-11.5-consistent audit trail; no credential or passkey material in logs (SEC-LOG-3).
- **Alerting**: Deprovision events and floor-refusals route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: Floor computation (three admins → allowed; two admins → refused; concurrent deprovisions cannot race below the floor); state machine (enabled → deprovisioned; no deprovisioned → enabled transition); policy decisions per actor role.
- **Integration Tests**: Full deprovision of a consultant mid-engagement asserting immediate denial of their existing session on the engaged subscriber's data (the SEC-AUTHN-13 verification case); passkey assertion refused after deprovisioning; audit entry shape; refusal path leaving all state untouched.
- **Security Tests**: Captured-token replay after deprovisioning across every protected route class; re-enable attempts via mass assignment of account state (SEC-INPUT-3), passkey re-registration, and direct flag mutation, asserting failure; deprovision attempts as `subscriber` and `consultant`, asserting denial; two-admin race test issuing simultaneous deprovisions of both admins, asserting at least two admins remain.
- **Compliance Tests / Evidence**: Audit-trail evidence for a deprovision cycle as SQ-12 accountability evidence.
- **Acceptance-Criteria Traceability**: AC-01 — deprovision integration suite; AC-02 — captured-token and passkey replay suite; AC-03 — floor unit and integration tests; AC-04 — re-enable and role-denial security suite; AC-05 — fault-injection partial-failure test.
- **Coverage Target**: Positive and negative coverage of every revocation step, the floor check, and every denial path (project threshold 90% line and branch, `CLAUDE.md`, 2026-08-03).
- **Required Test Environment**: Vitest and HTTP test client; fixtures for two- and three-admin populations, a consultant with active engagement and live sessions, registered-passkey state; fault-injection hooks for mid-sequence failure.

## Dependencies

- **Upstream Requirements**: REQ-AUTH-140 (invitation provisioning — the only re-enable path), REQ-AUTH-150 (two-admin/two-passkey minimums the floor enforces), REQ-CONSULT-020 (engagement-ending revocation semantics)
- **Downstream Requirements**: REQ-PRIVACY-090 (FR-9.4 refuses admin deletion below the floor — "a replacement is provisioned, or the account deprovisioned under FR-2.17, first")
- **External Dependencies**: None
- **Dependency Assumptions**: Session records are server-side and individually invalidatable (SEC-SESSION-3); ending an engagement revokes data access immediately (SEC-AUTHZ-3, SEC-SESSION-4).
- **Failure Impact**: Without immediate session termination, a deprovisioned consultant retains health-data access until token expiry (TM-I-2); without the floor, an attacker with one admin account can destroy administrative control of the system.

## Implementation Notes

- **Constraints**: Node.js with Fastify; the floor check and the disable step MUST be atomic against concurrent deprovisions (and against FR-9.4 admin deletions) so the admin count can never pass below two.
- **Prohibited Approaches**: Waiting for token expiry instead of invalidating session records (SEC-SESSION-4); soft-disabling only the login path while existing sessions continue; leaving passkeys registered on a disabled account; exposing a re-enable toggle (SEC-AUTHN-13 — fresh invitation only); enforcing the floor or the confirmation only in the Browser Client (DR-2); deleting the account record — deprovisioning disables, it does not delete (FR-9.4 governs deletion).
- **Implementation Guidance**: Implement as a single server-side operation: floor check → disable → terminate sessions → invalidate passkeys → end engagements → audit, with idempotent re-execution for partial-failure recovery. Reuse REQ-CONSULT-020's engagement-termination path rather than duplicating revocation logic (DR-9-adjacent single-path discipline). The DESIGN.md destructive page states the consequences and uses an explicit text-verb action.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-ABAC`, `REF-PROMPT-QUALITY`; `CLAUDE.md` working rules.
- **Required Human Review**: Security review of the revocation sequence, floor atomicity, and re-enable prohibition; product review of the destructive page against DESIGN.md.
- **Open Decisions**: None — FR-2.17 and SEC-AUTHN-13 fix the behavior; per-issue standards mappings await the SQ-10 assessment.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 350–700.
**Recommended model**: Claude Opus (`claude-opus-5`) — a revocation operation where a missed step leaves a hostile privileged account with live access to subscriber health data.
