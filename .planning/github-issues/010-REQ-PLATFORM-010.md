# [REQ-PLATFORM-010] Design tokens for color, typography, and spacing

## Metadata

- **ID**: REQ-PLATFORM-010
- **Title**: Design tokens for color, typography, and spacing
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `DESIGN.md` Color Palette, Typography, Layout and Spacing

## Requirement

- **Statement**: The Browser Client MUST expose the `DESIGN.md` palette, type scale, and spacing scale as a single set of named tokens, and all client styling MUST consume those tokens rather than literal color, font-size, or spacing values.
- **Rationale**: `DESIGN.md` fixes exact hex values, a 1.25 type scale on a 16px base, and a 4px spacing scale, and states that new hues must not be introduced without extending the palette table. A single token source is the only way that constraint is enforceable in review.
- **Assumptions**: The Vue.js client is the only rendering surface (`ARCHITECTURE.md`, Browser Client).
- **Out of Scope**: Dark mode (`DESIGN.md` OQ-3); additional tints and shades (`DESIGN.md`, "Additional tints/shades are TO BE DECIDED"); brand typography (`DESIGN.md`, Typography — UNKNOWN); component markup.
- **Design Traceability**: `DESIGN.md` — Color Palette (all seven tokens and their contrast values); Typography (system UI stack, fallback, 1.25 scale xs…2xl, weights 400/500/700, line heights 1.5 / 1.25, tabular figures for numeric data); Layout and Spacing (4, 8, 12, 16, 24, 32, 48, 64).
- **Architecture Traceability**: `ARCHITECTURE.md` — Browser Client; DR-1.
- **Security Traceability**: N/A — no security rule governs token definition. Contrast obligations are accessibility, covered by AC-02 and REQ-PLATFORM-030.

## Scope

- **Applies To**: Web Client
- **Components**: Browser Client (Vue.js)
- **Interfaces / Operations**: All rendered views; no API operation
- **Actors**: `subscriber`, `consultant`, `admin`, unauthenticated visitor
- **Preconditions**: None
- **Data Classification**: Public
- **Personal or Regulated Data**: None
- **Jurisdiction / Regulatory Scope**: N/A

## Security Context

- **Security Objectives**: N/A
- **Control Layers**: Other — presentation foundation
- **Threat References**: N/A
- **Abuse / Misuse Case**: N/A
- **Trust Boundary**: N/A — tokens carry no data and cross no boundary.
- **Untrusted Inputs or Assertions**: None
- **Authoritative Enforcement Point**: N/A
- **Independent Verification**: N/A
- **Zero Trust Relevance**: N/A

## Standards Alignment

- **OWASP ASVS 5.0.0**: N/A — no ASVS requirement governs design tokens.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: N/A
- **NIST SP 800-207**: N/A
- **Regulatory**: N/A
- **Other**: WCAG 2.2 AA contrast thresholds as restated in `DESIGN.md` Accessibility (4.5:1 body text, 3:1 large text and UI boundaries).
- **Mapping Basis**: `DESIGN.md` states WCAG 2.2 AA as the conformance target and publishes the contrast ratio of every palette token; the token set is the artifact that makes those ratios auditable.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given the token set, when a view renders body copy, then it resolves `text` on `background` or `text` on `surface`, and the computed contrast ratio is at least 4.5:1.
2. **AC-02 — Boundary or failure behavior**: Given an automated contrast check over every documented pairing (`primary`, `secondary`, `error`, `success` as text on `background` and on `surface`; white text on each as a background), when the check runs, then every pairing meets or exceeds 4.5:1 and the check fails the build if any does not.
3. **AC-03 — Prohibited behavior**: Given a lint rule over client styles, when a literal hex color, a font-size outside the xs…2xl scale, or a spacing value outside {4, 8, 12, 16, 24, 32, 48, 64} appears, then the lint rule reports an error and the value is not merged.
4. **AC-04 — Additional criterion**: Given a view that renders logged numeric values (weight, measurements, sets, reps, calories, macros), when it renders, then the typeface features requested include tabular/lining figures so values align in columns.

## Failure Behavior

- **On Invalid Input**: N/A — no runtime input.
- **On Authentication Failure**: N/A
- **On Authorization Failure**: N/A
- **On Security-Decision Failure**: N/A
- **On External Dependency Failure**: N/A — the system font stack is local to the browser and downloads nothing (`DESIGN.md`, Typography).
- **On System Error**: N/A
- **Logging / Audit**: N/A
- **Alerting**: N/A

## Test Strategy

- **Unit Tests**: Token module exports exactly the seven documented color tokens, six type steps, three weights, two line heights, and eight spacing steps, with the documented values.
- **Integration Tests**: Rendered snapshot of a representative view resolves tokens rather than literals.
- **Security Tests**: N/A
- **Compliance Tests / Evidence**: Automated contrast report for all documented pairings, retained as accessibility evidence for the WCAG 2.2 AA target.
- **Acceptance-Criteria Traceability**: AC-01 and AC-02 — contrast test suite; AC-03 — style lint rule with a deliberate failing fixture; AC-04 — rendered numeric table test.
- **Coverage Target**: Every documented token and every documented pairing exercised.
- **Required Test Environment**: Client build and test runner — TO BE DECIDED (`CLAUDE.md`, Repository state).

## Dependencies

- **Upstream Requirements**: None
- **Downstream Requirements**: REQ-PLATFORM-020, REQ-PLATFORM-030, REQ-CATALOG-010, REQ-CATALOG-020, REQ-CATALOG-030, REQ-PROGRESS-030, REQ-PRIVACY-040
- **External Dependencies**: None
- **Dependency Assumptions**: None
- **Failure Impact**: N/A

## Implementation Notes

- **Constraints**: Vue.js client (`ARCHITECTURE.md`). Client build tooling, styling approach, and directory layout are TO BE DECIDED (`CLAUDE.md`) and MUST NOT be selected within this issue without an explicit decision recorded there.
- **Prohibited Approaches**: Introducing hues, weights, sizes, or spacing steps absent from `DESIGN.md`; encoding meaning in color alone (`DESIGN.md`, Accessibility); web font downloads while brand typography is UNKNOWN.
- **Implementation Guidance**: Name tokens exactly as `DESIGN.md` names them (`primary`, `secondary`, `background`, `surface`, `text`, `error`, `success`) so review against the table is mechanical.
- **AI Development Guidance**: `REF-PROMPT-VUE`; `CLAUDE.md`.
- **Required Human Review**: Design review against `DESIGN.md`; accessibility review of the contrast report.
- **Open Decisions**: `DESIGN.md` OQ-2 (brand personality), OQ-3 (dark mode); brand typography UNKNOWN. None block this issue: the interim system font stack and light-mode palette are documented decisions.

**Estimated effort**: 0.5–1 engineer-day. **Estimated changed lines**: 150–350.
**Recommended model**: Claude Opus (`claude-opus-5`) — a bounded, rule-dense translation of an exact specification table into code, where fidelity to documented values matters more than autonomy.
