# [REQ-CATALOG-010] Browse and view published exercise plans

## Metadata

- **ID**: REQ-CATALOG-010
- **Title**: Browse and view published exercise plans
- **Version**: 1.1.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-5.1, FR-4.6, FR-4.7, FR-5.4

## Requirement

- **Statement**: A subscriber MUST be able to browse published exercise plans and view a plan's full contents, including its exercises — each referencing an exercise-catalog entry (FR-5.4) — and prescribed sets and repetitions, together with the plan's citations; only published plans MUST be shown.
- **Rationale**: FR-5.1 states the browse and view behavior and names the content that must be visible; FR-4.6 requires citations to be displayed when a plan is viewed; FR-4.7 restricts what subscribers see to published plans. FR-5.4 (added 2026-08-03) fixes how a plan composes its exercises: by reference to admin-curated catalog entries with stable identifiers, so the view displays catalog-referenced exercises.
- **Assumptions**: The exercise plan model and publication state exist (REQ-PLAN-010, REQ-PLAN-050), and plan exercises reference catalog entries (FR-5.4, REQ-PLAN-080).
- **Out of Scope**: Selecting a plan to follow (FR-5.2, FR-5.3; `REQUIREMENTS.md` OQ-6 RESOLVED) — planned as REQ-SELECT-010; the subscription entitlement gate (FR-3.1, FR-3.2; OQ-1 RESOLVED) — planned as REQ-ENTITLE-010, which gates this surface; exercise-catalog management itself (FR-5.4, REQ-PLAN-080); the safe rendering mechanics of plan content and citation links (REQ-CATALOG-030); diet plans (REQ-CATALOG-020). Citation and verification presentation is resolved by `DESIGN.md` OQ-5 and followed here.
- **Design Traceability**: `DESIGN.md` — Typography and Iconography (data face with lining tabular figures for sets × reps; values and units do not wrap apart); Layout, Spacing, and Responsive Behavior (80rem maximum application content, 68ch prose, data tables become labelled record cards below 48rem, 320px/200% one-column reflow); Core Components → Status, feedback, and loading (empty states), Cards, lists, and tables; Product Patterns → Plans, citations, and review state (Evidence reviewed line, inline numbered references, in-page Evidence section); Accessibility.
- **Architecture Traceability**: `ARCHITECTURE.md` — data flow 2 ("Client → REST API → entitlement and publication checks → Persistence → plan content with citations and verification record → Client"); Browser Client; REST API Application.
- **Security Traceability**: SEC-AUTHN-1, SEC-AUTHZ-1, SEC-DATA-5, SEC-RENDER-1, SEC-RENDER-3, SEC-HTTP-2.

## Scope

- **Applies To**: Server-Side Application, API, Web Client
- **Components**: REST API Application; Relational Persistence; Browser Client (plan library and plan detail views)
- **Interfaces / Operations**: List published exercise plans; retrieve one published exercise plan with its contents and citations
- **Actors**: `subscriber`; `consultant` and `admin` insofar as they are authenticated readers
- **Preconditions**: Authenticated session
- **Data Classification**: Internal — plan content itself is not personal data
- **Personal or Regulated Data**: None in the plan content. Which plans a subscriber views is behavioral data; the FR-9.12 health-data definition covers log entries, plan copies, and active selections — not browsing published plans — so this issue creates no audited health-data access.
- **Jurisdiction / Regulatory Scope**: N/A

## Security Context

- **Security Objectives**: Confidentiality, Integrity, Availability
- **Control Layers**: Authorization, Output Encoding, Data Protection
- **Threat References**: `SECURITY.md` TM-T-4 (stored XSS via admin-authored plan content rendered here), TM-I-3 (excessive data exposure); CWE-79 (cross-site scripting), CWE-200 (exposure of sensitive information)
- **Abuse / Misuse Case**: An unpublished or draft plan becomes reachable through a direct identifier or a listing filter; or admin-authored markup executes in the reader's browser when the plan renders.
- **Trust Boundary**: Boundary 1 — the response crosses to an untrusted client; the stored content is untrusted at render time regardless of its admin origin.
- **Untrusted Inputs or Assertions**: Listing filter and pagination parameters; the plan identifier; the stored plan content at render time.
- **Authoritative Enforcement Point**: REST API Application — the published-only filter is applied in the query, not in the client.
- **Independent Verification**: The publication filter is part of the query rather than a post-retrieval filter (SEC-AUTHZ-2 pattern).
- **Zero Trust Relevance**: N/A

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: N/A
- **Regulatory**: N/A
- **Other**: WCAG 2.2 AA as adopted by `DESIGN.md`, for the plan views' structure, contrast, and keyboard operability.
- **Mapping Basis**: FR-5.1, FR-4.6, and FR-4.7 are the normative sources; `DESIGN.md` supplies the presentation obligations. No control catalog identifier is asserted without verification.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated subscriber, when they browse the exercise plan library, then only published exercise plans are listed, and when they open one, its full contents — exercises named from their referenced catalog entries (FR-5.4), with prescribed sets and repetitions — and its citations are displayed.
2. **AC-02 — Boundary or failure behavior**: Given an unpublished exercise plan, when a subscriber requests it directly by identifier, then the response is indistinguishable from that for a plan that does not exist, and it never appears in a listing regardless of filter or pagination parameters.
3. **AC-03 — Prohibited behavior**: Given the plan library, when it is served, then it MUST NOT return unpublished plans through any parameter combination, MUST NOT return fields beyond those the view requires (SEC-DATA-5), and the client MUST NOT filter unpublished plans as its own responsibility (DR-2).
4. **AC-04 — Additional criterion**: Given the plan library with no published plans, when it renders, then an empty state says what belongs in the region and provides the action that creates it where one is available to the actor (`DESIGN.md`, Core Components → Status, feedback, and loading).
5. **AC-05 — Additional criterion**: Given a plan detail view, when it renders at a 320px viewport and at 200% zoom, then all content including the exercise table reflows without horizontal page scrolling and without loss of content, and numeric prescriptions render with tabular figures (`DESIGN.md`).

## Failure Behavior

- **On Invalid Input**: Reject malformed listing or identifier parameters per REQ-API-010 with field-level detail.
- **On Authentication Failure**: Denied upstream by REQ-AUTHZ-010.
- **On Authorization Failure**: Deny without disclosing plan existence (REQ-AUTHZ-040).
- **On Security-Decision Failure**: If publication state cannot be determined for a plan, exclude it from results (fail closed).
- **On External Dependency Failure**: If persistence is unavailable, return a generic error with a correlation identifier; the client shows an error state rather than a partial library.
- **On System Error**: Generic error; diagnostics stay server-side (SEC-ERR-1).
- **Logging / Audit**: No audit entry is required — plan content is not the subscriber's health data. Errors are logged with a correlation identifier.
- **Alerting**: N/A — a read-only published-content view defines no alert condition of its own; authorization denials are SEC-LOG-4 events logged upstream and route to the security lead as SEC-OPS-2 detection inputs.

## Test Strategy

- **Unit Tests**: The listing query includes the published predicate; the response serializer emits only the declared fields; the detail assembler includes citations.
- **Integration Tests**: Browse and view as a subscriber with a mix of published and unpublished plans seeded; assert listing contents, detail contents, and citation presence.
- **Security Tests**: Direct retrieval of an unpublished plan; filter and pagination parameter manipulation attempting to surface unpublished plans; response-shape assertion against the persisted entity; stored-markup rendering asserted safe in conjunction with REQ-CATALOG-030.
- **Compliance Tests / Evidence**: Accessibility evidence for the plan detail view at 320px and 200% zoom.
- **Acceptance-Criteria Traceability**: AC-01 — browse and view suite; AC-02 — unpublished access suite; AC-03 — parameter manipulation and response-shape tests; AC-04 — empty state test; AC-05 — reflow and typography tests.
- **Coverage Target**: Published and unpublished states × listing and detail operations; the empty case.
- **Required Test Environment**: A subscriber identity with an active subscription period, seeded catalog entries (FR-5.4), and seeded published and unpublished exercise plans with citations. Runs against PostgreSQL on Vitest.

## Dependencies

- **Upstream Requirements**: REQ-PLAN-010, REQ-PLAN-050, REQ-PLAN-080, REQ-ENTITLE-010, REQ-AUTHZ-010, REQ-PRIVACY-060, REQ-PLATFORM-020, REQ-PLATFORM-030, REQ-CATALOG-030
- **Downstream Requirements**: REQ-CUSTOM-010, REQ-PROGRESS-020, REQ-SELECT-010
- **External Dependencies**: None
- **Dependency Assumptions**: Citations are retrievable with the plan (REQ-PLAN-040).
- **Failure Impact**: A leak of unpublished plans exposes unverified health guidance — content that has deliberately not passed the FR-4.4 and FR-4.5 gates.

## Implementation Notes

- **Constraints**: PostgreSQL with Drizzle ORM (`CLAUDE.md`); the client is a Vite-built single-page application with `vue-router`. Subscription entitlement (FR-3.1, FR-3.2; `REQUIREMENTS.md` OQ-1 RESOLVED) is enforced at this boundary by REQ-ENTITLE-010's gate — this issue delivers the browse and view surface behind it and MUST NOT introduce a second, competing entitlement check (SEC-AUTHZ-5).
- **Prohibited Approaches**: Returning all plans and filtering in the client; a publication filter applied after retrieval; exposing the verification record's internal fields beyond what the view needs; using `v-html` or any raw HTML binding for plan content (SEC-RENDER-1, enforced in REQ-CATALOG-030).
- **Implementation Guidance**: Reuse the published-only query predicate established in REQ-PLAN-050 rather than writing a second one, so unpublication takes effect everywhere at once.
- **AI Development Guidance**: `REF-PROMPT-VUE`, `REF-PROMPT-API`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Design review against `DESIGN.md`; accessibility review of the plan detail view.
- **Open Decisions**: None — `REQUIREMENTS.md` OQ-1 (entitlement, gated by REQ-ENTITLE-010), OQ-6 (one active selection per type, REQ-SELECT-010), and `DESIGN.md` OQ-5 (citation and verification presentation) are all resolved; FR-4.6 is satisfied against the fixed Evidence-section pattern.

**Estimated effort**: 1–2 engineer-days. **Estimated changed lines**: 350–750.
**Recommended model**: Claude Fable (`claude-fable-5`) — spans server query, response shaping, and an accessible responsive client view, where consistency with the diet plan counterpart matters.
