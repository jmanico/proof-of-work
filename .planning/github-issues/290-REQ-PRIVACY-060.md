# [REQ-PRIVACY-060] Response field minimization

## Metadata

- **ID**: REQ-PRIVACY-060
- **Title**: Response field minimization
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: High
- **Requirement Type**: Privacy
- **Source / Parent**: REQ-EPIC-001; `SECURITY.md` SEC-DATA-5; `REQUIREMENTS.md` FR-9.1; threat TM-I-3

## Requirement

- **Statement**: Every response MUST return only the fields the operation requires, and bulk retrieval of other subscribers' health data MUST NOT be possible through any role.
- **Rationale**: SEC-DATA-5 states the rule as least-privilege collection and return; the threat model rates excessive data exposure and bulk retrieval (TM-I-3) as high severity, and notes that admin capabilities over subscriber health data are an assumption `REQUIREMENTS.md` does not fully define.
- **Assumptions**: Owner scoping is enforced per operation (REQ-AUTHZ-020) and engagement scoping for consultants (REQ-CONSULT-010). This issue constrains what a *permitted* response may contain and how much of it may be retrieved at once.
- **Out of Scope**: The authorization decisions themselves; data export (FR-9.3), which is a deliberate bulk operation scoped to the requesting actor and is blocked by `SECURITY.md` SQ-5; pagination limits as an availability control (SEC-HTTP-5, blocked by SQ-3) — this issue bounds disclosure, not load.
- **Design Traceability**: `DESIGN.md` — Components → Empty states and the progress views, insofar as the client is built against the minimized shapes rather than expecting whole records.
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application outputs; DR-2 (responses carry structured detail sufficient for presentation, which bounds what is needed); data flow 5.
- **Security Traceability**: SEC-DATA-5; supports SEC-AUTHZ-2, SEC-DATA-3, SEC-LOG-3, SEC-ERR-1.

## Scope

- **Applies To**: API, Server-Side Application
- **Components**: REST API Application
- **Interfaces / Operations**: Every read operation, including list and detail responses for plans, plan copies, and all log entry types
- **Actors**: `subscriber`, `consultant`, `admin`
- **Preconditions**: The operation is already authorized
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED (`SECURITY.md` SQ-1, `REQUIREMENTS.md` OQ-3)

## Security Context

- **Security Objectives**: Confidentiality, Privacy
- **Control Layers**: Data Protection, Authorization, Output Encoding
- **Threat References**: `SECURITY.md` TM-I-3 (excessive data exposure and bulk retrieval), TM-I-1 (BOLA), TM-P-1 (identifiability); CWE-213 (exposure of sensitive information due to incompatible policies), CWE-200 (exposure of sensitive information to an unauthorized actor)
- **Abuse / Misuse Case**: An endpoint returns the full persisted record — including fields the view never shows — so that one authorized read discloses more than the operation needed; or a list endpoint accepts an unbounded page size and becomes a bulk-extraction tool for whichever subject scope the actor holds.
- **Trust Boundary**: Boundary 1 — the response crosses to an untrusted client that may be inspected, logged, or cached.
- **Untrusted Inputs or Assertions**: Field-selection, filter, and page-size parameters supplied by the client.
- **Authoritative Enforcement Point**: The response shaping layer of the REST API Application, applied per operation.
- **Independent Verification**: Response-shape assertions per endpoint, independent of handler code.
- **Zero Trust Relevance**: NIST SP 800-207 — least-privilege access granted per request. Exact tenet: TO BE DECIDED (not verified against the publication in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: TO BE DECIDED — data minimization is a principle in the regimes `REQUIREMENTS.md` names, but the governing set is blocked by `SECURITY.md` SQ-1.
- **Other**: `REF-API-2023`, `REF-PROMPT-API` as cited by SEC-DATA-5.
- **Mapping Basis**: SEC-DATA-5 names these references; the CWE identifiers name the over-exposure classes.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given any read operation, when it responds, then the body contains only the fields declared for that operation, and every field is one the operation's purpose requires.
2. **AC-02 — Boundary or failure behavior**: Given a list operation, when it is requested without a page size or with an excessive one, then a bounded default applies and the response cannot return an unbounded number of records.
3. **AC-03 — Prohibited behavior**: Given any response, when it is inspected, then it MUST NOT contain credential material, session material, internal identifiers beyond those the client needs, audit fields, or health data belonging to a subject outside the actor's authorized scope; and no role, including `admin`, may retrieve other subscribers' health records in bulk.
4. **AC-04 — Additional criterion**: Given a client-supplied field-selection or filter parameter, when it is processed, then it may only narrow the declared field set, never widen it.
5. **AC-05 — Additional criterion**: Given a response containing health or personal data, when it is produced, then it carries `Cache-Control: no-store` (REQ-PLATFORM-040).

## Failure Behavior

- **On Invalid Input**: An unrecognized field-selection or filter parameter is rejected per REQ-API-010 rather than ignored, so a typo cannot silently widen a response.
- **On Authentication Failure**: Denied upstream; no body.
- **On Authorization Failure**: Denied per REQ-AUTHZ-020 and REQ-AUTHZ-040; the denial body discloses nothing about the target.
- **On Security-Decision Failure**: If the declared field set for an operation cannot be resolved, refuse the response rather than serialize the whole record (fail closed).
- **On External Dependency Failure**: Generic error with a correlation identifier; no partial record dump.
- **On System Error**: Generic error; diagnostics stay server-side (SEC-ERR-1).
- **Logging / Audit**: Health-data reads produce audit entries via REQ-AUDIT-020. Responses are never logged in full (SEC-LOG-3, REQ-AUDIT-040).
- **Alerting**: TO BE DECIDED — a volume threshold on health-record reads would be a natural detection for TM-I-3, but thresholds are blocked by `SECURITY.md` SQ-3.

## Test Strategy

- **Unit Tests**: The serializer for each operation emits exactly the declared fields for a fully populated entity; a field-selection parameter can narrow but not widen.
- **Integration Tests**: Response-shape assertions per endpoint; list endpoints assert the bounded default and a maximum.
- **Security Tests**: Excessive-exposure sweep comparing each response against the persisted entity to confirm withheld fields; bulk-retrieval attempts as `subscriber`, `consultant`, and `admin` asserting no cross-subject health records are returned; assertion that no credential, session, or audit field appears in any response.
- **Compliance Tests / Evidence**: The per-endpoint response-shape report, retained as minimization evidence.
- **Acceptance-Criteria Traceability**: AC-01 — shape assertions; AC-02 — pagination bound tests; AC-03 — exposure sweep and bulk-retrieval matrix; AC-04 — field-selection narrowing test; AC-05 — cache header assertion.
- **Coverage Target**: Every read operation covered by a response-shape assertion.
- **Required Test Environment**: Fully populated entities for every type, identities for all three roles, and a second subscriber. Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-AUTHZ-020, REQ-API-010, REQ-PLATFORM-040
- **Downstream Requirements**: REQ-CATALOG-010, REQ-CATALOG-020, REQ-CUSTOM-020, REQ-PROGRESS-010, REQ-PROGRESS-020, REQ-CONSULT-010
- **External Dependencies**: None
- **Dependency Assumptions**: None
- **Failure Impact**: Over-broad responses defeat minimization even where authorization is correct, and they widen the blast radius of every other disclosure path — logs, caches, and token capture.

## Implementation Notes

- **Constraints**: PostgreSQL with Drizzle ORM; Fastify response schemas govern serialization, so a field absent from the schema cannot be emitted. `SECURITY.md` records an explicit assumption that admin capabilities are limited to plan and account administration and that `REQUIREMENTS.md` does not fully define admin access to health data; this issue enforces the bulk-retrieval prohibition without resolving that gap.
- **Prohibited Approaches**: Serializing the persisted entity directly; a generic serializer that includes every column by default; a client-controlled field parameter that can request additional fields; unbounded list endpoints.
- **Implementation Guidance**: Declare the response shape next to the operation's input schema (REQ-API-010) and binding allow-list (REQ-API-020), so a new persisted field is not automatically exposed in three directions at once.
- **AI Development Guidance**: `REF-API-2023`, `REF-PROMPT-API`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Privacy review of each declared response shape; security review of the bulk-retrieval matrix.
- **Open Decisions**: What health-data access, if any, an `admin` legitimately has is not defined by `REQUIREMENTS.md` (recorded as an assumption in the `SECURITY.md` threat model). Consultant capabilities are open (`REQUIREMENTS.md` OQ-12). Both bound what these response shapes should contain for those roles and must be revisited when resolved.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 250–550.
**Recommended model**: Claude Opus (`claude-opus-5`) — a per-endpoint discipline where the default (serialize everything) is the vulnerability.
