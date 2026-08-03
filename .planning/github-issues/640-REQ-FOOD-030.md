# [REQ-FOOD-030] Daily intake versus selected diet plan targets

## Metadata

- **ID**: REQ-FOOD-030
- **Title**: Daily intake versus selected diet plan targets
- **Version**: 1.0.1
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-8.5, FR-6.2, FR-6.4, FR-4.9, FR-9.7; `SECURITY.md` SEC-DATA-5; `DESIGN.md` D-PRINCIPLE-3 and "Progress and target comparison"

## Requirement

- **Statement**: The system MUST display, for a subscriber-chosen day, that subscriber's total logged calories and FR-6.2 macronutrients — protein, carbohydrate, and fat — against the daily targets of their currently selected diet plan (FR-6.4), and when no diet plan selection is active MUST present the no-target state (FR-4.9) showing the logged totals without a comparison.
- **Rationale**: FR-8.5 is the payoff of food logging: intake in context of the plan the subscriber chose. FR-6.4 fixes which plan supplies targets (the current selection, published plan or own customized copy), and FR-4.9 defines what happens when unpublication ends that selection, so the comparison is total — every state is specified, including having no target at all. D-PRINCIPLE-3 requires the comparison to stay neutral context, never a moral grade.
- **Assumptions**: Food entries with attributed values exist per REQ-FOOD-020; the active diet plan selection, including its replacement semantics and ending on unpublication, is maintained by REQ-SELECT-020 and REQ-SELECT-030; authentication, entitlement, and owner scoping are enforced by REQ-AUTHZ-010, REQ-ENTITLE-010, and REQ-AUTHZ-020.
- **Out of Scope**: Creating or mutating food entries or selections (this is a read-only view); trend charts over time ranges (FR-8.14, REQ-PROGRESS-060); diet plan authoring and target definition (REQ-PLAN-* per FR-6.2); ending selections on unpublication (REQ-SELECT-030); consultant access to this view within an engagement (REQ-CONSULT-040).
- **Design Traceability**: `DESIGN.md` — "Progress and target comparison": explicit text such as "82 g of 110 g target" beside a linear indicator; an over-target value continues the bar with a labelled over-target amount and does not turn red or use failure language; D-PRINCIPLE-3 (targets are context, not grades); Color rules (variance uses neutral brand tones; `error` reserved for actual validation/destructive states); Components → Status and empty states ("No entries yet", never "You're falling behind").
- **Architecture Traceability**: `ARCHITECTURE.md` — data flow 5 (progress viewing: owner scoping, logged intake compared against the selected plan's targets, audit entry written); REST API Application (owns food log entry and plan selections; sole authority for the comparison); Browser Client (presentation only, DR-2).
- **Security Traceability**: SEC-DATA-5 (least-privilege response shape); SEC-AUTHZ-1, SEC-AUTHZ-2 (owner-scoped read); SEC-LOG-1/FR-9.7 (audit of health-data access); SEC-ERR-1.

## Scope

- **Applies To**: Multiple — API, Server-Side Application, Web Client
- **Components**: REST API Application; Relational Persistence; Browser Client
- **Interfaces / Operations**: Daily intake-versus-target read operation; the daily food view in the Log/Progress areas
- **Actors**: Subscriber (owner); consultant (same view within an active engagement per FR-11.6, delivered by REQ-CONSULT-040); admin (prohibited, FR-10.3); anonymous attacker (denied)
- **Preconditions**: Authenticated session; active subscription entitlement (FR-3.1). A diet plan selection MAY or MAY NOT exist — both states are in scope.
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data — food log entries and the active diet plan selection are health data (FR-9.12)
- **Jurisdiction / Regulatory Scope**: Global service under the `SECURITY.md` SQ-1 regime set — GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, Washington My Health My Data, and the FTC Health Breach Notification Rule for US users; HIPAA evaluated and not applicable

## Security Context

- **Security Objectives**: Confidentiality, Privacy, Accountability, Integrity
- **Control Layers**: Authorization, Data Protection, Logging and Monitoring, Output Encoding
- **Threat References**: `SECURITY.md` TM-I-1 (BOLA — reading another subscriber's daily intake), TM-I-3 (excessive data exposure through an aggregate view); CWE-639 (authorization bypass through user-controlled key), CWE-359 (exposure of private personal information)
- **Abuse / Misuse Case**: Subscriber A requests subscriber B's daily comparison by identifier or date-scoped query; an over-broad response leaks entry fields, plan internals, or another user's targets beyond what the view needs; an admin uses the aggregate view to read subscriber health data.
- **Trust Boundary**: Boundary 1 (Browser Client → REST API Application) — the requested day and any subject identifier are untrusted; boundary 3 for persistence reads.
- **Untrusted Inputs or Assertions**: The requested date and any client-supplied subject or selection identifier; the comparison subject is always resolved from session identity (or engagement scope under REQ-CONSULT-040), never from a client assertion.
- **Authoritative Enforcement Point**: REST API Application — it computes totals, resolves the current selection, and shapes the least-privilege response; the client renders but never derives entitlement to the data (DR-2, DR-3).
- **Independent Verification**: Owner scoping is applied in the query (SEC-AUTHZ-2), and the selection whose targets are used is read from persisted state (FR-6.4), not from a client-named plan.
- **Zero Trust Relevance**: TO BE DECIDED — per-issue NIST SP 800-207 mapping is deferred to the independent pre-launch assessment (`SECURITY.md` SQ-10).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — per-issue mapping deferred to the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A — no AI-enabled component is involved in this requirement.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — deferred with SQ-10.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: GDPR/UK GDPR (EU/UK data subjects), CCPA/CPRA, Washington My Health My Data, FTC Health Breach Notification Rule (US users) per `SECURITY.md` SQ-1 — this view is a health-data access under FR-9.12/FR-9.7. Section-level mappings: TO BE DECIDED pending the SQ-1 counsel review.
- **Other**: `REF-API-2023`, `REF-PROMPT-API` as cited by SEC-DATA-5.
- **Mapping Basis**: SEC-DATA-5 names these references for least-privilege response shaping; the CWE identifiers describe the BOLA and private-data-exposure classes for an aggregate health-data read.

## Acceptance Criteria

1. **AC-01 — Expected behavior (comparison)**: Given a subscriber with an active diet plan selection and food entries logged for a chosen day, when they open the daily view, then it shows, for calories and each of protein, carbohydrate, and fat, the day's total beside the selected plan's daily target as explicit text (e.g., "82 g of 110 g target") with a linear indicator, and the totals equal the sum of that day's entries' stored values.
2. **AC-02 — Expected behavior (customized copy and selection change)**: Given the active selection names the subscriber's own customized diet plan copy, when the view renders, then the copy's targets are used; and given the subscriber replaces their selection, when the view is reloaded, then the new selection's targets apply while all logged totals remain unchanged (FR-6.4 — selection change never alters logged history).
3. **AC-03 — Boundary behavior (over target and no entries)**: Given a day whose total exceeds a target, when the view renders, then the indicator continues past the target with a labelled over-target value using neutral tones — no `error` styling and no failure language; and given a day with no entries, then the view shows the neutral empty state with zero totals, not an error or judgment.
4. **AC-04 — Boundary behavior (no-target state)**: Given no active diet plan selection — never selected, or ended because the selected plan was unpublished (FR-4.9) — when the view renders, then it presents the no-target state: logged totals remain visible, no comparison or fabricated target is shown, the state explains that no diet plan is selected, and it offers the action to select one.
5. **AC-05 — Prohibited behavior (scope and exposure)**: Given subscriber A authenticated, when the daily view is requested for subscriber B by any identifier or parameter manipulation, then it is denied without disclosing whether B or B's data exists (SEC-AUTHZ-2); and the response contains only the fields this view requires — totals, targets, and entry summaries for the owner's chosen day — never other days' data beyond the request, other subscribers' data, or unpublished plan internals (SEC-DATA-5). An admin request for a subscriber's daily view is denied (FR-10.3).
6. **AC-06 — Additional criterion (audit)**: Given any successful daily-view read, when the request completes, then exactly one FR-9.7 audit entry is recorded with the acting account, the action, the affected subject, and the time — including the subscriber's own reads.

## Failure Behavior

- **On Invalid Input**: A malformed or out-of-schema date parameter is rejected by allow-list validation (REQ-API-010) naming the failing field; future dates are a valid query (they simply have no entries) unless schema bounds say otherwise — no write occurs anywhere on this path.
- **On Authentication Failure**: Deny per REQ-AUTHZ-010 with a uniform response.
- **On Authorization Failure**: Deny without confirming the target subscriber's existence; lapsed entitlement returns the subscription-required state of FR-3.2 via REQ-ENTITLE-010.
- **On Security-Decision Failure**: Deny by default — an error resolving identity, entitlement, or scope returns a denial, never a partial view (SEC-AUTHZ-7).
- **On External Dependency Failure**: N/A — the view reads in-boundary persistence only.
- **On System Error**: Return a generic error with a correlation identifier (SEC-ERR-1); no internal detail, no partial cross-subscriber data. A failure writing the audit entry fails the read (DR-9).
- **Logging / Audit**: One FR-9.7 audit entry per daily-view request (acting account, action, affected subject, time). Neither logs nor audit entries contain the totals, targets, or any food values (SEC-LOG-3).
- **Alerting**: Repeated authorization denials on this read path (BOLA probing) route as threshold alerts to the security lead as SEC-OPS-2 detection inputs (SQ-11).

## Test Strategy

- **Unit Tests**: Daily aggregation over stored entry values (single entry, multiple entries, day boundaries, empty day); target resolution from published plan versus customized copy; no-target state derivation when no selection exists or the selection was ended; over-target value computation.
- **Integration Tests**: End-to-end view through the API with fixtures — entries plus selection asserting AC-01/AC-02; selection replacement asserting targets switch while totals persist; unpublication via REQ-SELECT-030 asserting the no-target state appears; audit entry written once per read.
- **Security Tests**: BOLA suite requesting another subscriber's day, asserting denial without existence disclosure; response-shape assertion enforcing the SEC-DATA-5 field allow-list (no extra entry fields, no other-day leakage, no plan internals); admin access attempt asserting denial (FR-10.3); log-content assertion that no health value appears in logs or audit entries.
- **Compliance Tests / Evidence**: Audit-granularity evidence (one entry per request, FR-9.7) retained for the SQ-1 counsel review; accessibility evidence for the indicator and its text equivalent rides the Playwright + axe suite (`CLAUDE.md`, DESIGN.md Design Verification).
- **Acceptance-Criteria Traceability**: AC-01 — aggregation and rendering suites; AC-02 — copy-target and selection-change suites; AC-03 — over-target and empty-day suites; AC-04 — no-target suite; AC-05 — BOLA and response-shape suites; AC-06 — audit suite.
- **Coverage Target**: Project coverage threshold is 90% line and branch (`CLAUDE.md`, 2026-08-03); every denial and state-derivation path MUST have positive and negative coverage.
- **Required Test Environment**: Two subscriber fixtures with entries and selections in each state (published plan, customized copy, no selection, selection ended by unpublication); admin fixture; PostgreSQL; Vitest, HTTP test client, and Playwright + axe for the presentation checks.

## Dependencies

- **Upstream Requirements**: REQ-FOOD-020 (food entries with attributed values); REQ-SELECT-020 (active diet plan selection supplying targets)
- **Downstream Requirements**: REQ-SELECT-030 (relies on this view's no-target state after unpublication ends a selection); REQ-CONSULT-040 (consultant read of this view within an engagement)
- **External Dependencies**: None
- **Dependency Assumptions**: REQ-SELECT-020 guarantees at most one active diet selection whose targets carry the FR-6.2 trio in grams; REQ-AUDIT-010 provides append-only audit persistence.
- **Failure Impact**: If selection resolution is wrong, subscribers are graded against a plan they did not choose; if scoping fails, the view becomes a health-data disclosure channel — the latter is a release-blocking defect.

## Implementation Notes

- **Constraints**: Fastify schema-bound response serialization is the delivery mechanism for the SEC-DATA-5 response shape (`CLAUDE.md`); comparison text and indicator follow DESIGN.md exactly, including the data face for numeric values and the account's unit conventions where units appear.
- **Prohibited Approaches**: Client-side target resolution or aggregation as the authority (DR-2); accepting a client-named plan for the comparison instead of the persisted selection; styling over-target as `error` or using failure language (D-PRINCIPLE-3); fetching all entries and filtering in the client; post-retrieval filtering instead of owner-scoped queries (SEC-AUTHZ-2).
- **Implementation Guidance**: Compute totals in the database query, scoped by owner and date, and let the serializer strip everything outside the declared response schema. Represent "no selection" as an explicit state in the response contract so the client renders FR-4.9's explanatory state without inventing semantics (DR-2's structured-detail rule).
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-NODE`, `REF-PROMPT-VUE`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the response schema against SEC-DATA-5; design review that over-target and no-target presentations match DESIGN.md's neutral-language rules.
- **Open Decisions**: None. The day boundary (account-local time versus a fixed timezone) is an implementation-level choice decided with the code; it changes no requirement above.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 250–500.
**Recommended model**: Claude Opus (`claude-opus-5`) — a read path whose response shape is the control: least-privilege serialization, owner scoping, and the fully specified no-target state.
