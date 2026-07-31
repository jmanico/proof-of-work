# [REQ-PROGRESS-030] Log entry validation and field-level error reporting

## Metadata

- **ID**: REQ-PROGRESS-030
- **Title**: Log entry validation and field-level error reporting
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-8.9; `SECURITY.md` SEC-INPUT-2; `DESIGN.md` Components → Form feedback and errors

## Requirement

- **Statement**: The system MUST reject a log entry whose required value is non-numeric, negative, or absent, MUST report the specific invalid field to the user without disclosing internal state, and the client MUST present that error inline beside the failing field with icon and text, programmatically associated with the field, moving focus to the first invalid field on submit failure.
- **Rationale**: FR-8.9 requires rejection and specific-field reporting; SEC-INPUT-2 restates it with the non-disclosure constraint; `DESIGN.md` fixes how the error is presented and announced. Together they make one testable contract that every log entry type reuses.
- **Assumptions**: Boundary schema validation exists (REQ-API-010); this issue defines the log-entry-specific numeric rules and the shared presentation contract.
- **Out of Scope**: The entity-specific field sets, which belong to each log type — body weight (REQ-PROGRESS-010), workouts (REQ-PROGRESS-020), measurements (blocked by `REQUIREMENTS.md` OQ-4), food (blocked by OQ-5); the generic error response shape for system faults (REQ-API-040); the unit system (OQ-4, `DESIGN.md` OQ-8).
- **Design Traceability**: `DESIGN.md` — Components → Form feedback and errors ("Validation errors appear inline, adjacent to the field that failed, in `error`, paired with an icon and text — never color alone. The message names the specific problem and how to fix it… Errors are programmatically associated with their field and announced to assistive technology. On submit failure, focus moves to the first invalid field"); Components → Inputs (persistent visible labels, units adjacent to numeric fields, required marked in the label); Color Palette (`error` token); Accessibility (color independence).
- **Architecture Traceability**: `ARCHITECTURE.md` — DR-2 ("Server responses MUST carry enough structured detail (failing field, reason) for the client to present errors per DESIGN.md without re-deriving the rule"); Browser Client ("inline field-level error presentation with focus moved to the first invalid field"); REST API Application outputs.
- **Security Traceability**: SEC-INPUT-2, SEC-INPUT-1, SEC-ERR-1, SEC-RENDER-4 (error text surfaced to assistive technology must not expose unauthorized data).

## Scope

- **Applies To**: Server-Side Application, API, Web Client
- **Components**: REST API Application; Browser Client (all log entry forms)
- **Interfaces / Operations**: Validation of every log entry create and update; the shared client error presentation for those forms
- **Actors**: `subscriber`
- **Preconditions**: Authenticated session
- **Data Classification**: Restricted — the values being validated are health data
- **Personal or Regulated Data**: Health Data
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED (`SECURITY.md` SQ-1)

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

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: N/A
- **Regulatory**: N/A
- **Other**: `REF-INPUT`, `REF-ERROR` as cited by SEC-INPUT-2 and SEC-ERR-1; WCAG 2.2 AA as adopted by `DESIGN.md` for the error association and announcement behavior.
- **Mapping Basis**: FR-8.9 and SEC-INPUT-2 are the normative sources and name these references; `DESIGN.md` supplies the presentation obligations under its stated WCAG 2.2 AA target.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a log entry submission with all required values numeric, non-negative, and present, when it is validated, then validation passes and the entry proceeds to the owner-scoping and consent checks.
2. **AC-02 — Boundary or failure behavior**: Given a submission where a required value is absent, non-numeric, or negative, when it is validated, then it is rejected, the response identifies the specific invalid field and the rule it violated, and nothing is persisted.
3. **AC-03 — Prohibited behavior**: Given any validation failure, when the response is produced, then it MUST NOT disclose internal state, stack frames, database detail, or any data belonging to another subject; the value MUST NOT be coerced into a valid one; and the client MUST NOT be the only place a rule is enforced (DR-2).
4. **AC-04 — Additional criterion**: Given a submit failure in the client, when the errors render, then each appears inline adjacent to its field using the `error` token with an icon and text, is programmatically associated with the field, is announced to assistive technology, and focus moves to the first invalid field.
5. **AC-05 — Additional criterion**: Given a boundary value of exactly zero, when it is submitted for a field where zero is meaningful, then it is accepted — FR-8.9 forbids negative values, not zero — and where zero is not meaningful for a field, that field's own rule states so explicitly rather than relying on the shared numeric rule.

## Failure Behavior

- **On Invalid Input**: This issue defines that behavior — reject, name the field and rule, persist nothing.
- **On Authentication Failure**: Denied upstream by REQ-AUTHZ-010.
- **On Authorization Failure**: Denied upstream by REQ-AUTHZ-020; a validation message MUST NOT be used to reveal that a foreign record exists.
- **On Security-Decision Failure**: If a field's rule cannot be resolved, reject the submission (fail closed) rather than accept an unvalidated value.
- **On External Dependency Failure**: N/A — validation precedes persistence.
- **On System Error**: Generic error with a correlation identifier (REQ-API-040); the structured field detail is reserved for genuine validation outcomes.
- **Logging / Audit**: Log validation rejections by field name and rule class only — never the submitted value, which is health data (SEC-LOG-3). No audit entry is written for a rejected submission, since no health data was accessed or modified.
- **Alerting**: N/A

## Test Strategy

- **Unit Tests**: The numeric rule accepts valid values and zero, rejects absent, non-numeric, negative, infinite, and over-precision values; the error builder produces the field identifier and rule class without internal detail.
- **Integration Tests**: Direct API submissions bypassing the client for each log type, asserting rejection and correct field attribution; multi-field failures return all failing fields.
- **Security Tests**: Assertion that no rejection message contains internal detail or foreign-subject data; assertion that no submitted value appears in a log record; coercion negative test confirming `"12abc"` and `"1e5"` are not silently converted.
- **Compliance Tests / Evidence**: Accessibility evidence for the error presentation — association, announcement, focus movement — retained under the WCAG 2.2 AA target.
- **Acceptance-Criteria Traceability**: AC-01 — happy-path validation; AC-02 — rejection matrix per field; AC-03 — disclosure and coercion negatives plus direct-API enforcement; AC-04 — client error presentation and focus tests; AC-05 — zero-boundary tests.
- **Coverage Target**: Every numeric field of every unblocked log type covered by the full boundary matrix.
- **Required Test Environment**: A subscriber identity, forms for each unblocked log type, assistive-technology testing capability. Client tooling and test framework TO BE DECIDED.

## Dependencies

- **Upstream Requirements**: REQ-API-010, REQ-API-040, REQ-PLATFORM-010, REQ-PLATFORM-030
- **Downstream Requirements**: REQ-PROGRESS-010, REQ-PROGRESS-020, and the blocked measurement and food-logging work when unblocked
- **External Dependencies**: None
- **Dependency Assumptions**: The boundary validation layer (REQ-API-010) can express per-field numeric constraints and return structured failures.
- **Failure Impact**: Silent coercion corrupts health records in a way that later reads cannot distinguish from genuine data.

## Implementation Notes

- **Constraints**: Node.js runtime and client tooling TO BE DECIDED. The unit system is open (`REQUIREMENTS.md` OQ-4, `DESIGN.md` OQ-8), so validation must not encode unit-specific ranges; range plausibility beyond "non-negative numeric" is not specified by any source document and MUST NOT be invented.
- **Prohibited Approaches**: Coercing strings to numbers; accepting `NaN` or infinity; a single generic "invalid input" message that does not name the field; enforcing the rule only in the client; echoing the submitted value back in the error message.
- **Implementation Guidance**: Express the rules once, server-side, and let the client render the returned structure (DR-2). The client may pre-validate for immediate feedback, but its rules must be derived from the same declaration rather than restated.
- **AI Development Guidance**: `REF-INPUT`, `REF-ERROR`, `REF-PROMPT-VUE`, `REF-PROMPT-API`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Accessibility review of the error presentation; security review of the message content.
- **Open Decisions**: Plausible value ranges and the unit system are undecided (`REQUIREMENTS.md` OQ-4); this issue implements only the rules FR-8.9 states. Measurement and food-log field sets are blocked (OQ-4, OQ-5) and will extend this contract when unblocked.

**Estimated effort**: 0.5–1.5 engineer-days. **Estimated changed lines**: 200–450.
**Recommended model**: Claude Opus (`claude-opus-5`) — a shared contract spanning server rules and accessible client presentation, where the disclosure limits are easy to violate while satisfying the functional requirement.
