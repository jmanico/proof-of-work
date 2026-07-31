# [REQ-PROGRESS-020] Workout completion logging

## Metadata

- **ID**: REQ-PROGRESS-020
- **Title**: Workout completion logging
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-8.3, FR-8.7, FR-8.8, FR-9.1, FR-9.7

## Requirement

- **Statement**: A subscriber MUST be able to record completion of a workout from their plan, including the sets, repetitions, and weight used for each exercise, MUST be able to edit and delete their own workout log entries, and MUST be able to record one for a past date; every entry MUST be owner-scoped and every access or modification MUST be audited.
- **Rationale**: FR-8.3 requires workout completion recording with per-exercise sets, repetitions, and weight used; FR-8.7 grants edit and delete; FR-8.8 permits past dates; FR-9.1 and FR-9.7 supply the scoping and audit obligations.
- **Assumptions**: Consent is active (REQ-PRIVACY-010, REQ-PRIVACY-020) and the subscriber holds a plan or a customized copy whose exercises the entry references (REQ-CUSTOM-010, REQ-CATALOG-010).
- **Out of Scope**: Body weight (REQ-PROGRESS-010) and measurements (blocked by `REQUIREMENTS.md` OQ-4); food logging (blocked by OQ-5); workout performance history over time (FR-8.6), blocked by OQ-7 and `DESIGN.md` OQ-4; the generic validation contract (REQ-PROGRESS-030); plan selection (blocked by OQ-6), which is why the entry references a plan or copy the subscriber can access rather than a designated "active" plan.
- **Design Traceability**: `DESIGN.md` — Components → Inputs (units adjacent to numeric fields for sets, reps, and weight), Buttons (destructive delete confirmed), Form feedback and errors; Typography (tabular figures so per-exercise values align in columns); Layout and Spacing (mobile reflow of a multi-exercise form); Accessibility.
- **Architecture Traceability**: `ARCHITECTURE.md` — data flow 4; REST API Application ("Owns … workout log entry"); DR-9.
- **Security Traceability**: SEC-DATA-2, SEC-AUTHZ-2, SEC-INPUT-1, SEC-INPUT-2, SEC-INPUT-3, SEC-LOG-1, SEC-DATA-5.

## Scope

- **Applies To**: Server-Side Application, API, Web Client
- **Components**: REST API Application; Relational Persistence; Browser Client (workout logging view)
- **Interfaces / Operations**: Create, read, update, and delete a workout log entry with its per-exercise performance records
- **Actors**: `subscriber`
- **Preconditions**: Authenticated session; active consent; the medical disclaimer acknowledged (REQ-PRIVACY-040); an accessible plan or customized copy
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED (`SECURITY.md` SQ-1, `REQUIREMENTS.md` OQ-3)

## Security Context

- **Security Objectives**: Confidentiality, Integrity, Privacy, Accountability
- **Control Layers**: Authorization, Input Validation, Business-Rule Validation, Data Protection, Logging and Monitoring
- **Threat References**: `SECURITY.md` TM-I-1 (BOLA), TM-I-3 (bulk retrieval), TM-I-5 (health values in logs), TM-T-1 (mass assignment); CWE-639, CWE-20
- **Abuse / Misuse Case**: A subscriber reads another's training history by substituting an entry identifier; or an entry is bound to a plan or exercise the subscriber cannot access, leaking the existence of another subscriber's copy or an unpublished plan.
- **Trust Boundary**: Boundary 1 — entry identifiers, exercise references, values, and dates all arrive untrusted.
- **Untrusted Inputs or Assertions**: The referenced plan or copy identifier, each referenced exercise identifier, per-exercise sets, repetitions and weight values, the date, and any owner field.
- **Authoritative Enforcement Point**: REST API Application — consent, owner scoping, reference validation, and the audit dependency before persistence.
- **Independent Verification**: The referenced plan or copy is re-checked for accessibility by the acting subscriber, not accepted because the client offered it.
- **Zero Trust Relevance**: NIST SP 800-207 — per-request authorization of every referenced resource, not only the addressed one. Exact tenet: TO BE DECIDED (not verified in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: TO BE DECIDED — blocked by `SECURITY.md` SQ-1.
- **Other**: `REF-INPUT`, `REF-API-2023`; ISO 8601 for the date representation.
- **Mapping Basis**: The FR references are normative; the security references are those `SECURITY.md` cites for validation and object-level authorization.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated subscriber with active consent and access to a plan or customized copy, when they record a completed workout with per-exercise sets, repetitions, and weight used, then the entry and its per-exercise records are persisted against their account, are retrievable by them, and an audit entry records the write.
2. **AC-02 — Boundary or failure behavior**: Given a workout entry where any per-exercise sets, repetitions, or weight value is absent, non-numeric, or negative, or where the date is absent or unparseable, when it is submitted, then it is rejected, the response names the specific invalid field including which exercise it belongs to, and nothing is persisted (FR-8.9).
3. **AC-03 — Prohibited behavior**: Given a workout entry referencing a plan, a customized copy, or an exercise the subscriber cannot access, when it is submitted, then it is refused without disclosing whether that plan, copy, or exercise exists; and the entry MUST NOT be bound to an owner supplied in the request or completed without an audit entry.
4. **AC-04 — Additional criterion**: Given a past date, when a workout entry is submitted for it, then it is accepted and stored against that date (FR-8.8).
5. **AC-05 — Additional criterion**: Given an owned workout entry, when the subscriber edits or deletes it, then the change takes effect atomically across the entry and its per-exercise records, is audited, and deletion requires explicit confirmation in the client.

## Failure Behavior

- **On Invalid Input**: Reject per REQ-API-010 and REQ-PROGRESS-030, naming the failing field and the exercise it belongs to; nothing persisted.
- **On Authentication Failure**: Denied upstream by REQ-AUTHZ-010.
- **On Authorization Failure**: Deny; a foreign entry, plan, copy, or exercise identifier yields a response indistinguishable from a nonexistent one (REQ-AUTHZ-020, REQ-AUTHZ-040).
- **On Security-Decision Failure**: If consent, ownership, or the accessibility of a referenced resource cannot be resolved, refuse the operation.
- **On External Dependency Failure**: If persistence or audit storage is unavailable, the operation fails atomically.
- **On System Error**: Roll back the entry, its per-exercise records, and its audit record together — a partially written workout is a corrupt training record.
- **Logging / Audit**: Audit entry per access and modification (FR-9.7). No performance values in log records (SEC-LOG-3).
- **Alerting**: N/A

## Test Strategy

- **Unit Tests**: Per-exercise validation across valid, absent, non-numeric, and negative values; reference validation rejects inaccessible plans, copies, and exercises; the owner resolver binds the session identity.
- **Integration Tests**: Record a multi-exercise workout end to end; edit and delete it; assert atomicity across child records; backdated entry accepted; audit entries asserted for each operation.
- **Security Tests**: Cross-subscriber entry access; references to another subscriber's copy, to an unpublished plan, and to an exercise from a different plan; owner field injection; consent-withdrawn write refused; audit-failure fault injection; log-content assertion.
- **Compliance Tests / Evidence**: Audit entries for workout-data access, retained per FR-9.7.
- **Acceptance-Criteria Traceability**: AC-01 — record-and-retrieve suite; AC-02 — per-exercise validation matrix; AC-03 — reference authorization and audit-dependency suites; AC-04 — backdating test; AC-05 — edit, delete, and atomicity tests.
- **Coverage Target**: Every operation and every validation branch, positive and negative; every reference type covered by an inaccessible-target test.
- **Required Test Environment**: Two subscribers each with a customized copy, seeded published and unpublished plans, consent variations, audit capture. Runs against PostgreSQL on Vitest.

## Dependencies

- **Upstream Requirements**: REQ-PROGRESS-010, REQ-PROGRESS-030, REQ-AUTHZ-020, REQ-AUDIT-020, REQ-PRIVACY-010, REQ-PRIVACY-020, REQ-PRIVACY-040, REQ-CUSTOM-010, REQ-CATALOG-010, REQ-PLATFORM-020, REQ-PLATFORM-030
- **Downstream Requirements**: None unblocked — workout performance history (FR-8.6) is blocked by `REQUIREMENTS.md` OQ-7
- **External Dependencies**: None
- **Dependency Assumptions**: Exercises are addressable child records of a plan or copy (REQ-PLAN-010, REQ-CUSTOM-010), so per-exercise performance can reference them.
- **Failure Impact**: This entry type carries the richest per-record health detail in the system; a scoping defect here discloses a subscriber's complete training history.

## Implementation Notes

- **Constraints**: PostgreSQL with Drizzle ORM (`CLAUDE.md`); client build tooling TO BE DECIDED. The weight unit is open (`REQUIREMENTS.md` OQ-4, `DESIGN.md` OQ-8) — store it explicitly per record. Plan selection is blocked (OQ-6), so an entry references a plan or copy the subscriber can access rather than a system-designated active plan; that choice is provisional and must be revisited when OQ-6 resolves.
- **Prohibited Approaches**: Accepting exercise references without re-checking accessibility server-side; storing per-exercise performance as an opaque blob, which would defeat the FR-8.6 history requirement when it unblocks; partial writes when one exercise fails validation; logging performance values.
- **Implementation Guidance**: Validate the whole entry before writing anything, so AC-02 and AC-05's atomicity hold together. Route the write through the health-data accessor from REQ-AUDIT-020.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-VUE`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of reference authorization; product review of the logging form's usability on mobile.
- **Open Decisions**: `REQUIREMENTS.md` OQ-6 (which plan a workout is logged against), OQ-4 and `DESIGN.md` OQ-8 (units), OQ-7 (history granularity). None block the write path; all bound the surrounding workflow.

**Estimated effort**: 1.5–2 engineer-days. **Estimated changed lines**: 400–900.
**Recommended model**: Claude Opus (`claude-opus-5`) — multi-record atomicity plus authorization of every referenced resource, on the most detailed health record in the system.
