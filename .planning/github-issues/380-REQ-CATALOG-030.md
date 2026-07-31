# [REQ-CATALOG-030] Safe rendering of plan content and citation links

## Metadata

- **ID**: REQ-CATALOG-030
- **Title**: Safe rendering of plan content and citation links
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `SECURITY.md` SEC-RENDER-1, SEC-RENDER-2, SEC-RENDER-3, SEC-RENDER-4; threat TM-T-4

## Requirement

- **Statement**: The Browser Client MUST render all dynamic content — plan content, citations, and subscriber-entered values — through the framework's contextual auto-escaping, MUST NOT use raw HTML injection interfaces for server- or user-originated content, MUST parse and scheme-check every data-derived URL before binding it to a link or resource attribute, and MUST open external links without granting opener access to the application window.
- **Rationale**: SEC-RENDER-1 forbids raw HTML binding for any server- or user-originated content including admin-authored plans; SEC-RENDER-3 requires scheme checking on data-derived URLs, naming citation links specifically; SEC-RENDER-4 forbids persisting health data or tokens in browser-local storage. The threat model rates stored XSS through admin-authored plan content, citation URLs, or subscriber-entered fields (TM-T-4) as high severity.
- **Assumptions**: Vue.js is the client framework (`ARCHITECTURE.md`), so contextual auto-escaping is available by default and the prohibited interface is the raw HTML binding.
- **Out of Scope**: The Content Security Policy, which is defense in depth delivered by REQ-PLATFORM-040; server-side citation URL validation (REQ-PLAN-040), which this issue duplicates deliberately at render time; whether rich-text plan content is required at all, which SEC-RENDER-2 marks `TO BE DECIDED`.
- **Design Traceability**: `DESIGN.md` — Components → Links ("Citation links on plan content are always visibly links"), Form feedback and errors; Accessibility (color independence, focus states on links).
- **Architecture Traceability**: `ARCHITECTURE.md` — Browser Client ("present plan content and citations"); trust boundary 1; DR-1.
- **Security Traceability**: SEC-RENDER-1, SEC-RENDER-2, SEC-RENDER-3, SEC-RENDER-4; supports SEC-HTTP-2, SEC-EXT-2.

## Scope

- **Applies To**: Web Client
- **Components**: Browser Client
- **Interfaces / Operations**: Every view that renders plan content, citations, customized copies, or logged entries
- **Actors**: `subscriber`, `consultant`, `admin` as readers
- **Preconditions**: Content has been retrieved from the REST API Application
- **Data Classification**: Restricted — the same views render health data alongside plan content
- **Personal or Regulated Data**: Health Data — logged values rendered in progress views
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED (`SECURITY.md` SQ-1)

## Security Context

- **Security Objectives**: Integrity, Confidentiality, Privacy
- **Control Layers**: Output Encoding, Sanitization, Data Protection
- **Threat References**: `SECURITY.md` TM-T-4 (stored XSS via admin-authored plan content, citation URLs, or subscriber-entered fields), TM-I-9 (health-data residue in browser storage), TM-S-5 (token theft via XSS); CWE-79 (cross-site scripting), CWE-83 (improper neutralization of script in attributes), CWE-1022 (use of web link to untrusted target with `window.opener`)
- **Abuse / Misuse Case**: A compromised admin stores markup in a plan description or a `javascript:` citation URL; it renders in every subscriber's browser, executes, and steals the session token — which, combined with SEC-SESSION-5's unresolved transport decision, is the most direct path to health data in the threat model.
- **Trust Boundary**: Boundary 1 — all content arriving from the server is untrusted at render time, regardless of which role authored it.
- **Untrusted Inputs or Assertions**: Every rendered string and every data-derived URL, including admin-authored content.
- **Authoritative Enforcement Point**: The Browser Client's rendering layer, backed by a lint rule that forbids raw HTML binding.
- **Independent Verification**: The scheme check is applied at render time even though REQ-PLAN-040 also applies it at storage time, so a record stored before a rule change cannot render as an active hostile link.
- **Zero Trust Relevance**: N/A — output encoding, not a resource-access decision.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: N/A
- **Regulatory**: N/A
- **Other**: `REF-XSS`, `REF-PROMPT-VUE` as cited by SEC-RENDER-1 through SEC-RENDER-3; RFC 3986 for URL parsing.
- **Mapping Basis**: The four SEC-RENDER rules name these references directly; the CWE identifiers name the injection and window-opener classes.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given plan content, citation text, or a subscriber-entered value containing HTML metacharacters, when it renders, then it is displayed as literal text through contextual auto-escaping and no markup is interpreted.
2. **AC-02 — Boundary or failure behavior**: Given a citation URL whose scheme is not on the allow-list — `javascript:`, `data:`, `vbscript:`, `file:`, or an unparseable value — when the plan renders, then the value does not render as an active link and no navigable or fetchable attribute is bound to it.
3. **AC-03 — Prohibited behavior**: Given the client codebase, when it is linted and reviewed, then no raw HTML binding is applied to server- or user-originated content, no custom sanitization is written, and no health data, personal data, or token is written to `localStorage`, `sessionStorage`, IndexedDB, or any other browser-local store (SEC-RENDER-4, SEC-SESSION-5).
4. **AC-04 — Additional criterion**: Given an external citation link, when it is rendered and followed, then it does not grant opener access to the application window, and it is visibly a link without relying on color alone (`DESIGN.md`, Components → Links; Accessibility).
5. **AC-05 — Additional criterion**: Given a decision that rich-text plan content is required, when it is implemented, then it passes through a vetted HTML sanitizer in the rendering path and no hand-written sanitization is used (SEC-RENDER-2) — until that decision is made, plan content renders as plain text.

## Failure Behavior

- **On Invalid Input**: A URL failing the render-time scheme check renders as inert text rather than a link; the content still displays.
- **On Authentication Failure**: N/A
- **On Authorization Failure**: N/A
- **On Security-Decision Failure**: If a URL cannot be parsed or its scheme classified, render it inert (fail closed).
- **On External Dependency Failure**: N/A — the client never fetches citation targets; the user's own browser navigates.
- **On System Error**: The view shows a generic error; no raw server content is injected into the DOM.
- **Logging / Audit**: No audit entry. The client MUST NOT log rendered health values to the console (REQ-AUDIT-040, AC-05 there).
- **Alerting**: TO BE DECIDED — CSP violation reporting in pre-production (REQ-PLATFORM-040) is the nearest signal; no alerting model exists.

## Test Strategy

- **Unit Tests**: Rendering helpers escape metacharacters in text, attribute, and URL contexts; the scheme classifier accepts allow-listed schemes and rejects hostile and malformed ones, including mixed-case and whitespace-obfuscated variants.
- **Integration Tests**: Render plans whose content and citations contain stored XSS vectors, asserting no script execution and no active hostile link.
- **Security Tests**: Stored-XSS suite injecting markup through admin-authored plan fields and subscriber-entered fields end to end (SEC-RENDER-1 verification); hostile-scheme binding suite (SEC-RENDER-3 verification); lint rule forbidding raw HTML binding with a deliberate failing fixture; runtime storage inspection asserting no health data or token is persisted client-side.
- **Compliance Tests / Evidence**: Retained stored-XSS suite results as evidence against TM-T-4.
- **Acceptance-Criteria Traceability**: AC-01 — escaping suite; AC-02 — hostile-scheme suite; AC-03 — lint rule and storage inspection; AC-04 — link behavior and appearance tests; AC-05 — sanitizer path test, applicable only if rich text is adopted.
- **Coverage Target**: Every rendering surface for server- or user-originated content covered by the XSS suite.
- **Required Test Environment**: Seeded plans containing XSS vectors in every content field and citation URL; browser automation for runtime storage inspection. Client tooling and test framework TO BE DECIDED.

## Dependencies

- **Upstream Requirements**: REQ-PLATFORM-010, REQ-PLATFORM-030, REQ-PLATFORM-040
- **Downstream Requirements**: REQ-CATALOG-010, REQ-CATALOG-020, REQ-CUSTOM-020, REQ-PROGRESS-010, REQ-PROGRESS-020
- **External Dependencies**: A vetted HTML sanitizer only if rich text is later adopted (SEC-RENDER-2), subject to DEP-1 (custom sanitization is forbidden) and DEP-3, DEP-6, DEP-8.
- **Dependency Assumptions**: The client framework's default binding is contextually escaped and the raw-HTML interface is the only unsafe path, so a lint rule can be exhaustive.
- **Failure Impact**: A single raw HTML binding on a plan field turns every admin-authored plan into a stored-XSS vector against every subscriber, and via token theft into a health-data breach.

## Implementation Notes

- **Constraints**: Vue.js client (`ARCHITECTURE.md`); client build and lint tooling TO BE DECIDED (`CLAUDE.md`). Whether rich text is needed at all is `TO BE DECIDED` in SEC-RENDER-2, so plain-text rendering is the delivered behavior and AC-05 is conditional.
- **Prohibited Approaches**: `v-html` or any equivalent raw HTML binding on server- or user-originated content; hand-written escaping or sanitization; substring checks on URLs instead of parsing; trusting the server-side scheme check as sufficient; storing tokens or health data in browser-local storage as a convenience.
- **Implementation Guidance**: Apply the same scheme allow-list used in REQ-PLAN-040 so storage and render agree. `DESIGN.md` requires citation links to remain visibly links; an inert rejected URL should be displayed as text with an indication that it could not be rendered as a link, rather than silently dropped, so the evidence trail is not lost.
- **AI Development Guidance**: `REF-XSS`, `REF-PROMPT-VUE`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of every rendering surface and of the lint rule's coverage.
- **Open Decisions**: SEC-RENDER-2 leaves rich-text plan content `TO BE DECIDED`; adopting it later requires a vetted sanitizer and re-running the XSS suite. `DESIGN.md` OQ-9 (imagery policy) would add resource-attribute URLs to this issue's scope.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 200–450.
**Recommended model**: Claude Opus (`claude-opus-5`) — the client-side control against the highest-severity injection threat in the model, where the lint rule's completeness matters as much as the code.
