# [REQ-PLAN-010] Exercise plan content model

## Metadata

- **ID**: REQ-PLAN-010
- **Title**: Exercise plan content model
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-4.1, FR-5.1

## Requirement

- **Statement**: The system MUST persist exercise plans as admin-authored records whose contents include their exercises and the prescribed sets and repetitions per exercise, together with the plan's citations, its verification record, and its publication state.
- **Rationale**: FR-4.1 requires a library of exercise plans authored by admins; FR-5.1 requires a subscriber to view a plan's full contents including its exercises and prescribed sets and repetitions, which fixes the minimum content model. Citations (FR-4.4), verification (FR-4.5), and publication state (FR-4.7) are attributes of the same record.
- **Assumptions**: Exercise plans and diet plans are distinct entities (`ARCHITECTURE.md`, Data model expectations, lists them separately).
- **Out of Scope**: Diet plans (REQ-PLAN-020); the authoring operations (REQ-PLAN-030); citation management behavior (REQ-PLAN-040); the publication gate (REQ-PLAN-050); subscriber-facing retrieval (REQ-CATALOG-010); whether a plan may be re-verified after edit (`REQUIREMENTS.md` OQ-10) and whether verification needs a second admin (OQ-16), both blocked; demonstration imagery (`DESIGN.md` OQ-9); rich-text plan content, which SEC-RENDER-2 marks `TO BE DECIDED`.
- **Design Traceability**: `DESIGN.md` — Typography (numeric data rendered with tabular/lining figures, which applies to sets and repetitions); Layout and Spacing (72ch for plan descriptions).
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application ("Owns … exercise plan … plan citations and verification records"); Relational Persistence ("Enforces referential integrity … in schema"); DR-4.
- **Security Traceability**: SEC-INPUT-1 (field constraints), SEC-INPUT-5 (parameterized access), SEC-RENDER-1 and SEC-RENDER-2 (content is rendered through auto-escaping; rich text would require a vetted sanitizer), SEC-EXT-2 (citation URLs are never fetched).

## Scope

- **Applies To**: Server-Side Application, API
- **Components**: Relational Persistence; REST API Application
- **Interfaces / Operations**: Exercise plan persistence and retrieval by the operations that own them
- **Actors**: `admin` as author; `subscriber` as reader of published plans
- **Preconditions**: None
- **Data Classification**: Internal — plan content is admin-authored and not personal data; which plan a subscriber follows is health data and is modelled elsewhere
- **Personal or Regulated Data**: None — the plan record itself carries no personal data beyond the verifying admin's identifier
- **Jurisdiction / Regulatory Scope**: N/A

## Security Context

- **Security Objectives**: Integrity, Safety
- **Control Layers**: Business-Rule Validation, Input Validation, Architecture
- **Threat References**: `SECURITY.md` TM-T-4 (stored XSS via admin-authored plan content), TM-T-5 (harmful published content); CWE-79 (cross-site scripting, as the sink for content stored here), CWE-20 (improper input validation)
- **Abuse / Misuse Case**: A compromised admin stores markup or a hostile citation URL in plan content that later renders in every subscriber's browser; or malformed set and repetition values are stored and presented as prescriptions.
- **Trust Boundary**: Boundary 1 for authored content — admin input is untrusted at the API boundary despite the elevated role; boundary 3 for persistence.
- **Untrusted Inputs or Assertions**: All authored plan content, including exercise names, descriptions, numeric prescriptions, and citation URLs.
- **Authoritative Enforcement Point**: REST API Application, validating content against the model before persistence.
- **Independent Verification**: Structural constraints are enforced in schema, not only in application code.
- **Zero Trust Relevance**: N/A — a data model, not an access decision.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: N/A
- **Regulatory**: N/A — plan content is not personal or regulated data.
- **Other**: `REF-INPUT` for field constraint design, as cited by SEC-INPUT-1.
- **Mapping Basis**: FR-4.1 and FR-5.1 fix the content; the security references are those `SECURITY.md` cites for the validation and rendering rules this model must support.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a persisted exercise plan, when it is retrieved, then it carries its identity, its title and description, its ordered exercises with prescribed sets and repetitions per exercise, its citations, its verification record where one exists, and its publication state.
2. **AC-02 — Boundary or failure behavior**: Given an attempt to persist an exercise plan with a non-numeric, negative, or absent value for prescribed sets or repetitions, or with an exercise carrying no name, when the write occurs, then it is rejected with the failing field identified and nothing is persisted.
3. **AC-03 — Prohibited behavior**: Given the model, when it is defined, then publication state, the verification record, and citation-verified status MUST NOT be settable from a request body (SEC-INPUT-3), and no plan may exist in a published state without the attributes REQ-PLAN-050 gates on.
4. **AC-04 — Additional criterion**: Given a plan's exercises, when they are retrieved, then their order is stable and reproducible across reads, since a prescription's sequence is part of its meaning.
5. **AC-05 — Additional criterion**: Given referential integrity, when a plan is deleted or unpublished, then its citations and verification record remain consistently associated and no orphaned rows result.

## Failure Behavior

- **On Invalid Input**: Rejected at the boundary per REQ-API-010 with field-level detail; no partial plan is persisted.
- **On Authentication Failure**: Denied upstream.
- **On Authorization Failure**: Denied per REQ-AUTHZ-030 — only admins reach authoring operations.
- **On Security-Decision Failure**: If publication or verification state cannot be resolved, treat the plan as unpublished (fail closed) so it is not shown to subscribers.
- **On External Dependency Failure**: If persistence is unavailable, the operation fails atomically.
- **On System Error**: Roll back so no partially written plan, exercise list, or citation set survives.
- **Logging / Audit**: Plan lifecycle actions are audited per REQ-AUDIT-030. The plan body itself is not copied into audit entries.
- **Alerting**: N/A

## Test Strategy

- **Unit Tests**: Model validation for each field — title, description, exercise name, sets, repetitions — covering valid, missing, non-numeric, negative, and over-length cases.
- **Integration Tests**: Persist and retrieve a plan with multiple exercises, asserting content fidelity and stable ordering; referential integrity on delete and unpublish.
- **Security Tests**: Mass-assignment attempts on publication state and verification record; markup-bearing content stored and retrieved verbatim, with the escaping obligation verified at the rendering boundary (REQ-CATALOG-030); assertion that no citation URL is dereferenced during persistence (SEC-EXT-2).
- **Compliance Tests / Evidence**: N/A
- **Acceptance-Criteria Traceability**: AC-01 — round-trip retrieval test; AC-02 — field validation matrix; AC-03 — mass-assignment and invariant tests; AC-04 — ordering stability test; AC-05 — referential integrity tests.
- **Coverage Target**: Every modelled field covered positive and negative.
- **Required Test Environment**: PostgreSQL with drizzle-kit migrations applied; an admin identity; Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-API-010, REQ-API-020, REQ-API-030
- **Downstream Requirements**: REQ-PLAN-030, REQ-PLAN-040, REQ-PLAN-050, REQ-PLAN-060, REQ-CATALOG-010, REQ-CUSTOM-010, REQ-PROGRESS-020
- **External Dependencies**: None
- **Dependency Assumptions**: The chosen RDBMS supports the constraints needed for AC-02 and AC-05 in schema.
- **Failure Impact**: An imprecise plan model propagates into customization copies, workout logging, and progress comparison, all of which reference its structure.

## Implementation Notes

- **Constraints**: PostgreSQL with Drizzle ORM and drizzle-kit migrations (`CLAUDE.md`). Schema design remains TO BE DECIDED, though the required content is fixed by FR-5.1.
- **Prohibited Approaches**: Storing exercise prescriptions as free text when FR-5.1 names sets and repetitions as distinct content; a shared polymorphic plan table that blurs exercise and diet plans, which `ARCHITECTURE.md` lists as separate entities; embedding rendered HTML in plan content while SEC-RENDER-2 leaves rich text undecided.
- **Implementation Guidance**: Model exercises as ordered child records rather than an opaque blob, so FR-7.1 customization and FR-8.3 workout logging can reference individual exercises. Keep units of prescription explicit where they exist, noting that `REQUIREMENTS.md` OQ-4 leaves the unit system open for measurements — do not generalize that decision to exercise prescriptions without a documented choice.
- **AI Development Guidance**: `REF-PROMPT-QUALITY`, `REF-PROMPT-NODE`; `CLAUDE.md`.
- **Required Human Review**: Architecture review of the schema; product review that the modelled content matches what admins need to author.
- **Open Decisions**: Whether plan content requires rich text (SEC-RENDER-2, `TO BE DECIDED`); whether demonstration imagery is in scope (`DESIGN.md` OQ-9); verification re-triggering on edit (`REQUIREMENTS.md` OQ-10). None prevent the minimum model FR-5.1 fixes, but each would extend it.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 250–500.
**Recommended model**: Claude Fable (`claude-fable-5`) — a foundational data model that several later issues build on, where consistency across customization, logging, and rendering matters more than local optimization.
