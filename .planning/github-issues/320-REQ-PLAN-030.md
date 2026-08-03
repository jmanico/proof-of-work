# [REQ-PLAN-030] Admin plan creation and editing

## Metadata

- **ID**: REQ-PLAN-030
- **Title**: Admin plan creation and editing
- **Version**: 1.1.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-4.3, FR-4.2, FR-4.8, FR-10.1, FR-10.2

## Requirement

- **Statement**: An `admin` MUST be able to create and edit exercise and diet plans, each such action MUST produce an audit entry, and no other role may perform them.
- **Rationale**: FR-4.3 grants admins create, edit, publish, and unpublish; FR-10.1 restricts those actions to admins; FR-4.2 forbids subscriber authoring; FR-10.2 requires an audit entry for each. Creation and editing are separated from publication because publication carries its own gate (REQ-PLAN-050). FR-4.8 fixes the edit semantics against verification and citations: an edit never clears or re-requires verification, and an edit that would leave a published plan without at least one citation is rejected.
- **Assumptions**: The plan content models exist (REQ-PLAN-010, REQ-PLAN-020), the exercise catalog exists (REQ-PLAN-080 — exercise plan payloads compose exercises by catalog reference, FR-5.4), and the role gate exists (REQ-AUTHZ-030).
- **Out of Scope**: Publication and unpublication (REQ-PLAN-050, REQ-PLAN-060); the verification operation (REQ-PLAN-070 — FR-4.8's one-time gate, `REQUIREMENTS.md` OQ-10 and OQ-16 RESOLVED); citation management, which is its own operation with its own URL rules (REQ-PLAN-040); catalog management (REQ-PLAN-080); the admin workspace's visual treatment, defined by `DESIGN.md` (OQ-7 RESOLVED: labelled "Admin workspace" with Plans among its destinations) and delivered with the client views.
- **Design Traceability**: `DESIGN.md` — Core Components → Forms and validation (persistent visible labels, unit suffixes adjacent to numeric fields, required fields marked with text), Actions (one primary action per region, busy state preventing repeat submission, specific verbs), Forms and validation (inline errors with icon and specific correction, summary linking to each invalid field, focus to first invalid field); Information Architecture (Admin workspace, Plans destination); Product Patterns → Plans, citations, and review state (draft admin views show a publication checklist with separate Citation present and Review recorded gates); Accessibility.
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application; data flow 6; DR-4 ("Published plans are mutable only by admin-role operations").
- **Security Traceability**: SEC-AUTHZ-4, SEC-INPUT-1, SEC-INPUT-3, SEC-INPUT-4, SEC-LOG-6, SEC-RENDER-1.

## Scope

- **Applies To**: Server-Side Application, API, Web Client
- **Components**: REST API Application; Relational Persistence; Browser Client (admin authoring view)
- **Interfaces / Operations**: Create exercise plan; create diet plan; edit exercise plan; edit diet plan
- **Actors**: `admin`
- **Preconditions**: Authenticated `admin` session established by passkey (REQ-AUTH-020)
- **Data Classification**: Internal
- **Personal or Regulated Data**: Personal Data — the acting admin's identifier in the audit entry
- **Jurisdiction / Regulatory Scope**: N/A

## Security Context

- **Security Objectives**: Integrity, Authorization, Accountability, Safety
- **Control Layers**: Authorization, Input Validation, Business-Rule Validation, Logging and Monitoring
- **Threat References**: `SECURITY.md` TM-T-5 (compromised admin publishes harmful content), TM-R-1 (unaudited admin actions), TM-T-4 (stored XSS via authored content), TM-T-1 (mass assignment); CWE-79, CWE-862, CWE-915
- **Abuse / Misuse Case**: A subscriber invokes the authoring endpoint directly; or an admin edit sets publication state or a verification record through the edit payload, bypassing the publication gate; or an edit strips the last citation from a published plan, leaving live content without its evidence basis.
- **Trust Boundary**: Boundary 1 — admin-supplied content is untrusted input despite the elevated role.
- **Untrusted Inputs or Assertions**: The entire authored payload, including any publication or verification field.
- **Authoritative Enforcement Point**: REST API Application — role gate, input schema, binding allow-list, citation invariant, and audit dependency all apply before persistence.
- **Independent Verification**: The acting admin is taken from the session; the audit entry is written in the same transaction.
- **Zero Trust Relevance**: NIST SP 800-207 — privilege evaluated per request rather than inferred from the admin interface. Exact tenet: TO BE DECIDED (per-issue mappings are verified during the SQ-10 pre-launch assessment).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — per-issue mappings are verified during the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — per-issue mappings are verified during the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: N/A
- **Other**: `REF-INPUT`, `REF-PROMPT-API`, `REF-LOG` as cited by the security rules this issue implements.
- **Mapping Basis**: FR-4.3, FR-4.8, FR-10.1, and FR-10.2 are the normative sources; the references are those `SECURITY.md` names for the validation, authorization, and audit rules applied here.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated `admin`, when they create a plan with valid content, then the plan is persisted in an unpublished state with the admin recorded as author, and exactly one audit entry records the create action, the admin, the plan, and the time.
2. **AC-02 — Boundary or failure behavior**: Given an edit whose payload violates the content model — an exercise referencing no catalog entry or a nonexistent one, non-numeric sets, an absent calorie or macronutrient target — when it is submitted, then it is rejected with the failing field named, no field is changed, and no audit entry is written for the failed edit.
3. **AC-03 — Prohibited behavior**: Given a create or edit payload, when it is processed, then it MUST NOT set publication state, the verification record, the author identity, or any audit field (SEC-INPUT-3), and a `subscriber` or `consultant` invoking these operations MUST be denied (FR-4.2, FR-10.1).
4. **AC-04 — Additional criterion**: Given an edit to a published plan, when it succeeds, then existing subscriber customized copies derived from it remain unchanged (FR-7.5, enforced in REQ-CUSTOM-030), and the edit is audited.
5. **AC-05 — Additional criterion**: Given the authoring form, when a validation error occurs, then the error appears inline next to the failing field with icon and text, focus moves to the first invalid field, and the error is programmatically associated with the field (`DESIGN.md`).
6. **AC-06 — Additional criterion**: Given an edit to a published plan, when it would leave the plan without at least one citation, then it is rejected (FR-4.8, FR-4.4), no field is changed, and no audit entry is written; and given any successful edit, then it neither clears nor re-requires the plan's verification record (FR-4.8).

## Failure Behavior

- **On Invalid Input**: Reject per REQ-API-010 with field-level detail; nothing persisted; no audit entry.
- **On Authentication Failure**: Denied upstream; privileged accounts require a passkey (REQ-AUTH-020).
- **On Authorization Failure**: Deny for non-admin roles; unpublished plan existence MUST NOT be disclosed to them (REQ-AUTHZ-030, REQ-AUTHZ-040).
- **On Security-Decision Failure**: Deny if role or acting identity cannot be resolved.
- **On External Dependency Failure**: If persistence or audit storage is unavailable, the operation fails atomically and nothing is written.
- **On System Error**: Roll back the plan write and its audit entry together.
- **Logging / Audit**: One audit entry per successful create or edit (REQ-AUDIT-030). Denials logged as security events (REQ-AUTHZ-040). The plan body is not copied into the entry.
- **Alerting**: Authorization-denial and admin-lifecycle audit events are SEC-LOG-4/SEC-LOG-6 detection inputs routed to the security lead under SEC-OPS-2 (`SECURITY.md` SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: Create and edit services validate against the content model; the binding allow-list excludes publication, verification, author, and audit fields; the citation invariant rejects an edit that would leave a published plan citation-less; the audit writer is invoked exactly once per success.
- **Integration Tests**: Create and edit both plan types end to end as an admin; assert unpublished initial state, audit entries, and that an omitted field is left unchanged on edit; assert an edit to a published, verified plan leaves the verification record exactly as it was.
- **Security Tests**: Role matrix asserting denial for `subscriber` and `consultant`; mass-assignment attempts on publication and verification; markup-bearing content stored verbatim with escaping asserted at the rendering boundary; audit-failure fault injection asserting the plan write is rolled back.
- **Compliance Tests / Evidence**: Audit entries for create and edit, retained as accountability evidence for FR-10.2.
- **Acceptance-Criteria Traceability**: AC-01 — create suite; AC-02 — edit validation suite; AC-03 — role and mass-assignment negative suites; AC-04 — copy-stability cross-check with REQ-CUSTOM-030; AC-05 — form error presentation test; AC-06 — citation-invariant and verification-stability suites.
- **Coverage Target**: Both plan types × create and edit, positive and negative; all three roles exercised against each operation.
- **Required Test Environment**: Two admin identities, one subscriber, one consultant; seeded plans in draft and published states; seeded catalog entries including a retired one. Runs against PostgreSQL on Vitest.

## Dependencies

- **Upstream Requirements**: REQ-PLAN-010, REQ-PLAN-020, REQ-PLAN-080, REQ-AUTHZ-030, REQ-AUDIT-030, REQ-API-010, REQ-API-020, REQ-PLATFORM-030
- **Downstream Requirements**: REQ-PLAN-040, REQ-PLAN-050, REQ-PLAN-060, REQ-PLAN-070, REQ-CATALOG-010, REQ-CATALOG-020
- **External Dependencies**: None
- **Dependency Assumptions**: Audit storage shares the transactional context of plan storage, so AC-01's atomicity holds.
- **Failure Impact**: Unaudited or under-validated authoring is the entry point for the highest safety-impact threat in the model (TM-T-5).

## Implementation Notes

- **Constraints**: PostgreSQL with Drizzle ORM and Fastify (`CLAUDE.md`); the client is a Vite-built single-page application with `vue-router`. Edit semantics are fixed by FR-4.8 (`REQUIREMENTS.md` OQ-10 and OQ-16 RESOLVED): verification is a one-time gate that an edit never clears or re-requires, and the FR-4.4 citation rule is a standing invariant enforced at edit time. Exercise plan payloads compose exercises by catalog reference (FR-5.4): a create or edit selects among catalog entries and sets prescriptions, and retired entries are excluded from new composition (`DESIGN.md`, admin Plans catalog pattern: retiring removes an entry from new plan composition, never from history).
- **Prohibited Approaches**: A combined "save and publish" operation that bypasses the publication gate; binding the request body wholesale; treating the admin role as a reason to skip input validation; writing the audit entry outside the transaction; clearing or re-requiring verification as a side effect of edit (FR-4.8 forbids both directions).
- **Implementation Guidance**: Keep create and edit as distinct operations with distinct audit action values so REQ-AUDIT-030's five-action enumeration stays exact. Enforce the citation invariant (AC-06) in the edit operation itself, not only in citation management (REQ-PLAN-040), since an edit payload that replaces plan content wholesale is where the last citation can silently vanish.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-VUE`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the binding allow-list and role gate; product review of the authoring form.
- **Open Decisions**: None. Verification-on-edit is resolved (FR-4.8; `REQUIREMENTS.md` OQ-10 and OQ-16 RESOLVED — the one-time verification operation itself is REQ-PLAN-070), and the admin workspace treatment is defined (`DESIGN.md` OQ-7 RESOLVED).

**Estimated effort**: 1.5–2 engineer-days. **Estimated changed lines**: 400–900.
**Recommended model**: Claude Opus (`claude-opus-5`) — bounded implementation across authorization, validation, audit, and an accessible form, with the FR-4.8 edit invariants that must hold exactly.
