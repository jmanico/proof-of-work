# [REQ-PLAN-020] Diet plan content model with calorie and macronutrient targets

## Metadata

- **ID**: REQ-PLAN-020
- **Title**: Diet plan content model with calorie and macronutrient targets
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-4.1, FR-6.1, FR-6.2

## Requirement

- **Statement**: The system MUST persist diet plans as admin-authored records that specify their meals and their daily calorie and macronutrient targets, together with the plan's citations, its verification record, and its publication state.
- **Rationale**: FR-6.2 requires each diet plan to specify its meals and its daily calorie and macronutrient targets; FR-6.1 requires a subscriber to view a plan's full contents; FR-4.1 places diet plans in the admin-authored library. The targets are also the comparison basis for FR-8.5, so their structure is load-bearing beyond display.
- **Assumptions**: Exercise plans and diet plans are distinct entities (`ARCHITECTURE.md`, Data model expectations).
- **Out of Scope**: Exercise plans (REQ-PLAN-010); authoring operations (REQ-PLAN-030); citation management (REQ-PLAN-040); the publication gate (REQ-PLAN-050); food logging and the comparison of logged intake against these targets (FR-8.4, FR-8.5), both blocked by `REQUIREMENTS.md` OQ-5 because no nutrition data source is defined; plan selection, blocked by OQ-6.
- **Design Traceability**: `DESIGN.md` — Typography (numeric data — calories and macros — rendered with tabular/lining figures so values align in columns); Layout and Spacing (72ch for plan descriptions).
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application ("Owns … diet plan"); Relational Persistence; DR-4; FR-6.1–FR-6.3 traceability row.
- **Security Traceability**: SEC-INPUT-1, SEC-INPUT-5, SEC-INPUT-3 (publication and verification not client-settable), SEC-RENDER-1 and SEC-RENDER-2.

## Scope

- **Applies To**: Server-Side Application, API
- **Components**: Relational Persistence; REST API Application
- **Interfaces / Operations**: Diet plan persistence and retrieval by the operations that own them
- **Actors**: `admin` as author; `subscriber` as reader of published plans
- **Preconditions**: None
- **Data Classification**: Internal — plan content is not personal data
- **Personal or Regulated Data**: None — the record carries no personal data beyond the verifying admin's identifier
- **Jurisdiction / Regulatory Scope**: N/A

## Security Context

- **Security Objectives**: Integrity, Safety
- **Control Layers**: Business-Rule Validation, Input Validation, Architecture
- **Threat References**: `SECURITY.md` TM-T-4 (stored XSS via admin-authored content), TM-T-5 (harmful published dietary content); CWE-79 (cross-site scripting), CWE-20 (improper input validation)
- **Abuse / Misuse Case**: A compromised admin publishes a diet plan with dangerous calorie targets, or stores markup in meal descriptions that renders in every subscriber's browser.
- **Trust Boundary**: Boundary 1 for authored content; boundary 3 for persistence.
- **Untrusted Inputs or Assertions**: All authored content — meal names and descriptions, calorie and macronutrient target values, citation URLs.
- **Authoritative Enforcement Point**: REST API Application, validating against the model before persistence.
- **Independent Verification**: Numeric and structural constraints enforced in schema as well as in application code.
- **Zero Trust Relevance**: N/A

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: N/A
- **Regulatory**: N/A — plan content is not personal or regulated data. Note that dietary prescriptions carry safety weight, which FR-9.6's disclaimer and FR-4.4's citation requirement address rather than a statute named in any source document.
- **Other**: `REF-INPUT` for field constraint design.
- **Mapping Basis**: FR-6.2 fixes the required content; the security references are those `SECURITY.md` cites for the validation and rendering rules the model must support.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a persisted diet plan, when it is retrieved, then it carries its identity, title and description, its meals, its daily calorie target, its daily macronutrient targets, its citations, its verification record where one exists, and its publication state.
2. **AC-02 — Boundary or failure behavior**: Given an attempt to persist a diet plan whose daily calorie or macronutrient target is absent, non-numeric, or negative, or that has no meals, when the write occurs, then it is rejected with the failing field identified and nothing is persisted.
3. **AC-03 — Prohibited behavior**: Given the model, when it is defined, then publication state and the verification record MUST NOT be settable from a request body (SEC-INPUT-3), and a diet plan MUST NOT be persistable without the daily calorie and macronutrient targets FR-6.2 requires.
4. **AC-04 — Additional criterion**: Given a plan's meals, when they are retrieved, then their order is stable and reproducible across reads.
5. **AC-05 — Additional criterion**: Given the macronutrient targets, when they are modelled, then each macronutrient is a distinct named numeric field rather than free text, so that FR-8.5's comparison against logged intake is computable once `REQUIREMENTS.md` OQ-5 resolves.

## Failure Behavior

- **On Invalid Input**: Rejected at the boundary per REQ-API-010 with field-level detail; no partial plan persisted.
- **On Authentication Failure**: Denied upstream.
- **On Authorization Failure**: Denied per REQ-AUTHZ-030.
- **On Security-Decision Failure**: If publication or verification state cannot be resolved, treat the plan as unpublished (fail closed).
- **On External Dependency Failure**: The operation fails atomically if persistence is unavailable.
- **On System Error**: Roll back so no partially written plan, meal set, or target set survives.
- **Logging / Audit**: Plan lifecycle actions are audited per REQ-AUDIT-030; the plan body is not copied into audit entries.
- **Alerting**: N/A

## Test Strategy

- **Unit Tests**: Validation for title, description, meal name, calorie target, and each macronutrient target across valid, missing, non-numeric, negative, and over-length cases.
- **Integration Tests**: Persist and retrieve a plan with multiple meals and a full target set, asserting content fidelity and stable ordering.
- **Security Tests**: Mass-assignment attempts on publication state and verification record; markup-bearing meal descriptions stored verbatim with escaping verified at the rendering boundary (REQ-CATALOG-030).
- **Compliance Tests / Evidence**: N/A
- **Acceptance-Criteria Traceability**: AC-01 — round-trip test; AC-02 — validation matrix; AC-03 — mass-assignment and required-target tests; AC-04 — ordering test; AC-05 — schema shape assertion.
- **Coverage Target**: Every modelled field covered positive and negative.
- **Required Test Environment**: PostgreSQL with drizzle-kit migrations applied; an admin identity; Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-API-010, REQ-API-020, REQ-API-030
- **Downstream Requirements**: REQ-PLAN-030, REQ-PLAN-040, REQ-PLAN-050, REQ-PLAN-060, REQ-CATALOG-020, REQ-CUSTOM-010
- **External Dependencies**: None — notably, no external nutrition database is in scope (`REQUIREMENTS.md`, External integrations; OQ-5).
- **Dependency Assumptions**: The chosen RDBMS supports the numeric and referential constraints AC-02 and AC-05 require.
- **Failure Impact**: Targets modelled as free text would make FR-8.5 uncomputable and would push the comparison logic into presentation code.

## Implementation Notes

- **Constraints**: PostgreSQL with Drizzle ORM and drizzle-kit migrations (`CLAUDE.md`); schema design remains TO BE DECIDED. Which macronutrients must be modelled is not enumerated by `REQUIREMENTS.md` beyond "macronutrient targets"; the conventional set is protein, carbohydrate, and fat, but this MUST be confirmed by product rather than assumed silently — record it as an open decision if it is not confirmed before implementation.
- **Prohibited Approaches**: A single free-text "targets" field; storing targets only as a computed percentage without absolute values; a shared plan table that merges exercise and diet plans; introducing an external nutrition data source, which is out of scope and would cross the FR-9.8 boundary.
- **Implementation Guidance**: Model meals as ordered child records so customization (FR-7.1) can edit individual meals. Keep units explicit on calorie and macronutrient values.
- **AI Development Guidance**: `REF-PROMPT-QUALITY`, `REF-PROMPT-NODE`; `CLAUDE.md`.
- **Required Human Review**: Product review of the macronutrient field set; architecture review of the schema.
- **Open Decisions**: The exact macronutrient set to model (not enumerated in `REQUIREMENTS.md`); the unit system (`REQUIREMENTS.md` OQ-4 leaves units open for measurements and `DESIGN.md` OQ-8 for numeric formatting generally); rich text (SEC-RENDER-2). Food logging and target comparison (OQ-5) are blocked and have no issue.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 250–500.
**Recommended model**: Claude Fable (`claude-fable-5`) — a foundational model that must stay consistent with the exercise plan model and with the blocked food-logging work it will later feed.
