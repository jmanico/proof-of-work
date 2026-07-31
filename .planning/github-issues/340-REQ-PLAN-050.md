# [REQ-PLAN-050] Publication gate requiring citation and verification record

## Metadata

- **ID**: REQ-PLAN-050
- **Title**: Publication gate requiring citation and verification record
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: Critical
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-4.4, FR-4.5, FR-4.7, FR-10.2; `SECURITY.md` SEC-INPUT-4

## Requirement

- **Statement**: The system MUST block publication of a plan that carries no citation or no admin verification record, MUST record which admin verified the plan and when, MUST make only published plans visible to subscribers, and MUST audit the publication action.
- **Rationale**: FR-4.4 requires at least one peer-reviewed citation before publication and explicitly requires blocking publication without one; FR-4.5 requires explicit admin verification with the verifying admin and time recorded; FR-4.7 restricts subscriber visibility to published plans; SEC-INPUT-4 requires these as server-side validation rather than UI affordance.
- **Assumptions**: Citations exist as records (REQ-PLAN-040) and a verification record can be present on a plan (REQ-PLAN-010, REQ-PLAN-020).
- **Out of Scope**: The verification *workflow* — who may verify relative to who authored, and whether an edit invalidates a prior verification — which `REQUIREMENTS.md` OQ-16 and OQ-10 leave open and which is blocked; unpublication (REQ-PLAN-060); how verification status is displayed (`DESIGN.md` OQ-5).
- **Design Traceability**: `DESIGN.md` — Components → Buttons (one primary action per view, busy state blocking repeat submission), Form feedback and errors; `DESIGN.md` OQ-5 leaves verification-badge presentation open.
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application ("Enforces publication gates (FR-4.4, FR-4.5, FR-4.7)"); data flow 6 ("role check, citation-presence gate, verification record → plan published").
- **Security Traceability**: SEC-INPUT-4, SEC-AUTHZ-4, SEC-INPUT-3, SEC-LOG-6.

## Scope

- **Applies To**: Server-Side Application, API
- **Components**: REST API Application; Relational Persistence
- **Interfaces / Operations**: Publish a plan; the published-only filter applied to subscriber-facing reads
- **Actors**: `admin` as publisher; `subscriber` as the audience the gate protects
- **Preconditions**: An existing plan and an authenticated `admin` session
- **Data Classification**: Internal
- **Personal or Regulated Data**: Personal Data — the verifying and publishing admins' identifiers
- **Jurisdiction / Regulatory Scope**: N/A

## Security Context

- **Security Objectives**: Integrity, Safety, Accountability, Authorization
- **Control Layers**: Business-Rule Validation, Authorization, Logging and Monitoring
- **Threat References**: `SECURITY.md` TM-T-5 (compromised admin publishes harmful exercise or diet content; author and verifier may be the same account), TM-R-1 (unaudited lifecycle actions), TM-T-1 (client-asserted state); CWE-602 (client-side enforcement of server-side security), CWE-862 (missing authorization)
- **Abuse / Misuse Case**: A plan reaches subscribers without evidence or verification — because publication was set through an edit payload, because the client hid rather than enforced the gate, or because the citation was removed after publication.
- **Trust Boundary**: Boundary 1 — the client cannot assert that the gate was satisfied.
- **Untrusted Inputs or Assertions**: Any publication or verification field in a request payload.
- **Authoritative Enforcement Point**: REST API Application — the gate is evaluated against persisted state at publication time.
- **Independent Verification**: The gate reads the plan's persisted citations and verification record rather than trusting the request.
- **Zero Trust Relevance**: N/A — a business-rule gate, not a resource-access decision.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: N/A
- **Regulatory**: N/A — no source document names a statute governing health-content publication; the control is product and threat-model driven.
- **Other**: `REF-PC-2024`, `REF-ASVS-5` as cited by SEC-INPUT-4; `REF-LOG` for the audit obligation.
- **Mapping Basis**: FR-4.4, FR-4.5, FR-4.7, and SEC-INPUT-4 are the normative sources and name these references.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a plan with at least one citation and an admin verification record, when an `admin` publishes it, then the plan becomes published, becomes visible to subscribers, and exactly one audit entry records the publish action, the acting admin, the plan, and the time.
2. **AC-02 — Boundary or failure behavior**: Given a plan with no citation, or with no verification record, or with neither, when publication is attempted, then it is refused, the response states which precondition failed, the plan remains unpublished, and no publish audit entry is written.
3. **AC-03 — Prohibited behavior**: Given any request, when it carries a publication state, a verification record, or a verifying-admin identity in its body, then those values MUST NOT be applied (SEC-INPUT-3); and the gate MUST NOT be satisfiable by any client-side check alone (SEC-INPUT-4).
4. **AC-04 — Additional criterion**: Given a verification record, when it is created, then it captures which admin verified the plan and when (FR-4.5), and that record is retrievable with the plan.
5. **AC-05 — Additional criterion**: Given an unpublished plan, when a `subscriber` requests the plan library or that plan directly, then it is not listed and not retrievable, and its existence is not disclosed (FR-4.7).

## Failure Behavior

- **On Invalid Input**: Reject per REQ-API-010; nothing persisted.
- **On Authentication Failure**: Denied upstream; admins authenticate by passkey (REQ-AUTH-020).
- **On Authorization Failure**: Denied for non-admin roles (REQ-AUTHZ-030); unpublished plan existence is not disclosed to them.
- **On Security-Decision Failure**: If citation presence or verification state cannot be determined, refuse publication (fail closed).
- **On External Dependency Failure**: If persistence or audit storage is unavailable, publication fails atomically.
- **On System Error**: Roll back the state change and its audit entry together.
- **Logging / Audit**: One audit entry per successful publish (FR-10.2, REQ-AUDIT-030). Refusals logged with the failing precondition.
- **Alerting**: TO BE DECIDED — given TM-T-5, a publication by the same admin who authored and verified the plan is a natural alert candidate, but `REQUIREMENTS.md` OQ-16 leaves dual control open and no alerting model exists.

## Test Strategy

- **Unit Tests**: The gate evaluates citation presence and verification presence independently and refuses when either is absent or unresolvable.
- **Integration Tests**: Publish with both preconditions satisfied; attempt publication with each precondition missing; assert subscriber visibility flips only on success; assert the audit entry.
- **Security Tests**: Direct API publication attempts as `subscriber` and `consultant`; mass-assignment of publication and verification fields through create and edit payloads; assertion that an unpublished plan is neither listed nor retrievable by a subscriber and returns an indistinguishable response from a nonexistent plan.
- **Compliance Tests / Evidence**: Verification records and publish audit entries, retained as evidence for FR-4.5 and FR-10.2.
- **Acceptance-Criteria Traceability**: AC-01 — publish success suite; AC-02 — precondition refusal matrix; AC-03 — mass-assignment and direct-API suites; AC-04 — verification record assertions; AC-05 — subscriber visibility tests.
- **Coverage Target**: Every combination of citation-present and verification-present exercised; all three roles against the publish operation.
- **Required Test Environment**: Two admin identities, one subscriber, one consultant; plans seeded in each precondition state. Runs against PostgreSQL on Vitest.

## Dependencies

- **Upstream Requirements**: REQ-PLAN-010, REQ-PLAN-020, REQ-PLAN-030, REQ-PLAN-040, REQ-AUTHZ-030, REQ-AUDIT-030
- **Downstream Requirements**: REQ-PLAN-060, REQ-CATALOG-010, REQ-CATALOG-020, REQ-CUSTOM-010
- **External Dependencies**: None
- **Dependency Assumptions**: The verification record exists as a persisted attribute of the plan; the operation that *creates* it is blocked, so this gate is testable using a seeded record until that operation is unblocked.
- **Failure Impact**: This gate is the only specified control standing between an admin account and health guidance reaching every subscriber, and TM-T-5 records that a single compromised admin can currently satisfy it alone.

## Implementation Notes

- **Constraints**: PostgreSQL with Drizzle ORM and Fastify (`CLAUDE.md`). The gate checks that a verification record is *present*; whether the verifier must differ from the author (`REQUIREMENTS.md` OQ-16) and whether an edit invalidates verification (OQ-10) are open and MUST NOT be decided here. This makes the issue's coverage of FR-4.5 partial: the record and the gate are delivered, the workflow is not.
- **Prohibited Approaches**: A combined save-and-publish path; enforcing the gate only in the admin UI; allowing publication state to be set through create or edit; treating a citation count cached on the plan row as authoritative without checking the citation records themselves.
- **Implementation Guidance**: Because REQ-PLAN-040 refuses removal of the last citation from a published plan, and this gate refuses publication without one, the invariant "published implies at least one citation" holds from both directions — assert it as a test in both issues.
- **AI Development Guidance**: `REF-PC-2024`, `REF-PROMPT-API`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the gate; product and safety review of the publication process while OQ-16 is open.
- **Open Decisions**: `REQUIREMENTS.md` OQ-16 (dual-control verification) and OQ-10 (re-verification after edit). Both are blocked; the verification-creation operation has no issue until they resolve.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 250–500.
**Recommended model**: Claude Opus (`claude-opus-5`) — the highest safety-impact business gate in the specification, with an adjacent open question that must be preserved rather than resolved.
