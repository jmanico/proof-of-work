# [REQ-PROGRESS-030] Log entry validation and field-level error reporting

## Metadata

- **ID**: REQ-PROGRESS-030
- **Title**: Log entry validation and field-level error reporting
- **Version**: 1.2.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-8.9, FR-8.8; `SECURITY.md` SEC-INPUT-2, SEC-INPUT-1; `DESIGN.md` Core Components → Forms and validation

## Requirement

- **Statement**: The system MUST reject a log entry whose required value is non-numeric, negative, or absent, whose date is in the future — later than the current calendar date evaluated at UTC+14, the named-constant earliest civil timezone (FR-8.9, fixed 2026-08-03) — or whose value falls outside its named-constant plausibility range — body weight 20–500 kg, body-measurement lengths 10–300 cm, body-fat percentage 2–75, workout load 0–600 kg, sets per exercise 1–100 and repetitions per set 1–1000 (both integers), each checked against the metric equivalent of the entered value — MUST report the specific invalid field to the user without disclosing internal state, and the client MUST present that error inline beside the failing field with icon and text, programmatically associated with the field, moving focus to the first invalid field on submit failure.
- **Rationale**: FR-8.9 requires rejection with the non-numeric, negative, absent-required, future-dated, and out-of-range classes, fixes the plausibility ranges as named constants, and requires storage to at most one decimal place for weights and lengths; FR-8.8 supplies the future-date rule; SEC-INPUT-2 restates the reporting contract with the non-disclosure constraint, and SEC-INPUT-1's range checks source from FR-8.9; `DESIGN.md` fixes how the error is presented and announced. Together they make one testable contract that every log entry type reuses.
- **Assumptions**: Boundary schema validation exists (REQ-API-010); this issue defines the log-entry-specific numeric rules and the shared presentation contract.
- **Out of Scope**: The entity-specific field sets, which belong to each log type — body weight (REQ-PROGRESS-010), workouts (REQ-PROGRESS-020), measurements (FR-8.2, REQ-PROGRESS-040), food (FR-8.4, REQ-FOOD-020); the generic error response shape for system faults (REQ-API-040); the per-account unit-system preference and display conversion (FR-8.10, REQ-PROGRESS-050 — this issue consumes the entered unit only to compute the metric equivalent for range checks).
- **Design Traceability**: `DESIGN.md` — Core Components → Forms and validation ("Validation appears inline with an error icon and a specific correction. After failed submission, a summary at the top links to each invalid field and focus moves to the first invalid field (FR-8.9)"; persistent visible labels, unit suffix tied to the account preference, required marked with text); Color System (`error` token reserved for invalid, failed, or destructive states); Accessibility (color independence; errors use assertive announcements only when submission fails).
- **Architecture Traceability**: `ARCHITECTURE.md` — DR-2 ("Server responses MUST carry enough structured detail (failing field, reason) for the client to present errors per DESIGN.md without re-deriving the rule"); Browser Client ("inline field-level error presentation with focus moved to the first invalid field"); REST API Application outputs.
- **Security Traceability**: SEC-INPUT-2, SEC-INPUT-1, SEC-ERR-1, SEC-RENDER-4 (error text surfaced to assistive technology must not expose unauthorized data).

## Scope

- **Applies To**: Server-Side Application, API, Web Client
- **Components**: REST API Application; Browser Client (all log entry forms)
- **Interfaces / Operations**: Validation of every log entry create and update; the shared client error presentation for those forms
- **Actors**: `subscriber`
- **Preconditions**: Authenticated session
- **Data Classification**: Restricted — the values being validated are health data (FR-9.12)
- **Personal or Regulated Data**: Health Data (FR-9.12)
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, Washington My Health My Data, and the FTC Health Breach Notification Rule for US users; HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED)

## Security Context

- **Security Objectives**: Integrity, Confidentiality, Privacy
- **Control Layers**: Input Validation, Business-Rule Validation, Output Encoding
- **Threat References**: `SECURITY.md` TM-T-1 (tampered payloads), TM-I-5 (health data leaking through error surfaces); CWE-20 (improper input validation), CWE-209 (information exposure through an error message), CWE-1287 (improper validation of specified type of input)
- **Abuse / Misuse Case**: A negative or non-numeric value is coerced and stored, corrupting a health record; or the error message echoes internal state, a stack frame, or another subject's data back to the user or to assistive technology.
- **Trust Boundary**: Boundary 1 — client-side validation is convenience only and is re-performed server-side (DR-2).
- **Untrusted Inputs or Assertions**: Every submitted numeric value and date; any client claim that validation already passed.
- **Authoritative Enforcement Point**: REST API Application — the server rejects and names the field; the client presents what the server returned.
- **Independent Verification**: The same rules are asserted by direct API tests that bypass the client entirely.
- **Zero Trust Relevance**: N/A — `REQUIREMENT_TEMPLATE.md` forbids using Zero Trust as a synonym for input validation.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — verified during the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — verified during the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **NIST SP 800-207**: N/A
- **Regulatory**: GDPR/UK GDPR (EU/UK data subjects); CCPA/CPRA, Washington My Health My Data, FTC Health Breach Notification Rule (US users); HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED) — the validated values are health data (FR-9.12), though rejected submissions are never stored. Statute-section precision: TO BE DECIDED — no source document states sections for this requirement.
- **Other**: `REF-INPUT`, `REF-ERROR` as cited by SEC-INPUT-2 and SEC-ERR-1; WCAG 2.2 AA as adopted by `DESIGN.md` for the error association and announcement behavior.
- **Mapping Basis**: FR-8.9 and SEC-INPUT-2 are the normative sources and name these references; `DESIGN.md` supplies the presentation obligations under its stated WCAG 2.2 AA target.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a log entry submission with all required values numeric, non-negative, and present, when it is validated, then validation passes and the entry proceeds to the owner-scoping and consent checks.
2. **AC-02 — Boundary or failure behavior**: Given a submission where a required value is absent, non-numeric, or negative, where the date is in the future (FR-8.8), or where a value falls outside its FR-8.9 plausibility range when checked against the metric equivalent of the entered value, when it is validated, then it is rejected, the response identifies the specific invalid field and the rule it violated, and nothing is persisted.
3. **AC-03 — Prohibited behavior**: Given any validation failure, when the response is produced, then it MUST NOT disclose internal state, stack frames, database detail, or any data belonging to another subject; the value MUST NOT be coerced into a valid one; and the client MUST NOT be the only place a rule is enforced (DR-2).
4. **AC-04 — Additional criterion**: Given a submit failure in the client, when the errors render, then each appears inline adjacent to its field using the `error` token with an icon and text, is programmatically associated with the field, is announced to assistive technology, a summary at the top links to each invalid field, and focus moves to the first invalid field.
5. **AC-05 — Additional criterion**: Given a boundary value of exactly zero, when it is submitted for a field where zero is inside the field's FR-8.9 range — workout load's range is 0–600 kg — then it is accepted; where zero falls outside the range — body weight below 20 kg, lengths below 10 cm, body-fat below 2 — the out-of-range rule rejects it and names the field.
6. **AC-06 — Additional criterion**: Given the plausibility ranges, when they are implemented, then each is a named constant per FR-8.9 — body weight 20–500 kg, lengths 10–300 cm, body-fat percentage 2–75 within FR-8.2's 0–100 bound, workout load 0–600 kg — the comparison uses the metric equivalent of the entered value, values exactly on a bound are accepted, and accepted weights and lengths are stored to at most one decimal place.

## Failure Behavior

- **On Invalid Input**: This issue defines that behavior — reject, name the field and rule, persist nothing.
- **On Authentication Failure**: Denied upstream by REQ-AUTHZ-010.
- **On Authorization Failure**: Denied upstream by REQ-AUTHZ-020; a validation message MUST NOT be used to reveal that a foreign record exists.
- **On Security-Decision Failure**: If a field's rule cannot be resolved, reject the submission (fail closed) rather than accept an unvalidated value.
- **On External Dependency Failure**: N/A — validation precedes persistence.
- **On System Error**: Generic error with a correlation identifier (REQ-API-040); the structured field detail is reserved for genuine validation outcomes.
- **Logging / Audit**: Log validation rejections by field name and rule class only — never the submitted value, which is health data (SEC-LOG-3). No audit entry is written for a rejected submission, since no health data was accessed or modified.
- **Alerting**: N/A — validation rejections are not a SEC-LOG-4 security-event class, and no source document attaches an alert condition to them; authentication and authorization alerting belongs to the REQ-AUTHZ and REQ-SESSION issues, routed to the security lead per SEC-OPS-2.

## Test Strategy

- **Unit Tests**: The numeric rule accepts valid values, rejects absent, non-numeric, negative, infinite, over-precision, future-dated, and out-of-range values; each range constant tested at both bounds, just inside, and just outside, in metric and imperial entry units; the error builder produces the field identifier and rule class without internal detail.
- **Integration Tests**: Direct API submissions bypassing the client for each log type, asserting rejection and correct field attribution; multi-field failures return all failing fields.
- **Security Tests**: Assertion that no rejection message contains internal detail or foreign-subject data; assertion that no submitted value appears in a log record; coercion negative test confirming `"12abc"` and `"1e5"` are not silently converted.
- **Compliance Tests / Evidence**: Accessibility evidence for the error presentation — association, announcement, summary links, focus movement — retained under the WCAG 2.2 AA target.
- **Acceptance-Criteria Traceability**: AC-01 — happy-path validation; AC-02 — rejection matrix per field including future-date and out-of-range classes; AC-03 — disclosure and coercion negatives plus direct-API enforcement; AC-04 — client error presentation and focus tests; AC-05 — zero-boundary tests per field range; AC-06 — named-constant, metric-equivalent, bound, and precision tests.
- **Coverage Target**: Every numeric field of every log type covered by the full boundary matrix, including both bounds of its FR-8.9 range.
- **Required Test Environment**: A subscriber identity, forms for each log type, assistive-technology testing capability. Vitest as the runner, with Playwright and axe-core where a real browser is required.

## Dependencies

- **Upstream Requirements**: REQ-API-010, REQ-API-040, REQ-PLATFORM-010, REQ-PLATFORM-030, REQ-PROGRESS-050 (the entered unit feeds the metric-equivalent range check)
- **Downstream Requirements**: REQ-PROGRESS-010, REQ-PROGRESS-020, REQ-PROGRESS-040, REQ-FOOD-020, REQ-FOOD-040 (estimation pre-fills are validated per FR-8.9 on save)
- **External Dependencies**: None
- **Dependency Assumptions**: The boundary validation layer (REQ-API-010) can express per-field numeric constraints and return structured failures.
- **Failure Impact**: Silent coercion corrupts health records in a way that later reads cannot distinguish from genuine data.

## Implementation Notes

- **Constraints**: Node.js runtime with Fastify (`CLAUDE.md`); the client is a Vite-built single-page application with `vue-router`. The unit system is resolved (`REQUIREMENTS.md` OQ-4, `DESIGN.md` OQ-8 — FR-8.10): validation converts the entered value to its metric equivalent and compares it against the FR-8.9 named constants, so the constants are expressed once, in metric, regardless of the entry unit. The conversion used for range checking never mutates the stored value (FR-8.10).
- **Prohibited Approaches**: Coercing strings to numbers; accepting `NaN` or infinity; a single generic "invalid input" message that does not name the field; enforcing the rule only in the client; echoing the submitted value back in the error message; inlining range literals per endpoint instead of the FR-8.9 named constants; duplicating the constants per unit system.
- **Implementation Guidance**: Express the rules once, server-side, and let the client render the returned structure (DR-2). The client may pre-validate for immediate feedback, but its rules must be derived from the same declaration rather than restated. Fastify route-level JSON Schema validation is the delivery mechanism for the boundary checks (SEC-INPUT-1, `CLAUDE.md`).
- **AI Development Guidance**: `REF-INPUT`, `REF-ERROR`, `REF-PROMPT-VUE`, `REF-PROMPT-API`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Accessibility review of the error presentation; security review of the message content.
- **Open Decisions**: None — `REQUIREMENTS.md` OQ-4 and OQ-5 are RESOLVED. The plausibility ranges are fixed as FR-8.9 named constants; the measurement field set is FR-8.2 (REQ-PROGRESS-040) and the food-log field set is FR-8.4 (REQ-FOOD-020), both extending this contract.

**Estimated effort**: 0.5–1.5 engineer-days. **Estimated changed lines**: 200–450.
**Recommended model**: Claude Opus (`claude-opus-5`) — a shared contract spanning server rules and accessible client presentation, where the disclosure limits are easy to violate while satisfying the functional requirement.
