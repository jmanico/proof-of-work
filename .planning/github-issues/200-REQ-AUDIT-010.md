# [REQ-AUDIT-010] Audit entry model and append-only enforcement

## Metadata

- **ID**: REQ-AUDIT-010
- **Title**: Audit entry model and append-only enforcement
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: Critical
- **Requirement Type**: Compliance
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-9.7; `SECURITY.md` SEC-LOG-1, SEC-LOG-2; threat TM-R-2

## Requirement

- **Statement**: The system MUST persist audit entries recording the acting account, the action, the affected subject, and the time, and those entries MUST be append-only from the application's perspective — no role, including `admin`, may edit or delete an audit entry through any application interface.
- **Rationale**: FR-9.7 requires an audit entry for each access to or modification of health data with those fields; SEC-LOG-1 restates the field set and extends it to consultant access; SEC-LOG-2 requires append-only semantics so that the record of an action cannot be erased by the actor.
- **Assumptions**: Audit entries live in Relational Persistence alongside the other business objects (`ARCHITECTURE.md`, Data model expectations).
- **Out of Scope**: Which operations must write an entry (REQ-AUDIT-020, REQ-AUDIT-030); redaction rules for log content (REQ-AUDIT-040); tamper-evidence against actors below the application (SEC-LOG-7, blocked by `SECURITY.md` SQ-8 and SQ-13); retention periods and access control for audit storage (SEC-LOG-5, blocked by SQ-8).
- **Design Traceability**: N/A — no source document specifies an audit-viewing interface, and none is created here.
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application ("Owns … audit entry"); Relational Persistence; DR-4; DR-9.
- **Security Traceability**: SEC-LOG-1, SEC-LOG-2; supports SEC-LOG-3 (entries reference data, never copy it), SEC-LOG-6, SEC-DATA-4 (retained audit entries MUST NOT hold health values).

## Scope

- **Applies To**: Server-Side Application, API
- **Components**: REST API Application; Relational Persistence
- **Interfaces / Operations**: Audit entry creation; the absence of any update or delete operation on audit entries
- **Actors**: `subscriber`, `consultant`, `admin` as audited actors; `admin` as the role explicitly denied mutation
- **Preconditions**: None
- **Data Classification**: Restricted — an audit trail of health-data access is itself sensitive (`SECURITY.md` TM-P-4)
- **Personal or Regulated Data**: Personal Data — actor and subject identifiers; Health Data values MUST NOT be copied into an entry
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED (`SECURITY.md` SQ-1, `REQUIREMENTS.md` OQ-3)

## Security Context

- **Security Objectives**: Accountability, Integrity, Privacy
- **Control Layers**: Logging and Monitoring, Data Protection
- **Threat References**: `SECURITY.md` TM-R-1 (repudiable admin actions), TM-R-2 (audit entries alterable below the application), TM-P-4 (the audit trail is itself sensitive); CWE-778 (insufficient logging), CWE-117 (improper output neutralization for logs), CWE-532 (sensitive information in a log)
- **Abuse / Misuse Case**: An actor who accessed another user's health data deletes or edits the entry recording it; or the audit table becomes a second copy of the health data it was meant to reference.
- **Trust Boundary**: Boundary 3 for persistence; boundary 5 for actors below the application, where this issue's guarantee explicitly does not reach.
- **Untrusted Inputs or Assertions**: Any request-derived value written into an entry; the acting identity MUST come from the session, not the request.
- **Authoritative Enforcement Point**: REST API Application for entry creation; Relational Persistence schema and privileges for append-only semantics.
- **Independent Verification**: Append-only is enforced by the absence of mutation operations and by database privileges, not by convention.
- **Zero Trust Relevance**: N/A

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: N/A
- **Regulatory**: TO BE DECIDED — FR-9.7 is asserted by `REQUIREMENTS.md` as behavior regardless of regime; the specific statutory basis is blocked by `SECURITY.md` SQ-1 and `REQUIREMENTS.md` OQ-3.
- **Other**: `REF-LOG` as cited by SEC-LOG-1 and SEC-LOG-2.
- **Mapping Basis**: FR-9.7 fixes the required fields; SEC-LOG-1 and SEC-LOG-2 fix the semantics and cite `REF-LOG`. No control identifier is asserted without verification.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given any audited action, when an audit entry is written, then it records the acting account, the action, the affected subject, and the time, and it is retrievable in the order written.
2. **AC-02 — Boundary or failure behavior**: Given an attempt to update or delete an audit entry through any application interface, when it is made as `subscriber`, `consultant`, or `admin`, then it is denied, no entry is altered or removed, and the attempt is logged.
3. **AC-03 — Prohibited behavior**: Given any audit entry, when it is inspected, then it MUST NOT contain a health value, credential, token, or full personal record — it references the data accessed rather than copying it (SEC-LOG-3) — and the acting account MUST NOT be taken from the request body.
4. **AC-04 — Additional criterion**: Given the database privileges used by the application, when they are reviewed, then the application's role has insert and select on audit storage and no update or delete privilege.
5. **AC-05 — Additional criterion**: Given a request-derived value written into an entry, when it is persisted, then it is stored so that delimiter or newline characters cannot forge an additional entry.

## Failure Behavior

- **On Invalid Input**: An entry that cannot be assembled with all four required fields MUST NOT be written as a partial record; the calling operation treats the failure per REQ-AUDIT-020.
- **On Authentication Failure**: N/A — audited actions occur after authentication.
- **On Authorization Failure**: Denied mutation attempts produce a denial per REQ-AUTHZ-040 and a log event per REQ-AUTH-050.
- **On Security-Decision Failure**: If the acting identity cannot be resolved, the entry MUST NOT be written with a placeholder actor; the calling operation fails closed.
- **On External Dependency Failure**: If persistence is unavailable, the audit write fails and the calling health-data operation MUST NOT complete (REQ-AUDIT-020, DR-9).
- **On System Error**: The audit write participates in the same transaction as the audited action, so a rollback removes both or neither.
- **Logging / Audit**: This issue defines the audit record itself. Attempts to mutate audit entries are logged as security events (SEC-LOG-4).
- **Alerting**: TO BE DECIDED — an attempted audit mutation is a plausible alert candidate, but no alerting model exists in any source document.

## Test Strategy

- **Unit Tests**: Entry builder requires all four fields and rejects a missing actor, action, subject, or time; builder refuses a health value in any field.
- **Integration Tests**: Write and read back entries preserving order; transactional rollback removes both the action and its entry.
- **Security Tests**: Attempted update and delete through every application interface as each role, asserting denial (SEC-LOG-2 verification); privilege review asserting no update or delete grant; entry-forgery test with delimiter-bearing input; content assertion that no health value appears.
- **Compliance Tests / Evidence**: Sample audit entries and the database privilege grant, retained as evidence for FR-9.7.
- **Acceptance-Criteria Traceability**: AC-01 — write-and-read suite; AC-02 — mutation denial matrix; AC-03 — content assertions; AC-04 — privilege review; AC-05 — forgery test.
- **Coverage Target**: Every role × every mutation interface exercised; all four required fields covered positive and negative.
- **Required Test Environment**: The selected RDBMS with a distinct application role, identities for all three roles. Engine and test framework TO BE DECIDED.

## Dependencies

- **Upstream Requirements**: REQ-API-030
- **Downstream Requirements**: REQ-AUDIT-020, REQ-AUDIT-030, REQ-AUTH-030, REQ-CONSULT-010, REQ-PRIVACY-010, REQ-PROGRESS-010, REQ-PROGRESS-020
- **External Dependencies**: None
- **Dependency Assumptions**: The chosen RDBMS supports per-role privilege grants fine enough to withhold update and delete on a single table.
- **Failure Impact**: Without append-only semantics the audit trail proves nothing, and FR-9.7's accountability guarantee is nominal.

## Implementation Notes

- **Constraints**: RDBMS engine and migration tooling TO BE DECIDED. `SECURITY.md` SEC-LOG-2 binds the application only; protection against a database-credential holder is explicitly a separate, unresolved rule (SEC-LOG-7) and this issue MUST NOT be described as delivering it.
- **Prohibited Approaches**: An `updated_at` column or any mutation path on audit entries; storing the accessed health values "for context"; taking the acting account from the request; writing the entry outside the audited action's transaction.
- **Implementation Guidance**: Model the affected subject as a reference (subject account identifier plus record type and identifier) rather than embedded content, which satisfies SEC-LOG-3 by construction and keeps FR-9.4's deletion obligations tractable.
- **AI Development Guidance**: `REF-LOG`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security and privacy review of the entry schema; database privilege review.
- **Open Decisions**: Retention and access control (`SECURITY.md` SQ-8); tamper-evidence against operators (SEC-LOG-7, SQ-13); what survives account deletion (SEC-DATA-4, SQ-5). All blocked; this issue is deliberately limited to the application-layer guarantee.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 250–500.
**Recommended model**: Claude Opus (`claude-opus-5`) — a compliance-critical data model whose value depends entirely on the mutation and content prohibitions holding.
