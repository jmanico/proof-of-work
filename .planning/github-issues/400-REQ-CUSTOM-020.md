# [REQ-CUSTOM-020] Persist, list, and retrieve a subscriber's customized plans

## Metadata

- **ID**: REQ-CUSTOM-020
- **Title**: Persist, list, and retrieve a subscriber's customized plans
- **Version**: 1.1.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-7.3, FR-7.4, FR-9.1

## Requirement

- **Statement**: The system MUST persist a subscriber's customized plans across sessions and make them retrievable by that subscriber, and MUST NOT allow a subscriber to view or modify another subscriber's customized plans.
- **Rationale**: FR-7.3 requires persistence across sessions and retrievability by the owner; FR-7.4 forbids cross-subscriber access; FR-9.1 scopes every plan copy to its owning subscriber.
- **Assumptions**: Copies are created as independent owned records (REQ-CUSTOM-010) and ownership scoping is enforced centrally (REQ-AUTHZ-020).
- **Out of Scope**: Creating and editing copies (REQ-CUSTOM-010); stability when the source plan changes (REQ-CUSTOM-030); consultant access to an engaged subscriber's copies (REQ-CONSULT-010 for engagement scoping; capabilities fixed by FR-11.6, REQ-CONSULT-040 — `REQUIREMENTS.md` OQ-12 RESOLVED); retention across subscription lapse (FR-3.4, REQ-ENTITLE-040); export (FR-9.3, REQ-PRIVACY-080).
- **Design Traceability**: `DESIGN.md` — Core Components → Status, feedback, and loading ("Empty states say what belongs in the region and provide the action that creates it"); Core Components → Cards, lists, and tables; Layout, Spacing, and Responsive Behavior; Typography and Iconography (tabular figures for numeric plan values); Accessibility.
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application ("Owns … user plan copy"); Relational Persistence ("Must retain progress records and plan customizations across subscription lapse (FR-3.4)"); DR-4.
- **Security Traceability**: SEC-AUTHZ-1, SEC-AUTHZ-2, SEC-DATA-5, SEC-LOG-1, SEC-RENDER-1.

## Scope

- **Applies To**: Server-Side Application, API, Web Client
- **Components**: REST API Application; Relational Persistence; Browser Client (my-plans views)
- **Interfaces / Operations**: List own customized plans; retrieve one owned customized plan; delete an owned customized plan
- **Actors**: `subscriber`
- **Preconditions**: Authenticated session
- **Data Classification**: Restricted — customized plan copies are health data (FR-9.12)
- **Personal or Regulated Data**: Health Data — a customized plan reveals what a subscriber is doing about their health (FR-9.12)
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, Washington My Health My Data, and the FTC Health Breach Notification Rule for US users; HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED, `REQUIREMENTS.md` OQ-3 RESOLVED)

## Security Context

- **Security Objectives**: Confidentiality, Privacy, Authorization, Availability
- **Control Layers**: Authorization, Data Protection, Logging and Monitoring
- **Threat References**: `SECURITY.md` TM-I-1 (BOLA), TM-I-3 (excessive data exposure and bulk retrieval), TM-P-1 (identifiability); CWE-639 (authorization bypass through user-controlled key), CWE-200
- **Abuse / Misuse Case**: Subscriber A enumerates copy identifiers and reads subscriber B's customized plans, learning what health regime B follows; or a listing endpoint returns copies across subscribers.
- **Trust Boundary**: Boundary 1 — copy identifiers are attacker-controlled input.
- **Untrusted Inputs or Assertions**: Copy identifiers; listing filter and pagination parameters; any owner identifier in the request.
- **Authoritative Enforcement Point**: REST API Application — the owner predicate is applied in the query (SEC-AUTHZ-2).
- **Independent Verification**: Ownership comes from persisted state compared against the session identity, never from the request (DR-3).
- **Zero Trust Relevance**: NIST SP 800-207 — per-request, per-resource authorization. Exact tenet: TO BE DECIDED (not verified in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — verified during the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — verified during the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: GDPR/UK GDPR (EU/UK data subjects); CCPA/CPRA, Washington My Health My Data, FTC Health Breach Notification Rule (US users); HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED). Statute-section precision: TO BE DECIDED — no source document states sections for this requirement.
- **Other**: `REF-API-2023`, `REF-PROMPT-API` as cited by SEC-AUTHZ-2 and SEC-DATA-5.
- **Mapping Basis**: FR-7.3, FR-7.4, and FR-9.1 are the normative sources; the references are those `SECURITY.md` cites for object-level authorization and response minimization.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a subscriber who created customized plans in an earlier session, when they authenticate again and list their plans, then all of their copies are returned with their content intact and unchanged.
2. **AC-02 — Boundary or failure behavior**: Given subscriber A authenticated, when A requests, modifies, or deletes a copy owned by subscriber B, then the operation is denied, nothing belonging to B is returned or changed, and the response does not disclose whether B's copy exists (FR-7.4).
3. **AC-03 — Prohibited behavior**: Given the listing operation, when it executes, then it MUST NOT retrieve copies outside the owner scope and filter afterwards, MUST NOT accept an owner identifier from the request, and MUST NOT return an unbounded number of records.
4. **AC-04 — Additional criterion**: Given a subscriber with no customized plans, when the view renders, then an empty state states what will appear there and names the action that creates one (`DESIGN.md`).
5. **AC-05 — Additional criterion**: Given any read of a customized plan — including the owner's own reads — when it completes, then an audit entry records the acting account, the action, the affected subject, and the time, at one entry per request (FR-9.7, REQ-AUDIT-020).

## Failure Behavior

- **On Invalid Input**: Reject malformed identifiers or listing parameters per REQ-API-010.
- **On Authentication Failure**: Denied upstream by REQ-AUTHZ-010.
- **On Authorization Failure**: Deny with a response indistinguishable from "does not exist" (REQ-AUTHZ-040).
- **On Security-Decision Failure**: If ownership cannot be resolved, deny.
- **On External Dependency Failure**: If persistence or audit storage is unavailable, the read fails closed with a generic error rather than returning unaudited data (REQ-AUDIT-020).
- **On System Error**: Generic error with a correlation identifier; no partial record dump.
- **Logging / Audit**: Audit entry per AC-05; denials logged as security events. Copy content is never written into logs or entries (SEC-LOG-3).
- **Alerting**: Repeated cross-owner denials are SEC-LOG-4 authorization-denial events; threshold alerts route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-3, SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: The listing query carries the owner predicate; the serializer emits only declared fields; the retrieval path denies on an ownership mismatch.
- **Integration Tests**: Create copies, end the session, re-authenticate, and assert full retrieval with content intact; delete an owned copy and assert it is gone while others remain.
- **Security Tests**: Cross-subscriber read, update, and delete attempts; response-equivalence between "another subscriber's copy" and "nonexistent copy"; listing parameter manipulation attempting to widen scope; assertion that the emitted query contains the owner predicate.
- **Compliance Tests / Evidence**: Audit entries for copy reads, retained per FR-9.7.
- **Acceptance-Criteria Traceability**: AC-01 — cross-session persistence suite; AC-02 — cross-subscriber matrix; AC-03 — query-shape and pagination tests; AC-04 — empty state test; AC-05 — audit assertion.
- **Coverage Target**: Every copy operation covered by a positive and a cross-owner negative test.
- **Required Test Environment**: Two subscriber identities each holding copies; audit capture. Runs against PostgreSQL on Vitest.

## Dependencies

- **Upstream Requirements**: REQ-CUSTOM-010, REQ-AUTHZ-020, REQ-AUDIT-020, REQ-PRIVACY-060, REQ-CATALOG-030, REQ-PLATFORM-030
- **Downstream Requirements**: REQ-CUSTOM-030, REQ-PROGRESS-020, REQ-CONSULT-010, REQ-CONSULT-040, REQ-SELECT-010, REQ-SELECT-020, REQ-ENTITLE-040, REQ-PRIVACY-080
- **External Dependencies**: None
- **Dependency Assumptions**: Copies are independent records, so retrieval does not depend on the source plan's current state or existence.
- **Failure Impact**: A cross-subscriber leak here discloses health-regime data directly, and the same defect would apply to every other owned record type.

## Implementation Notes

- **Constraints**: PostgreSQL with Drizzle ORM (`CLAUDE.md`); the client is a Vite-built single-page application with `vue-router`. FR-3.4 requires copies to survive a subscription lapse; the lapse semantics are resolved (`REQUIREMENTS.md` OQ-1 — FR-3.5, FR-3.6) and retention across lapse is REQ-ENTITLE-040. This issue guarantees persistence across sessions; REQ-ENTITLE-040 guarantees it across lapse.
- **Prohibited Approaches**: Retrieve-then-filter; treating an unguessable identifier as the access control; unbounded listings; returning the source plan's current content in place of the copy's stored content.
- **Implementation Guidance**: Reuse the scope resolver from REQ-AUTHZ-020 rather than writing a copy-specific ownership check, so a single review covers all owned-record types.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-VUE`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the ownership scoping; privacy review of the response shape.
- **Open Decisions**: None — `REQUIREMENTS.md` OQ-12 is RESOLVED (FR-11.6): an engaged consultant may view these copies and edit them, audited, which REQ-CONSULT-040 implements over REQ-CONSULT-010's engagement scoping. OQ-1 is RESOLVED; retention across lapse is REQ-ENTITLE-040.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 250–550.
**Recommended model**: Claude Opus (`claude-opus-5`) — a straightforward CRUD surface whose entire risk is object-level authorization.
