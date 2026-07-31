# [REQ-CATALOG-020] Browse and view published diet plans

## Metadata

- **ID**: REQ-CATALOG-020
- **Title**: Browse and view published diet plans
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-6.1, FR-6.2, FR-4.6, FR-4.7

## Requirement

- **Statement**: A subscriber MUST be able to browse published diet plans and view a plan's full contents, including its meals and its daily calorie and macronutrient targets, together with the plan's citations; only published plans MUST be shown.
- **Rationale**: FR-6.1 states the browse and view behavior; FR-6.2 fixes the content that must be visible; FR-4.6 requires citations to be displayed when a plan is viewed; FR-4.7 restricts subscriber visibility to published plans.
- **Assumptions**: The diet plan model and publication state exist (REQ-PLAN-020, REQ-PLAN-050).
- **Out of Scope**: Selecting a diet plan to follow (FR-6.3), blocked by `REQUIREMENTS.md` OQ-6; comparing logged intake against these targets (FR-8.5), blocked by OQ-5; subscription entitlement (FR-3.1, FR-3.2), blocked by OQ-1; safe rendering mechanics (REQ-CATALOG-030); citation and verification presentation (`DESIGN.md` OQ-5).
- **Design Traceability**: `DESIGN.md` — Typography (tabular/lining figures so calorie and macro values align in columns; xs step for citation attribution); Layout and Spacing (1200px application width, 72ch long-form, mobile reflow); Components → Links, Empty states; Accessibility.
- **Architecture Traceability**: `ARCHITECTURE.md` — data flow 2; Browser Client; REST API Application; FR-6.1–FR-6.3 traceability row.
- **Security Traceability**: SEC-AUTHN-1, SEC-AUTHZ-1, SEC-DATA-5, SEC-RENDER-1, SEC-RENDER-3, SEC-HTTP-2.

## Scope

- **Applies To**: Server-Side Application, API, Web Client
- **Components**: REST API Application; Relational Persistence; Browser Client (diet plan library and detail views)
- **Interfaces / Operations**: List published diet plans; retrieve one published diet plan with its meals, targets, and citations
- **Actors**: `subscriber`; `consultant` and `admin` as authenticated readers
- **Preconditions**: Authenticated session
- **Data Classification**: Internal
- **Personal or Regulated Data**: None in the plan content
- **Jurisdiction / Regulatory Scope**: N/A

## Security Context

- **Security Objectives**: Confidentiality, Integrity, Availability, Safety
- **Control Layers**: Authorization, Output Encoding, Data Protection
- **Threat References**: `SECURITY.md` TM-T-4 (stored XSS via admin-authored content), TM-I-3 (excessive data exposure), TM-T-5 (unverified dietary guidance reaching readers); CWE-79, CWE-200
- **Abuse / Misuse Case**: An unpublished diet plan with unreviewed calorie targets becomes reachable by identifier; or markup stored in a meal description executes in the reader's browser.
- **Trust Boundary**: Boundary 1.
- **Untrusted Inputs or Assertions**: Listing and pagination parameters; the plan identifier; stored plan content at render time.
- **Authoritative Enforcement Point**: REST API Application — published-only filtering in the query.
- **Independent Verification**: The publication predicate is part of the query, not a post-retrieval filter.
- **Zero Trust Relevance**: N/A

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: N/A
- **Regulatory**: N/A
- **Other**: WCAG 2.2 AA as adopted by `DESIGN.md`.
- **Mapping Basis**: FR-6.1, FR-6.2, FR-4.6, and FR-4.7 are the normative sources; `DESIGN.md` supplies the presentation obligations.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated subscriber, when they browse the diet plan library, then only published diet plans are listed, and when they open one, its meals, daily calorie target, daily macronutrient targets, and citations are displayed.
2. **AC-02 — Boundary or failure behavior**: Given an unpublished diet plan, when a subscriber requests it directly by identifier, then the response is indistinguishable from that for a plan that does not exist, and it never appears in a listing under any parameter combination.
3. **AC-03 — Prohibited behavior**: Given the diet plan library, when it is served, then it MUST NOT return unpublished plans, MUST NOT return fields beyond those the view requires (SEC-DATA-5), and the client MUST NOT be responsible for filtering unpublished plans (DR-2).
4. **AC-04 — Additional criterion**: Given calorie and macronutrient targets, when they render, then each is labelled with its unit and rendered with tabular figures so values align down a list (`DESIGN.md`, Typography).
5. **AC-05 — Additional criterion**: Given the diet plan detail view, when it renders at a 320px viewport and at 200% zoom, then meals and targets reflow without horizontal page scrolling and without loss of content.

## Failure Behavior

- **On Invalid Input**: Reject malformed listing or identifier parameters per REQ-API-010 with field-level detail.
- **On Authentication Failure**: Denied upstream by REQ-AUTHZ-010.
- **On Authorization Failure**: Deny without disclosing plan existence (REQ-AUTHZ-040).
- **On Security-Decision Failure**: If publication state cannot be determined, exclude the plan (fail closed).
- **On External Dependency Failure**: Generic error with a correlation identifier; the client shows an error state rather than partial targets, since a partially rendered dietary target is misleading.
- **On System Error**: Generic error; diagnostics stay server-side.
- **Logging / Audit**: No audit entry required — plan content is not the subscriber's health data.
- **Alerting**: N/A

## Test Strategy

- **Unit Tests**: The listing query includes the published predicate; the detail assembler includes meals, all target fields, and citations; the serializer emits only declared fields.
- **Integration Tests**: Browse and view as a subscriber with published and unpublished diet plans seeded; assert full target set present and correct.
- **Security Tests**: Direct retrieval of an unpublished plan; parameter manipulation attempting to surface unpublished plans; response-shape assertion; stored-markup rendering asserted safe with REQ-CATALOG-030.
- **Compliance Tests / Evidence**: Accessibility evidence for the detail view at 320px and 200% zoom.
- **Acceptance-Criteria Traceability**: AC-01 — browse and view suite; AC-02 — unpublished access suite; AC-03 — parameter and shape tests; AC-04 — unit labelling and typography tests; AC-05 — reflow tests.
- **Coverage Target**: Published and unpublished states × listing and detail; the empty case; every target field rendered.
- **Required Test Environment**: A subscriber identity, seeded published and unpublished diet plans with citations and full target sets. Engine and test framework TO BE DECIDED.

## Dependencies

- **Upstream Requirements**: REQ-PLAN-020, REQ-PLAN-050, REQ-AUTHZ-010, REQ-PRIVACY-060, REQ-PLATFORM-020, REQ-PLATFORM-030, REQ-CATALOG-030
- **Downstream Requirements**: REQ-CUSTOM-010
- **External Dependencies**: None — no nutrition database is in scope (`REQUIREMENTS.md`, External integrations; OQ-5).
- **Dependency Assumptions**: Citations are retrievable with the plan (REQ-PLAN-040); targets are modelled as distinct numeric fields (REQ-PLAN-020).
- **Failure Impact**: Exposing unpublished diet plans surfaces calorie and macronutrient prescriptions that have not passed the citation and verification gates — the dietary analogue of TM-T-5.

## Implementation Notes

- **Constraints**: RDBMS engine and client tooling TO BE DECIDED. Subscription entitlement is blocked by `REQUIREMENTS.md` OQ-1, so this surface is gated by authentication only; that gap must be stated when the issue is closed.
- **Prohibited Approaches**: Client-side filtering of unpublished plans; deriving displayed targets by recomputation rather than showing the authored values; rendering meal content through a raw HTML binding (SEC-RENDER-1); rounding or reformatting target values in a way that changes their meaning.
- **Implementation Guidance**: Share the published-only query predicate with REQ-CATALOG-010 and REQ-PLAN-050. Present targets as authored, with units, so that the later FR-8.5 comparison has an unambiguous reference once OQ-5 resolves.
- **AI Development Guidance**: `REF-PROMPT-VUE`, `REF-PROMPT-API`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Design review; accessibility review; product review that the displayed target set matches FR-6.2.
- **Open Decisions**: `REQUIREMENTS.md` OQ-1 (entitlement), OQ-5 (nutrition source, which blocks the comparison this view's targets feed), OQ-6 (plan selection), OQ-4 and `DESIGN.md` OQ-8 (unit system); `DESIGN.md` OQ-5 (citation and verification presentation).

**Estimated effort**: 1–2 engineer-days. **Estimated changed lines**: 350–700.
**Recommended model**: Claude Fable (`claude-fable-5`) — mirrors the exercise catalog across server and client; consistency between the two is the main risk.
