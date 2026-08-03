# [REQ-AUDIT-020] Mandatory audit write on every health-data access path

## Metadata

- **ID**: REQ-AUDIT-020
- **Title**: Mandatory audit write on every health-data access path
- **Version**: 1.1.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Compliance
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-9.7, FR-9.12, FR-11.4; `ARCHITECTURE.md` DR-9; `SECURITY.md` SEC-LOG-1

## Requirement

- **Statement**: Every path that reads or modifies a user's health data (FR-9.12) — including the user's own reads — MUST produce exactly one audit entry per request recording the acting account, the action, the affected subject, and the time, and the audit write MUST be a structural dependency of the access path rather than an optional caller responsibility that an individual path may omit — including consultant access.
- **Rationale**: FR-9.7 requires an audit entry for each access to or modification of health data, including the user's own reads, and fixes granularity at one entry per request (an FR-9.3 export produces one entry for the whole operation, referencing its scope); FR-11.4 extends it to consultant access; DR-9 states that audit writing must be a dependency of every health-data access path, not a caller option; SEC-LOG-1 requires the entry with the required fields including the affected subject.
- **Assumptions**: The audit entry model and its append-only semantics exist (REQ-AUDIT-010). The health-data boundary is defined by FR-9.12 (REQ-PRIVACY-070): workout, food, body-weight, and body-measurement log entries; customized plan copies; and active plan selections.
- **Out of Scope**: Admin plan lifecycle auditing, which concerns plan content rather than health data (REQ-AUDIT-030); retention and tamper-evidence below the application (SEC-LOG-5, SEC-LOG-7; `SECURITY.md` SQ-8 and SQ-13 RESOLVED — delivered by REQ-INFRA-040); the export and deletion operations themselves, which execute synchronously (SQ-5 RESOLVED) and are delivered by REQ-PRIVACY-080 and REQ-PRIVACY-090 — both adopt this issue's accessor, with export producing its one whole-operation entry per FR-9.7; the health-data definition itself (FR-9.12, REQ-PRIVACY-070).
- **Design Traceability**: `DESIGN.md` OQ-7 RESOLVED — subscribers see consultant access on Home, in Account → Consultant access, and as edit provenance on affected plan copies; that presentation is delivered elsewhere, but the audit trail is the record such views rest on.
- **Architecture Traceability**: `ARCHITECTURE.md` — DR-9; REST API Application ("audit writes on health-data access (FR-9.7, FR-11.4)"); data flows 4, 5 (progress viewing now names its audit write — one per request), 7, and 8.
- **Security Traceability**: SEC-LOG-1, SEC-LOG-2, SEC-LOG-3; supports SEC-AUTHZ-3, SEC-DATA-3.

## Scope

- **Applies To**: Server-Side Application, API
- **Components**: REST API Application; Relational Persistence
- **Interfaces / Operations**: Every read, create, update, and delete of the FR-9.12 health-data set — body weight entries, body measurement entries, workout log entries, food log entries, customized plan copies, and active plan selections — plus consent records as health-data-adjacent state
- **Actors**: `subscriber` (own data), `consultant` (engaged subscriber's data), `admin` where any access exists
- **Preconditions**: An authorized health-data operation is about to execute
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED, `REQUIREMENTS.md` OQ-3 RESOLVED)

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

- **OWASP ASVS 5.0.0**: TO BE DECIDED — mapped only when verified during the independent pre-launch assessment (`SECURITY.md` SQ-10 RESOLVED).
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — mapped only when verified during the independent pre-launch assessment (`SECURITY.md` SQ-10 RESOLVED).
- **NIST SP 800-207**: N/A
- **Regulatory**: The SQ-1 regime set governs (GDPR/UK GDPR for EU/UK data subjects; CCPA/CPRA, Washington My Health My Data, FTC HBNR for US users; HIPAA not applicable); FR-9.7 and FR-11.4 are required as behavior for all users regardless of jurisdiction. Statute-section precision: TO BE DECIDED — per-issue mappings await the SQ-1 pre-launch counsel review.
- **Other**: `REF-LOG` as cited by SEC-LOG-1.
- **Mapping Basis**: FR-9.7, FR-11.4, and DR-9 are the normative sources; SEC-LOG-1 cites `REF-LOG`. No control identifier is asserted without verification.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authorized read or modification of health data (FR-9.12) by any actor — including a subscriber reading their own records — when the request completes, then exactly one audit entry exists for that request recording the acting account, the action, the affected subject, and the time.
2. **AC-02 — Boundary or failure behavior**: Given the audit write fails, when a health-data read or modification is attempted, then the operation does not complete — no data is returned and no modification is persisted — and the failure is surfaced as a generic error with a correlation identifier.
3. **AC-03 — Prohibited behavior**: Given a health-data access path, when it is implemented, then it MUST NOT be able to return or write health data without producing an entry, MUST NOT produce more than one entry per request (FR-9.7), and the entry MUST NOT contain the health values themselves (SEC-LOG-3).
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
- **Alerting**: Audit-write failures are logged as security events (SEC-LOG-4); threshold alerts route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: The access wrapper invokes the audit writer exactly once per call; refuses to proceed when the writer fails; refuses when actor or subject is unresolved.
- **Integration Tests**: For each health-data operation — including plan-copy and selection reads and writes and a subscriber's own reads (FR-9.12, FR-9.7) — assert exactly one entry per request with correct actor, action, subject, and time; consultant access produces a distinguishable entry.
- **Security Tests**: Fault-injected audit failure asserting no data returned and no write persisted; a path inventory test that fails when a health-data operation bypasses the wrapper; content assertion that entries carry no health values.
- **Compliance Tests / Evidence**: The per-path audit coverage report, retained as evidence for FR-9.7 and FR-11.4.
- **Acceptance-Criteria Traceability**: AC-01 — per-operation entry assertions; AC-02 — audit-failure fault injection; AC-03 — single-entry and content assertions plus the structural check; AC-04 — consultant access test; AC-05 — path inventory test.
- **Coverage Target**: 100% of health-data operations covered by the inventory test.
- **Required Test Environment**: Two subscribers, one consultant with an active engagement, seeded health records, audit storage, and a fault-injectable audit writer. Runs against PostgreSQL on Vitest.

## Dependencies

- **Upstream Requirements**: REQ-AUDIT-010, REQ-AUTHZ-020, REQ-API-030, REQ-PRIVACY-070
- **Downstream Requirements**: REQ-PROGRESS-010, REQ-PROGRESS-020, REQ-PRIVACY-010, REQ-PRIVACY-020, REQ-PRIVACY-030, REQ-PRIVACY-080, REQ-PRIVACY-090, REQ-CONSULT-010, REQ-CUSTOM-020, REQ-SELECT-010, REQ-SELECT-020
- **External Dependencies**: None
- **Dependency Assumptions**: Audit storage shares the transactional context of the operational data store, so AC-02 and the rollback behavior are achievable.
- **Failure Impact**: An unaudited path defeats FR-9.7 for that path permanently — the missing record cannot be reconstructed later.

## Implementation Notes

- **Constraints**: PostgreSQL with Drizzle ORM (`CLAUDE.md`). DR-9 requires the audit write to be a *dependency* of the access path, so a convention of "remember to call the audit function" does not satisfy this issue.
- **Prohibited Approaches**: Audit calls scattered through handlers; a middleware that infers health-data access from the URL rather than from the data layer; asynchronous best-effort audit writes that let the operation succeed when the entry fails; copying health values into the entry for context.
- **Implementation Guidance**: Route every health-data read and write through a single accessor that takes the audit context as a required argument, so that omitting it is a compile-time or construction-time failure rather than a review miss. That structure is what makes AC-05 enforceable.
- **AI Development Guidance**: `REF-LOG`, `REF-PROMPT-QUALITY`, `REF-PROMPT-API`; `CLAUDE.md`.
- **Required Human Review**: Security and privacy review of the accessor design and the path inventory.
- **Open Decisions**: None. Export (FR-9.3) and deletion (FR-9.4) execute synchronously (`SECURITY.md` SQ-5 RESOLVED) and are delivered by REQ-PRIVACY-080 and REQ-PRIVACY-090; both adopt this accessor, with export producing one audit entry for the whole operation, referencing its scope (FR-9.7).

**Estimated effort**: 1–2 engineer-days. **Estimated changed lines**: 300–650.
**Recommended model**: Claude Opus (`claude-opus-5`) — the correctness bar is structural (omission must be impossible, not merely discouraged), which rewards careful design over speed.
