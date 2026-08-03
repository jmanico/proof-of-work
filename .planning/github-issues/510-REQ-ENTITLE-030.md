# [REQ-ENTITLE-030] Admin subscription-period grant, extension, and revocation

## Metadata

- **ID**: REQ-ENTITLE-030
- **Title**: Admin subscription-period grant, extension, and revocation
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-3.5 (OQ-1 RESOLVED); `SECURITY.md` SEC-AUTHZ-8, SEC-INPUT-3; threat TM-E-4

## Requirement

- **Statement**: The system MUST allow only an `admin` account to grant, extend, and revoke a subscription period on a subscriber account, and MUST record an audit entry for each such action capturing the acting admin, the affected account, the action, the period bounds, and the time — refusing the action if the audit entry cannot be written.
- **Rationale**: FR-3.5 defines the only mechanism by which subscription periods come into existence, change, or end, with a full audit record; because FR-3.6 makes granted periods the sole activation mechanism, this operation is the entire attack surface for entitlement forgery (threat TM-E-4), so it is admin-only, audited, and never client-assignable outside this path (SEC-INPUT-3). Payment collection is out of band in v1 (OQ-1); an admin MAY grant a time-boxed courtesy period through this ordinary mechanism (OQ-2).
- **Assumptions**: The subscription-period record exists (REQ-ENTITLE-010); the audit-entry model and append-only enforcement exist (REQ-AUDIT-010); role restriction is evaluated in the central policy module (REQ-AUTHZ-050).
- **Out of Scope**: The entitlement gate that consumes the periods (REQ-ENTITLE-010); the subscriber's status view (REQ-ENTITLE-020); retention across lapse (REQ-ENTITLE-040); admin listing and search of subscriber accounts by administrative fields, which this operation uses to find the target account (FR-10.3, REQ-AUTHZ-060); consultant-engagement creation and revocation, recorded analogously under FR-11.5 (REQ-CONSULT-030); self-serve payments (OQ-18).
- **Design Traceability**: `DESIGN.md` — Access (admin): "Subscription-period grants and consultant-engagement actions name the affected subscriber by administrative identity, show the period or engagement bounds being written, and confirm with 'Recorded and audited' (FR-3.5, FR-11.5)"; the admin workspace's Access destination (Information Architecture and Navigation).
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application (owns the `subscription` entity; "Subscription activation is resolved: admin-granted periods, audited"); DR-4 (only the owning component mutates business objects); traceability row FR-3.1–FR-3.3, FR-3.5, FR-3.6.
- **Security Traceability**: SEC-AUTHZ-8; SEC-INPUT-3 (subscription state is a server-controlled field settable only through this admin operation); SEC-AUTHZ-1/SEC-AUTHZ-4-style admin-only restriction via the SEC-AUTHZ-5 enforcement point; audit under the FR-3.5 field set with SEC-LOG-2 append-only semantics; supports SEC-SESSION-4 (a revocation must take effect immediately through the REQ-ENTITLE-010 gate).

## Scope

- **Applies To**: API, Server-Side Application
- **Components**: REST API Application; Relational Persistence; Browser Client (admin Access views)
- **Interfaces / Operations**: Grant, extend, and revoke operations on subscription periods; the audit write bound to each
- **Actors**: `admin` (permitted); `subscriber` and `consultant` (denied); a compromised or malicious actor attempting entitlement forgery (TM-E-4)
- **Preconditions**: Valid authenticated `admin` session (passkey-authenticated per FR-2.8); the target subscriber account exists
- **Data Classification**: Confidential — subscription state is treated as sensitive (`SECURITY.md`, Sensitive or regulated data)
- **Personal or Regulated Data**: Personal Data — subscription records are personal data but not health data (FR-9.12)
- **Jurisdiction / Regulatory Scope**: `SECURITY.md` SQ-1 RESOLVED — GDPR/UK GDPR for EU/UK data subjects; CCPA/CPRA, Washington My Health My Data, and the FTC Health Breach Notification Rule for US users; HIPAA not applicable. Specific sections: TO BE DECIDED.

## Security Context

- **Security Objectives**: Authorization, Integrity, Accountability
- **Control Layers**: Authorization, Input Validation, Business-Rule Validation, Logging and Monitoring
- **Threat References**: `SECURITY.md` TM-E-4 (subscription entitlement bypass via the activation mechanism), TM-T-1 (mass assignment of server-controlled fields); CWE-862 (missing authorization), CWE-915 (mass assignment), CWE-778 (insufficient logging)
- **Abuse / Misuse Case**: A subscriber or consultant invokes the grant operation directly; a request smuggles period fields into an unrelated write to mint entitlement (mass assignment); a hostile admin grants themselves-adjacent accounts unaudited access by suppressing the audit write; a revocation is performed but the affected subscriber's session keeps serving gated content.
- **Trust Boundary**: Boundary 1 (Browser Client → REST API Application) — the admin client is untrusted like any client; role comes from the session, target and bounds are validated server-side.
- **Untrusted Inputs or Assertions**: The target account identifier, the period bounds, and the action selector in the request; any role assertion (role comes from Identity and Session Handling only, DR-3).
- **Authoritative Enforcement Point**: REST API Application — admin-role check at the SEC-AUTHZ-5 enforcement point; period writes and their audit entries in the same transactional path (DR-9 discipline: the audit write is a dependency of the operation, not an optional caller responsibility).
- **Independent Verification**: Role from the verified session record; the target account's existence and role verified against persisted state before any write.
- **Zero Trust Relevance**: NIST SP 800-207 — administrative mutation of entitlement is itself a per-request authorization decision. Exact tenet: TO BE DECIDED (not verified against the publication in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — mapping deferred to the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A — no AI-enabled component is involved.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — mapping deferred per SQ-10.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: The SQ-1 regime set applies to the personal data handled; the audit record supports accountability obligations. Specific articles/sections: TO BE DECIDED.
- **Other**: `REF-ASVS-5`, `REF-PROMPT-ABAC` (SEC-AUTHZ-8); `REF-API-2023` (SEC-INPUT-3); `REF-LOG` (audit discipline).
- **Mapping Basis**: The cited SEC rules name these references; the CWE identifiers name the missing-authorization, mass-assignment, and insufficient-logging classes this operation must close.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated `admin` and an existing subscriber account, when the admin grants a period with valid bounds, extends an existing period, or revokes a period, then the period records change accordingly, and exactly one audit entry is written for the action capturing the acting admin, the affected account, the action, the period bounds, and the time (FR-3.5).
2. **AC-02 — Boundary or failure behavior**: Given a grant or extension whose bounds are malformed — end not after start, or values failing schema validation — or a revocation naming a nonexistent period, when the operation is attempted, then it is rejected with a structured field-level error (SEC-INPUT-1, SEC-ERR-1), no period record changes, and no partial audit entry is left behind.
3. **AC-03 — Prohibited behavior**: Given an authenticated `subscriber` or `consultant`, when they invoke any of these operations, then the request is denied and no state changes; and given any non-admin write operation anywhere in the API, when its payload includes subscription-period or subscription-state fields, then those fields MUST NOT be bound (SEC-INPUT-3) — this admin operation is the only path that mutates periods.
4. **AC-04 — Audit is mandatory**: Given the audit write fails, when a grant, extension, or revocation is attempted, then the period mutation MUST NOT commit — the action and its audit entry succeed or fail together.
5. **AC-05 — Revocation bites immediately**: Given a subscriber with an active session inside a granted period, when an admin revokes that period, then the subscriber's next gated request is denied by the REQ-ENTITLE-010 gate without waiting for token expiry (SEC-SESSION-4).

## Failure Behavior

- **On Invalid Input**: Reject with field-level errors per SEC-INPUT-1/SEC-ERR-1; no period or audit state change.
- **On Authentication Failure**: Uniform unauthenticated denial (REQ-AUTHZ-010, SEC-AUTHN-3).
- **On Authorization Failure**: Deny for non-admin actors with no state change; the denial does not disclose whether the target account or period exists (SEC-AUTHZ-1).
- **On Security-Decision Failure**: Deny by default; an unresolvable role or target attribute denies (SEC-AUTHZ-7).
- **On External Dependency Failure**: If Relational Persistence is unavailable, the operation fails whole with a generic error; no partial write (period without audit, or audit without period).
- **On System Error**: Transaction rolls back; generic message with correlation identifier (SEC-ERR-1).
- **Logging / Audit**: One audit entry per successful action with the FR-3.5 field set, append-only (SEC-LOG-2, REQ-AUDIT-010); denials logged as authorization denials (SEC-LOG-4); no credentials or tokens in any record (SEC-LOG-3).
- **Alerting**: Threshold alerts on anomalous grant/revocation volume or repeated non-admin attempts route to the security lead as SEC-OPS-2 detection inputs (SQ-3, SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: Bound validation (end after start, schema types); action-to-audit field mapping; extension and revocation state transitions over period fixtures.
- **Integration Tests**: Grant → gate passes (with REQ-ENTITLE-010); revoke → next request denied (AC-05); audit entry persisted with all five FR-3.5 fields; transactional failure injection asserting atomicity (AC-04, AC-02).
- **Security Tests**: Role matrix — subscriber and consultant invoking each operation, asserting denial (AC-03); mass-assignment suite submitting period/subscription fields on non-admin writes, asserting they are ignored (SEC-INPUT-3); attempt to mutate or delete the resulting audit entry through every role, asserting denial (SEC-LOG-2).
- **Compliance Tests / Evidence**: Audit-entry samples for each action type as accountability evidence; the mass-assignment suite result.
- **Acceptance-Criteria Traceability**: AC-01 — action integration suite; AC-02 — validation and atomicity suite; AC-03 — role-matrix and mass-assignment suites; AC-04 — audit-failure injection test; AC-05 — revocation-propagation test.
- **Coverage Target**: Project coverage threshold is TO BE DECIDED (`CLAUDE.md`); every action × role combination and every failure path MUST have positive and negative tests.
- **Required Test Environment**: PostgreSQL with migrations applied; identities for all three roles (admin passkey-authenticated fixture); a controllable clock; fault-injection capability for the audit write; Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-ENTITLE-010, REQ-AUDIT-010, REQ-AUTHZ-050
- **Downstream Requirements**: REQ-ENTITLE-020 (bounds shown to the subscriber), REQ-ENTITLE-040 (reactivation path), REQ-AUTHZ-060 (admin account views display subscription periods)
- **External Dependencies**: None — payment collection is out of band in v1 (OQ-1, OQ-18).
- **Dependency Assumptions**: REQ-AUDIT-010's append-only audit table is transactionally co-located with period records so the succeed-or-fail-together property (AC-04) is achievable with an ordinary transaction.
- **Failure Impact**: If this operation is reachable by non-admins or bypassable via mass assignment, entitlement is forgeable (TM-E-4); if the audit write is skippable, subscription manipulation becomes repudiable.

## Implementation Notes

- **Constraints**: Node.js with Fastify; PostgreSQL with Drizzle ORM (`CLAUDE.md`). Period mutation and audit write in one transaction (AC-04). Admin UI surfaces this under the Access destination with the "Recorded and audited" confirmation (`DESIGN.md`).
- **Prohibited Approaches**: Any code path that creates or alters period rows outside this operation (FR-3.6); binding period fields from non-admin request bodies (SEC-INPUT-3); fire-and-forget audit writes; deleting period history on revocation instead of recording the revocation (the audit trail and FR-3.4 retention depend on the record surviving).
- **Implementation Guidance**: Model revocation as ending or annulling a period row rather than deleting it, so the audit entry's period bounds always reference a real record. OQ-2: a courtesy period is just an ordinary grant — build no separate trial machinery. This operation's shape is the OQ-18 seam: a future payment processor would drive these same grant/revoke primitives.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-ABAC`, `REF-PROMPT-QUALITY`, `REF-LOG`; `CLAUDE.md`.
- **Required Human Review**: Security review of the admin-only restriction, the mass-assignment allow-lists on every write route touching accounts, and the transactional audit coupling.
- **Open Decisions**: None — OQ-1 and OQ-2 are resolved; OQ-18 payments remain deferred and do not block this issue.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 300–600.
**Recommended model**: Claude Opus (`claude-opus-5`) — the single mutation path for entitlement, where a missed role check, a bindable server-controlled field, or a skippable audit write is a direct TM-E-4 compromise.
