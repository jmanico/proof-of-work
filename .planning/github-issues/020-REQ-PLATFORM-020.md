# [REQ-PLATFORM-020] Responsive layout and reflow without loss of function

## Metadata

- **ID**: REQ-PLATFORM-020
- **Title**: Responsive layout and reflow without loss of function
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-1.1, FR-1.2

## Requirement

- **Statement**: The Browser Client MUST present a usable layout at desktop and mobile viewport sizes using the documented grid and breakpoints, and MUST reflow rather than truncate, so that no function or content is unavailable at small viewport widths.
- **Rationale**: FR-1.1 and FR-1.2 require delivery as a responsive web application with no loss of function or content on mobile; `DESIGN.md` fixes the grid, gutters, breakpoints, content widths, and touch-target minimum that make that testable.
- **Assumptions**: Current desktop and mobile browsers (FR-1.1). No native client (`ARCHITECTURE.md`, Runtime environment).
- **Out of Scope**: Offline use (`REQUIREMENTS.md` OQ-14, out of scope); localization and right-to-left layout (`DESIGN.md` OQ-8); client-side rendering/routing strategy (`ARCHITECTURE.md`, Browser Client open decisions).
- **Design Traceability**: `DESIGN.md` — Layout and Spacing (12-column desktop / 4-column mobile grid, 24px gutter reducing to 16px below `md`, max 1200px application width, max 72ch long-form width, breakpoints `sm` 480 / `md` 768 / `lg` 1024 / `xl` 1280, single column below `md`, 44×44px touch targets, reflow not truncation); Accessibility (200% zoom and 320px viewport without horizontal scrolling or loss of content).
- **Architecture Traceability**: `ARCHITECTURE.md` — Browser Client responsibility ("responsive reflow with no loss of function on mobile (FR-1.2)"); DR-1.
- **Security Traceability**: N/A

## Scope

- **Applies To**: Web Client
- **Components**: Browser Client (Vue.js)
- **Interfaces / Operations**: The application layout shell — page container, grid, navigation region, main region
- **Actors**: `subscriber`, `consultant`, `admin`, unauthenticated visitor
- **Preconditions**: REQ-PLATFORM-010 tokens exist
- **Data Classification**: Public
- **Personal or Regulated Data**: None
- **Jurisdiction / Regulatory Scope**: N/A

## Security Context

- **Security Objectives**: N/A
- **Control Layers**: Other — presentation foundation
- **Threat References**: N/A
- **Abuse / Misuse Case**: N/A
- **Trust Boundary**: N/A
- **Untrusted Inputs or Assertions**: None
- **Authoritative Enforcement Point**: N/A
- **Independent Verification**: N/A
- **Zero Trust Relevance**: N/A

## Standards Alignment

- **OWASP ASVS 5.0.0**: N/A
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: N/A
- **NIST SP 800-207**: N/A
- **Regulatory**: N/A
- **Other**: WCAG 2.2 AA reflow and target-size expectations as restated in `DESIGN.md` Accessibility and Layout and Spacing.
- **Mapping Basis**: `DESIGN.md` sets WCAG 2.2 AA as the conformance target and states the reflow and touch-target values this issue implements; no external control catalog is asserted.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a viewport at or above `md` (768px), when any application page renders, then content lays out on the 12-column grid with a 24px gutter, constrained to at most 1200px, and long-form regions to at most 72ch.
2. **AC-02 — Boundary or failure behavior**: Given a 320px-wide viewport, and separately a 1280px viewport at 200% zoom, when any application page renders, then the page produces no horizontal scrolling, every interactive control remains reachable and operable, and no content present at desktop width is absent.
3. **AC-03 — Prohibited behavior**: Given any breakpoint, when a control is rendered, then it MUST NOT have a hit target smaller than 44×44px, and content MUST NOT be removed, clipped, or replaced by an ellipsis to fit a narrow viewport.
4. **AC-04 — Additional criterion**: Given a viewport below `md`, when a page renders, then it uses the 4-column grid, a 16px gutter, and a single-column flow.

## Failure Behavior

- **On Invalid Input**: N/A
- **On Authentication Failure**: N/A
- **On Authorization Failure**: N/A
- **On Security-Decision Failure**: N/A
- **On External Dependency Failure**: N/A
- **On System Error**: N/A
- **Logging / Audit**: N/A
- **Alerting**: N/A

## Test Strategy

- **Unit Tests**: Breakpoint helper returns the documented column count and gutter for each of the five ranges.
- **Integration Tests**: Rendered-layout tests at 320, 480, 768, 1024, and 1280px asserting column count, gutter, and maximum content width.
- **Security Tests**: N/A
- **Compliance Tests / Evidence**: Reflow evidence at 320px and at 200% zoom, and a touch-target audit, retained for the WCAG 2.2 AA target.
- **Acceptance-Criteria Traceability**: AC-01 and AC-04 — breakpoint layout suite; AC-02 — reflow and zoom suite; AC-03 — target-size audit plus an overflow assertion.
- **Coverage Target**: All five breakpoints and both reflow conditions exercised.
- **Required Test Environment**: Client test runner and a browser automation target — TO BE DECIDED (`CLAUDE.md`).

## Dependencies

- **Upstream Requirements**: REQ-PLATFORM-010
- **Downstream Requirements**: REQ-PLATFORM-030 and every client-rendering issue (REQ-CATALOG-010, REQ-CATALOG-020, REQ-PROGRESS-030, REQ-PRIVACY-030, REQ-PRIVACY-040)
- **External Dependencies**: None
- **Dependency Assumptions**: None
- **Failure Impact**: N/A

## Implementation Notes

- **Constraints**: Vue.js client; responsive web only, no native application. Client build tooling and routing strategy TO BE DECIDED.
- **Prohibited Approaches**: Serving a reduced mobile feature set; hiding content behind a viewport query; fixed pixel widths that defeat 200% zoom; suppressing reflow with horizontal scroll containers at the page level.
- **Implementation Guidance**: Relative units throughout so user font-size preferences are respected (`DESIGN.md`, Accessibility — Text sizing). Wide tabular data may scroll inside its own container so long as the page body does not.
- **AI Development Guidance**: `REF-PROMPT-VUE`; `CLAUDE.md`.
- **Required Human Review**: Design and accessibility review.
- **Open Decisions**: `DESIGN.md` OQ-8 (localization, RTL, unit system) may later alter numeric formatting and layout; it does not block the grid and reflow behavior specified here.

**Estimated effort**: 1–2 engineer-days. **Estimated changed lines**: 300–700.
**Recommended model**: Claude Opus (`claude-opus-5`) — bounded implementation against exact numeric layout constraints with a demanding reflow test matrix.
