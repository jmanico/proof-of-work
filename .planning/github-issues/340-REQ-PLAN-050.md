# [REQ-PLAN-050] Publication gate requiring citation and verification record

## Metadata

- **ID**: REQ-PLAN-050
- **Title**: Publication gate requiring citation and verification record
- **Version**: 1.1.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-4.4, FR-4.5, FR-4.7, FR-4.8, FR-10.2; `SECURITY.md` SEC-INPUT-4

## Requirement

- **Statement**: The system MUST block publication of a plan that carries no citation or no admin verification record, MUST record which admin verified the plan and when, MUST make only published plans visible to subscribers, and MUST audit the publication action.
- **Rationale**: FR-4.4 requires at least one peer-reviewed citation before publication and explicitly requires blocking publication without one; FR-4.5 requires explicit admin verification with the verifying admin and time recorded; FR-4.7 restricts subscriber visibility to published plans; SEC-INPUT-4 requires these as server-side validation rather than UI affordance. FR-4.8 fixes verification as a one-time gate before first publication — performable by any `admin`, including the plan's author — so this gate checks record presence and never re-requires verification after an edit.
- **Assumptions**: Citations exist as records (REQ-PLAN-040) and a verification record can be present on a plan (REQ-PLAN-010, REQ-PLAN-020).
- **Out of Scope**: The verification *operation* that creates the verification record — a one-time gate under FR-4.8 (`REQUIREMENTS.md` OQ-10 and OQ-16 RESOLVED), planned as REQ-PLAN-070; unpublication (REQ-PLAN-060) and its selection-ending consequence (FR-4.9, REQ-SELECT-030); how verification status is displayed — resolved by `DESIGN.md` OQ-5 (Product Patterns → Plans, citations, and review state).
- **Design Traceability**: `DESIGN.md` — Core Components → Actions (one primary action per region; busy state prevents repeat submission; specific verbs such as "Publish plan"); Forms and validation; Product Patterns → Plans, citations, and review state ("Draft admin views show a publication checklist with separate Citation present and Review recorded gates"; admin views show the verifier identity and timestamp required by FR-4.5); `DESIGN.md` OQ-5 RESOLVED (2026-08-01).
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
- **Alerting**: Publish audit entries (SEC-LOG-6) and refusals are SEC-LOG-4/SEC-OPS-2 detection inputs; threshold alerts route to the security lead (`SECURITY.md` SQ-11 RESOLVED). Same-admin author-verify-publish is not alerted by rule: `REQUIREMENTS.md` OQ-16 resolved dual control as consciously declined, with post-hoc audit review as the compensating control.

## Test Strategy

- **Unit Tests**: The gate evaluates citation presence and verification presence independently and refuses when either is absent or unresolvable.
- **Integration Tests**: Publish with both preconditions satisfied; attempt publication with each precondition missing; assert subscriber visibility flips only on success; assert the audit entry.
- **Security Tests**: Direct API publication attempts as `subscriber` and `consultant`; mass-assignment of publication and verification fields through create and edit payloads; assertion that an unpublished plan is neither listed nor retrievable by a subscriber and returns an indistinguishable response from a nonexistent plan.
- **Compliance Tests / Evidence**: Verification records and publish audit entries, retained as evidence for FR-4.5 and FR-10.2.
- **Acceptance-Criteria Traceability**: AC-01 — publish success suite; AC-02 — precondition refusal matrix; AC-03 — mass-assignment and direct-API suites; AC-04 — verification record assertions; AC-05 — subscriber visibility tests.
- **Coverage Target**: Every combination of citation-present and verification-present exercised; all three roles against the publish operation.
- **Required Test Environment**: Two admin identities, one subscriber, one consultant; plans seeded in each precondition state. Runs against PostgreSQL on Vitest.

## Dependencies

- **Upstream Requirements**: REQ-PLAN-010, REQ-PLAN-020, REQ-PLAN-030, REQ-PLAN-040, REQ-PLAN-070, REQ-AUTHZ-030, REQ-AUDIT-030
- **Downstream Requirements**: REQ-PLAN-060, REQ-CATALOG-010, REQ-CATALOG-020, REQ-CUSTOM-010
- **External Dependencies**: None
- **Dependency Assumptions**: The verification record exists as a persisted attribute of the plan; the operation that *creates* it is REQ-PLAN-070 (FR-4.8), so this gate is testable using a seeded record until that issue lands.
- **Failure Impact**: This gate is the only specified control standing between an admin account and health guidance reaching every subscriber. TM-T-5 — a single compromised admin can author, verify, and publish alone — is RISK ACCEPTED (OQ-16, 2026-08-01), with FR-10.2/SEC-LOG-6 audit, passkey-only admin authentication (SEC-AUTHN-2), and the two-admin minimum (FR-2.15) as the compensating controls.

## Implementation Notes

- **Constraints**: PostgreSQL with Drizzle ORM and Fastify (`CLAUDE.md`). The gate checks that a verification record is *present*. The workflow is fixed by FR-4.8 (`REQUIREMENTS.md` OQ-10 and OQ-16 RESOLVED): verification is required once, before first publication, MAY be performed by any `admin` including the plan's author, and an edit to a published plan never clears or re-requires it — so this gate MUST NOT invalidate a verification record on edit. The verification operation itself is delivered by REQ-PLAN-070.
- **Prohibited Approaches**: A combined save-and-publish path; enforcing the gate only in the admin UI; allowing publication state to be set through create or edit; treating a citation count cached on the plan row as authoritative without checking the citation records themselves.
- **Implementation Guidance**: Because REQ-PLAN-040 refuses removal of the last citation from a published plan, and this gate refuses publication without one, the invariant "published implies at least one citation" holds from both directions — assert it as a test in both issues.
- **AI Development Guidance**: `REF-PC-2024`, `REF-PROMPT-API`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the gate; product and safety review that the compensating controls for the accepted TM-T-5 risk (audit, passkey-only admins, two-admin minimum) are in place around it.
- **Open Decisions**: None — `REQUIREMENTS.md` OQ-16 (dual control declined, risk accepted) and OQ-10 (verification never re-triggers on edit) are resolved by FR-4.8, and the verification-creation operation is planned as REQ-PLAN-070.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 250–500.
**Recommended model**: Claude Opus (`claude-opus-5`) — the highest safety-impact business gate in the specification, sitting on a consciously accepted single-admin risk whose compensating controls must hold exactly.
