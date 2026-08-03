# [REQ-FOOD-020] Food log entry with calorie and macronutrient attribution

## Metadata

- **ID**: REQ-FOOD-020
- **Title**: Food log entry with calorie and macronutrient attribution
- **Version**: 1.0.1
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-8.4, FR-6.2 (macronutrient set), FR-8.7, FR-8.8, FR-8.9, FR-9.7, FR-9.12; `SECURITY.md` SEC-INPUT-2, SEC-LOG-1

## Requirement

- **Statement**: The system MUST allow a subscriber to create, edit, and delete their own food log entries, where every entry carries a date and attributed calories and the FR-6.2 macronutrients — protein, carbohydrate, and fat, each in grams — and where an entry arriving from dataset search with a quantity, from manually typed values, or from a confirmed AI estimate is validated identically on save per FR-8.9.
- **Rationale**: FR-8.4 makes food intake a first-class progress record with a fixed macronutrient trio (FR-6.2, resolved 2026-08-03) so the FR-8.5 comparison is well defined; validating all three entry paths identically on save (FR-8.12) means no path bypasses FR-8.9, and storing attributed values on the entry itself (FR-8.11) keeps logged history immutable under dataset updates.
- **Assumptions**: Consent capture and refusal-without-consent are enforced by REQ-PRIVACY-010 and the FR-9.12 health-data binding (REQ-PRIVACY-070); email-verification gating by REQ-AUTH-090's verification flow per SEC-AUTHN-8; owner scoping by REQ-AUTHZ-020; the generic validation and inline error-presentation contract by REQ-PROGRESS-030; audit persistence by REQ-AUDIT-010.
- **Out of Scope**: Dataset import and search themselves (REQ-FOOD-010); generating the AI estimate and the transient photo flow (REQ-FOOD-040, REQ-FOOD-050); the daily target comparison (REQ-FOOD-030); weight, measurement, and workout logging (REQ-PROGRESS-010, REQ-PROGRESS-040, and the workout logging issue); the consent-withdrawal flow itself (REQ-PRIVACY-020).
- **Design Traceability**: `DESIGN.md` — "Logging and AI-assisted food estimates": three equal entry methods, AI never the default; confirmation saves the subscriber-edited values; persistent `Estimate` status chip (Components → Status vocabulary); date near the top, Save in a stable location, no prefilled previous health value (D-PRINCIPLE-2); form validation per Components → Forms and validation.
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application (owns food log entry; enforces log-entry validation FR-8.9, consent capture FR-9.2, audit writes FR-9.7); Relational Persistence (entity "food log entry"); data flow 4 (progress logging: consent check, owner scoping, field validation, log entry written, audit entry written); DR-4 (log entries mutable only by their owning subscriber); DR-9 (audit writing is a dependency of every health-data path).
- **Security Traceability**: SEC-INPUT-1, SEC-INPUT-2, SEC-INPUT-3 (allow-listed client-assignable fields), SEC-AUTHZ-2 (object-level authorization), SEC-DATA-2 (consent precondition), SEC-LOG-1/FR-9.7 (audit), SEC-AI-3 (estimate label carried end-to-end); SEC-ERR-1.

## Scope

- **Applies To**: Multiple — API, Server-Side Application, Web Client (form presentation)
- **Components**: REST API Application; Relational Persistence; Browser Client
- **Interfaces / Operations**: Food log entry create, edit, and delete operations; the food logging form with its three entry methods
- **Actors**: Subscriber (owner); consultant (read-only within an engagement, FR-11.6 — no log writes); admin (prohibited, FR-10.3); anonymous attacker (denied)
- **Preconditions**: Authenticated session; active subscription entitlement (FR-3.1); recorded, unwithdrawn health-data consent and verified email address for creation (FR-9.12 — edit and delete of existing entries remain available under withdrawn consent per FR-9.9)
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data — food log entries are health data by definition (FR-9.12)
- **Jurisdiction / Regulatory Scope**: Global service under the `SECURITY.md` SQ-1 regime set — GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, Washington My Health My Data, and the FTC Health Breach Notification Rule for US users; HIPAA evaluated and not applicable

## Security Context

- **Security Objectives**: Integrity, Confidentiality, Privacy, Accountability
- **Control Layers**: Input Validation, Business-Rule Validation, Authorization, Logging and Monitoring, Data Protection
- **Threat References**: `SECURITY.md` TM-I-1 (BOLA on log entries), TM-T-1 (tampered payloads, mass assignment), TM-I-5 (health data leaking into logs); CWE-639 (authorization bypass through user-controlled key), CWE-20 (improper input validation)
- **Abuse / Misuse Case**: Subscriber A creates, edits, or deletes an entry under subscriber B's identifier; a tampered payload sets server-controlled fields (owner, estimate label, audit fields); a client bypasses the estimate-confirmation flow to store unvalidated model output; new entries are written for an account without consent, with consent withdrawn, or with an unverified address.
- **Trust Boundary**: Boundary 1 (Browser Client → REST API Application) — all values, including confirmed estimate values, arrive as untrusted client input; boundary 3 for persistence.
- **Untrusted Inputs or Assertions**: Entry date, calories, protein, carbohydrate, fat, description, quantity, referenced dataset item identifier, and any claimed entry provenance; none is trusted until validated server-side.
- **Authoritative Enforcement Point**: REST API Application — schema validation, FR-8.9 field rules, FR-9.12 gates, and owner scoping all execute before any write; the client's inline validation is convenience only (DR-2).
- **Independent Verification**: Ownership comes from persisted state and session identity, never from the request body (DR-3, SEC-INPUT-3); the estimate label is set server-side from the flow that produced the entry, not from a client claim.
- **Zero Trust Relevance**: TO BE DECIDED — per-issue NIST SP 800-207 mapping is deferred to the independent pre-launch assessment (`SECURITY.md` SQ-10).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — per-issue mapping deferred to the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: TO BE DECIDED — this issue persists the estimate label end-to-end (SEC-AI-3); mapping deferred with SQ-10. The AI inference component itself is REQ-FOOD-040/REQ-FOOD-050.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — deferred with SQ-10.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: GDPR/UK GDPR (EU/UK data subjects), CCPA/CPRA, Washington My Health My Data, FTC Health Breach Notification Rule (US users) per `SECURITY.md` SQ-1; consent precondition per FR-9.2. Section-level mappings: TO BE DECIDED pending the SQ-1 counsel review.
- **Other**: `REF-INPUT`, `REF-LOG` as cited by SEC-INPUT-2 and SEC-LOG-1.
- **Mapping Basis**: SEC-INPUT-2 and SEC-LOG-1 name these references; the CWE identifiers describe the BOLA and input-validation classes this write path must defeat.

## Acceptance Criteria

1. **AC-01 — Expected behavior (dataset path)**: Given a consenting, email-verified, entitled subscriber, when they save a food entry from a dataset item and quantity (REQ-FOOD-010), then the entry persists with the computed calories, protein, carbohydrate, and fat stored on the entry itself, and a later dataset re-import does not alter the stored values.
2. **AC-02 — Expected behavior (manual and estimate paths)**: Given the same preconditions, when the subscriber saves manually typed values, or confirms an editable estimate pre-fill, then the entry persists with exactly the subscriber-confirmed values; an entry created from an estimate carries the `Estimate` label in persistence and in every subsequent read, and an entry with no estimate requested carries no such label.
3. **AC-03 — Boundary or failure behavior (validation)**: Given a save on any of the three paths with a non-numeric calorie value, a negative macronutrient value, an absent required value, a future date, calories outside the 0–10,000 kcal plausibility range, or any macronutrient outside 0–2,000 g (FR-8.9 named constants, fixed 2026-08-03), when validation runs, then the request is rejected naming the specific invalid field (FR-8.9, SEC-INPUT-2), nothing persists, and the client presents the error inline per REQ-PROGRESS-030.
4. **AC-04 — Boundary or failure behavior (FR-9.12 gates)**: Given an account with no consent record, withdrawn consent, or an unverified email address, when a food entry creation or an edit that adds new intake data is attempted, then it is refused with the reason class; and given withdrawn consent, when the subscriber edits or deletes an existing entry, then the operation succeeds as an exercise of the FR-9.9 correction right.
5. **AC-05 — Prohibited behavior (ownership and mass assignment)**: Given subscriber A authenticated, when a create, edit, or delete names subscriber B's entry identifier, or a request body supplies owner, estimate-label, or audit fields, then the operation is denied or the server-controlled fields are ignored (SEC-AUTHZ-2, SEC-INPUT-3), with no disclosure that B's entry exists; a consultant or admin attempting a food entry write is denied (FR-11.6, FR-10.3).
6. **AC-06 — Additional criterion (edit, delete, backdating, audit)**: Given an existing entry, when the owner edits it with valid values, deletes it, or creates an entry for a past date, then the operation succeeds; every create, edit, delete, and read request on food entries produces exactly one audit entry per request with acting account, action, affected subject, and time (FR-9.7).

## Failure Behavior

- **On Invalid Input**: Reject with the specific failing field and validation class (FR-8.9, SEC-INPUT-2, SEC-ERR-1); no partial write, no side effect. Rejection responses disclose no internal state.
- **On Authentication Failure**: Deny per REQ-AUTHZ-010 with a uniform response.
- **On Authorization Failure**: Deny per SEC-AUTHZ-2 without confirming whether the target entry exists; consultant and admin write attempts are denied and logged as authorization denials (SEC-LOG-4).
- **On Security-Decision Failure**: Deny by default — an error resolving consent state, email-verification state, entitlement, or ownership refuses the write (SEC-AUTHZ-7).
- **On External Dependency Failure**: N/A — no external dependency exists on this path; dataset values arrive from in-boundary persistence and estimates from the in-boundary flow of REQ-FOOD-040.
- **On System Error**: The entry write and its audit entry are committed atomically (DR-9); on failure, neither persists, and the response carries a generic message with a correlation identifier (SEC-ERR-1).
- **Logging / Audit**: One FR-9.7 audit entry per request (acting account, action, affected subject, time) for every food entry read and write, including the owner's own operations. Audit entries and operational logs reference the entry; they never contain food descriptions, calorie or macronutrient values, or other health values (SEC-LOG-3).
- **Alerting**: Repeated authorization denials and validation-failure bursts on this path route as threshold alerts to the security lead as SEC-OPS-2 detection inputs (SQ-11).

## Test Strategy

- **Unit Tests**: FR-8.9 validation matrix for calories and each macronutrient (non-numeric, negative, absent, future-dated); estimate-label assignment from flow context and its immutability to client input; gate evaluation for consent absent, withdrawn, and unverified-email states including the FR-9.9 edit/delete carve-out.
- **Integration Tests**: Create/edit/delete round trips through the API and PostgreSQL for all three entry paths; backdated entry accepted, future-dated rejected; dataset re-import leaves stored entries unchanged (with REQ-FOOD-010); audit entry written atomically with each operation and exactly once per request.
- **Security Tests**: BOLA suite — subscriber A against B's entry identifiers on create/edit/delete/read, asserting denial without existence disclosure; mass-assignment suite submitting owner, label, and audit fields; consultant and admin write attempts asserting denial; log-content assertion that no health value or food description appears in logs or audit entries.
- **Compliance Tests / Evidence**: Test evidence that no health data is recorded without consent (FR-9.2/SEC-DATA-2) and that audit granularity is one entry per request (FR-9.7), retained for the SQ-1 counsel review.
- **Acceptance-Criteria Traceability**: AC-01 — dataset-path suite; AC-02 — manual/estimate-path suite; AC-03 — validation matrix; AC-04 — gate suite; AC-05 — BOLA and mass-assignment suites; AC-06 — edit/delete/backdate and audit-atomicity suites.
- **Coverage Target**: Project coverage threshold is 90% line and branch (`CLAUDE.md`, 2026-08-03); every gate, validation, and denial path MUST have positive and negative coverage.
- **Required Test Environment**: Subscriber fixtures in each consent/verification state; consultant and admin fixtures; a second subscriber for BOLA tests; dataset fixture from REQ-FOOD-010; PostgreSQL; Vitest and the HTTP test client.

## Dependencies

- **Upstream Requirements**: REQ-FOOD-010 (dataset search and computed values); REQ-PROGRESS-030 (validation and field-level error-reporting contract); REQ-PRIVACY-010 (consent capture and refusal without consent)
- **Downstream Requirements**: REQ-FOOD-030 (daily totals aggregate these entries); REQ-FOOD-040 (estimate confirmation saves through this path); REQ-PRIVACY-080 (export includes these entries); REQ-PRIVACY-090 (deletion removes them)
- **External Dependencies**: None
- **Dependency Assumptions**: REQ-PRIVACY-070 binds the FR-9.12 definition into the shared gates so this path and every other health-data write refuse identically; REQ-AUDIT-010 provides append-only audit persistence.
- **Failure Impact**: If validation or the gates fail open, unconsented or malformed health data enters persistence; if audit writing is skipped, FR-9.7 accountability is lost for food intake — both are release-blocking defects.

## Implementation Notes

- **Constraints**: Fastify route-level JSON Schema validation and schema-bound response serialization (`CLAUDE.md`); values stored with at most the precision the schema declares; the entry stores its values — it never references the live dataset for display (FR-8.11).
- **Prohibited Approaches**: Client-side-only validation (DR-2); trusting a client-supplied estimate label or provenance claim; recomputing entry values from the current dataset at read time; filtering another subscriber's entries after retrieval instead of scoping the query (SEC-AUTHZ-2); logging food descriptions or nutrient values.
- **Implementation Guidance**: Model the three entry paths as one save operation with a server-derived provenance attribute, so "validated identically" is true by construction rather than by discipline. Wire the audit write as a required step of the shared health-data persistence path (DR-9), not a per-handler call.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-NODE`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the gate ordering (consent, verification, entitlement, ownership before write) and of audit atomicity; privacy review that the FR-9.9 edit/delete carve-out is scoped to existing entries only.
- **Open Decisions**: None. Calorie and macronutrient plausibility bounds beyond non-negative numeric are not specified by FR-8.9 (its named ranges cover weight, lengths, body-fat, and load); introducing bounds for food values would be a spec change, not an implementation choice.

**Estimated effort**: 1.5–2 engineer-days. **Estimated changed lines**: 400–800.
**Recommended model**: Claude Opus (`claude-opus-5`) — the food-logging write path composes consent, verification, entitlement, ownership, validation, and audit; ordering mistakes are silent privacy defects.
