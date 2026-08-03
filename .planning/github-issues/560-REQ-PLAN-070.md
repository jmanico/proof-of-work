# [REQ-PLAN-070] One-time plan verification operation

## Metadata

- **ID**: REQ-PLAN-070
- **Title**: One-time plan verification operation
- **Version**: 1.0.1
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-4.5, FR-4.8, FR-10.2 (OQ-10 and OQ-16 RESOLVED); `SECURITY.md` SEC-LOG-6, threat TM-T-5 (RISK ACCEPTED)

## Requirement

- **Statement**: The system MUST provide an admin-only verification operation that records, exactly once per plan and before first publication, which `admin` verified the plan and when; the operation MAY be performed by any `admin` including the plan's author, MUST produce an audit entry, and a later edit to the plan MUST NOT clear the verification record or re-require verification.
- **Rationale**: FR-4.5 requires explicit admin verification before publication with the verifying admin and time recorded; FR-4.8 resolves OQ-10 and OQ-16 by fixing verification as a one-time gate that any admin — including the author — may perform and that edits never re-trigger; FR-10.2 and SEC-LOG-6 make the action auditable, which is the compensating accountability control for the risk-accepted TM-T-5 (no dual control).
- **Assumptions**: Plans exist as draft records with lifecycle state (REQ-PLAN-030); the publication gate (REQ-PLAN-050) consumes the verification record this operation creates; the audit pipeline for admin plan lifecycle actions exists (REQ-AUDIT-030).
- **Out of Scope**: The publication operation itself and its citation-presence gate (REQ-PLAN-050); the citation invariant on edits to published plans (FR-4.4/FR-4.8 edit-time enforcement, REQ-PLAN-030/REQ-PLAN-040); how the review state is displayed to subscribers versus admins (`DESIGN.md` "Plans, citations, and review state"); dual-control verification — consciously declined by OQ-16 and MUST NOT be reintroduced here.
- **Design Traceability**: `DESIGN.md` — "Plans, citations, and review state": admin views show the verifier identity and timestamp required by FR-4.5; subscriber copy shows "Evidence reviewed by Proof of Work" plus the review date without the admin's identity; draft admin views show a publication checklist with separate "Citation present" and "Review recorded" gates.
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application ("Enforces publication gates (FR-4.4, FR-4.5, FR-4.7)"); data flow 6 (admin publication: role check, citation-presence gate, verification record); DR-4 (published plans mutable only by admin-role operations).
- **Security Traceability**: SEC-LOG-6, SEC-AUTHZ-4, SEC-INPUT-3 (verification record is a server-controlled field), SEC-INPUT-4; threat TM-T-5 context (risk accepted with audit as compensating control).

## Scope

- **Applies To**: Server-Side Application, API
- **Components**: REST API Application; Relational Persistence
- **Interfaces / Operations**: Verify-plan operation on exercise and diet plans; the verification-record read used by the publication gate and by admin plan views
- **Actors**: `admin` (permitted); `subscriber` and `consultant` (denied)
- **Preconditions**: An authenticated `admin` session; a plan record that has not yet been verified
- **Data Classification**: Internal
- **Personal or Regulated Data**: Personal Data — the verifying admin's identity in the verification record and audit entry; plan content itself is not personal data
- **Jurisdiction / Regulatory Scope**: `SECURITY.md` SQ-1 regime set applies to the platform generally; no health data is involved in this operation, so no health-specific obligation attaches

## Security Context

- **Security Objectives**: Integrity, Accountability, Safety, Authorization
- **Control Layers**: Authorization, Business-Rule Validation, Logging and Monitoring
- **Threat References**: `SECURITY.md` TM-T-5 (compromised admin publishes harmful content — risk accepted, this audit trail is a named compensating control), TM-R-1 (repudiable admin lifecycle actions); CWE-862 (missing authorization), CWE-778 (insufficient logging)
- **Abuse / Misuse Case**: A subscriber or consultant attempts to verify a plan to push unreviewed content toward publication; a client supplies a forged verification record in a request body; a hostile admin later disputes having verified a harmful plan (repudiation).
- **Trust Boundary**: Boundary 1 — Browser Client → REST API Application; the verification decision and record are created server-side only.
- **Untrusted Inputs or Assertions**: Any client-supplied verifier identity, verification timestamp, or verification-state field — all are server-controlled (SEC-INPUT-3); the plan identifier in the request path.
- **Authoritative Enforcement Point**: REST API Application, behind the single authorization enforcement point (SEC-AUTHZ-5); the verifier identity comes from the authenticated session, the timestamp from the server clock.
- **Independent Verification**: Role is resolved from Identity and Session Handling and persisted state, never from the request (DR-3); the publication gate independently re-reads the persisted verification record (REQ-PLAN-050).
- **Zero Trust Relevance**: NIST SP 800-207 — access to the verification operation is evaluated per request against server-held role state. Exact tenet: TO BE DECIDED (not verified against the publication; SQ-10 assessment).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — per-issue mapping deferred to the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A — no AI-enabled component is involved.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — per `SECURITY.md` SQ-10.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: N/A — no health or consumer-privacy obligation attaches to plan verification itself; the SQ-1 regime set governs the platform generally.
- **Other**: `REF-LOG`, `REF-ASVS-5` as cited by SEC-LOG-6.
- **Mapping Basis**: SEC-LOG-6 names these references for the audit obligation; CWE identifiers describe the missing-authorization and insufficient-logging classes this requirement closes.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated `admin` and an unverified plan, when the admin performs the verification operation, then the plan carries a verification record naming that admin and the server-recorded time, and exactly one audit entry is written capturing the acting admin, the `verify` action, the affected plan, and the time (FR-10.2, SEC-LOG-6).
2. **AC-02 — Author self-verification**: Given an unverified plan whose author is the requesting `admin`, when that admin performs the verification operation, then it succeeds identically to AC-01 — no dual-control check is applied (FR-4.8, OQ-16).
3. **AC-03 — One-time gate**: Given a plan that already carries a verification record, when any `admin` attempts the verification operation again, then the operation is refused and the existing record's verifier and timestamp remain unchanged.
4. **AC-04 — Edits never re-trigger**: Given a verified plan (published or not), when an admin edits the plan, then the verification record is neither cleared nor invalidated, and the plan does not re-enter an unverified state (FR-4.8).
5. **AC-05 — Prohibited behavior**: Given an authenticated `subscriber` or `consultant`, when they attempt the verification operation, then it is denied (SEC-AUTHZ-4); and given any request body containing verifier identity, timestamp, or verification-state fields, when any write operation processes it, then those fields are ignored and never bound (SEC-INPUT-3).

## Failure Behavior

- **On Invalid Input**: Reject a verification request targeting a nonexistent plan or carrying a malformed plan identifier with a schema-validation error (SEC-INPUT-1); no state change occurs.
- **On Authentication Failure**: Deny per the deny-by-default guard (REQ-AUTHZ-010) with a uniform response (SEC-AUTHN-3).
- **On Authorization Failure**: Deny for non-admin roles at the single enforcement point; resource existence MAY be disclosed to authenticated admins only — non-admins receive the standard denial without confirming the plan exists.
- **On Security-Decision Failure**: Deny by default; an error inside policy evaluation denies (SEC-AUTHZ-7).
- **On External Dependency Failure**: N/A — no external dependency; persistence unavailability is a system error.
- **On System Error**: The verification record and its audit entry are written atomically — if the audit write fails, the verification does not commit (DR-9, REQ-AUDIT-030); the response carries a generic message with a correlation identifier (SEC-ERR-1).
- **Logging / Audit**: One audit entry per successful verification (acting admin, action, plan, time — FR-10.2, SEC-LOG-6); authorization denials logged per SEC-LOG-4; no plan content copied into logs (SEC-LOG-3).
- **Alerting**: Threshold alerts on repeated authorization denials route to the security lead as SEC-OPS-2 detection inputs (SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: Verification-state transition rules — unverified→verified succeeds, verified→verified refused without record mutation; edit operations leave the verification record untouched; server-controlled field stripping for verifier, timestamp, and verification state.
- **Integration Tests**: End-to-end verify as admin (including as the plan's author) asserting the persisted record and the single audit entry; verify-then-edit-then-publish flow asserting the gate still passes without re-verification; atomicity test asserting a failed audit write rolls back the verification.
- **Security Tests**: Verification attempts as subscriber and consultant asserting denial (SEC-AUTHZ-4); mass-assignment test submitting verification-record fields in create and edit bodies asserting they are ignored (SEC-INPUT-3); replay of the verify operation asserting the original verifier and timestamp survive.
- **Compliance Tests / Evidence**: Audit-entry field assertions (acting admin, action, plan, time) as evidence for the TM-T-5 compensating control.
- **Acceptance-Criteria Traceability**: AC-01 — integration verify suite; AC-02 — author-self-verify test; AC-03 — repeat-verify unit and integration tests; AC-04 — edit-after-verify suite; AC-05 — role-denial and mass-assignment security tests.
- **Coverage Target**: Project coverage threshold is 90% line and branch (`CLAUDE.md`, 2026-08-03); all state transitions and denial paths MUST have positive and negative tests.
- **Required Test Environment**: Vitest; HTTP test client; fixtures for admin (author and non-author), subscriber, and consultant identities; draft, verified, and published plan fixtures.

## Dependencies

- **Upstream Requirements**: REQ-PLAN-030 (plan creation and editing), REQ-PLAN-050 (publication gate that consumes the verification record), REQ-AUDIT-030 (admin plan lifecycle audit entries)
- **Downstream Requirements**: None beyond REQ-PLAN-050's consumption of the record; subscriber-facing review-state display rides with the plan-view issues (REQ-CATALOG-010, REQ-CATALOG-020)
- **External Dependencies**: None
- **Dependency Assumptions**: REQ-AUDIT-030 provides an atomic audit write the verification operation can participate in (DR-9); REQ-PLAN-050 refuses publication when no verification record exists, independently of this operation.
- **Failure Impact**: If verification can be repeated or forged, the FR-4.5 accountability record loses meaning and the TM-T-5 compensating control (attributable, audited verification) collapses.

## Implementation Notes

- **Constraints**: Node.js/Fastify with the single authorization enforcement point (`CLAUDE.md`, SEC-AUTHZ-5); verifier identity MUST come from the session context and the timestamp from the server clock, both persisted with the plan.
- **Prohibited Approaches**: Accepting verifier, timestamp, or verification-state values from any request body (SEC-INPUT-3); implementing verification as a client-side checklist state (DR-2); clearing or resetting verification on edit or unpublication (FR-4.8); adding a second-admin approval step — OQ-16 declined dual control and reintroducing it is a spec change, not an implementation choice.
- **Implementation Guidance**: Model the verification record as an immutable, nullable-once structure on the plan (verifier account identifier, timestamp) rather than a boolean, so REQ-PLAN-050 and admin views read one authoritative source; the refusal on re-verification keeps the record append-never-update.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-QUALITY`; `CLAUDE.md` working rules (trace every change; do not resolve open questions unilaterally).
- **Required Human Review**: Security review of the authorization and server-controlled-field handling; product review that the one-time semantics match FR-4.8.
- **Open Decisions**: None — OQ-10 and OQ-16 are resolved; per-issue standards mappings await the SQ-10 assessment.

**Estimated effort**: 0.5–1 engineer-day. **Estimated changed lines**: 150–350.
**Recommended model**: Claude Opus (`claude-opus-5`) — a small operation, but it is the accountability record compensating for the risk-accepted TM-T-5 and must be exactly once, immutable, and audited.
