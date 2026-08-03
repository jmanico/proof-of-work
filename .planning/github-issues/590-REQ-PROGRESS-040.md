# [REQ-PROGRESS-040] Body measurement entry logging

## Metadata

- **ID**: REQ-PROGRESS-040
- **Title**: Body measurement entry logging
- **Version**: 1.0.1
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-8.2, FR-8.7, FR-8.8, FR-8.9, FR-8.10, FR-9.1, FR-9.7, FR-9.12

## Requirement

- **Statement**: A subscriber MUST be able to log a body measurement entry with a date carrying any subset of the fields waist, chest, hips, upper arm, thigh (each a length in the account's unit system), and body-fat percentage (unitless, within 0–100) — where an entry MUST carry at least one value and each field holds a single value — MUST be able to edit and delete their own entries and record entries for past dates, and the system MUST reject invalid values per FR-8.9, scope every entry to its owning subscriber, and audit every access and modification.
- **Rationale**: FR-8.2 defines the measurement fields, their per-entry optionality, the at-least-one-value floor, and the single-value (no laterality) shape; FR-8.7 grants edit and delete of one's own entries; FR-8.8 permits past dates and forbids future ones; FR-8.9 supplies the rejection classes and plausibility ranges; FR-9.1 scopes entries to their owner; FR-9.7 requires one audit entry per request for each access or modification; FR-9.12 classifies measurement entries as health data, binding the consent and email-verification gates.
- **Assumptions**: Consent has been captured and is active (REQ-PRIVACY-010, REQ-PRIVACY-020) and the account's email address is verified (REQ-AUTH-090), per the FR-9.12 health-data-write gates. The unit-system preference and display-only conversion exist (REQ-PROGRESS-050). The shared validation and error-presentation contract exists (REQ-PROGRESS-030).
- **Out of Scope**: Body weight entries (REQ-PROGRESS-010); workout logging (REQ-PROGRESS-020); food logging (REQ-FOOD-020); history display over time (REQ-PROGRESS-060); the unit-system preference itself and the conversion mechanics (REQ-PROGRESS-050); the generic validation and error-presentation contract (REQ-PROGRESS-030); the health-data definition that fixes the gates this issue relies on (REQ-PRIVACY-070); left/right laterality per field, which FR-8.2 explicitly excludes in v1.
- **Design Traceability**: `DESIGN.md` — Core Components → Forms and validation ("Numeric health fields use the data face and an adjacent unit suffix tied to the account preference (FR-8.10)"; dates use a visible calendar control plus an editable text value; inline validation with an error icon and a specific correction; focus moves to the first invalid field); Product Patterns → Logging and AI-assisted food estimates ("Logging opens directly to the chosen entry type, with the date near the top and Save kept in a stable location. The interface does not prefill a previous health value. A successful save shows the recorded value and an Edit link"); Core Components → Actions (destructive delete uses an `error`-treatment button with explicit verb and confirmation); Typography (tabular figures; values and units do not wrap apart).
- **Architecture Traceability**: `ARCHITECTURE.md` — data flow 4 ("consent check, owner scoping, field validation → log entry written → audit entry written"); REST API Application ("Owns … body measurement entry"); Relational Persistence; DR-9 (audit writing is a dependency of every health-data access path).
- **Security Traceability**: SEC-DATA-2, SEC-AUTHN-8 (verified-email gate), SEC-AUTHZ-2, SEC-INPUT-1, SEC-INPUT-2, SEC-INPUT-3, SEC-LOG-1, SEC-DATA-5.

## Scope

- **Applies To**: Server-Side Application, API, Web Client
- **Components**: REST API Application; Relational Persistence; Browser Client (measurement logging view)
- **Interfaces / Operations**: Create, read, update, and delete a body measurement entry
- **Actors**: Subscriber (owner); consultant reads occur only through REQ-CONSULT-010/REQ-CONSULT-040 scoping and are not granted here; `admin` has no access (FR-10.3)
- **Preconditions**: Authenticated subscriber session; active subscription entitlement (REQ-ENTITLE-010); recorded, unwithdrawn consent and verified email for create operations (FR-9.12)
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, Washington My Health My Data and comparable state consumer-health laws, and the FTC Health Breach Notification Rule for US users; HIPAA not applicable (`SECURITY.md` SQ-1)

## Security Context

- **Security Objectives**: Confidentiality, Integrity, Accountability, Privacy
- **Control Layers**: Authorization, Input Validation, Business-Rule Validation, Data Protection, Logging and Monitoring
- **Threat References**: `SECURITY.md` TM-I-1 (BOLA against another subscriber's entries), TM-T-1 (tampered payloads, mass assignment of owner or audit fields), TM-P-2 (writes despite withdrawn consent); CWE-639 (authorization bypass through user-controlled key), CWE-20 (improper input validation)
- **Abuse / Misuse Case**: A malicious subscriber addresses another subscriber's measurement entry by identifier to read, alter, or delete it; a client submits implausible or future-dated values to corrupt history; a script writes health data for an account that never consented or never verified its address.
- **Trust Boundary**: Boundary 1 (Browser Client → REST API Application); boundary 3 for the persisted write.
- **Untrusted Inputs or Assertions**: Every field of the entry payload, the entry date, the addressed entry identifier, and any client assertion of ownership, consent state, or verification state.
- **Authoritative Enforcement Point**: REST API Application — consent and verification gates, owner scoping, and field validation before any write; queries constrained by owner (SEC-AUTHZ-2).
- **Independent Verification**: Ownership, consent state, and email-verification state are read from persisted state and Identity and Session Handling, never from the request (DR-3).
- **Zero Trust Relevance**: TO BE DECIDED — specific NIST SP 800-207 tenet mapping not verified against the publication (`SECURITY.md` SQ-10 gates all mapping work).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — per-issue mapping deferred to the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A — no AI-enabled component in this flow.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — per-issue mapping deferred (`SECURITY.md` SQ-10).
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: GDPR/UK GDPR, CCPA/CPRA, Washington My Health My Data, FTC HBNR per `SECURITY.md` SQ-1; specific article/section citations TO BE DECIDED pending the SQ-1 counsel review.
- **Other**: `REF-INPUT`, `REF-API-2023` as cited by SEC-INPUT-2 and SEC-AUTHZ-2.
- **Mapping Basis**: The regime set is fixed by SQ-1 for all health-data behavior; the cited references are those `SECURITY.md` names for the validation and object-authorization rules this issue implements.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated, entitled subscriber with recorded consent and a verified email, when they save a measurement entry dated today or in the past carrying any non-empty subset of waist, chest, hips, upper arm, thigh, and body-fat percentage with in-range values, then the entry is persisted with each value stored to at most one decimal place together with the unit it was entered in (FR-8.10), the client shows the recorded values with an Edit link, and exactly one audit entry records the acting account, action, affected subject, and time.
2. **AC-02 — Boundary or failure behavior**: Given a create or update payload, when a length field's metric equivalent is exactly 10 cm or 300 cm or body-fat is exactly 2 or 75, then the value is accepted; when a length's metric equivalent falls below 10 cm or above 300 cm, body-fat falls outside 2–75 (including values such as 80 that satisfy the 0–100 bound but fail plausibility), a value is non-numeric or negative, the entry date is in the future, or the entry carries no values at all, then the request is rejected identifying the specific invalid field (or the at-least-one-value violation) without disclosing internal state, and nothing is persisted.
3. **AC-03 — Prohibited behavior**: Given subscriber A's measurement entry, when subscriber B attempts to read, edit, or delete it by identifier, then the operation is denied without revealing whether the entry exists; and given an account without recorded consent, with consent withdrawn, or with an unverified email address, when it submits a new measurement entry, then the write is refused (FR-9.12) — while editing or deleting that account's existing entries remains available under withdrawal as the FR-9.9 correction right.
4. **AC-04 — Additional criterion**: Given an existing entry, when the owner edits a value or deletes the entry (delete requiring explicit confirmation in the client), then the change is applied only to that entry, is validated identically to creation on edit, produces exactly one audit entry per request, and payload fields outside the declared schema — including any owner, audit, or laterality field — are rejected rather than bound (SEC-INPUT-1, SEC-INPUT-3).

## Failure Behavior

- **On Invalid Input**: Reject with the field-level error contract of REQ-PROGRESS-030 (HTTP 4xx, specific failing field, no coercion, no internal detail); no entry is created or modified.
- **On Authentication Failure**: Deny per REQ-AUTHZ-010 with a uniform response; no existence disclosure.
- **On Authorization Failure**: Deny; the response MUST NOT reveal whether the addressed entry exists (SEC-AUTHZ-2).
- **On Security-Decision Failure**: Deny by default — an error resolving consent state, verification state, or ownership refuses the write (SEC-AUTHZ-7 discipline).
- **On External Dependency Failure**: N/A — no external dependency participates in this flow.
- **On System Error**: The write is atomic with its audit entry (DR-9); on failure the transaction rolls back, the client receives a generic error with a correlation identifier (SEC-ERR-1), and no partial entry persists.
- **Logging / Audit**: One audit entry per request for every access and modification, recording acting account, action, affected subject, and time (FR-9.7, SEC-LOG-1); audit entries and logs never contain the measurement values themselves (SEC-LOG-3).
- **Alerting**: Repeated validation failures and authorization denials surface through SEC-LOG-4 structured events; threshold alerts route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-3, SQ-11).

## Test Strategy

- **Unit Tests**: At-least-one-value rule; per-field optionality; single-value shape; plausibility boundaries at 10/300 cm metric equivalent and 2/75 body-fat plus the 0–100 bound; one-decimal-place storage; future-date rejection; imperial-entry values checked against their metric equivalent.
- **Integration Tests**: Create/read/update/delete round trips including backdated entries; consent, withdrawal, and unverified-email gates on create; edit-under-withdrawal allowed; audit entry emitted exactly once per request; stored value-plus-unit persistence.
- **Security Tests**: BOLA suite — subscriber B against subscriber A's entry identifiers for read, edit, and delete, asserting denial without existence disclosure; mass-assignment probes submitting owner, audit, and undeclared laterality fields; write attempts as consultant and admin asserting denial (FR-10.3, FR-11.6).
- **Compliance Tests / Evidence**: Audit-trail evidence that every access and modification produced the FR-9.7 fields; consent-gate refusal evidence for the SQ-1 regimes.
- **Acceptance-Criteria Traceability**: AC-01 — create round-trip suite; AC-02 — boundary and rejection suite; AC-03 — BOLA and gate suites; AC-04 — edit/delete and mass-assignment suites.
- **Coverage Target**: Project coverage threshold is 90% line and branch (`CLAUDE.md`, 2026-08-03); all validation boundaries, gates, and denial paths MUST have positive and negative tests regardless.
- **Required Test Environment**: Vitest with an HTTP test client and PostgreSQL fixtures; accounts in consented, unconsented, withdrawn, and unverified states; two subscriber identities for BOLA tests; Playwright for the client form, error presentation, and delete confirmation.

## Dependencies

- **Upstream Requirements**: REQ-PROGRESS-010 (logging pattern and shared audit/scoping plumbing), REQ-PROGRESS-030 (validation and error contract), REQ-PROGRESS-050 (unit preference, stored unit, metric-equivalent checks), REQ-PRIVACY-010, REQ-PRIVACY-020, REQ-AUTH-090, REQ-AUTHZ-010, REQ-AUTHZ-020, REQ-AUDIT-020, REQ-ENTITLE-010
- **Downstream Requirements**: REQ-PROGRESS-060 (measurement trend charts), REQ-PRIVACY-080 (export includes these entries), REQ-PRIVACY-090 (deletion removes them), REQ-CONSULT-040 (consultant reads them under engagement)
- **External Dependencies**: None
- **Dependency Assumptions**: REQ-PROGRESS-050 exposes the metric-equivalent conversion used by validation so ranges are checked consistently for metric and imperial entries.
- **Failure Impact**: Without owner scoping or the gates, another subscriber's health data is readable or writable; without validation, implausible values corrupt lifetime history that FR-8.14 preserves at full granularity.

## Implementation Notes

- **Constraints**: TypeScript on Fastify with route-level JSON Schema validation and schema-bound response serialization (`CLAUDE.md`); Drizzle ORM against PostgreSQL; plausibility ranges are named constants sourced from FR-8.9.
- **Prohibited Approaches**: Client-only validation (DR-2); filtering another subscriber's rows after retrieval instead of constraining the query (SEC-AUTHZ-2); mutating stored values during unit conversion (FR-8.10); accepting undeclared fields; skipping the audit write on any path (DR-9); prefilling the previous measurement value in the form (`DESIGN.md`, Logging).
- **Implementation Guidance**: Reuse the entry-logging module structure established by REQ-PROGRESS-010; represent the six fields as nullable columns with a check constraint enforcing at least one non-null value; validate against the metric equivalent before storing the entered value and unit.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-NODE`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the owner-scoping and gate enforcement; privacy review that audit entries carry no measurement values.
- **Open Decisions**: None.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 350–700.
**Recommended model**: Claude Opus (`claude-opus-5`) — a well-patterned CRUD flow whose risk concentrates in gates, scoping, and boundary validation.
