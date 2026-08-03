# [REQ-PLATFORM-010] Design tokens for color, typography, and spacing

## Metadata

- **ID**: REQ-PLATFORM-010
- **Title**: Design tokens for color, typography, and spacing
- **Version**: 1.1.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `DESIGN.md` Color System, Typography and Iconography, Layout, Spacing, and Responsive Behavior

## Requirement

- **Statement**: The Browser Client MUST expose the `DESIGN.md` semantic color tokens with their paired light and dark theme values, the two font stacks and seven-step type scale, and the spacing scale as a single set of named tokens, and all client styling MUST consume those tokens rather than literal color, font-size, or spacing values.
- **Rationale**: `DESIGN.md` fixes nineteen semantic color tokens with exact paired light and dark hex values, a fluid type ramp on two local system font stacks, and a nine-step spacing scale, and states that all interface color is assigned by semantic token and components MUST NOT reference raw palette values. A single token source is the only way that constraint is enforceable in review.
- **Assumptions**: The Vue.js client is the only rendering surface (`ARCHITECTURE.md`, Browser Client).
- **Out of Scope**: Component markup; the user-facing appearance preference control (`Light`/`Dark`/`System`, initial `System` — an Account feature, not a token concern); chart-specific application of the emphasis rules, owned by the progress-display issue (REQ-PROGRESS-060, planned 2026-08-03); the focus-outline behavior itself, owned by REQ-PLATFORM-030 (it consumes the `brand` token defined here).
- **Design Traceability**: `DESIGN.md` — Color System (all nineteen semantic tokens with paired light/dark values; the verified core contrast pairs; the color rules, including the 2026-08-03 `signal` restriction on light surfaces); Typography and Iconography (`--font-sans` and `--font-data` local system stacks; `display`, `h1`, `h2`, `h3`, `body`, `small`, `micro` with their fluid sizes, line heights, and weights; lining tabular figures for numeric data); Layout, Spacing, and Responsive Behavior (spacing scale 4, 8, 12, 16, 24, 32, 48, 64, 96); Required Design Inputs (light and dark modes with `System` as the initial preference).
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
- **Other**: WCAG 2.2 AA contrast thresholds as restated in `DESIGN.md` Accessibility (4.5:1 text except qualifying large text; 3:1 meaningful graphics and control boundaries).
- **Mapping Basis**: `DESIGN.md` states WCAG 2.2 AA as the conformance target and publishes verified contrast ratios for its core token pairs; the token set is the artifact that makes those ratios auditable.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given the token set, when a view renders body copy, then it resolves `ink` on `canvas` or `ink` on `surface` in the active theme, and the computed contrast ratio is at least 4.5:1 in both light and dark modes (documented: 15.01:1 light, 16.66:1 dark for `ink` on `canvas`).
2. **AC-02 — Boundary or failure behavior**: Given an automated contrast check over every documented token pairing in both light and dark modes — including the verified core pairs (`ink` and `ink-muted` on `canvas`; white on light `brand`; dark `canvas` on dark `brand`; `control-border` against its adjacent surface) and every status-on-soft-background pairing actually used — when the check runs, then every text pairing meets or exceeds 4.5:1 (except qualifying large text at 3:1), every meaningful-graphics and control-boundary pairing meets or exceeds 3:1, and the check fails the build if any does not.
3. **AC-03 — Prohibited behavior**: Given a lint rule over client styles, when a literal color value, a font size outside the `display`…`micro` scale, or a spacing value outside {4, 8, 12, 16, 24, 32, 48, 64, 96} appears, then the lint rule reports an error and the value is not merged.
4. **AC-04 — Additional criterion**: Given a view that renders logged numeric values (weight, measurements, sets, reps, calories, macros), when it renders, then it uses the data face (`--font-data`) with lining tabular figures so values align in columns, and the data face is not used for paragraphs or form labels.
5. **AC-05 — Additional criterion**: Given the `signal` token, when it is used, then it is paired with `ink` and never white, and in light mode it does not meet `canvas`, `surface`, or `surface-subtle` without a 2px `ink` boundary carrying the 3:1 graphics requirement — chart emphasis without such a boundary resolves `brand` instead (`DESIGN.md` Color System, 2026-08-03 rule).

## Failure Behavior

- **On Invalid Input**: N/A — no runtime input.
- **On Authentication Failure**: N/A
- **On Authorization Failure**: N/A
- **On Security-Decision Failure**: N/A
- **On External Dependency Failure**: N/A — both font stacks are local to the browser and download nothing (`DESIGN.md`, Typography and Iconography).
- **On System Error**: N/A
- **Logging / Audit**: N/A
- **Alerting**: N/A — no runtime alert condition exists for token definition.

## Test Strategy

- **Unit Tests**: Token module exports exactly the nineteen documented color tokens, each with its documented light and dark value, the two font stacks, the seven type-scale tokens with documented sizes, line heights, and weights, and the nine spacing steps — no more, no fewer.
- **Integration Tests**: Rendered snapshot of a representative view resolves tokens rather than literals in both themes; switching theme changes color only — spacing, hierarchy, border meaning, and elevation remain stable (`DESIGN.md` Color System).
- **Security Tests**: N/A
- **Compliance Tests / Evidence**: Automated contrast report for all documented pairings in both modes, retained as accessibility evidence for the WCAG 2.2 AA target.
- **Acceptance-Criteria Traceability**: AC-01 and AC-02 — contrast test suite over both themes; AC-03 — style lint rule with a deliberate failing fixture; AC-04 — rendered numeric table test; AC-05 — `signal` pairing assertions and a light-mode chart-emphasis fixture.
- **Coverage Target**: Every documented token and every documented pairing exercised in both light and dark modes.
- **Required Test Environment**: Vitest as the runner, against the Vite build (`CLAUDE.md`, Repository state).

## Dependencies

- **Upstream Requirements**: None
- **Downstream Requirements**: REQ-PLATFORM-020, REQ-PLATFORM-030, REQ-CATALOG-010, REQ-CATALOG-020, REQ-CATALOG-030, REQ-PROGRESS-030, REQ-PRIVACY-040; REQ-PROGRESS-060 (planned 2026-08-03 — trend-chart emphasis consumes the `signal`/`brand` rules)
- **External Dependencies**: None
- **Dependency Assumptions**: None
- **Failure Impact**: N/A

## Implementation Notes

- **Constraints**: Vue.js client written in TypeScript under `apps/web` (`CLAUDE.md`). Vite builds it as a single-page application with `vue-router`, and styling is plain CSS custom properties with scoped single-file-component styles, onto which the `DESIGN.md` token table maps directly. A CSS framework or component library MUST NOT be introduced within this issue.
- **Prohibited Approaches**: Introducing hues, weights, sizes, or spacing steps absent from `DESIGN.md`; referencing raw palette values from components; encoding meaning in color alone (`DESIGN.md`, Accessibility); pairing `signal` with white or placing it bare on light surfaces (AC-05); web font downloads — `DESIGN.md` fixes the local system stacks and no web font is required; exposing the light and dark values as separate component APIs rather than paired theme values.
- **Implementation Guidance**: Name tokens exactly as `DESIGN.md` names them (`canvas`, `surface`, `surface-subtle`, `ink`, `ink-muted`, `border`, `control-border`, `brand`, `brand-strong`, `brand-soft`, `signal`, `error`, `error-soft`, `success`, `success-soft`, `warning`, `warning-soft`, `info`, `info-soft`) so review against the table is mechanical. Dark values follow the viewer's mode per the `System` initial preference (`DESIGN.md`, Required Design Inputs); new combinations MUST be contrast-tested before use (`DESIGN.md`, Color System).
- **AI Development Guidance**: `REF-PROMPT-VUE`; `CLAUDE.md`.
- **Required Human Review**: Design review against `DESIGN.md`; accessibility review of the contrast report.
- **Open Decisions**: None — `DESIGN.md` has no open design questions (its OQ-1…OQ-9 are all RESOLVED); the palette, both themes, the type scale, and the spacing scale are fixed.

**Estimated effort**: 0.5–1 engineer-day. **Estimated changed lines**: 200–450.
**Recommended model**: Claude Opus (`claude-opus-5`) — a bounded, rule-dense translation of an exact specification table into code, where fidelity to documented values matters more than autonomy.
