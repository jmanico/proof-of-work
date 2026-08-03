# [REQ-PROGRESS-020] Workout completion logging

## Metadata

- **ID**: REQ-PROGRESS-020
- **Title**: Workout completion logging
- **Version**: 1.2.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-8.3, FR-5.4, FR-8.7, FR-8.8, FR-8.9, FR-8.10, FR-9.1, FR-9.7, FR-9.12

## Requirement

- **Statement**: A subscriber MUST be able to record completion of a workout from their plan, including the sets, repetitions, and weight used for each exercise, where each logged exercise references its FR-5.4 catalog entry and stores a snapshot of the exercise name at log time; the subscriber MUST be able to edit and delete their own workout log entries and to record one for a past date — never a future date; every entry MUST be owner-scoped and every access or modification MUST be audited.
- **Rationale**: FR-8.3 requires workout completion recording with per-exercise sets, repetitions, and weight used, each logged exercise referencing its catalog entry (FR-5.4) with a name snapshot so later catalog renames never alter logged history; FR-8.7 grants edit and delete; FR-8.8 permits past dates and rejects future dates; FR-8.9 supplies the validation classes and the 0–600 kg load range; FR-8.10 requires load values stored with their entry unit; FR-9.1 and FR-9.7 supply the scoping and audit obligations (FR-9.12).
- **Assumptions**: Consent is active (REQ-PRIVACY-010, REQ-PRIVACY-020), the account's email address is verified (FR-2.11, FR-9.12), the exercise catalog exists (REQ-PLAN-080, FR-5.4), and the subscriber holds a plan or a customized copy from which the workout is performed (REQ-CUSTOM-010, REQ-CATALOG-010).
- **Out of Scope**: Body weight (REQ-PROGRESS-010) and measurements (FR-8.2, REQ-PROGRESS-040); food logging (REQ-FOOD-010, REQ-FOOD-020); workout performance history over time (FR-8.6, FR-8.14 — REQ-PROGRESS-060, where the charted value is the top-set weight per catalog entry); the generic validation contract (REQ-PROGRESS-030); active plan selection (FR-5.3, REQ-SELECT-010); catalog management (FR-5.4, REQ-PLAN-080).
- **Design Traceability**: `DESIGN.md` — Core Components → Forms and validation (numeric health fields use the data face with an adjacent unit suffix tied to the account preference per FR-8.10), Core Components → Actions (destructive delete confirmed); Product Patterns → Logging and AI-assisted food estimates (logging opens directly to the chosen entry type, date near the top, Save stable, no prefilled previous health value); Typography and Iconography (tabular figures so per-exercise values align in columns); Layout, Spacing, and Responsive Behavior (mobile reflow of a multi-exercise form); Accessibility.
- **Architecture Traceability**: `ARCHITECTURE.md` — data flow 4; REST API Application ("Owns … workout log entry"); DR-9.
- **Security Traceability**: SEC-DATA-2, SEC-AUTHZ-2, SEC-INPUT-1, SEC-INPUT-2, SEC-INPUT-3, SEC-LOG-1, SEC-DATA-5.

## Scope

- **Applies To**: Server-Side Application, API, Web Client
- **Components**: REST API Application; Relational Persistence; Browser Client (workout logging view)
- **Interfaces / Operations**: Create, read, update, and delete a workout log entry with its per-exercise performance records
- **Actors**: `subscriber`
- **Preconditions**: Authenticated session; active consent; verified email address (FR-2.11, FR-9.12); the medical disclaimer acknowledged (REQ-PRIVACY-040); an accessible plan or customized copy
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data (FR-9.12)
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, Washington My Health My Data, and the FTC Health Breach Notification Rule for US users; HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED, `REQUIREMENTS.md` OQ-3 RESOLVED)

## Security Context

- **Security Objectives**: Confidentiality, Integrity, Privacy, Accountability
- **Control Layers**: Authorization, Input Validation, Business-Rule Validation, Data Protection, Logging and Monitoring
- **Threat References**: `SECURITY.md` TM-I-1 (BOLA), TM-I-3 (bulk retrieval), TM-I-5 (health values in logs), TM-T-1 (mass assignment); CWE-639, CWE-20
- **Abuse / Misuse Case**: A subscriber reads another's training history by substituting an entry identifier; or an entry is bound to a plan or copy the subscriber cannot access, leaking the existence of another subscriber's copy or an unpublished plan; or a forged catalog reference corrupts the per-exercise identity FR-8.14 trends depend on.
- **Trust Boundary**: Boundary 1 — entry identifiers, catalog-entry references, plan or copy references, values, and dates all arrive untrusted.
- **Untrusted Inputs or Assertions**: The referenced plan or copy identifier, each referenced catalog-entry identifier (FR-5.4), per-exercise sets, repetitions and weight values with their unit, the date, and any owner field.
- **Authoritative Enforcement Point**: REST API Application — consent, owner scoping, reference validation, and the audit dependency before persistence.
- **Independent Verification**: The referenced plan or copy is re-checked for accessibility by the acting subscriber, and each catalog reference is re-checked for existence, not accepted because the client offered it. The stored exercise-name snapshot is taken server-side from the catalog record, never from the request.
- **Zero Trust Relevance**: NIST SP 800-207 — per-request authorization of every referenced resource, not only the addressed one. Exact tenet: TO BE DECIDED (not verified in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — verified during the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — verified during the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: GDPR/UK GDPR (EU/UK data subjects); CCPA/CPRA, Washington My Health My Data, FTC Health Breach Notification Rule (US users); HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED). Statute-section precision: TO BE DECIDED — no source document states sections for this requirement.
- **Other**: `REF-INPUT`, `REF-API-2023`; ISO 8601 for the date representation.
- **Mapping Basis**: The FR references are normative; the security references are those `SECURITY.md` cites for validation and object-level authorization.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated subscriber with active consent, a verified email address, and access to a plan or customized copy, when they record a completed workout with per-exercise sets, repetitions, and weight used, then the entry and its per-exercise records are persisted against their account — each per-exercise record referencing its FR-5.4 catalog entry, storing a server-derived snapshot of the exercise name at log time, and storing the load value with the unit it was entered in (FR-8.10) — are retrievable by them, and an audit entry records the write.
2. **AC-02 — Boundary or failure behavior**: Given a workout entry where any per-exercise sets, repetitions, or weight value is absent, non-numeric, or negative, or where the weight is outside the 0–600 kg load range checked against the metric equivalent, or where sets are outside 1–100 or repetitions outside 1–1000 or either is non-integer (FR-8.9), or where the date is absent, unparseable, or in the future per the FR-8.9 UTC+14 calendar-date rule (FR-8.8), when it is submitted, then it is rejected, the response names the specific invalid field including which exercise it belongs to, and nothing is persisted.
3. **AC-03 — Prohibited behavior**: Given a workout entry referencing a plan or customized copy the subscriber cannot access, or a catalog entry that does not exist, when it is submitted, then it is refused without disclosing whether that plan, copy, or catalog entry exists; and the entry MUST NOT be bound to an owner supplied in the request, MUST NOT carry a client-supplied exercise-name snapshot, and MUST NOT complete without an audit entry.
4. **AC-04 — Additional criterion**: Given a past date, when a workout entry is submitted for it, then it is accepted and stored against that date (FR-8.8).
5. **AC-05 — Additional criterion**: Given an owned workout entry, when the subscriber edits or deletes it, then the change takes effect atomically across the entry and its per-exercise records, is audited, and deletion requires explicit confirmation in the client.
6. **AC-06 — Additional criterion**: Given a logged workout whose catalog entry is later renamed by an admin, when the entry is retrieved, then it presents the name snapshot taken at log time and its catalog reference is unchanged, so logged history is never altered by a rename (FR-8.3, FR-5.4).

## Failure Behavior

- **On Invalid Input**: Reject per REQ-API-010 and REQ-PROGRESS-030, naming the failing field and the exercise it belongs to; nothing persisted.
- **On Authentication Failure**: Denied upstream by REQ-AUTHZ-010.
- **On Authorization Failure**: Deny; a foreign entry, plan, copy, or catalog-entry identifier yields a response indistinguishable from a nonexistent one (REQ-AUTHZ-020, REQ-AUTHZ-040).
- **On Security-Decision Failure**: If consent, ownership, or the accessibility of a referenced resource cannot be resolved, refuse the operation.
- **On External Dependency Failure**: If persistence or audit storage is unavailable, the operation fails atomically.
- **On System Error**: Roll back the entry, its per-exercise records, and its audit record together — a partially written workout is a corrupt training record.
- **Logging / Audit**: Audit entry per access and modification, including the owner's own reads, one entry per request (FR-9.7). No performance values in log records (SEC-LOG-3).
- **Alerting**: Authorization denials and consent-refused writes are SEC-LOG-4 security events; threshold alerts route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-3, SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: Per-exercise validation across valid, absent, non-numeric, negative, out-of-range (above 600 kg metric-equivalent), and future-dated values; reference validation rejects inaccessible plans and copies and nonexistent catalog entries; the owner resolver binds the session identity; the name snapshot resolves from the catalog record, not the payload.
- **Integration Tests**: Record a multi-exercise workout end to end; edit and delete it; assert atomicity across child records; backdated entry accepted; future-dated entry rejected; rename the catalog entry and assert the logged snapshot and reference are unchanged; audit entries asserted for each operation.
- **Security Tests**: Cross-subscriber entry access; references to another subscriber's copy, to an unpublished plan, and to a nonexistent catalog entry; a client-supplied exercise-name snapshot ignored; owner field injection; write refused without consent, with consent withdrawn, and with an unverified email; audit-failure fault injection; log-content assertion.
- **Compliance Tests / Evidence**: Audit entries for workout-data access, retained per FR-9.7.
- **Acceptance-Criteria Traceability**: AC-01 — record-and-retrieve suite; AC-02 — per-exercise validation matrix; AC-03 — reference authorization and audit-dependency suites; AC-04 — backdating test; AC-05 — edit, delete, and atomicity tests; AC-06 — catalog-rename stability test.
- **Coverage Target**: Every operation and every validation branch, positive and negative; every reference type covered by an inaccessible- or nonexistent-target test.
- **Required Test Environment**: Two subscribers each with a customized copy, seeded published and unpublished plans, a seeded exercise catalog with an admin able to rename entries, consent and email-verification variations, audit capture. Runs against PostgreSQL on Vitest.

## Dependencies

- **Upstream Requirements**: REQ-PROGRESS-010, REQ-PROGRESS-030, REQ-AUTHZ-020, REQ-AUDIT-020, REQ-PRIVACY-010, REQ-PRIVACY-020, REQ-PRIVACY-040, REQ-PRIVACY-070, REQ-CUSTOM-010, REQ-CATALOG-010, REQ-PLAN-080, REQ-PROGRESS-050, REQ-PLATFORM-020, REQ-PLATFORM-030
- **Downstream Requirements**: REQ-PROGRESS-060 (FR-8.6, FR-8.14 per-exercise trends over these records, charted as top-set weight per catalog entry)
- **External Dependencies**: None
- **Dependency Assumptions**: Exercise identity is the FR-5.4 catalog entry (REQ-PLAN-080), so per-exercise performance references a stable identifier that survives plan switches, copies, and renames.
- **Failure Impact**: This entry type carries the richest per-record health detail in the system; a scoping defect here discloses a subscriber's complete training history.

## Implementation Notes

- **Constraints**: PostgreSQL with Drizzle ORM (`CLAUDE.md`); the client is a Vite-built single-page application with `vue-router`. Units are resolved (FR-8.10, REQ-PROGRESS-050): the load value is stored with the unit it was entered in and converted only at display time; range checks compare the metric equivalent against the 0–600 kg named constants (FR-8.9); weights are stored to at most one decimal place. Each per-exercise record carries the FR-5.4 catalog-entry reference plus the server-derived name snapshot (FR-8.3); a retired catalog entry never blocks retrieval of history that references it (FR-5.4: retire, never delete).
- **Prohibited Approaches**: Accepting plan or copy references without re-checking accessibility server-side, or catalog references without re-checking existence; accepting a client-supplied name snapshot; storing per-exercise performance as an opaque blob, which would defeat FR-8.14's per-exercise trends; partial writes when one exercise fails validation; logging performance values.
- **Implementation Guidance**: Validate the whole entry before writing anything, so AC-02 and AC-05's atomicity hold together. Route the write through the health-data accessor from REQ-AUDIT-020. Store sets, repetitions, and weight per set-group as structured records so REQ-PROGRESS-060 can derive the top-set weight (FR-8.14) without reparsing.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-VUE`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of reference authorization; product review of the logging form's usability on mobile.
- **Open Decisions**: None — `REQUIREMENTS.md` OQ-4, OQ-6, and OQ-7 and `DESIGN.md` OQ-8 are RESOLVED. Exercise identity is the FR-5.4 catalog entry; active selection is REQ-SELECT-010; units are REQ-PROGRESS-050; history display is REQ-PROGRESS-060.

**Estimated effort**: 1.5–2 engineer-days. **Estimated changed lines**: 400–900.
**Recommended model**: Claude Opus (`claude-opus-5`) — multi-record atomicity plus authorization of every referenced resource, on the most detailed health record in the system.
