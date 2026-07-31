# [REQ-PROGRESS-010] Body weight entry logging, editing, deletion, and backdating

## Metadata

- **ID**: REQ-PROGRESS-010
- **Title**: Body weight entry logging, editing, deletion, and backdating
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-8.1, FR-8.7, FR-8.8, FR-9.1, FR-9.7

## Requirement

- **Statement**: A subscriber MUST be able to log a body weight entry with a date, to edit and delete their own entries, and to record an entry for a past date; every such entry MUST be scoped to its owning subscriber and every access to or modification of it MUST be audited.
- **Rationale**: FR-8.1 requires body weight logging with a date; FR-8.7 grants edit and delete of one's own entries; FR-8.8 permits past dates; FR-9.1 scopes entries to their owner; FR-9.7 requires an audit entry for each access or modification of health data.
- **Assumptions**: Consent has been captured and is active (REQ-PRIVACY-010, REQ-PRIVACY-020).
- **Out of Scope**: Body measurements (FR-8.2), blocked by `REQUIREMENTS.md` OQ-4 because the measurement fields and unit system are undecided; workout logging (REQ-PROGRESS-020); food logging (FR-8.4), blocked by OQ-5; history display over time (FR-8.6), blocked by OQ-7 and `DESIGN.md` OQ-4; the generic validation and error-presentation contract (REQ-PROGRESS-030); the unit system for weight, which `REQUIREMENTS.md` OQ-4 and `DESIGN.md` OQ-8 leave open.
- **Design Traceability**: `DESIGN.md` — Components → Inputs ("Numeric log entry fields (weight, measurements, sets, reps, calories, macros) show their unit adjacent to the field"), Buttons (destructive delete requires explicit confirmation), Form feedback and errors; Typography (tabular figures); Accessibility.
- **Architecture Traceability**: `ARCHITECTURE.md` — data flow 4 ("consent check, owner scoping, field validation → log entry written → audit entry written"); REST API Application ("Owns … body weight entry"); DR-9.
- **Security Traceability**: SEC-DATA-2, SEC-AUTHZ-2, SEC-INPUT-1, SEC-INPUT-2, SEC-INPUT-3, SEC-LOG-1, SEC-DATA-5.

## Scope

- **Applies To**: Server-Side Application, API, Web Client
- **Components**: REST API Application; Relational Persistence; Browser Client (weight logging view)
- **Interfaces / Operations**: Create, read, update, and delete a body weight entry
- **Actors**: `subscriber`
- **Preconditions**: Authenticated session; active consent (FR-9.2, FR-9.9)
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED (`SECURITY.md` SQ-1, `REQUIREMENTS.md` OQ-3)

## Security Context

- **Security Objectives**: Confidentiality, Integrity, Privacy, Accountability
- **Control Layers**: Authorization, Input Validation, Business-Rule Validation, Data Protection, Logging and Monitoring
- **Threat References**: `SECURITY.md` TM-I-1 (BOLA on log entries), TM-I-3 (bulk retrieval), TM-I-5 (health values in logs), TM-P-1 (identifiability); CWE-639 (authorization bypass through user-controlled key), CWE-20 (improper input validation)
- **Abuse / Misuse Case**: A subscriber reads or alters another subscriber's weight history by substituting an entry identifier; or entries are written with no audit trail, so an unauthorized read is undetectable.
- **Trust Boundary**: Boundary 1 — entry identifiers, values, and dates all arrive from an untrusted client.
- **Untrusted Inputs or Assertions**: The weight value, the date, the entry identifier, and any owner field in the payload.
- **Authoritative Enforcement Point**: REST API Application — consent check, owner scoping, validation, and the audit dependency, in that order, before persistence.
- **Independent Verification**: Owner identity from the session; audit write structurally required (REQ-AUDIT-020).
- **Zero Trust Relevance**: NIST SP 800-207 — per-request authorization of each addressed resource. Exact tenet: TO BE DECIDED (not verified in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: TO BE DECIDED — blocked by `SECURITY.md` SQ-1; body weight is health data under every regime `REQUIREMENTS.md` names.
- **Other**: `REF-INPUT` for validation; `REF-API-2023` for object-level authorization; ISO 8601 for the date representation.
- **Mapping Basis**: The FR references are normative; the security references are those `SECURITY.md` cites for the validation and authorization rules applied here. ISO 8601 is named because a dated entry requires an unambiguous date format, which no source document otherwise fixes.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated subscriber with active consent, when they log a body weight entry with a valid numeric value and a date, then the entry is persisted against their account, is retrievable by them, and an audit entry records the write.
2. **AC-02 — Boundary or failure behavior**: Given a weight entry whose value is absent, non-numeric, or negative, or whose date is absent or unparseable, when it is submitted, then it is rejected, the response names the specific invalid field, nothing is persisted, and the client moves focus to the first invalid field (FR-8.9, REQ-PROGRESS-030).
3. **AC-03 — Prohibited behavior**: Given any entry operation, when it is processed, then it MUST NOT act on an owner identifier supplied in the request, MUST NOT permit a subscriber to read, edit, or delete another subscriber's entry, and MUST NOT complete without an audit entry.
4. **AC-04 — Additional criterion**: Given a date earlier than today, when an entry is submitted for it, then the entry is accepted and stored against that date (FR-8.8).
5. **AC-05 — Additional criterion**: Given an owned entry, when the subscriber edits or deletes it, then the change takes effect, is audited, and deletion requires explicit confirmation in the client as a destructive action (`DESIGN.md`, Components → Buttons).

## Failure Behavior

- **On Invalid Input**: Reject per REQ-API-010 and REQ-PROGRESS-030 with the specific invalid field named; nothing persisted.
- **On Authentication Failure**: Denied upstream by REQ-AUTHZ-010.
- **On Authorization Failure**: Deny; a foreign entry identifier produces a response indistinguishable from a nonexistent entry (REQ-AUTHZ-020, REQ-AUTHZ-040).
- **On Security-Decision Failure**: If consent state or ownership cannot be resolved, refuse the operation (fail closed).
- **On External Dependency Failure**: If persistence or audit storage is unavailable, the operation fails atomically and nothing is written (REQ-AUDIT-020).
- **On System Error**: Roll back the entry and its audit record together.
- **Logging / Audit**: Audit entry per access and modification (FR-9.7). The weight value MUST NOT appear in any log record (SEC-LOG-3).
- **Alerting**: N/A

## Test Strategy

- **Unit Tests**: Validation of the value and date fields across valid, absent, non-numeric, negative, and unparseable cases; the owner resolver binds the session identity; the audit writer is invoked once per operation.
- **Integration Tests**: Create, read, update, delete an owned entry end to end; backdated entry accepted; assert audit entries for each operation.
- **Security Tests**: Cross-subscriber read, edit, and delete attempts; owner field injection; write attempted without consent and after consent withdrawal; audit-failure fault injection asserting no entry is written; log-content assertion that no weight value appears.
- **Compliance Tests / Evidence**: Audit entries for weight-data access, retained per FR-9.7.
- **Acceptance-Criteria Traceability**: AC-01 — create-and-retrieve suite; AC-02 — validation matrix; AC-03 — cross-subscriber and audit-dependency suites; AC-04 — backdating test; AC-05 — edit, delete, and confirmation tests.
- **Coverage Target**: Every operation positive and negative; every validation branch.
- **Required Test Environment**: Two subscribers with consent, one without consent, one with consent withdrawn; audit capture. Runs against PostgreSQL on Vitest.

## Dependencies

- **Upstream Requirements**: REQ-AUTHZ-010, REQ-AUTHZ-020, REQ-API-010, REQ-API-020, REQ-AUDIT-020, REQ-PRIVACY-010, REQ-PRIVACY-020, REQ-PROGRESS-030, REQ-PLATFORM-030
- **Downstream Requirements**: None unblocked — history display (FR-8.6) is blocked by `REQUIREMENTS.md` OQ-7 and `DESIGN.md` OQ-4
- **External Dependencies**: None
- **Dependency Assumptions**: None
- **Failure Impact**: This is the first health-data write path; a defect in its consent, scoping, or audit handling establishes the pattern every later log type would copy.

## Implementation Notes

- **Constraints**: PostgreSQL with Drizzle ORM (`CLAUDE.md`); client build tooling TO BE DECIDED. The unit system for weight is open (`REQUIREMENTS.md` OQ-4, `DESIGN.md` OQ-8) — store the unit explicitly with each entry rather than assuming one, so resolving OQ-4 does not require reinterpreting stored data.
- **Prohibited Approaches**: Deriving the owner from the request; a shared "log entry" endpoint that switches on a client-supplied type discriminator without per-type validation; logging the value for debugging; allowing a future date without a documented decision, since no source document addresses it — reject future dates and record the choice as provisional.
- **Implementation Guidance**: Route the write through the health-data accessor established by REQ-AUDIT-020 so consent and audit obligations are carried structurally rather than by convention.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-VUE`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of scoping and audit; privacy review of the stored field set.
- **Open Decisions**: Unit system (`REQUIREMENTS.md` OQ-4, `DESIGN.md` OQ-8); whether future-dated entries are permitted (unspecified); history granularity (OQ-7), which is blocked and governs how these entries are later displayed.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 250–550.
**Recommended model**: Claude Opus (`claude-opus-5`) — the reference implementation for every health-data write path, where consent, scoping, and audit must compose correctly.
