# [REQ-CONSULT-030] Consultant engagement lifecycle by admin action

## Metadata

- **ID**: REQ-CONSULT-030
- **Title**: Consultant engagement lifecycle by admin action
- **Version**: 1.0.1
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-11.1, FR-11.5 (OQ-13 purchase half RESOLVED); `SECURITY.md` SEC-AUTHZ-3, SEC-INPUT-3

## Requirement

- **Statement**: The system MUST create a consultant engagement only through an `admin` action that names the consultant and the subscriber, MUST treat the engagement as active from creation until it is ended by the subscriber under FR-11.3 or revoked by an `admin`, and MUST record an audit entry for each creation and each administrative revocation.
- **Rationale**: FR-11.5 fixes the engagement lifecycle: admin-only creation, active-from-creation semantics, and audited creation and revocation, with payment out of band in v1 (OQ-1, OQ-18). FR-11.1 makes the engagement the paid consultant option and requires the subscriber's current engagement state to be visible to them. The engagement record is the sole predicate SEC-AUTHZ-3 evaluates, so its lifecycle is itself security-enforcing: a forgeable or client-assignable engagement is consultant access to health data without a grant.
- **Assumptions**: Consultant accounts exist via privileged invitation (FR-2.10, REQ-AUTH-050 lineage) and authenticate with passkeys (REQ-AUTH-020). The admin performing the action is authenticated and authorized through the policy module (REQ-AUTHZ-050). Payment for the option occurs out of band in v1 and has no system representation (`REQUIREMENTS.md` OQ-1, OQ-18; `DESIGN.md` states payment collection is not represented).
- **Out of Scope**: Enforcing engagement scope on consultant data access (REQ-CONSULT-010); the subscriber-initiated end of an engagement and its revocation effect (FR-11.3 — REQ-CONSULT-020); what a consultant may do within an active engagement (FR-11.6 — REQ-CONSULT-040); ending engagements as part of consultant deprovisioning (FR-2.17 — REQ-AUTH-170) or consultant account deletion (FR-9.4 — REQ-PRIVACY-090); self-serve purchase (OQ-18, deferred).
- **Design Traceability**: `DESIGN.md` — Information Architecture and Navigation: admin "Access" contains consultant engagements; "Access (admin)" pattern: engagement actions name the affected subscriber by administrative identity, show the engagement bounds being written, and confirm with "Recorded and audited" (FR-3.5, FR-11.5). Subscriber visibility: consultant access appears in a Home summary and Account → Consultant access, both naming the consultant, stating the exact access granted by FR-11.6, and offering **End access** (OQ-7 RESOLVED).
- **Architecture Traceability**: `ARCHITECTURE.md` — consultant engagement entity (Data model expectations); REST API Application ("consultant engagement scoping (FR-11.2, FR-11.3) before any data access"; owns the consultant engagement object per DR-4); data flow 7 relies on the engagement state this issue writes.
- **Security Traceability**: SEC-AUTHZ-3 (the predicate this record feeds), SEC-INPUT-3 (engagement state is a server-controlled field), SEC-SESSION-4 (revocation takes effect without waiting for token expiry), SEC-LOG-4; capability restriction to `admin` via SEC-AUTHZ-5/SQ-4 (REQ-AUTHZ-050).

## Scope

- **Applies To**: Server-Side Application, API
- **Components**: REST API Application; Relational Persistence; Browser Client (admin Access views; subscriber Home and Account visibility)
- **Interfaces / Operations**: Admin engagement-creation and engagement-revocation operations; subscriber engagement-state read; admin engagement listing for FR-11.5
- **Actors**: `admin` (acting); `consultant` and `subscriber` (named parties); `subscriber` as viewer of their own engagement state
- **Preconditions**: Authenticated `admin` session (passkey, REQ-AUTH-020); the named consultant account has role `consultant` and the named subscriber account has role `subscriber`
- **Data Classification**: Confidential — the engagement record is personal data linking two identities, not health data (FR-9.12)
- **Personal or Regulated Data**: Personal Data
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED)

## Security Context

- **Security Objectives**: Authorization, Integrity, Accountability
- **Control Layers**: Authorization, Business-Rule Validation, Input Validation, Logging and Monitoring
- **Threat References**: `SECURITY.md` TM-E-3 (consultant capability creep), TM-I-2 (consultant retains access after engagement ends), TM-E-1 (privilege escalation via provisioning flows), TM-T-1 (mass assignment of server-controlled fields); CWE-862 (missing authorization), CWE-915 (improperly controlled modification of dynamically-determined object attributes)
- **Abuse / Misuse Case**: A consultant creates or reactivates an engagement naming themselves and a chosen subscriber; a request body sets engagement state or parties directly; an admin revocation leaves the consultant's existing session with working access; an engagement is created or revoked with no audit trail, making the grant repudiable.
- **Trust Boundary**: Boundary 1 — the consultant and subscriber identifiers in the admin's request are untrusted input; the engagement state is never client-assignable.
- **Untrusted Inputs or Assertions**: Named consultant and subscriber identifiers; any client-supplied engagement state, bounds, or identifier.
- **Authoritative Enforcement Point**: REST API Application — the admin-only capability is evaluated at the REQ-AUTHZ-050 enforcement point; party roles are verified from persisted state before the engagement row is written.
- **Independent Verification**: The roles of both named parties are re-read from persisted accounts, not taken from the request; engagement state transitions are computed server-side (SEC-INPUT-3, DR-3).
- **Zero Trust Relevance**: NIST SP 800-207 — access grants derive from current recorded relationship state, administered explicitly. Exact tenet: TO BE DECIDED (not verified against the publication in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — per-issue mappings are verified during the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — per-issue mappings are verified during the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: The engagement is the recorded basis for third-party access to health data, so the `SECURITY.md` SQ-1 regime set applies — GDPR/UK GDPR for EU/UK data subjects; CCPA/CPRA, Washington My Health My Data, and the FTC Health Breach Notification Rule for US users; HIPAA not applicable. Statute-section precision: TO BE DECIDED (SQ-1 counsel review, SQ-10).
- **Other**: `REF-PROMPT-ABAC`, `REF-ASVS-5` as cited by SEC-AUTHZ-3; `REF-PROMPT-API` for SEC-INPUT-3.
- **Mapping Basis**: FR-11.1 and FR-11.5 are the normative sources; SEC-AUTHZ-3 consumes the record; SEC-INPUT-3 governs its fields; the CWE identifiers name the missing-authorization and mass-assignment classes.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated `admin` and accounts with roles `consultant` and `subscriber`, when the admin creates an engagement naming both, then an engagement record is persisted in the active state effective immediately, exactly one audit entry records the acting admin, the action, both named accounts, and the time, and the subscriber's Home summary and Account → Consultant access views reflect the engagement naming the consultant and the FR-11.6 scope.
2. **AC-02 — Expected behavior**: Given an active engagement, when an `admin` revokes it, then the engagement leaves the active state immediately, the consultant's access to that subscriber's data is denied on their very next request without waiting for session expiry (SEC-SESSION-4, via REQ-CONSULT-010/REQ-CONSULT-020 enforcement), and exactly one audit entry records the acting admin, the action, both accounts, and the time.
3. **AC-03 — Boundary or failure behavior**: Given a creation request naming a party with the wrong role (a subscriber named as consultant, an admin named as subscriber, a nonexistent account) or naming a pair that already has an active engagement, when the admin submits it, then the request is rejected with a specific structured reason, no engagement row is written, and no audit entry claims a creation occurred.
4. **AC-04 — Prohibited behavior**: Given a request from a `subscriber` or `consultant` session, when it attempts engagement creation or revocation, then it is denied by the policy module; and given any request whose body supplies an engagement state, party substitution, or engagement identifier for creation, then those server-controlled fields MUST NOT be bound from the request (SEC-INPUT-3) — an engagement MUST NOT come into existence or change state through any path other than this admin action, FR-11.3 subscriber ending, FR-2.17 deprovisioning, or FR-9.4 deletion.

## Failure Behavior

- **On Invalid Input**: Reject per REQ-API-010 schema validation with the specific failing field; no engagement state change; malformed identifiers are rejected, not coerced.
- **On Authentication Failure**: Deny per REQ-AUTHZ-010 with the uniform unauthenticated response.
- **On Authorization Failure**: Deny via the REQ-AUTHZ-050 policy module; the response does not disclose whether the named accounts exist beyond what the admin's own administrative listing (FR-10.3) already shows.
- **On Security-Decision Failure**: Deny by default; an exception during role verification or state transition denies and rolls back (SEC-AUTHZ-7 discipline).
- **On External Dependency Failure**: N/A — no external dependency; persistence unavailability fails the operation atomically.
- **On System Error**: The engagement write and its audit entry succeed or fail together — no unaudited engagement can exist; generic error with correlation identifier (SEC-ERR-1).
- **Logging / Audit**: One audit entry per creation and per administrative revocation: acting admin, action, consultant account, subscriber account, time (FR-11.5; SEC-LOG-4 for the security event). Entries reference identities; they contain no health values (SEC-LOG-3).
- **Alerting**: Threshold alerts route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-3, SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: State-transition rules (created → active; active → revoked; no client-supplied state accepted); party-role validation for both named accounts; duplicate-active-engagement rejection; capability mapping restricting creation and revocation to `admin`.
- **Integration Tests**: Create-and-revoke flow against persistence asserting the engagement row, its state transitions, and the paired audit entries; subscriber engagement-state read reflecting creation and revocation (FR-11.1 visibility); revocation followed by an immediate consultant request asserting denial (with REQ-CONSULT-010).
- **Security Tests**: Attempted creation and revocation as `subscriber` and `consultant`, asserting denial; mass-assignment probes supplying engagement state, parties, and identifiers in bodies, asserting the fields are not bound (SEC-INPUT-3); audit-omission probe asserting a failed audit write fails the operation.
- **Compliance Tests / Evidence**: Audit-entry field assertions (acting admin, action, both accounts, time) retained as evidence for the FR-11.5 record.
- **Acceptance-Criteria Traceability**: AC-01 — creation integration suite; AC-02 — revocation and immediate-denial suite; AC-03 — party-validation and duplicate-engagement suite; AC-04 — role-denial and mass-assignment suites.
- **Coverage Target**: Project coverage threshold is 90% line and branch (`CLAUDE.md`, 2026-08-03); all transition, denial, and audit paths MUST have positive and negative tests.
- **Required Test Environment**: Vitest; HTTP test client; fixture accounts for all three roles; persistence with the consultant engagement and audit entry tables.

## Dependencies

- **Upstream Requirements**: REQ-AUTHZ-050, REQ-AUDIT-010, REQ-API-010, REQ-API-020
- **Downstream Requirements**: REQ-CONSULT-010 (evaluates the engagement state this issue writes), REQ-CONSULT-020, REQ-CONSULT-040, REQ-AUTH-170 (deprovisioning ends engagements), REQ-PRIVACY-090 (consultant deletion first ends engagements)
- **External Dependencies**: None — payment is out of band in v1 (OQ-1, OQ-18).
- **Dependency Assumptions**: The audit entry model (REQ-AUDIT-010) provides an append-only write the engagement transaction can join; the policy module exposes admin-scoped engagement capabilities.
- **Failure Impact**: A forgeable engagement is unaudited third-party access to health data (TM-E-3); a revocation that does not take immediate effect is TM-I-2 realized.

## Implementation Notes

- **Constraints**: TypeScript on Node.js with Fastify; PostgreSQL via Drizzle (`CLAUDE.md`). Engagement state and parties are server-controlled fields under SEC-INPUT-3. The engagement write and audit write are one transaction.
- **Prohibited Approaches**: Client-assignable engagement state or parties; consultant- or subscriber-initiated creation; representing payment state (out of band in v1); soft "inactive" flags that FR-11.3/REQ-CONSULT-020 enforcement does not read; caching engagement state in the session or token (SEC-SESSION-4, SEC-SESSION-6); re-enabling a revoked engagement in place rather than creating a new audited engagement.
- **Implementation Guidance**: Model the lifecycle as explicit states (active; ended-by-subscriber; revoked-by-admin) with the creating and ending actor and times on the record, so the SEC-AUTHZ-3 predicate is a single state comparison and the `DESIGN.md` Access views can show engagement bounds. The admin listing needed to pick the parties is the FR-10.3 administrative view — administrative account fields only (REQ-AUTHZ-060).
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-ABAC`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the lifecycle state machine, the transaction joining engagement and audit writes, and the policy-module capabilities.
- **Open Decisions**: None — OQ-13's purchase half and OQ-1 are RESOLVED; self-serve payments remain deferred (OQ-18) with the engagement record as the seam.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 350–650.
**Recommended model**: Claude Opus (`claude-opus-5`) — the single mutation path for the predicate that grants third-party access to health data, where a bindable state field or a skippable audit write is a direct TM-E-3 compromise.
