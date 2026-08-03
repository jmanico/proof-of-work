# [REQ-PLAN-010] Exercise plan content model

## Metadata

- **ID**: REQ-PLAN-010
- **Title**: Exercise plan content model
- **Version**: 1.1.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-4.1, FR-5.1, FR-5.4

## Requirement

- **Statement**: The system MUST persist exercise plans as admin-authored records whose contents include their ordered exercises — each composed by reference to an exercise catalog entry (FR-5.4) with prescribed sets and repetitions — together with the plan's citations, its verification record, and its publication state.
- **Rationale**: FR-4.1 requires a library of exercise plans authored by admins; FR-5.1 requires a subscriber to view a plan's full contents including its exercises and prescribed sets and repetitions, which fixes the minimum content model. FR-5.4 fixes the composition: exercise plans and copies compose their exercises by referencing catalog entries, giving each exercise a stable identity that FR-8.14's per-exercise trends and FR-8.3's logging depend on. Citations (FR-4.4), verification (FR-4.5), and publication state (FR-4.7) are attributes of the same record.
- **Assumptions**: Exercise plans and diet plans are distinct entities (`ARCHITECTURE.md`, Data model expectations, lists them separately, and lists the exercise catalog entry as its own entity). The catalog and its management operations exist (REQ-PLAN-080).
- **Out of Scope**: Diet plans (REQ-PLAN-020); the authoring operations (REQ-PLAN-030); citation management behavior (REQ-PLAN-040); the publication gate (REQ-PLAN-050); the verification operation (REQ-PLAN-070 — FR-4.8, a one-time gate, `REQUIREMENTS.md` OQ-10 and OQ-16 RESOLVED); catalog management itself — create, edit, retire (REQ-PLAN-080); subscriber-facing retrieval (REQ-CATALOG-010); exercise demonstration diagrams, which `DESIGN.md` OQ-9 resolves as admin-approved neutral vector diagrams under its imagery rules and which extend rather than change this model.
- **Design Traceability**: `DESIGN.md` — Typography (numeric data rendered with lining tabular figures, which applies to sets and repetitions; long-form text limited to 68ch for plan descriptions); Product Patterns → Plans, citations, and review state (structured plain-text content: headings, ordered steps, lists, tables, approved diagrams).
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application ("Owns … exercise catalog entry, exercise plan … plan citations and verification records"); Relational Persistence ("Enforces referential integrity … in schema"); DR-4.
- **Security Traceability**: SEC-INPUT-1 (field constraints), SEC-INPUT-5 (parameterized access), SEC-RENDER-1 and SEC-RENDER-2 (content is rendered through auto-escaping; v1 plan content is structured plain text with no rich-text rendering path — Confirmed), SEC-EXT-2 (citation URLs are never fetched).

## Scope

- **Applies To**: Server-Side Application, API
- **Components**: Relational Persistence; REST API Application
- **Interfaces / Operations**: Exercise plan persistence and retrieval by the operations that own them
- **Actors**: `admin` as author; `subscriber` as reader of published plans
- **Preconditions**: None
- **Data Classification**: Internal — plan content is admin-authored and not personal data; which plan a subscriber follows is health data (FR-9.12) and is modelled elsewhere
- **Personal or Regulated Data**: None — the plan record itself carries no personal data beyond the verifying admin's identifier
- **Jurisdiction / Regulatory Scope**: N/A

## Security Context

- **Security Objectives**: Integrity, Safety
- **Control Layers**: Business-Rule Validation, Input Validation, Architecture
- **Threat References**: `SECURITY.md` TM-T-4 (stored XSS via admin-authored plan content), TM-T-5 (harmful published content); CWE-79 (cross-site scripting, as the sink for content stored here), CWE-20 (improper input validation)
- **Abuse / Misuse Case**: A compromised admin stores markup or a hostile citation URL in plan content that later renders in every subscriber's browser; or malformed set and repetition values are stored and presented as prescriptions.
- **Trust Boundary**: Boundary 1 for authored content — admin input is untrusted at the API boundary despite the elevated role; boundary 3 for persistence.
- **Untrusted Inputs or Assertions**: All authored plan content, including catalog-entry references, descriptions, numeric prescriptions, and citation URLs.
- **Authoritative Enforcement Point**: REST API Application, validating content against the model before persistence.
- **Independent Verification**: Structural constraints are enforced in schema, not only in application code.
- **Zero Trust Relevance**: N/A — a data model, not an access decision.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — per-issue mappings are verified during the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — per-issue mappings are verified during the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **NIST SP 800-207**: N/A
- **Regulatory**: N/A — plan content is not personal or regulated data.
- **Other**: `REF-INPUT` for field constraint design, as cited by SEC-INPUT-1.
- **Mapping Basis**: FR-4.1 and FR-5.1 fix the content and FR-5.4 the composition; the security references are those `SECURITY.md` cites for the validation and rendering rules this model must support.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a persisted exercise plan, when it is retrieved, then it carries its identity, its title and description, its ordered exercises — each referencing an exercise catalog entry (FR-5.4) with prescribed sets and repetitions — its citations, its verification record where one exists, and its publication state.
2. **AC-02 — Boundary or failure behavior**: Given an attempt to persist an exercise plan with a non-numeric, negative, or absent value for prescribed sets or repetitions, or with an exercise that references no catalog entry or a nonexistent catalog entry, when the write occurs, then it is rejected with the failing field identified and nothing is persisted.
3. **AC-03 — Prohibited behavior**: Given the model, when it is defined, then publication state, the verification record, and citation-verified status MUST NOT be settable from a request body (SEC-INPUT-3), and no plan may exist in a published state without the attributes REQ-PLAN-050 gates on.
4. **AC-04 — Additional criterion**: Given a plan's exercises, when they are retrieved, then their order is stable and reproducible across reads, since a prescription's sequence is part of its meaning.
5. **AC-05 — Additional criterion**: Given referential integrity, when a plan is deleted or unpublished, then its citations and verification record remain consistently associated and no orphaned rows result; and a catalog entry referenced by any plan MUST NOT be deletable (FR-5.4 — retire, never delete), so a plan's exercise references never dangle.

## Failure Behavior

- **On Invalid Input**: Rejected at the boundary per REQ-API-010 with field-level detail; no partial plan is persisted.
- **On Authentication Failure**: Denied upstream.
- **On Authorization Failure**: Denied per REQ-AUTHZ-030 — only admins reach authoring operations.
- **On Security-Decision Failure**: If publication or verification state cannot be resolved, treat the plan as unpublished (fail closed) so it is not shown to subscribers.
- **On External Dependency Failure**: If persistence is unavailable, the operation fails atomically.
- **On System Error**: Roll back so no partially written plan, exercise list, or citation set survives.
- **Logging / Audit**: Plan lifecycle actions are audited per REQ-AUDIT-030. The plan body itself is not copied into audit entries.
- **Alerting**: N/A — a data model has no runtime alert condition of its own; lifecycle-action audit events are REQ-AUDIT-030's concern.

## Test Strategy

- **Unit Tests**: Model validation for each field — title, description, catalog-entry reference, sets, repetitions — covering valid, missing, non-numeric, negative, over-length, and nonexistent-reference cases.
- **Integration Tests**: Persist and retrieve a plan with multiple exercises referencing catalog entries, asserting content fidelity and stable ordering; referential integrity on delete and unpublish; a delete attempt on a referenced catalog entry fails in schema.
- **Security Tests**: Mass-assignment attempts on publication state and verification record; markup-bearing content stored and retrieved verbatim, with the escaping obligation verified at the rendering boundary (REQ-CATALOG-030); assertion that no citation URL is dereferenced during persistence (SEC-EXT-2).
- **Compliance Tests / Evidence**: N/A
- **Acceptance-Criteria Traceability**: AC-01 — round-trip retrieval test; AC-02 — field validation matrix including catalog references; AC-03 — mass-assignment and invariant tests; AC-04 — ordering stability test; AC-05 — referential integrity tests including the catalog no-delete constraint.
- **Coverage Target**: Every modelled field covered positive and negative.
- **Required Test Environment**: PostgreSQL with drizzle-kit migrations applied; an admin identity; seeded catalog entries. Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-API-010, REQ-API-020, REQ-API-030, REQ-PLAN-080
- **Downstream Requirements**: REQ-PLAN-030, REQ-PLAN-040, REQ-PLAN-050, REQ-PLAN-060, REQ-CATALOG-010, REQ-CUSTOM-010, REQ-PROGRESS-020
- **External Dependencies**: None
- **Dependency Assumptions**: The chosen RDBMS supports the constraints needed for AC-02 and AC-05 in schema, including the foreign-key relationship to catalog entries.
- **Failure Impact**: An imprecise plan model propagates into customization copies, workout logging, and progress comparison, all of which reference its structure — and, through the catalog reference, into FR-8.14's per-exercise trend identity.

## Implementation Notes

- **Constraints**: PostgreSQL with Drizzle ORM and drizzle-kit migrations (`CLAUDE.md`). Schema design and indexing are implementation-level, decided with the code (`ARCHITECTURE.md`); the required content is fixed by FR-5.1 and the composition by FR-5.4. V1 plan content is structured plain text — headings, steps, lists, tables, and approved diagrams — with no rich-text rendering path (SEC-RENDER-2, Confirmed; `DESIGN.md`).
- **Prohibited Approaches**: Storing exercise prescriptions as free text when FR-5.1 names sets and repetitions as distinct content; embedding a free-text exercise name in the plan instead of a catalog reference (FR-5.4 — the catalog entry owns the name, and renames must not change identity); a shared polymorphic plan table that blurs exercise and diet plans, which `ARCHITECTURE.md` lists as separate entities; embedding rendered HTML in plan content (SEC-RENDER-2).
- **Implementation Guidance**: Model exercises as ordered child records, each carrying a foreign key to its catalog entry plus the prescription (sets, repetitions, order), so FR-7.1 customization can select among catalog exercises and edit prescriptions, and FR-8.3 workout logging can reference the same catalog identity. Sets and repetitions are unitless counts; workout load units belong to logging under FR-8.10 (`REQUIREMENTS.md` OQ-4 RESOLVED) and are not part of the plan prescription FR-5.1 defines.
- **AI Development Guidance**: `REF-PROMPT-QUALITY`, `REF-PROMPT-NODE`; `CLAUDE.md`.
- **Required Human Review**: Architecture review of the schema; product review that the modelled content matches what admins need to author.
- **Open Decisions**: None. Rich text is settled (SEC-RENDER-2 Confirmed: v1 is structured plain text); demonstration imagery is settled (`DESIGN.md` OQ-9 RESOLVED); verification semantics are settled (FR-4.8, `REQUIREMENTS.md` OQ-10 and OQ-16 RESOLVED — REQ-PLAN-070); exercise identity is settled (FR-5.4 — REQ-PLAN-080).

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 250–500.
**Recommended model**: Claude Fable (`claude-fable-5`) — a foundational data model that several later issues build on, where consistency across customization, logging, and rendering matters more than local optimization.
