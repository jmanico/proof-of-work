# [REQ-PROGRESS-060] Progress history display with trend charts and paired tables

## Metadata

- **ID**: REQ-PROGRESS-060
- **Title**: Progress history display with trend charts and paired tables
- **Version**: 1.0.1
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-8.6, FR-8.14, FR-5.4 (catalog identity), FR-9.1, FR-9.7; `SECURITY.md` SEC-DATA-5; `DESIGN.md` Progress and target comparison

## Requirement

- **Statement**: The system MUST display a subscriber's logged history at full entry granularity for the lifetime of the account as trend charts over selectable ranges of 4 weeks, 3 months, 1 year, and all time — covering body weight, each body-measurement field, and per-exercise workout load keyed by catalog entry with the session's top-set weight as the charted value — MUST pair every chart with an accessible data table presenting the same data, MUST return only the requesting subscriber's own entries with only the fields the view requires, and MUST record one audit entry per history request.
- **Rationale**: FR-8.6 requires history display for weight, measurements, and workout performance; FR-8.14 fixes full-granularity lifetime retention, the four ranges, the chart-plus-paired-table presentation, catalog-entry identity so trends survive plan switches, copies, and renames, and top-set weight — the heaviest weight logged for the exercise in the session — as the charted load value; FR-9.7 makes each history read an audited health-data access, one entry per request, including the subscriber's own reads; SEC-DATA-5 bounds the response to least privilege.
- **Assumptions**: Weight, workout, and measurement entries exist to display (REQ-PROGRESS-010, REQ-PROGRESS-020, REQ-PROGRESS-040); logged exercises reference stable catalog entries with name snapshots (FR-8.3, REQ-PLAN-080 for catalog management); display-time unit conversion is available (REQ-PROGRESS-050); the platform accessibility and responsive baselines hold (REQ-PLATFORM-020, REQ-PLATFORM-030).
- **Out of Scope**: The daily calorie and macronutrient comparison against the selected diet plan (FR-8.5, REQ-FOOD-030); the raw entry list with edit and delete, owned by each logging issue (FR-8.7); food-log history aggregation, which FR-8.14 does not name as a charted series; export (REQ-PRIVACY-080); consultant views of an engaged subscriber's progress, which reuse these views under REQ-CONSULT-040's scoping and audit; retention mechanics themselves (Relational Persistence stores entries for the account lifetime — no aggregation or pruning exists to configure).
- **Design Traceability**: `DESIGN.md` — Product Patterns → Progress and target comparison ("Progress pages pair every trend chart with a data table showing the same full-granularity entries (FR-8.14). Range controls are 4 weeks, 3 months, 1 year, and all time. The selected range is programmatically exposed."; charts use direct labels where space allows, visible point shapes, stroke of at least 2px; multiple series differ by both color and dash or point shape; the emphasized or most-recent point uses `brand` in light mode — or a `signal` fill inside a 2px `ink` stroke — and `signal` directly in dark mode; keyboard focus reaches each data point and exposes its label and value; tooltips duplicate information, never containing unique data; reduced motion removes drawing animations); Color System (`signal` restriction on light surfaces, 2026-08-03); Accessibility ("the paired table is the complete text alternative, but the chart itself also has a concise accessible name and range summary"; no animated chart drawing required to understand state); Core Components → Cards, lists, and tables (real table semantics, tabular figures, record cards below 48rem); Typography (data face for chart labels); Status, feedback, and loading (empty states say what belongs and provide the creating action); admin exercise-catalog pattern (retiring an entry never removes it from history or charts, FR-5.4).
- **Architecture Traceability**: `ARCHITECTURE.md` — data flow 5 ("Client → REST API → owner scoping → entry-level log history over the requested range … → audit entry written (FR-9.7, one per request) → Client"); Browser Client responsibility (present progress history; trend charts paired with accessible tables at entry granularity); REST API Application; Relational Persistence (retains records across subscription lapse, FR-3.4); DR-9.
- **Security Traceability**: SEC-DATA-5, SEC-LOG-1 (one audit entry per request, self-reads included), SEC-AUTHZ-2, SEC-INPUT-1 (range and catalog-entry parameters validated), SEC-RENDER-4 (no health data persisted in browser storage; accessibility affordances expose nothing unauthorized).

## Scope

- **Applies To**: Multiple — Web Client, Server-Side Application, API
- **Components**: Browser Client (Progress views); REST API Application (history read endpoints); Relational Persistence
- **Interfaces / Operations**: History read operations for body weight, each body-measurement field, and per-exercise workout load over a requested range; the Progress pages rendering charts, range controls, and paired tables
- **Actors**: Subscriber (own history); consultant access is granted only through REQ-CONSULT-040's engagement scoping; `admin` has no access (FR-10.3)
- **Preconditions**: Authenticated subscriber session; active subscription entitlement (REQ-ENTITLE-010); entries may be absent (empty state)
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, Washington My Health My Data and comparable state consumer-health laws, and the FTC Health Breach Notification Rule for US users; HIPAA not applicable (`SECURITY.md` SQ-1)

## Security Context

- **Security Objectives**: Confidentiality, Accountability, Privacy
- **Control Layers**: Authorization, Input Validation, Data Protection, Logging and Monitoring, Output Encoding
- **Threat References**: `SECURITY.md` TM-I-1 (BOLA against another subscriber's history), TM-I-3 (excessive data exposure / bulk retrieval), TM-D-1 (resource exhaustion via expensive history aggregation); CWE-639 (authorization bypass through user-controlled key), CWE-213 (exposure of sensitive information beyond what the operation requires)
- **Abuse / Misuse Case**: A malicious subscriber varies identifiers or range parameters to read another subscriber's trends; any role attempts bulk retrieval of history across accounts; an attacker hammers the all-time aggregation as a denial-of-service amplifier; a client bug caches health values in browser storage after the view ends.
- **Trust Boundary**: Boundary 1 (Browser Client → REST API Application) — range, series, and catalog-entry parameters are untrusted input; boundary 3 for the constrained query.
- **Untrusted Inputs or Assertions**: Range selection, catalog-entry identifier, series selection, any pagination or date parameters, and any client assertion of ownership.
- **Authoritative Enforcement Point**: REST API Application — owner-scoped queries (SEC-AUTHZ-2) with schema-bound response serialization returning only the fields the view requires (SEC-DATA-5), and the audit write on every request (DR-9).
- **Independent Verification**: Ownership comes from persisted state resolved against the session identity, never from a client parameter (DR-3).
- **Zero Trust Relevance**: TO BE DECIDED — specific NIST SP 800-207 tenet mapping not verified against the publication (`SECURITY.md` SQ-10 gates all mapping work).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — per-issue mapping deferred to the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A — no AI-enabled component in this flow.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — per-issue mapping deferred (`SECURITY.md` SQ-10).
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: GDPR/UK GDPR, CCPA/CPRA, Washington My Health My Data, FTC HBNR per `SECURITY.md` SQ-1; FR-9.7's audit obligation applies to these reads; specific article/section citations TO BE DECIDED pending the SQ-1 counsel review.
- **Other**: WCAG 2.2 AA (`DESIGN.md` Accessibility target; `REQUIREMENTS.md` OQ-11 RESOLVED); `REF-API-2023`, `REF-PROMPT-API` as cited by SEC-DATA-5.
- **Mapping Basis**: WCAG 2.2 AA is the fixed conformance target for the chart, table, and range-control accessibility behavior; the cited references govern the excessive-data-exposure rule this issue's response shapes implement; the SQ-1 regime set applies to all health-data access.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a subscriber with logged history, when they open a Progress view and select each of 4 weeks, 3 months, 1 year, and all time, then trend charts render for body weight, each body-measurement field with recorded values, and per-exercise workout load, each showing full entry granularity for the range with no lossy aggregation, each paired with a data table presenting the same data (the workout table showing the full sets, repetitions, and weight while the chart plots the session's top-set weight), values rendered in the account's unit system, and the selected range programmatically exposed.
2. **AC-02 — Boundary or failure behavior**: Given a subscriber whose logged exercise spans a plan switch, a customized copy, and a catalog rename, when its trend is displayed, then the series continues unbroken keyed by the FR-5.4 catalog-entry identifier (a retired entry still charts its history); and given a range containing no entries, when the view loads, then an empty state names what belongs there and offers the action that creates it, with no error styling.
3. **AC-03 — Prohibited behavior**: Given subscriber A's session, when any combination of range, series, catalog-entry, or identifier parameters references subscriber B's data, then the response contains none of it and discloses nothing about its existence; responses MUST NOT carry fields beyond what the view requires (SEC-DATA-5); no role may bulk-retrieve history across accounts; and no health value or photo remains in browser storage after the view ends (SEC-RENDER-4).
4. **AC-04 — Additional criterion**: Given the rendered charts, when inspected for the `DESIGN.md` contract, then every chart has a concise accessible name and range summary; each data point is keyboard-reachable exposing its label and value; multiple series differ by both color and dash or point shape; strokes are at least 2px; emphasis uses `brand` in light mode (or `signal` only inside a 2px `ink` boundary) and `signal` in dark mode; tooltips contain no unique data; drawing animations are absent under `prefers-reduced-motion`; and below 48rem the paired tables become labelled record cards without page-level horizontal scrolling.
5. **AC-05 — Additional criterion**: Given any history request, when it is served, then exactly one audit entry records the acting account, the action, the affected subject, and the time (FR-9.7, self-reads included), and the audit entry contains no health values.

## Failure Behavior

- **On Invalid Input**: An unrecognized range, series, or catalog-entry parameter is rejected by allow-list schema validation (SEC-INPUT-1) with the specific failing field; no query executes.
- **On Authentication Failure**: Deny per REQ-AUTHZ-010; no history shape or existence information leaks.
- **On Authorization Failure**: Deny without disclosing whether the referenced data exists (SEC-AUTHZ-2); lapsed entitlement shows the exact reason and next step per `DESIGN.md` status patterns without exposing data.
- **On Security-Decision Failure**: Deny by default — an error resolving ownership or entitlement returns no data.
- **On External Dependency Failure**: N/A — the flow involves only the REST API Application and Relational Persistence.
- **On System Error**: The client shows a generic error with the correlation identifier (SEC-ERR-1); partial chart data is not presented as complete; if the audit write fails the read fails with it (DR-9).
- **Logging / Audit**: One FR-9.7/SEC-LOG-1 audit entry per history request (acting account, action, affected subject, time); request logs carry no health values (SEC-LOG-3).
- **Alerting**: High-volume history-read patterns are covered by the SEC-HTTP-5 per-account rate limits (REQ-API-050); threshold alerts route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-3, SQ-11).

## Test Strategy

- **Unit Tests**: Top-set-weight derivation (heaviest weight logged for the exercise in a session, including ties and single-set sessions); range windowing at the 4-week/3-month/1-year boundaries; series construction keyed by catalog entry across renames and retirement; unit conversion applied at display only; response-shape field allow-lists.
- **Integration Tests**: History endpoints over fixtures spanning plan switches, copies, and renames; full-granularity assertions (every entry in range present, none aggregated away); empty-range behavior; audit entry emitted exactly once per request including self-reads; entitlement-lapse denial with records intact on reactivation (FR-3.4).
- **Security Tests**: BOLA suite across all history parameters; response-shape assertions for SEC-DATA-5 (no extra fields, no cross-account rows); bulk-retrieval attempts as subscriber, consultant (no engagement), and admin asserting denial; browser-storage inspection after leaving the view (SEC-RENDER-4).
- **Compliance Tests / Evidence**: Playwright + axe-core coverage of the Progress pages — chart accessible names and range summaries, keyboard reach of data points, programmatic range exposure, record-card reflow at 320px and 200% zoom, reduced-motion behavior — per the `DESIGN.md` Design Verification list.
- **Acceptance-Criteria Traceability**: AC-01 — range and granularity integration suite; AC-02 — catalog-continuity and empty-state suites; AC-03 — BOLA, response-shape, and storage-inspection suites; AC-04 — Playwright + axe chart-contract suite; AC-05 — audit-granularity suite.
- **Coverage Target**: Project coverage threshold is 90% line and branch (`CLAUDE.md`, 2026-08-03); every denial path, range boundary, and accessibility commitment above MUST have automated coverage regardless.
- **Required Test Environment**: Vitest with PostgreSQL fixtures containing multi-year, multi-plan, renamed- and retired-exercise history for two subscribers; an HTTP test client; Playwright with axe-core in light, dark, and reduced-motion configurations at 320px and desktop widths.

## Dependencies

- **Upstream Requirements**: REQ-PROGRESS-010, REQ-PROGRESS-020, REQ-PROGRESS-040 (the entries displayed), REQ-PROGRESS-050 (display-time conversion), REQ-PLATFORM-020, REQ-PLATFORM-030 (responsive and accessibility baselines), REQ-PLATFORM-010 (tokens), REQ-AUTHZ-010, REQ-AUTHZ-020, REQ-AUDIT-020, REQ-ENTITLE-010, REQ-PLAN-080 (catalog identity)
- **Downstream Requirements**: REQ-CONSULT-040 (consultant progress views reuse these displays under engagement scoping); REQ-FOOD-030 (the daily-target view sits beside these Progress patterns)
- **External Dependencies**: None
- **Dependency Assumptions**: Logged exercises carry catalog-entry references and name snapshots (FR-8.3), so series identity never depends on mutable names.
- **Failure Impact**: A scoping or response-shape defect exposes another subscriber's health history; a missing audit write breaks FR-9.7's accountability for reads; an aggregation shortcut violates FR-8.14's full-granularity guarantee.

## Implementation Notes

- **Constraints**: TypeScript; Vue single-file components with scoped styles and CSS custom-property tokens — no component or charting framework may be adopted without an explicit decision (`CLAUDE.md`, DEP-1, DEP-2); Fastify schema-bound response serialization is the SEC-DATA-5 delivery mechanism; all-time queries must remain time-bounded (SEC-HTTP-5).
- **Prohibited Approaches**: Client-side filtering of over-fetched cross-account data (SEC-AUTHZ-2); server-side pre-aggregation that discards entries (FR-8.14); keying exercise series by name instead of catalog identifier; charts whose tooltip or hover state carries unique data; conveying series identity by color alone; caching health values in `localStorage`/`sessionStorage` (SEC-RENDER-4); animated chart drawing that ignores `prefers-reduced-motion`.
- **Implementation Guidance**: Render charts as inline SVG built from the same response the paired table consumes, so chart and table cannot diverge; compute top-set weight server-side in the history query; treat the paired table as the primary DOM structure and the chart as its visual companion per `DESIGN.md`.
- **AI Development Guidance**: `REF-PROMPT-VUE`, `REF-PROMPT-API`, `REF-PROMPT-QUALITY`; `CLAUDE.md`; the repository `dataviz` skill guidance where chart construction begins.
- **Required Human Review**: Accessibility review of the chart/table pairing against WCAG 2.2 AA; security review of the response shapes and owner scoping; design review of chart emphasis in both color modes.
- **Open Decisions**: None.

**Estimated effort**: 1.5–2 engineer-days. **Estimated changed lines**: 600–1200.
**Recommended model**: Claude Fable (`claude-fable-5`) — cross-boundary work spanning API response design, SVG chart construction, and a demanding accessibility contract benefits from the stronger design-and-frontend model.
