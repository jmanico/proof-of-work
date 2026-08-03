# [REQ-PROGRESS-050] Per-account unit system and display-only conversion

## Metadata

- **ID**: REQ-PROGRESS-050
- **Title**: Per-account unit system and display-only conversion
- **Version**: 1.0.1
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-8.10, FR-8.9 (metric-equivalent plausibility checks); `DESIGN.md` Forms and validation, Account patterns

## Requirement

- **Statement**: The system MUST maintain a per-account unit-system preference — metric (centimetres, kilograms) or imperial (inches, pounds) — chosen by the user as a required step of registration with no silent default (FR-8.10, fixed 2026-08-03; the registration form is REQ-AUTH-080), applied to body measurements, body weight, and workout load, MUST store every logged value together with the unit it was entered in, and MUST convert between unit systems only at display time, never by mutating stored values.
- **Rationale**: FR-8.10 fixes the preference, the store-with-entered-unit rule, and display-only conversion, resolving `REQUIREMENTS.md` OQ-4's unit half with `DESIGN.md` OQ-8. Storing the entered value with its unit keeps logged history exact and auditable; converting only at display time means a preference switch can never corrupt or drift historical data. FR-8.9's plausibility ranges are defined in metric, so validation MUST check the metric equivalent of the entered value regardless of the entry unit.
- **Assumptions**: An account exists to carry the preference (REQ-AUTH-080). Body-fat percentage is unitless (FR-8.2) and is unaffected by the preference.
- **Out of Scope**: The plausibility ranges themselves and per-entry validation flows, which each logging issue applies (REQ-PROGRESS-010, REQ-PROGRESS-020, REQ-PROGRESS-040, REQ-PROGRESS-030); chart and table rendering of converted values (REQ-PROGRESS-060); calorie and macronutrient units, which FR-6.2/FR-8.4 fix as grams and are not part of the metric/imperial preference; localization of number formatting beyond the unit suffix (`DESIGN.md` defers additional languages beyond v1).
- **Design Traceability**: `DESIGN.md` — Core Components → Forms and validation ("Numeric health fields use the data face and an adjacent unit suffix tied to the account preference (FR-8.10)"); Typography ("Values and their units do not wrap onto separate lines"); Product Patterns → Account, privacy, and destructive actions ("Account groups Profile, Units and appearance, Security, Subscription, Consultant access, and Privacy" — the preference lives under Units and appearance).
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application (sole authority for business rules; owns the account record the preference lives on); Relational Persistence (stores value-with-unit on every log entry); Browser Client (display-time presentation); DR-2 (the conversion rule is server-defined; the client never re-derives validation).
- **Security Traceability**: SEC-INPUT-1 (the preference value and entry units are allow-list validated), SEC-INPUT-3 (only the preference itself is client-assignable; stored historical values are not), SEC-AUTHZ-2 (the preference is readable and writable only by its owning account).

## Scope

- **Applies To**: Server-Side Application, API, Web Client
- **Components**: REST API Application; Relational Persistence; Browser Client (Account → Units and appearance; every numeric health field and history view)
- **Interfaces / Operations**: Read and update the account unit-system preference; unit capture on every measurement, weight, and workout-load write; display-time conversion on every read path that presents those values
- **Actors**: Subscriber (own preference); consultant and admin views render values under their own display context without altering stored data
- **Preconditions**: Authenticated session for reading or changing the preference
- **Data Classification**: Confidential (the preference alone); the values it applies to are Restricted health data
- **Personal or Regulated Data**: Personal Data (the preference); it governs presentation of Health Data
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, Washington My Health My Data and comparable state consumer-health laws, and the FTC Health Breach Notification Rule for US users; HIPAA not applicable (`SECURITY.md` SQ-1)

## Security Context

- **Security Objectives**: Integrity, Privacy
- **Control Layers**: Input Validation, Business-Rule Validation, Data Protection
- **Threat References**: `SECURITY.md` TM-T-1 (tampered payloads — a forged unit or preference value skewing validation or stored history); CWE-20 (improper input validation), CWE-1284 (improper validation of specified quantity)
- **Abuse / Misuse Case**: A client submits a value with a false unit so a dangerous or implausible quantity passes validation (600 lb logged as "kg", or a range check applied to the raw imperial number instead of its metric equivalent); a preference change is abused to trigger bulk rewrites of stored history.
- **Trust Boundary**: Boundary 1 (Browser Client → REST API Application) — the entered unit and the preference are client input; the conversion rule and stored values are server truth.
- **Untrusted Inputs or Assertions**: The preference value, the unit accompanying each logged value, and any client-side conversion result.
- **Authoritative Enforcement Point**: REST API Application — validates the unit against the allow-list, computes the metric equivalent for FR-8.9 range checks, and persists the entered value with its unit unchanged.
- **Independent Verification**: The server performs its own conversion for validation and display; a client-computed conversion is never trusted or stored (DR-2, DR-3).
- **Zero Trust Relevance**: N/A — no resource-access decision is made by this requirement beyond owner scoping already covered by REQ-AUTHZ-020.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — per-issue mapping deferred to the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A — no AI-enabled component.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — per-issue mapping deferred (`SECURITY.md` SQ-10).
- **NIST SP 800-207**: N/A — see Zero Trust Relevance.
- **Regulatory**: The stored-value integrity this rule protects underpins the FR-9.5 correction right and FR-9.3 export accuracy under the SQ-1 regimes (GDPR/UK GDPR; CCPA/CPRA, Washington My Health My Data, FTC HBNR); specific article/section citations TO BE DECIDED pending the SQ-1 counsel review.
- **Other**: `REF-INPUT` as cited by SEC-INPUT-1.
- **Mapping Basis**: The regime set is fixed by SQ-1 wherever health data is presented or exported; `REF-INPUT` governs the allow-list validation of units and the preference enum.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an account whose preference is metric, when the subscriber switches to imperial under Account → Units and appearance, then subsequent displays of body measurements, body weight, and workout load — forms, saved-value confirmations, and history — render converted values with the imperial unit suffix adjacent to each value, and every previously stored row remains byte-identical (value and entered unit unchanged).
2. **AC-02 — Boundary or failure behavior**: Given an entry submitted under an imperial preference, when its value's metric equivalent is checked against the FR-8.9 named-constant ranges (e.g., a body weight whose kg equivalent falls below 20 or above 500, or a length whose cm equivalent falls outside 10–300), then out-of-range values are rejected naming the specific field and in-range values are stored as entered — the imperial value with its imperial unit, to at most one decimal place — never as a pre-converted metric rewrite.
3. **AC-03 — Prohibited behavior**: Given any preference change, when it is processed, then the system MUST NOT mutate, rewrite, or re-round any stored logged value or its stored unit, MUST NOT apply conversion more than once in any display path, and MUST NOT accept a unit outside the allow-list (cm/kg for metric entry, in/lb for imperial entry) or a preference value other than metric or imperial.
4. **AC-04 — Additional criterion**: Given mixed history — entries logged under metric and entries logged under imperial on the same account — when history is displayed under either preference, then every value renders in the current preference's units via display-time conversion from its stored value and unit, and body-fat percentage renders identically under both preferences.

## Failure Behavior

- **On Invalid Input**: An unrecognized unit or preference value is rejected by allow-list schema validation (SEC-INPUT-1) with the specific failing field; no state change occurs.
- **On Authentication Failure**: Deny per REQ-AUTHZ-010; the preference is never readable or writable without a session.
- **On Authorization Failure**: Deny — only the owning account may read or change its preference (SEC-AUTHZ-2); no existence disclosure.
- **On Security-Decision Failure**: Deny by default; if the entered unit cannot be resolved for conversion, the write is refused rather than assumed metric.
- **On External Dependency Failure**: N/A — conversion is arithmetic within the REST API Application.
- **On System Error**: A failed preference update rolls back atomically; display paths that cannot resolve the preference fail with a generic error and correlation identifier (SEC-ERR-1) rather than guessing a unit system.
- **Logging / Audit**: The preference update is an account modification logged as a structured event without health values (SEC-LOG-3, SEC-LOG-4); reads and writes of the log entries it governs carry their own FR-9.7 audit entries under the owning logging issues.
- **Alerting**: N/A — a unit-preference change has no alert condition of its own; anomalous write patterns are covered by the logging issues' SEC-LOG-4 events, whose threshold alerts route to the security lead as SEC-OPS-2 detection inputs.

## Test Strategy

- **Unit Tests**: Conversion functions in both directions with one-decimal-place storage rounding; metric-equivalent computation used by FR-8.9 range checks at the exact boundaries (20/500 kg, 10/300 cm, 0/600 kg load) from imperial inputs; allow-list rejection of unknown units and preference values; body-fat pass-through.
- **Integration Tests**: Preference switch followed by verification that stored rows are unchanged and displayed values are converted; mixed-unit history rendering under both preferences; entry stored with entered value and unit; preference persisted per account and applied across sessions.
- **Security Tests**: Forged-unit payloads attempting to smuggle out-of-range values past validation; mass-assignment probes attempting to set another account's preference or rewrite a stored value/unit through the preference endpoint; client-supplied "converted" values asserting the server ignores them.
- **Compliance Tests / Evidence**: Evidence that export (REQ-PRIVACY-080) carries stored values with their stored units, demonstrating unmutated records.
- **Acceptance-Criteria Traceability**: AC-01 — preference-switch integration suite; AC-02 — metric-equivalent boundary suite; AC-03 — immutability and allow-list suites; AC-04 — mixed-history rendering suite.
- **Coverage Target**: Project coverage threshold is 90% line and branch (`CLAUDE.md`, 2026-08-03); every conversion path and boundary MUST have positive and negative tests regardless.
- **Required Test Environment**: Vitest with PostgreSQL fixtures containing mixed metric and imperial entries; an HTTP test client; Playwright for the Account → Units and appearance flow and unit-suffix rendering.

## Dependencies

- **Upstream Requirements**: REQ-AUTH-080 (the account carrying the preference), REQ-API-010 (allow-list schema validation), REQ-AUTHZ-010, REQ-AUTHZ-020
- **Downstream Requirements**: REQ-PROGRESS-010, REQ-PROGRESS-020, REQ-PROGRESS-040 (store value-with-unit and validate against metric equivalents), REQ-PROGRESS-060 (renders converted history), REQ-PRIVACY-080 (exports stored values with units)
- **External Dependencies**: None
- **Dependency Assumptions**: Logging issues call this issue's conversion for validation rather than re-implementing it, so ranges are enforced identically everywhere.
- **Failure Impact**: A wrong or double conversion silently corrupts displayed health data; a mutation-on-switch bug destroys the exact logged history FR-8.14 requires for the lifetime of the account.

## Implementation Notes

- **Constraints**: TypeScript on Fastify (`CLAUDE.md`); conversion factors and the FR-8.9 ranges are named constants in one shared module (`packages/shared` may carry the contract shape, but the enforced rule lives in `apps/api` per `CLAUDE.md`).
- **Prohibited Approaches**: Normalizing storage to metric by converting entered imperial values (violates FR-8.10's store-as-entered rule); trusting a client-computed conversion; applying range checks to the raw entered number instead of its metric equivalent; batch-rewriting rows on preference change; inferring a unit when none is supplied.
- **Implementation Guidance**: Model each stored quantity as (value, unit) and expose a single display-conversion helper the client consumes via API responses; round only at display and at the one-decimal-place storage boundary, never both cumulatively on the stored value.
- **AI Development Guidance**: `REF-PROMPT-NODE`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Architecture review that no write path stores a converted value; test review of the conversion boundary cases.
- **Open Decisions**: None. The initial preference question is RESOLVED (FR-8.10, 2026-08-03): the unit system is a required choice at registration, no silent default exists, and the choice remains changeable in Account settings.

**Estimated effort**: 0.5–1 engineer-days. **Estimated changed lines**: 200–450.
**Recommended model**: Claude Opus (`claude-opus-5`) — small surface, but the immutability and metric-equivalent rules must be exact.
