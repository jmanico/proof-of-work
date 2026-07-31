# [REQ-AUDIT-020] Mandatory audit write on every health-data access path

## Metadata

- **ID**: REQ-AUDIT-020
- **Title**: Mandatory audit write on every health-data access path
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: Critical
- **Requirement Type**: Compliance
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-9.7, FR-11.4; `ARCHITECTURE.md` DR-9; `SECURITY.md` SEC-LOG-1

## Requirement

- **Statement**: Every path that reads or modifies a user's health data MUST produce exactly one audit entry, and the audit write MUST be a structural dependency of the access path rather than an optional caller responsibility that an individual path may omit — including consultant access.
- **Rationale**: FR-9.7 requires an audit entry for each access to or modification of health data; FR-11.4 extends it to consultant access; DR-9 states that audit writing must be a dependency of every health-data access path, not a caller option; SEC-LOG-1 requires exactly one entry with the required fields per path.
- **Assumptions**: The audit entry model and its append-only semantics exist (REQ-AUDIT-010).
- **Out of Scope**: Admin plan lifecycle auditing, which concerns plan content rather than health data (REQ-AUDIT-030); retention (SEC-LOG-5, blocked by `SECURITY.md` SQ-8); tamper-evidence below the application (SEC-LOG-7, blocked by SQ-13); export and deletion paths, blocked by SQ-5 and the unresolved synchronous-versus-deferred decision in `ARCHITECTURE.md`.
- **Design Traceability**: `DESIGN.md` OQ-7 asks how a subscriber sees that a consultant has access to their data; that presentation is open and not required here, but the audit trail is the record such a view would draw on.
- **Architecture Traceability**: `ARCHITECTURE.md` — DR-9; REST API Application ("audit writes on health-data access (FR-9.7, FR-11.4)"); data flows 4, 7, and 8.
- **Security Traceability**: SEC-LOG-1, SEC-LOG-2, SEC-LOG-3; supports SEC-AUTHZ-3, SEC-DATA-3.

## Scope

- **Applies To**: Server-Side Application, API
- **Components**: REST API Application; Relational Persistence
- **Interfaces / Operations**: Every read, create, update, and delete of body weight entries, body measurement entries, workout log entries, food log entries, consent records, and any plan association that reveals what a subscriber follows
- **Actors**: `subscriber` (own data), `consultant` (engaged subscriber's data), `admin` where any access exists
- **Preconditions**: An authorized health-data operation is about to execute
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED (`SECURITY.md` SQ-1, `REQUIREMENTS.md` OQ-3)

## Security Context

- **Security Objectives**: Accountability, Privacy, Integrity
- **Control Layers**: Logging and Monitoring, Architecture
- **Threat References**: `SECURITY.md` TM-R-1 (repudiable actions), TM-I-2 (consultant retains access), TM-I-8 (undetected access), TM-P-4 (the trail is itself sensitive); CWE-778 (insufficient logging), CWE-223 (omission of security-relevant information)
- **Abuse / Misuse Case**: A consultant or an attacker with a valid session reads a subscriber's health records through a path whose author forgot the audit call, leaving the access unrecorded and the subscriber unable to learn of it.
- **Trust Boundary**: Boundary 1 for the request; boundary 3 for the write. The audit obligation exists on the trusted side, so it cannot be skipped by the client.
- **Untrusted Inputs or Assertions**: The acting identity and subject identity MUST come from the session and persisted state, never from the request.
- **Authoritative Enforcement Point**: The health-data access layer of the REST API Application, structured so that data cannot be returned or written without the entry.
- **Independent Verification**: A test enumerating health-data paths asserts one entry per access, independently of each handler's own code.
- **Zero Trust Relevance**: N/A — accountability, not an access decision.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: N/A
- **Regulatory**: TO BE DECIDED — FR-9.7 and FR-11.4 are required as behavior regardless of regime; the statutory basis is blocked by `SECURITY.md` SQ-1 and `REQUIREMENTS.md` OQ-3.
- **Other**: `REF-LOG` as cited by SEC-LOG-1.
- **Mapping Basis**: FR-9.7, FR-11.4, and DR-9 are the normative sources; SEC-LOG-1 cites `REF-LOG`. No control identifier is asserted without verification.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authorized read or modification of health data by any actor, when it completes, then exactly one audit entry exists recording the acting account, the action, the affected subject, and the time.
2. **AC-02 — Boundary or failure behavior**: Given the audit write fails, when a health-data read or modification is attempted, then the operation does not complete — no data is returned and no modification is persisted — and the failure is surfaced as a generic error with a correlation identifier.
3. **AC-03 — Prohibited behavior**: Given a health-data access path, when it is implemented, then it MUST NOT be able to return or write health data without producing an entry, and MUST NOT produce more than one entry per access, and the entry MUST NOT contain the health values themselves (SEC-LOG-3).
4. **AC-04 — Additional criterion**: Given a consultant reading an engaged subscriber's health data, when the access occurs, then the entry records the consultant as the acting account and the subscriber as the affected subject, distinguishable from the subscriber's own access (FR-11.4).
5. **AC-05 — Additional criterion**: Given a path inventory test over all health-data operations, when it runs, then every operation is covered and a newly added path with no audit dependency fails the test.

## Failure Behavior

- **On Invalid Input**: Rejected upstream by REQ-API-010 before any access occurs, so no entry is written for a rejected request.
- **On Authentication Failure**: Denied upstream; no access, no entry.
- **On Authorization Failure**: Denied per REQ-AUTHZ-020 or REQ-CONSULT-010; the denial is logged as a security event (REQ-AUTHZ-040) and MUST NOT be recorded as a health-data access.
- **On Security-Decision Failure**: If the acting or subject identity cannot be resolved, deny the operation rather than write an entry with an unknown actor.
- **On External Dependency Failure**: If audit persistence is unavailable, health-data operations fail closed (AC-02). This is a deliberate availability trade recorded here because FR-9.7 admits no unaudited access.
- **On System Error**: The entry and the audited action share a transaction; a rollback removes both.
- **Logging / Audit**: This issue defines the audit obligation. Operational failures of the audit path are logged as security events (SEC-LOG-4) without health values.
- **Alerting**: TO BE DECIDED — audit-write failure is a plausible alert candidate, but no alerting model exists in the source documents.

## Test Strategy

- **Unit Tests**: The access wrapper invokes the audit writer exactly once per call; refuses to proceed when the writer fails; refuses when actor or subject is unresolved.
- **Integration Tests**: For each health-data operation, assert exactly one entry with correct actor, action, subject, and time; consultant access produces a distinguishable entry.
- **Security Tests**: Fault-injected audit failure asserting no data returned and no write persisted; a path inventory test that fails when a health-data operation bypasses the wrapper; content assertion that entries carry no health values.
- **Compliance Tests / Evidence**: The per-path audit coverage report, retained as evidence for FR-9.7 and FR-11.4.
- **Acceptance-Criteria Traceability**: AC-01 — per-operation entry assertions; AC-02 — audit-failure fault injection; AC-03 — single-entry and content assertions plus the structural check; AC-04 — consultant access test; AC-05 — path inventory test.
- **Coverage Target**: 100% of health-data operations covered by the inventory test.
- **Required Test Environment**: Two subscribers, one consultant with an active engagement, seeded health records, audit storage, and a fault-injectable audit writer. Runs against PostgreSQL on Vitest.

## Dependencies

- **Upstream Requirements**: REQ-AUDIT-010, REQ-AUTHZ-020, REQ-API-030
- **Downstream Requirements**: REQ-PROGRESS-010, REQ-PROGRESS-020, REQ-PRIVACY-010, REQ-PRIVACY-020, REQ-PRIVACY-030, REQ-CONSULT-010, REQ-CUSTOM-020
- **External Dependencies**: None
- **Dependency Assumptions**: Audit storage shares the transactional context of the operational data store, so AC-02 and the rollback behavior are achievable.
- **Failure Impact**: An unaudited path defeats FR-9.7 for that path permanently — the missing record cannot be reconstructed later.

## Implementation Notes

- **Constraints**: PostgreSQL with Drizzle ORM (`CLAUDE.md`). DR-9 requires the audit write to be a *dependency* of the access path, so a convention of "remember to call the audit function" does not satisfy this issue.
- **Prohibited Approaches**: Audit calls scattered through handlers; a middleware that infers health-data access from the URL rather than from the data layer; asynchronous best-effort audit writes that let the operation succeed when the entry fails; copying health values into the entry for context.
- **Implementation Guidance**: Route every health-data read and write through a single accessor that takes the audit context as a required argument, so that omitting it is a compile-time or construction-time failure rather than a review miss. That structure is what makes AC-05 enforceable.
- **AI Development Guidance**: `REF-LOG`, `REF-PROMPT-QUALITY`, `REF-PROMPT-API`; `CLAUDE.md`.
- **Required Human Review**: Security and privacy review of the accessor design and the path inventory.
- **Open Decisions**: Export (FR-9.3) and deletion (FR-9.4) are health-data paths that also require entries, but both are blocked by `SECURITY.md` SQ-5 and the unresolved synchronous-versus-deferred decision in `ARCHITECTURE.md`; they must adopt this accessor when unblocked.

**Estimated effort**: 1–2 engineer-days. **Estimated changed lines**: 300–650.
**Recommended model**: Claude Opus (`claude-opus-5`) — the correctness bar is structural (omission must be impossible, not merely discouraged), which rewards careful design over speed.
