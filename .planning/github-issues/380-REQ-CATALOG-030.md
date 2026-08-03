# [REQ-CATALOG-030] Safe rendering of plan content and citation links

## Metadata

- **ID**: REQ-CATALOG-030
- **Title**: Safe rendering of plan content and citation links
- **Version**: 1.1.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `SECURITY.md` SEC-RENDER-1, SEC-RENDER-2, SEC-RENDER-3, SEC-RENDER-4; threat TM-T-4

## Requirement

- **Statement**: The Browser Client MUST render all dynamic content — plan content, citations, and subscriber-entered values — through the framework's contextual auto-escaping, MUST NOT use raw HTML injection interfaces for server- or user-originated content, MUST parse and scheme-check every data-derived URL before binding it to a link or resource attribute, and MUST open external links without granting opener access to the application window.
- **Rationale**: SEC-RENDER-1 forbids raw HTML binding for any server- or user-originated content including admin-authored plans; SEC-RENDER-3 requires scheme checking on data-derived URLs, naming citation links specifically; SEC-RENDER-4 forbids persisting health data or tokens in browser-local storage. The threat model rates stored XSS through admin-authored plan content, citation URLs, or subscriber-entered fields (TM-T-4) as high severity.
- **Assumptions**: Vue.js is the client framework (`ARCHITECTURE.md`), so contextual auto-escaping is available by default and the prohibited interface is the raw HTML binding.
- **Out of Scope**: The Content Security Policy — defense in depth delivered by REQ-PLATFORM-040, with the directive set now fixed in SEC-HTTP-2 (Confirmed 2026-08-03); server-side citation URL validation (REQ-PLAN-040), which this issue duplicates deliberately at render time; any rich-text rendering path — SEC-RENDER-2 is Confirmed that v1 plan content is structured plain text, and rich text would be a future design change requiring a vetted sanitizer.
- **Design Traceability**: `DESIGN.md` — Color System color rules ("Links are underlined"); Product Patterns → Plans, citations, and review state (Evidence section with a safe external link, SEC-RENDER-3; "The UI never fetches citation URLs server-side"); Forms and validation; Accessibility (color independence, visible focus).
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
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED)

## Security Context

- **Security Objectives**: Integrity, Confidentiality, Privacy
- **Control Layers**: Output Encoding, Sanitization, Data Protection
- **Threat References**: `SECURITY.md` TM-T-4 (stored XSS via admin-authored plan content, citation URLs, or subscriber-entered fields), TM-I-9 (health-data residue in browser storage), TM-S-5 (token theft via XSS); CWE-79 (cross-site scripting), CWE-83 (improper neutralization of script in attributes), CWE-1022 (use of web link to untrusted target with `window.opener`)
- **Abuse / Misuse Case**: A compromised admin stores markup in a plan description or a `javascript:` citation URL; it renders in every subscriber's browser and executes. The session token itself is unreadable to script (SEC-SESSION-5's `HttpOnly` cookie, SQ-2 RESOLVED), so the payload's path to health data is riding the victim's live session — still the most direct client-side path to health data in the threat model (TM-T-4, TM-S-5).
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
4. **AC-04 — Additional criterion**: Given an external citation link, when it is rendered and followed, then it does not grant opener access to the application window, and it is visibly a link — underlined, not distinguished by color alone (`DESIGN.md`, Color System color rules; Accessibility).
5. **AC-05 — Additional criterion**: Given a future design change that introduces rich-text plan content (SEC-RENDER-2's future clause), when it is implemented, then it passes through a vetted HTML sanitizer in the rendering path and no hand-written sanitization is used — absent such a change, plan content renders as the structured plain-text fields, headings, steps, lists, tables, and approved diagrams `DESIGN.md` defines.

## Failure Behavior

- **On Invalid Input**: A URL failing the render-time scheme check renders as inert text rather than a link; the content still displays.
- **On Authentication Failure**: N/A
- **On Authorization Failure**: N/A
- **On Security-Decision Failure**: If a URL cannot be parsed or its scheme classified, render it inert (fail closed).
- **On External Dependency Failure**: N/A — the client never fetches citation targets; the user's own browser navigates.
- **On System Error**: The view shows a generic error; no raw server content is injected into the DOM.
- **Logging / Audit**: No audit entry. The client MUST NOT log rendered health values to the console (REQ-AUDIT-040, AC-05 there).
- **Alerting**: CSP violation reporting runs report-only in staging before enforcement (SEC-HTTP-2, REQ-PLATFORM-040); violation reports and SEC-LOG-4 security events route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: Rendering helpers escape metacharacters in text, attribute, and URL contexts; the scheme classifier accepts allow-listed schemes and rejects hostile and malformed ones, including mixed-case and whitespace-obfuscated variants.
- **Integration Tests**: Render plans whose content and citations contain stored XSS vectors, asserting no script execution and no active hostile link.
- **Security Tests**: Stored-XSS suite injecting markup through admin-authored plan fields and subscriber-entered fields end to end (SEC-RENDER-1 verification); hostile-scheme binding suite (SEC-RENDER-3 verification); lint rule forbidding raw HTML binding with a deliberate failing fixture; runtime storage inspection asserting no health data or token is persisted client-side.
- **Compliance Tests / Evidence**: Retained stored-XSS suite results as evidence against TM-T-4.
- **Acceptance-Criteria Traceability**: AC-01 — escaping suite; AC-02 — hostile-scheme suite; AC-03 — lint rule and storage inspection; AC-04 — link behavior and appearance tests; AC-05 — sanitizer path test, applicable only if a future design change adopts rich text (SEC-RENDER-2).
- **Coverage Target**: Every rendering surface for server- or user-originated content covered by the XSS suite.
- **Required Test Environment**: Seeded plans containing XSS vectors in every content field and citation URL; browser automation for runtime storage inspection. Vitest as the runner, with Playwright and axe-core where a real browser is required.

## Dependencies

- **Upstream Requirements**: REQ-PLATFORM-010, REQ-PLATFORM-030, REQ-PLATFORM-040
- **Downstream Requirements**: REQ-CATALOG-010, REQ-CATALOG-020, REQ-CUSTOM-020, REQ-PROGRESS-010, REQ-PROGRESS-020
- **External Dependencies**: A vetted HTML sanitizer only if a future design change adopts rich text (SEC-RENDER-2's future clause), subject to DEP-1 (custom sanitization is forbidden) and DEP-3, DEP-6, DEP-8. None in v1.
- **Dependency Assumptions**: The client framework's default binding is contextually escaped and the raw-HTML interface is the only unsafe path, so a lint rule can be exhaustive.
- **Failure Impact**: A single raw HTML binding on a plan field turns every admin-authored plan into a stored-XSS vector against every subscriber, and via token theft into a health-data breach.

## Implementation Notes

- **Constraints**: Vue.js client written in TypeScript (`ARCHITECTURE.md`); ESLint with eslint-plugin-vue supplies the `vue/no-v-html` rule this issue's verification depends on (`CLAUDE.md`), and Vite builds the client as a single-page application. SEC-RENDER-2 is Confirmed: v1 plan content is structured plain text with headings, steps, lists, tables, and approved diagrams, and no rich-text rendering path exists — plain-text rendering is the delivered behavior and AC-05 applies only to a future design change.
- **Prohibited Approaches**: `v-html` or any equivalent raw HTML binding on server- or user-originated content; hand-written escaping or sanitization; substring checks on URLs instead of parsing; trusting the server-side scheme check as sufficient; storing tokens or health data in browser-local storage as a convenience.
- **Implementation Guidance**: Apply the same scheme allow-list used in REQ-PLAN-040 so storage and render agree. `DESIGN.md` renders citations as underlined links in the Evidence section; an inert rejected URL should be displayed as text with an indication that it could not be rendered as a link, rather than silently dropped, so the evidence trail is not lost.
- **AI Development Guidance**: `REF-XSS`, `REF-PROMPT-VUE`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of every rendering surface and of the lint rule's coverage.
- **Open Decisions**: None — SEC-RENDER-2 is Confirmed (structured plain text in v1; a future rich-text change requires a vetted sanitizer and a re-run of the XSS suite), and `DESIGN.md` OQ-9 is resolved: admin-approved neutral vector exercise diagrams and the transient, unsaved food-photo preview (served under SEC-HTTP-2's `img-src 'self' data: blob:`) are the resource-attribute rendering surfaces this issue's URL rules cover.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 200–450.
**Recommended model**: Claude Opus (`claude-opus-5`) — the client-side control against the highest-severity injection threat in the model, where the lint rule's completeness matters as much as the code.
