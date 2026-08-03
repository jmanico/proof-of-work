# [REQ-AUTHZ-020] Object-level ownership scoping

## Metadata

- **ID**: REQ-AUTHZ-020
- **Title**: Object-level ownership scoping
- **Version**: 1.1.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-7.4, FR-9.1; `SECURITY.md` SEC-AUTHZ-2; threat TM-I-1

## Requirement

- **Statement**: Every operation that addresses a subscriber-owned record by identifier MUST verify that the authenticated actor is authorized for that specific record before performing the operation, and every query returning such records MUST be constrained by the authorized owner scope in the query rather than filtered after retrieval.
- **Rationale**: FR-9.1 requires every plan copy and log entry to be scoped to its owning subscriber and never exposed to another; FR-7.4 forbids viewing or modifying another subscriber's customized plans. SEC-AUTHZ-2 states the in-query constraint. The threat model rates BOLA/IDOR (TM-I-1) as high severity.
- **Assumptions**: The actor's identity comes from the verified session (REQ-AUTHZ-010).
- **Out of Scope**: Consultant engagement scope (REQ-CONSULT-010); admin role restrictions (REQ-AUTHZ-030); subscription entitlement (FR-3.1, FR-3.2; REQ-ENTITLE-010 — `REQUIREMENTS.md` OQ-1 RESOLVED); the central typed authorization policy module (SEC-AUTHZ-5–SEC-AUTHZ-7; REQ-AUTHZ-050 — `SECURITY.md` SQ-4 RESOLVED).
- **Design Traceability**: N/A
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application ("enforces … owner scoping (FR-7.4, FR-9.1) … before any data access"); DR-3 ("ownership comes from persisted state"); DR-4.
- **Security Traceability**: SEC-AUTHZ-1, SEC-AUTHZ-2, SEC-DATA-3, SEC-DATA-5.

## Scope

- **Applies To**: API, Server-Side Application
- **Components**: REST API Application; Relational Persistence
- **Interfaces / Operations**: All read, update, and delete operations on user plan copies, workout log entries, food log entries, body weight entries, body measurement entries, consent records, and data export
- **Actors**: `subscriber`, `consultant`, `admin`
- **Preconditions**: Authenticated session (REQ-AUTHZ-010)
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED)

## Security Context

- **Security Objectives**: Confidentiality, Integrity, Authorization, Privacy
- **Control Layers**: Authorization, Business-Rule Validation
- **Threat References**: `SECURITY.md` TM-I-1 (BOLA/IDOR), TM-I-3 (excessive data exposure); CWE-639 (authorization bypass through user-controlled key), CWE-566 (authorization bypass through user-controlled SQL primary key)
- **Abuse / Misuse Case**: An authenticated subscriber substitutes another subscriber's record identifier into a read, update, delete, or export request and receives or mutates that subscriber's health data.
- **Trust Boundary**: Boundary 1 — record identifiers are attacker-controlled input.
- **Untrusted Inputs or Assertions**: Every record identifier in a path, query, or body; any owner identifier in a payload.
- **Authoritative Enforcement Point**: REST API Application, before data access, using persisted ownership.
- **Independent Verification**: Ownership is read from persisted state and compared to the session identity; a client-supplied owner assertion is never used (DR-3).
- **Zero Trust Relevance**: NIST SP 800-207 — per-request, per-resource access decisions. Exact tenet: TO BE DECIDED (not verified in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: The SQ-1 regime set applies to the health data this control scopes — GDPR/UK GDPR, CCPA/CPRA, Washington My Health My Data, FTC HBNR; HIPAA not applicable (`SECURITY.md` SQ-1 and `REQUIREMENTS.md` OQ-3 RESOLVED). Statute-section mappings: TO BE DECIDED (SQ-1 counsel review).
- **Other**: `REF-API-2023`, `REF-PROMPT-API`, `REF-PROMPT-ABAC` as cited by SEC-AUTHZ-1 and SEC-AUTHZ-2.
- **Mapping Basis**: The cited rules name these references; the CWE identifiers name the object-level authorization failure class.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given subscriber A authenticated, when A requests, updates, or deletes a record A owns, then the operation succeeds and returns only A's data.
2. **AC-02 — Boundary or failure behavior**: Given subscriber A authenticated, when A submits subscriber B's record identifier to any read, update, delete, or export operation, then the operation is denied, no data belonging to B is returned or modified, and the response does not disclose whether B's record exists.
3. **AC-03 — Prohibited behavior**: Given any listing or query operation, when it executes, then it MUST NOT retrieve records outside the authorized scope and then filter them in application code, and it MUST NOT accept an owner identifier from the request as the scope.
4. **AC-04 — Additional criterion**: Given an operation on a record that does not exist at all, when it is requested, then the response is indistinguishable from the response for a record owned by another subscriber.

## Failure Behavior

- **On Invalid Input**: A malformed identifier is rejected by REQ-API-010 before authorization is evaluated.
- **On Authentication Failure**: Denied upstream by REQ-AUTHZ-010.
- **On Authorization Failure**: Deny the operation with no state change; resource existence MUST NOT be disclosed (AC-04). The response class is uniform across "not found" and "not yours".
- **On Security-Decision Failure**: If ownership cannot be determined, deny (SEC-AUTHZ-7).
- **On External Dependency Failure**: If persistence is unavailable, deny with a generic error; MUST NOT proceed on an assumption of ownership.
- **On System Error**: Roll back; generic error with a correlation identifier.
- **Logging / Audit**: Log every ownership denial with actor, operation, and target class — never the target's health values (SEC-LOG-3). A successful access to health data additionally emits an audit entry via REQ-AUDIT-020 (FR-9.7).
- **Alerting**: Threshold alerts route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-3, SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: Scope resolver returns the session subject for a subscriber; denies when the persisted owner differs; denies when ownership is unresolvable.
- **Integration Tests**: For every owned record type, a two-subscriber fixture exercising A-on-A (allow) and A-on-B (deny) for read, update, delete, and list.
- **Security Tests**: BOLA/IDOR suite over every identifier-addressed operation; a test asserting the emitted query carries the owner predicate; a response-equivalence test for AC-04.
- **Compliance Tests / Evidence**: Evidence that no cross-subject access is possible, retained for FR-9.1.
- **Acceptance-Criteria Traceability**: AC-01 and AC-02 — two-subscriber suite; AC-03 — query-shape assertion and a static check for post-retrieval filtering; AC-04 — response-equivalence test.
- **Coverage Target**: Every identifier-addressed operation covered by a positive and a cross-subject negative test.
- **Required Test Environment**: At least two subscriber identities, one consultant, one admin, and seeded owned records; Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-AUTHZ-010, REQ-API-030
- **Downstream Requirements**: REQ-CUSTOM-010, REQ-CUSTOM-020, REQ-PROGRESS-010, REQ-PROGRESS-020, REQ-PRIVACY-030, REQ-CONSULT-010
- **External Dependencies**: None
- **Dependency Assumptions**: Persistence enforces the ownership relationships in schema (`ARCHITECTURE.md`, Relational Persistence).
- **Failure Impact**: A single unscoped operation exposes every subscriber's health data to every other subscriber.

## Implementation Notes

- **Constraints**: PostgreSQL accessed through Drizzle ORM. Scoping MUST be expressed in the query (SEC-AUTHZ-2).
- **Prohibited Approaches**: Retrieve-then-check; trusting an owner identifier from the request; relying on unguessable identifiers as the access control; distinguishing "not found" from "forbidden" in the response.
- **Implementation Guidance**: A single scope-resolution function used by every owned-record operation makes AC-03 mechanically reviewable and keeps the rule out of individual handlers (`SECURITY.md` code-quality resolution).
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-ABAC`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of every owned-record operation and of the scope resolver.
- **Open Decisions**: None. Consultant capabilities are fixed (`REQUIREMENTS.md` OQ-12 RESOLVED; FR-11.6 — views plus plan-copy edits only, audited); this issue covers subscriber-owner scope only and defers engagement scope to REQ-CONSULT-010 and REQ-CONSULT-040.

**Estimated effort**: 1–2 engineer-days. **Estimated changed lines**: 300–700.
**Recommended model**: Claude Opus (`claude-opus-5`) — the highest-severity authorization control in the threat model; correctness and exhaustive negative coverage dominate.
