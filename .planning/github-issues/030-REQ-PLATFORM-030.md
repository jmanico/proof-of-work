# [REQ-PLATFORM-030] Keyboard operability, focus management, and reduced motion baseline

## Metadata

- **ID**: REQ-PLATFORM-030
- **Title**: Keyboard operability, focus management, and reduced motion baseline
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `DESIGN.md` Accessibility, Components (Focus states); `ARCHITECTURE.md` Browser Client responsibility

## Requirement

- **Statement**: The Browser Client MUST make every function operable by keyboard in an order matching the visual reading order, MUST render a visible focus indicator on every interactive element, MUST provide a skip link ahead of repeated navigation, and MUST honor `prefers-reduced-motion` by removing non-essential motion.
- **Rationale**: `DESIGN.md` sets WCAG 2.2 AA as the conformance target — which `ARCHITECTURE.md` records as resolving `REQUIREMENTS.md` OQ-11 — and specifies the focus indicator, keyboard, skip-link, and reduced-motion behavior exactly.
- **Assumptions**: The client is the only user-facing surface.
- **Out of Scope**: Per-component markup and labelling, which each feature issue owns; screen-reader announcement of validation errors, owned by REQ-PROGRESS-030; contrast, owned by REQ-PLATFORM-010.
- **Design Traceability**: `DESIGN.md` — Accessibility (target conformance WCAG 2.2 AA, keyboard, structure, reduced motion, zoom and reflow, text sizing); Components → Focus states (2px `primary` outline offset 2px, at least 3:1 against its background, never suppressed without an equivalent replacement, focus order follows visual reading order).
- **Architecture Traceability**: `ARCHITECTURE.md` — Browser Client responsibility ("visible focus indicators, keyboard operability, … reduced-motion handling, and 200%-zoom/320px reflow").
- **Security Traceability**: SEC-RENDER-4 — accessibility affordances MUST NOT expose data the actor is not authorized to see.

## Scope

- **Applies To**: Web Client
- **Components**: Browser Client (Vue.js)
- **Interfaces / Operations**: Application shell, navigation landmarks, all focusable controls, all dialogs and content that opens, closes, or replaces itself
- **Actors**: `subscriber`, `consultant`, `admin`, unauthenticated visitor
- **Preconditions**: REQ-PLATFORM-010 and REQ-PLATFORM-020 complete
- **Data Classification**: Public
- **Personal or Regulated Data**: None directly; announcements may reference the acting user's own data
- **Jurisdiction / Regulatory Scope**: N/A

## Security Context

- **Security Objectives**: Confidentiality — only insofar as SEC-RENDER-4 bounds what announcements may contain
- **Control Layers**: Output Encoding, Other — accessibility
- **Threat References**: N/A — no threat-model entry addresses accessibility affordances beyond SEC-RENDER-4's constraint.
- **Abuse / Misuse Case**: A status or focus announcement discloses data belonging to another actor, or discloses internal detail contrary to SEC-ERR-1.
- **Trust Boundary**: Browser Client → user; no server boundary is crossed by this issue.
- **Untrusted Inputs or Assertions**: Server-supplied message text rendered into live regions.
- **Authoritative Enforcement Point**: REST API Application decides what data may be returned (SEC-DATA-5); the client only renders what it received.
- **Independent Verification**: N/A — the client performs no authorization.
- **Zero Trust Relevance**: N/A

## Standards Alignment

- **OWASP ASVS 5.0.0**: N/A
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: N/A
- **NIST SP 800-207**: N/A
- **Regulatory**: TO BE DECIDED — accessibility statutes are not named in any source document; `DESIGN.md` states WCAG 2.2 AA as a product target.
- **Other**: WCAG 2.2 AA, as adopted by `DESIGN.md` Accessibility.
- **Mapping Basis**: `DESIGN.md` explicitly names WCAG 2.2 AA; `ARCHITECTURE.md` records that this resolves `REQUIREMENTS.md` OQ-11. No statutory mapping is asserted because none is documented.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given any page, when the user traverses it with the keyboard alone, then every interactive element is reachable in visual reading order, every focused element shows a 2px `primary` outline offset 2px meeting at least 3:1 against its background, and every function can be completed without a pointer.
2. **AC-02 — Boundary or failure behavior**: Given content that opens, closes, or replaces itself, when the change occurs, then focus is moved deliberately to the new content and returned to the invoking control on close, and no keyboard trap exists at any point.
3. **AC-03 — Prohibited behavior**: Given any interactive element, when it receives focus, then the focus indicator MUST NOT be suppressed without an equivalent visible replacement, and no status or error announcement may contain data the actor is not authorized to see or internal diagnostic detail (SEC-RENDER-4, SEC-ERR-1).
4. **AC-04 — Additional criterion**: Given `prefers-reduced-motion: reduce`, when any view renders or transitions, then non-essential transitions, parallax, and auto-playing motion are removed and only instantaneous state change or a plain opacity fade remains; no task requires motion to understand or complete.
5. **AC-05 — Additional criterion**: Given the application shell, when the first Tab is pressed on any page, then a skip link precedes the repeated navigation and moves focus to the main landmark when activated.

## Failure Behavior

- **On Invalid Input**: N/A
- **On Authentication Failure**: N/A
- **On Authorization Failure**: N/A
- **On Security-Decision Failure**: N/A
- **On External Dependency Failure**: N/A
- **On System Error**: The client renders the generic error text supplied by the server (SEC-ERR-1) into a live region without exposing correlation-only internal detail beyond the correlation identifier.
- **Logging / Audit**: N/A — client-side accessibility behavior produces no audit event.
- **Alerting**: N/A

## Test Strategy

- **Unit Tests**: Focus-management helper moves and restores focus for open/close/replace transitions; reduced-motion helper reports the media-query state.
- **Integration Tests**: Keyboard traversal of the shell and a representative form asserting order, reachability, and absence of traps; skip-link activation moves focus to main.
- **Security Tests**: Assertion that live-region content is limited to server-supplied user-facing text and contains no stack trace, internal identifier, or foreign-subject data.
- **Compliance Tests / Evidence**: Automated accessibility scan plus a manual keyboard-only walkthrough recorded as WCAG 2.2 AA evidence.
- **Acceptance-Criteria Traceability**: AC-01 — keyboard traversal suite; AC-02 — focus-management suite; AC-03 — focus-indicator lint plus announcement-content assertion; AC-04 — reduced-motion suite; AC-05 — skip-link test.
- **Coverage Target**: Every shell control and every focus transition type exercised.
- **Required Test Environment**: Client test runner, accessibility scanner, and browser automation — all TO BE DECIDED (`CLAUDE.md`).

## Dependencies

- **Upstream Requirements**: REQ-PLATFORM-010, REQ-PLATFORM-020
- **Downstream Requirements**: REQ-CATALOG-010, REQ-CATALOG-020, REQ-CATALOG-030, REQ-PROGRESS-030, REQ-PRIVACY-030, REQ-PRIVACY-040
- **External Dependencies**: None
- **Dependency Assumptions**: None
- **Failure Impact**: N/A

## Implementation Notes

- **Constraints**: Vue.js client. Client tooling TO BE DECIDED.
- **Prohibited Approaches**: Removing the default focus ring without replacement; positive tab indices to fake an order; motion as the only signal of a state change; announcing raw server diagnostics.
- **Implementation Guidance**: Correct heading hierarchy, landmark regions, and labelled controls are part of this baseline (`DESIGN.md`, Accessibility — Structure). The logo is decorative wherever adjacent text already names the product.
- **AI Development Guidance**: `REF-PROMPT-VUE`; `CLAUDE.md`.
- **Required Human Review**: Accessibility review, including a manual assistive-technology pass.
- **Open Decisions**: None blocking. `DESIGN.md` OQ-1 (product name) affects the logo's text alternative only.

**Estimated effort**: 1–2 engineer-days. **Estimated changed lines**: 300–650.
**Recommended model**: Claude Opus (`claude-opus-5`) — bounded work with precise, verifiable rules and a security constraint (SEC-RENDER-4) on announcement content that rewards careful reasoning over autonomy.
