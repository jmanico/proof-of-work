# [REQ-PLATFORM-020] Responsive layout and reflow without loss of function

## Metadata

- **ID**: REQ-PLATFORM-020
- **Title**: Responsive layout and reflow without loss of function
- **Version**: 1.1.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-1.1, FR-1.2

## Requirement

- **Statement**: The Browser Client MUST present a usable layout at desktop and mobile viewport sizes using the documented grid and breakpoints, and MUST reflow rather than truncate, so that no function or content is unavailable at small viewport widths.
- **Rationale**: FR-1.1 and FR-1.2 require delivery as a responsive web application with no loss of function or content on mobile; `DESIGN.md` fixes the grid, gutters, breakpoints, content widths, and touch-target minimum that make that testable.
- **Assumptions**: Current desktop and mobile browsers (FR-1.1). No native client (`ARCHITECTURE.md`, Runtime environment).
- **Out of Scope**: Offline use (`REQUIREMENTS.md` OQ-14 RESOLVED — confirmed out of scope); additional languages and active right-to-left support (`DESIGN.md` OQ-8 RESOLVED — deferred beyond v1; components stay logical-property ready and text-expansion tolerant); the per-role navigation destinations and workspace labels (`DESIGN.md` Information Architecture — owned by the feature and workspace issues).
- **Design Traceability**: `DESIGN.md` — Layout, Spacing, and Responsive Behavior (12-column grid at 64rem and above, 8 columns from 48rem, 4 columns below 48rem; gutters 24px desktop and 16px below 48rem; content widths 80rem application maximum, 68ch prose, 40rem single-column form, 32rem authentication; breakpoints 30rem, 48rem, 64rem, 80rem responding to content pressure; 44×44 CSS-pixel targets; one-column reflow at 320px and 200% zoom with no page-level horizontal scrolling; data tables become labelled record cards below 48rem; sticky regions leave room for content and the on-screen keyboard); Information Architecture and Navigation (15rem left rail at 64rem and above; compact top bar plus bottom navigation below 64rem, labels always visible); Accessibility (reflow, relative units).
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
- **Other**: WCAG 2.2 AA reflow and target-size expectations as restated in `DESIGN.md` Accessibility and Layout, Spacing, and Responsive Behavior.
- **Mapping Basis**: `DESIGN.md` sets WCAG 2.2 AA as the conformance target and states the reflow and touch-target values this issue implements; no external control catalog is asserted.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a viewport at or above 64rem, when any application page renders, then content lays out on the 12-column grid with a 24px gutter, application content is constrained to at most 80rem, prose to at most 68ch, and a single-column form to at most 40rem.
2. **AC-02 — Boundary or failure behavior**: Given a 320-CSS-pixel-wide viewport, and separately a desktop viewport at 200% zoom, when any application page renders, then content reflows into one column, the page produces no page-level horizontal scrolling, every interactive control remains reachable and operable, and no content present at desktop width is absent.
3. **AC-03 — Prohibited behavior**: Given any breakpoint, when a control is rendered, then it MUST NOT have a hit target smaller than 44×44 CSS pixels, and content MUST NOT be removed, clipped, or replaced by an ellipsis to fit a narrow viewport — data tables become labelled record cards below 48rem rather than forcing horizontal scrolling.
4. **AC-04 — Additional criterion**: Given a viewport below 48rem, when a page renders, then it uses the 4-column grid and a 16px gutter; given a viewport from 48rem up to 64rem, it uses the 8-column grid; and multi-column forms stack with action groups wrapping primary-action-first.

## Failure Behavior

- **On Invalid Input**: N/A
- **On Authentication Failure**: N/A
- **On Authorization Failure**: N/A
- **On Security-Decision Failure**: N/A
- **On External Dependency Failure**: N/A
- **On System Error**: N/A
- **Logging / Audit**: N/A
- **Alerting**: N/A — no runtime alert condition exists for layout behavior.

## Test Strategy

- **Unit Tests**: Breakpoint helper returns the documented column count and gutter for each range delimited by the 30rem, 48rem, 64rem, and 80rem breakpoints.
- **Integration Tests**: Rendered-layout tests at 320px and at the 30rem (480px), 48rem (768px), 64rem (1024px), and 80rem (1280px) breakpoints asserting column count, gutter, and maximum content widths.
- **Security Tests**: N/A
- **Compliance Tests / Evidence**: Reflow evidence at 320px and at 200% zoom, and a touch-target audit, retained for the WCAG 2.2 AA target.
- **Acceptance-Criteria Traceability**: AC-01 and AC-04 — breakpoint layout suite; AC-02 — reflow and zoom suite; AC-03 — target-size audit plus an overflow assertion and a record-card transformation test.
- **Coverage Target**: All four breakpoints, the 320px floor, and both reflow conditions exercised.
- **Required Test Environment**: Vitest for component tests; Playwright across viewports for the reflow assertions (`CLAUDE.md` — real-browser commitments are Playwright's responsibility).

## Dependencies

- **Upstream Requirements**: REQ-PLATFORM-010
- **Downstream Requirements**: REQ-PLATFORM-030 and every client-rendering issue (REQ-CATALOG-010, REQ-CATALOG-020, REQ-PROGRESS-030, REQ-PRIVACY-030, REQ-PRIVACY-040; REQ-PROGRESS-060, planned 2026-08-03)
- **External Dependencies**: None
- **Dependency Assumptions**: None
- **Failure Impact**: N/A

## Implementation Notes

- **Constraints**: Vue.js client written in TypeScript; responsive web only, no native application, built by Vite as a single-page application with `vue-router` (`CLAUDE.md`). Spacing uses only the documented steps {4, 8, 12, 16, 24, 32, 48, 64, 96} (REQ-PLATFORM-010).
- **Prohibited Approaches**: Serving a reduced mobile feature set; hiding content behind a viewport query; fixed pixel widths that defeat 200% zoom; suppressing reflow with horizontal scroll containers at the page level; icon-only primary navigation (`DESIGN.md`, Information Architecture — labels remain visible); sticky regions that crowd out content or the on-screen keyboard.
- **Implementation Guidance**: Relative units throughout so user font-size preferences are respected (`DESIGN.md`, Accessibility). Use logical layout properties and tolerate 30% text expansion (`DESIGN.md`, Required Design Inputs). Wide tabular data becomes labelled record cards below 48rem; where a wide element must scroll, it scrolls inside its own container so long as the page body does not. Breakpoints respond to content pressure, not named device models.
- **AI Development Guidance**: `REF-PROMPT-VUE`; `CLAUDE.md`.
- **Required Human Review**: Design and accessibility review.
- **Open Decisions**: None — `DESIGN.md` has no open design questions. Additional translations and active RTL support are deferred beyond v1 by decision (OQ-8 RESOLVED), not open.

**Estimated effort**: 1–2 engineer-days. **Estimated changed lines**: 300–700.
**Recommended model**: Claude Opus (`claude-opus-5`) — bounded implementation against exact numeric layout constraints with a demanding reflow test matrix.
