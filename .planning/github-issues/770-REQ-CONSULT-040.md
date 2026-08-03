# [REQ-CONSULT-040] Consultant capabilities within an active engagement

## Metadata

- **ID**: REQ-CONSULT-040
- **Title**: Consultant capabilities within an active engagement
- **Version**: 1.0.1
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-11.6, FR-11.4 (OQ-12 RESOLVED); `SECURITY.md` SEC-AUTHZ-3, threat TM-E-3

## Requirement

- **Statement**: Within an active engagement, the system MUST allow the consultant to view the engaged subscriber's selected plans, customized plan copies, and progress logs and to edit that subscriber's customized plan copies, MUST NOT allow the consultant any other operation on the subscriber's data — including creating, editing, or deleting log entries — and MUST produce an audit entry for every consultant view and edit.
- **Rationale**: FR-11.6 fixes the consultant capability set as views plus plan-copy edits and nothing else, with ownership of every copy remaining with the subscriber (FR-7.2) and every view and edit audited per FR-11.4. `REQUIREMENTS.md` OQ-12 resolved that there is no log-entry write, no messaging, and no access outside the engagement; threat TM-E-3 (consultant capability creep) is bounded by exactly this enumeration.
- **Assumptions**: The active-engagement predicate is enforced by REQ-CONSULT-010 and evaluated in the policy module (REQ-AUTHZ-050); engagements exist via REQ-CONSULT-030; subscriber plan copies exist via REQ-CUSTOM-010. Consultant edits to a plan copy are health-data writes on the subscriber's record (FR-9.12) already gated by the subscriber's consent and verification state at collection time — this issue adds no consent flow.
- **Out of Scope**: The engagement-existence predicate itself and its audit obligation as an access condition (REQ-CONSULT-010); engagement ending and revocation effects (REQ-CONSULT-020, REQ-CONSULT-030); the copy-on-customize semantics and copy ownership model (REQ-CUSTOM-010); subscriber-side plan-copy editing (REQ-CUSTOM-020 lineage); any consultant messaging channel — a new requirements change per OQ-12.
- **Design Traceability**: `DESIGN.md` — Information Architecture and Navigation: the consultant workspace is "Clients, Account" with a persistent selected-client context bar stating the subscriber name, the active-engagement status, and the scope "View progress and selected plans; edit plan copies"; changing clients is an explicit action. A customized plan edited by a consultant shows a neutral "Edited by your consultant" provenance line with the time — not an endorsement badge (OQ-7 RESOLVED). Dense tables are allowed in the consultant workspace at desktop sizes.
- **Architecture Traceability**: `ARCHITECTURE.md` — data flow 7 ("Consultant client → REST API → active-engagement check → scoped subscriber data → audit entry written"); DR-4 ("subscriber plan copies are mutable only by their owning subscriber or, under FR-11.6, by a consultant with an active engagement"); DR-9 (audit writing as a dependency of every health-data access path).
- **Security Traceability**: SEC-AUTHZ-3, SEC-AUTHZ-2 (object-level scoping in the query), SEC-LOG-1 (audit on every view and edit), SEC-DATA-5 (least-privilege response shape), SEC-INPUT-3 (copy ownership never reassignable).

## Scope

- **Applies To**: Server-Side Application, API, Web Client
- **Components**: REST API Application; Relational Persistence; Browser Client (consultant workspace; subscriber-visible provenance line)
- **Interfaces / Operations**: Consultant reads of an engaged subscriber's selected plans, plan copies, and workout/food/weight/measurement logs; consultant edits of that subscriber's plan copies; every other operation on subscriber data as the denied set
- **Actors**: `consultant` (acting); `subscriber` (data subject and copy owner)
- **Preconditions**: Authenticated `consultant` session (passkey, REQ-AUTH-020); an active engagement between the consultant and the target subscriber (REQ-CONSULT-010, REQ-CONSULT-030)
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED)

## Security Context

- **Security Objectives**: Authorization, Confidentiality, Integrity, Accountability, Privacy
- **Control Layers**: Authorization, Business-Rule Validation, Logging and Monitoring, Data Protection
- **Threat References**: `SECURITY.md` TM-E-3 (consultant capability creep beyond read-plus-copy-edit), TM-I-2 (residual access), TM-I-3 (bulk retrieval), TM-I-1 (BOLA); CWE-863 (incorrect authorization), CWE-269 (improper privilege management), CWE-778 (insufficient logging)
- **Abuse / Misuse Case**: A compromised consultant account tries to write or delete log entries for an engaged subscriber, to reassign a plan copy's ownership to themselves, to bulk-export an engaged subscriber's history beyond the views, or to read or edit data of a subscriber they are not engaged with; or performs permitted views through a path that omits the audit write, leaving the disclosure unrecorded.
- **Trust Boundary**: Boundary 1 — the target subscriber identifier, copy identifier, and edit payload are untrusted input.
- **Untrusted Inputs or Assertions**: Target subscriber and object identifiers; edit payloads against plan copies; any client assertion of engagement or capability.
- **Authoritative Enforcement Point**: REST API Application — the policy module (REQ-AUTHZ-050) maps consultant capabilities from the engaged-consultant relationship: view capabilities over selected plans, plan copies, and progress logs, and `plan_copy.edit`; no log-entry write capability exists for the `consultant` role.
- **Independent Verification**: Engagement state and copy ownership are read from persisted state on every request (SEC-AUTHZ-2, DR-3); the query is constrained to the engaged subscriber's records rather than filtered after retrieval.
- **Zero Trust Relevance**: NIST SP 800-207 — least-privilege per-request access scoped to the current relationship. Exact tenet: TO BE DECIDED (not verified against the publication in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — per-issue mappings are verified during the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — per-issue mappings are verified during the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: Consultant views and edits are third-party processing of health data under the `SECURITY.md` SQ-1 regime set — GDPR/UK GDPR for EU/UK data subjects; CCPA/CPRA, Washington My Health My Data, and the FTC Health Breach Notification Rule for US users; HIPAA not applicable. The FR-11.4/FR-9.7 audit trail is the accountability record. Statute-section precision: TO BE DECIDED (SQ-1 counsel review, SQ-10).
- **Other**: `REF-PROMPT-ABAC`, `REF-ASVS-5` as cited by SEC-AUTHZ-3; `REF-LOG` as cited by SEC-LOG-1.
- **Mapping Basis**: FR-11.6 and FR-11.4 are the normative sources; SEC-AUTHZ-3 and SEC-LOG-1 restate the access and audit obligations; the CWE identifiers name the incorrect-authorization, privilege-management, and insufficient-logging classes.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an active engagement, when the consultant views the engaged subscriber's selected plans, customized plan copies, or progress logs, then the requested data scoped to that subscriber is returned with a least-privilege response shape (SEC-DATA-5), and each request produces exactly one audit entry recording the acting consultant, the action, the affected subscriber, and the time (FR-9.7 granularity).
2. **AC-02 — Expected behavior**: Given an active engagement and a plan copy owned by the engaged subscriber, when the consultant edits that copy, then the edit is persisted with the copy's ownership unchanged (FR-7.2), exactly one audit entry records the edit, and the subscriber's view of the copy shows the "Edited by your consultant" provenance line with the time per `DESIGN.md`.
3. **AC-03 — Prohibited behavior**: Given an active engagement, when the consultant attempts to create, edit, or delete any workout, food, body-weight, or body-measurement log entry of the subscriber, to delete a plan copy, to change a plan selection, or to perform any operation outside the FR-11.6 enumeration, then the operation is denied, no state changes, and the denial is logged (SEC-LOG-4).
4. **AC-04 — Boundary or failure behavior**: Given no engagement or an ended or revoked engagement with a target subscriber, when the consultant requests any view or edit of that subscriber's data, then the request is denied without disclosing whether the objects exist (REQ-CONSULT-010); and an edit payload attempting to set the copy's owner, or any server-controlled field, MUST NOT bind those fields (SEC-INPUT-3).
5. **AC-05 — Additional criterion**: Given a permitted consultant view or edit path, when the audit write fails, then the operation fails with it — no consultant access to subscriber health data completes unaudited (DR-9, REQ-AUDIT-020).

## Failure Behavior

- **On Invalid Input**: Edit payloads are validated against the allow-list schema (REQ-API-010); rejection identifies the failing field without internal detail; no partial write.
- **On Authentication Failure**: Deny per REQ-AUTHZ-010 with the uniform unauthenticated response.
- **On Authorization Failure**: Deny via the policy module without confirming the existence of the target subscriber's objects; the client presents the ended-engagement state per `DESIGN.md` ("No client data remains visible after an engagement ends") from its own engagement-state read, not from leaked detail.
- **On Security-Decision Failure**: Deny by default; a missing or unresolvable engagement or ownership attribute denies (SEC-AUTHZ-7).
- **On External Dependency Failure**: N/A — no external dependency; persistence unavailability fails the request without partial state.
- **On System Error**: Edits roll back atomically with their audit entry; generic error with correlation identifier (SEC-ERR-1).
- **Logging / Audit**: One audit entry per consultant view or edit request: acting consultant account, action, affected subscriber, time (FR-11.4, FR-9.7, SEC-LOG-1). Entries reference the data accessed and never copy health values (SEC-LOG-3). Denials log per SEC-LOG-4.
- **Alerting**: Threshold alerts route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-3, SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: Capability map assertions — consultant capabilities are exactly the FR-11.6 views plus `plan_copy.edit`; no log-entry write capability resolves for the `consultant` role; ownership immutability on the edit path; policy denial when engagement state is anything but active.
- **Integration Tests**: View and edit flows against persistence with an active engagement, asserting scoped data, unchanged ownership, provenance metadata for the `DESIGN.md` line, and exactly one audit entry per request; the same flows after engagement end/revocation asserting denial (with REQ-CONSULT-010/020).
- **Security Tests**: Attempted log-entry create/edit/delete, copy deletion, and selection change as consultant, asserting denial and no state change; BOLA probes against a non-engaged subscriber's object identifiers asserting denial with no information disclosure; mass-assignment probe on copy ownership; audit-failure injection asserting the operation fails closed (AC-05); response-shape assertions for SEC-DATA-5.
- **Compliance Tests / Evidence**: Audit-entry field assertions per view and edit, retained as FR-11.4 evidence.
- **Acceptance-Criteria Traceability**: AC-01 — view integration suite with audit assertions; AC-02 — copy-edit suite with ownership and provenance assertions; AC-03 — prohibited-operation suite; AC-04 — non-engaged BOLA and mass-assignment suites; AC-05 — audit-failure injection test.
- **Coverage Target**: Project coverage threshold is 90% line and branch (`CLAUDE.md`, 2026-08-03); every permitted and prohibited capability MUST have positive and negative tests.
- **Required Test Environment**: Vitest; HTTP test client; fixtures for a consultant with an active engagement, an ended engagement, and no engagement; subscriber fixtures with plan copies, selections, and log entries.

## Dependencies

- **Upstream Requirements**: REQ-CONSULT-010, REQ-CONSULT-030, REQ-CUSTOM-010, REQ-AUTHZ-050, REQ-AUDIT-020, REQ-API-010, REQ-API-020
- **Downstream Requirements**: None — this issue closes the consultant capability surface; client presentation consumes its provenance metadata.
- **External Dependencies**: None
- **Dependency Assumptions**: REQ-CONSULT-010 has already denied any request outside an active engagement; REQ-AUDIT-020 makes the audit write a structural dependency of every health-data path this issue opens.
- **Failure Impact**: A capability beyond the FR-11.6 enumeration is TM-E-3 realized — third-party modification or disclosure of health data outside the agreed scope; a missed audit write makes a regulated disclosure unrecorded.

## Implementation Notes

- **Constraints**: TypeScript on Node.js with Fastify; PostgreSQL via Drizzle (`CLAUDE.md`). Consultant capabilities are expressed only in the REQ-AUTHZ-050 capability map (engaged-consultant relationship), never as per-endpoint role checks. Copy ownership is a server-controlled field (SEC-INPUT-3).
- **Prohibited Approaches**: Granting the consultant a generic "read subscriber" capability instead of the enumerated views; any log-entry write path for the consultant role; transferring or sharing copy ownership; filtering unauthorized rows after retrieval instead of constraining the query (SEC-AUTHZ-2); rendering the provenance line as an endorsement badge; caching engagement or capability state client-side as authority (DR-2).
- **Implementation Guidance**: Reuse the subscriber-facing read models for the consultant views with the engagement predicate substituted for the owner predicate, so the response shapes stay least-privilege and identical in structure (SEC-DATA-5). Record the editing actor and time on the plan copy so the client can render "Edited by your consultant" without a second audit-table read. The consultant workspace context bar content ("View progress and selected plans; edit plan copies") is static scope text from `DESIGN.md`, not derived from policy output.
- **AI Development Guidance**: `REF-PROMPT-ABAC`, `REF-PROMPT-API`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the consultant capability entries in the policy map and of the copy-edit path; privacy review that the audit trail captures every view.
- **Open Decisions**: None — OQ-12 is RESOLVED; a messaging channel would be a new requirements change and is explicitly not included.

**Estimated effort**: 1–2 engineer-days. **Estimated changed lines**: 400–800.
**Recommended model**: Claude Opus (`claude-opus-5`) — an enumerated-capability surface over health data where the failure mode is a quiet extra capability or a missed audit write, both regulated-disclosure defects.
