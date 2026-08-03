# [REQ-PRIVACY-090] Synchronous account deletion with deletion-ledger write

## Metadata

- **ID**: REQ-PRIVACY-090
- **Title**: Synchronous account deletion with deletion-ledger write
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Privacy
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-9.4 (execution clock, re-authentication, privileged floor, and ledger resolved 2026-08-03); `SECURITY.md` SEC-DATA-4, SEC-AUTHN-7, SQ-5 RESOLVED

## Requirement

- **Statement**: The system MUST execute an account-deletion request as a synchronous hard delete of all of the account's personal and health data, effective at execution time — immediate for an authenticated request after fresh re-authentication no older than 5 minutes, and after the 14-day window for an FR-9.11 request — writing at execution a deletion-ledger entry (the keyed tombstone and the execution time, no plaintext identifier) to the log-archive account; full completion, including expiry of backup copies, occurs within 35 days of execution, and any backup restore MUST re-apply ledger deletions before the restored system serves traffic. Deletion of an `admin` account MUST be refused while it would leave fewer than two `admin` accounts, and deletion of a `consultant` account MUST first end its active engagements with access revoked.
- **Rationale**: FR-9.4 grants the deletion right (a GDPR-grade data-subject right under SQ-1) and fixes its mechanics: synchronous hard delete keeps the promise observable at request time, the 35-day clock from execution bounds backup residue — a completion-clock position recorded for confirmation at the SQ-1 counsel review (SQ-5), the deletion ledger makes restore re-deletion mechanical rather than best-effort (SEC-DATA-4, threats TM-P-1, TM-I-6), the two-admin floor preserves FR-2.15's operability invariant, and ending consultant engagements first prevents dangling access records (FR-11.3).
- **Assumptions**: The keyed tombstone derivation exists (REQ-PRIVACY-100); the log-archive account and its storage exist (REQ-INFRA-040); engagement termination with immediate revocation exists (REQ-CONSULT-020); the FR-9.11 channel (REQ-PRIVACY-110) invokes this execution path after its window expires.
- **Out of Scope**: The out-of-band request channel, its identity checks, notices, and 14-day cancellation window (FR-9.11; REQ-PRIVACY-110) — this issue receives its expired requests as an execution trigger; the tombstone derivation itself and audit-entry rewriting (REQ-PRIVACY-100); admin-initiated deprovisioning, which disables but does not delete (FR-2.17; REQ-AUTH-170); data export (REQ-PRIVACY-080); backup infrastructure configuration (REQ-INFRA-040 and Terraform work).
- **Design Traceability**: `DESIGN.md` — Account, privacy, and destructive actions: deletion uses a dedicated page, not a small dialog; it summarizes immediate live-system deletion, the 35-day backup horizon, audit tombstoning, and loss of access; the final action requires fresh re-authentication and an explicit destructive button labelled **Permanently delete account**. Core Components → Actions (destructive button treatment); Re-authentication pattern (prompt names the operation, 5-minute window).
- **Architecture Traceability**: `ARCHITECTURE.md` — data flow 8 (hard deletion executed synchronously, audit entry written) and data flow 9 (FR-9.11 window expiry evaluated by a scheduled execution, then FR-9.4 deletion executes); "No background-processing boundary exists for user-facing operations"; the deletion ledger lives in the log-archive account, outside the restored dataset (Data model expectations); DR-5 (scheduled executions are sanctioned persistence paths).
- **Security Traceability**: SEC-DATA-4, SEC-AUTHN-7, SEC-AUTHN-12/SEC-SESSION-3 (all sessions end with the account), SEC-SESSION-4 (engagement revocation immediacy), SEC-OPS-2 (restore re-deletion is a runbook step), SEC-LOG-1; FR-9.10/SEC-LOG-3 via REQ-PRIVACY-100.

## Scope

- **Applies To**: Multiple — API, Server-Side Application, Web Client (dedicated deletion page)
- **Components**: REST API Application (including its scheduled executions); Identity and Session Handling (re-authentication, session termination); Relational Persistence; Browser Client
- **Interfaces / Operations**: The dedicated deletion page and its execute operation; the FR-9.11 window-expiry execution trigger; the restore re-deletion procedure
- **Actors**: Any authenticated account deleting itself — subscriber, consultant, or admin; the scheduled execution for expired FR-9.11 requests; operators performing a restore
- **Preconditions**: For the in-session path, an authenticated session and fresh re-authentication no older than 5 minutes; for the out-of-band path, an FR-9.11 request whose 14-day window has expired uncancelled
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED)

## Security Context

- **Security Objectives**: Privacy, Confidentiality, Integrity, Accountability, Availability
- **Control Layers**: Authorization, Authentication, Session Management, Data Protection, Business-Rule Validation, Logging and Monitoring
- **Threat References**: `SECURITY.md` TM-P-1 (identifiability — deletion may not de-identify backups and derived copies), TM-I-6 (backups hold over-retained health data), TM-D-2 analogue (deletion as a destruction attack against a chosen victim — mitigated here by re-authentication and by FR-9.11's window living in REQ-PRIVACY-110); CWE-459 (incomplete cleanup)
- **Abuse / Misuse Case**: An attacker with a stolen but idle session destroys the victim's account and history; a request-supplied identifier deletes a different account; a restore silently resurrects deleted health data; deleting the second-to-last admin bricks privileged administration; a deleted consultant's engagement records leave residual access.
- **Trust Boundary**: Boundary 1 for the request; boundary 2 for the re-authentication assertion; boundary 5 for the restore procedure, which handles health data below the application.
- **Untrusted Inputs or Assertions**: Any request-supplied account identifier (the target is only ever the authenticated account, SEC-INPUT-3); the claim that re-authentication is fresh — verified against the server-side session record.
- **Authoritative Enforcement Point**: REST API Application — it verifies freshness, the admin floor, and engagement termination before executing; Relational Persistence enforces referential deletion; the ledger write is part of execution, not an afterthought.
- **Independent Verification**: The deletion target is bound to the authenticated identity from Identity and Session Handling (DR-3); the floor check counts `admin` accounts from persisted state; restore re-deletion is verified against the ledger, not against operator memory.
- **Zero Trust Relevance**: NIST SP 800-207 — a security-critical destructive change is re-authorized with fresh authentication rather than inherited from an existing session. Exact tenet: TO BE DECIDED (not verified against the publication in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session (SQ-10 pre-launch assessment).
- **OWASP AISVS 1.0**: N/A — no AI component participates in deletion.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: The SQ-1 regime set — GDPR/UK GDPR (erasure right motivates FR-9.4), CCPA/CPRA, Washington My Health My Data, FTC HBNR; HIPAA not applicable. The 35-day completion clock from execution is recorded for confirmation at the SQ-1 counsel review (SQ-5). Statute-section mappings: TO BE DECIDED (SQ-1 counsel review).
- **Other**: `REF-ASVS-5` as cited by SEC-DATA-4; `REF-AUTH`, `REF-63B` as cited by SEC-AUTHN-7.
- **Mapping Basis**: SEC-DATA-4 and SEC-AUTHN-7 name these references; CWE-459 describes the incomplete-cleanup class the ledger and restore contract defeat.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated user whose re-authentication is no older than 5 minutes, when they confirm deletion on the dedicated page, then all of the account's personal and health rows — account record, credentials, MFA enrolment, sessions, tokens, consent records, plan copies, selections, and all log entries — are hard-deleted synchronously in the live system; a deletion-ledger entry containing the keyed tombstone and the execution time, and no plaintext identifier, is written to the log-archive account; audit entries are tombstoned per REQ-PRIVACY-100; the deletion itself produces an audit entry; and no subsequent authentication or data access for the account succeeds.
2. **AC-02 — Boundary or failure behavior**: Given an `admin` account requesting self-deletion while exactly two `admin` accounts exist, when it confirms, then the deletion is refused with the reason and the remedy — provision a replacement (FR-2.10) or deprovision under FR-2.17 first — and no state changes; and given a `consultant` account with active engagements, when its deletion executes, then every engagement is ended with the consultant's access revoked immediately (FR-11.3, SEC-SESSION-4) before the account rows are deleted.
3. **AC-03 — Prohibited behavior**: Given a session whose re-authentication is absent or older than the 5-minute constant, when deletion is confirmed, then nothing is deleted and the refusal is logged; given any request shape, when deletion executes, then a request-supplied account identifier MUST NOT select the target — only the authenticated account is ever deleted; after execution, no health record of the account is queryable through any application interface, and no recoverable soft-delete state remains in the live system; and the ledger entry MUST NOT contain a plaintext account identifier.
4. **AC-04 — Additional criterion**: Given a backup restore containing accounts deleted since that backup, when the restore procedure runs, then before the restored system serves traffic it derives tombstones for restored accounts, hard-deletes every ledger match, and records the re-deletion (SEC-DATA-4; SEC-OPS-2 runbook step); and backup copies of the deleted data expire within 35 days of execution. For an FR-9.11 request whose 14-day window expired uncancelled, the scheduled execution triggers this same execution path and clock.

## Failure Behavior

- **On Invalid Input**: The operation binds no client-assignable target fields; unexpected parameters are rejected per SEC-INPUT-1 and SEC-INPUT-3 with no state change.
- **On Authentication Failure**: Stale or failed re-authentication refuses the deletion with a uniform response (SEC-AUTHN-3) and a logged refusal; the account and its data are untouched.
- **On Authorization Failure**: Deny — there is no path by which one account deletes another through this operation; denial follows REQ-AUTHZ-040 semantics. The admin-floor refusal is a business-rule denial stating reason and remedy (`DESIGN.md` refusal-with-remedy pattern).
- **On Security-Decision Failure**: Deny by default — if freshness, the admin-floor count, engagement termination, the ledger write, or the tombstoning step cannot be completed, the deletion does not execute; a deletion MUST NOT complete without its ledger entry, since restore re-deletion depends on it.
- **On External Dependency Failure**: If the log-archive account is unreachable, the deletion is refused and retriable rather than executed unledgered; the failure is logged and alerted. For the scheduled FR-9.11 execution, the request remains pending and is retried.
- **On System Error**: The hard delete is transactionally consistent — a mid-operation failure rolls back to the pre-deletion state rather than leaving the account half-deleted; the user receives a generic error with a correlation identifier (SEC-ERR-1).
- **Logging / Audit**: One audit entry for the deletion (actor, action, subject, time — subject subsequently tombstoned per REQ-PRIVACY-100); refusals (stale re-auth, admin floor, ledger unavailability) logged with reason class (SEC-LOG-4). Logs MUST NOT contain deleted health values or the ledger key material (SEC-LOG-3).
- **Alerting**: Ledger-write failures and any restore performed without a completed re-deletion step alert the security lead as SEC-OPS-2 detection inputs; every restore is a runbook event (`SECURITY.md` SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: Freshness check at and beyond the 5-minute boundary; admin-floor check refuses at two admins and permits at three; target binding ignores request-supplied identifiers; ledger-entry construction contains tombstone and execution time only; execution ordering — engagements end before consultant rows delete, ledger write completes within execution.
- **Integration Tests**: Full deletion fixture asserting zero remaining rows for the account across every table in Relational Persistence and denial of all subsequent authentication and data access; consultant-deletion fixture asserting engagement termination and immediate access revocation first; FR-9.11 expiry trigger executing the same path; rollback fixture asserting a forced mid-deletion failure leaves the account fully intact.
- **Security Tests**: Stolen-session scenario — a valid session without fresh re-authentication cannot delete (SEC-AUTHN-7); attempt to delete another account via forged identifiers is denied without disclosure (SEC-INPUT-3, TM-I-1 analogue); post-deletion probing of every read path asserting no residue (SEC-DATA-4, CWE-459); restore drill — restore a backup containing a deleted account and verify re-deletion completes before traffic is served, with the re-deletion recorded.
- **Compliance Tests / Evidence**: The deletion fixture and restore-drill record as erasure evidence for the SQ-1 counsel review; documentation of the 35-day backup expiry configuration (with REQ-INFRA-040).
- **Acceptance-Criteria Traceability**: AC-01 — full deletion integration suite; AC-02 — admin-floor and consultant-engagement fixtures; AC-03 — stale-re-auth, forged-target, residue, and ledger-content tests; AC-04 — restore drill and FR-9.11 expiry trigger test.
- **Coverage Target**: Project-defined; every refusal path and the rollback path covered positive and negative.
- **Required Test Environment**: Vitest and HTTP test client; fixtures for subscriber, consultant-with-engagements, and admin accounts at the two-admin floor; a log-archive stand-in for ledger writes with a failure mode; restorable backup fixture for the drill.

## Dependencies

- **Upstream Requirements**: REQ-PRIVACY-100, REQ-AUTH-170, REQ-INFRA-040, REQ-CONSULT-020, REQ-SESSION-040, REQ-PRIVACY-070
- **Downstream Requirements**: REQ-PRIVACY-110 (the out-of-band channel triggers this execution path)
- **External Dependencies**: None at runtime beyond the system's own log-archive account (in-boundary; REQ-INFRA-040).
- **Dependency Assumptions**: The tombstone derivation is deterministic and keyed (REQ-PRIVACY-100) so ledger matching works on restore; the log-archive account is writable by the sanctioned path and outside the restored dataset (SEC-DATA-4, DR-5).
- **Failure Impact**: An unledgered deletion resurrects on restore, breaching the erasure right silently (TM-P-1); a deletion without the admin floor check can strand privileged administration below FR-2.15's minimum; incomplete cleanup leaves health data queryable after the user was told it was gone.

## Implementation Notes

- **Constraints**: Node.js runtime with Fastify, PostgreSQL with Drizzle ORM (`CLAUDE.md`). Execution is synchronous — no background-processing boundary exists for user-facing operations (ARCHITECTURE.md); the FR-9.11 expiry evaluation is a scheduled execution of the same application. The 5-minute freshness window, 14-day window, and 35-day horizon are named constants fixed by SEC-AUTHN-7, FR-9.11, and SQ-5.
- **Prohibited Approaches**: Soft delete or flag-based "deletion" in the live system; deferring the hard delete to a queue; accepting a target account identifier from the request; executing without the ledger write; writing a plaintext identifier to the ledger; skipping restore re-deletion because a restore is urgent (it is a SEC-OPS-2 runbook step, not optional).
- **Implementation Guidance**: Order execution as: verify freshness → check role-specific preconditions (admin floor; end consultant engagements) → compute tombstone (REQ-PRIVACY-100) → write ledger entry → hard-delete rows in one transaction with audit tombstoning → terminate sessions → write the deletion audit entry. Model foreign keys so ownership cascades make the hard delete complete by construction; the post-deletion residue test is the check on that construction.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-NODE`, `REF-PROMPT-TF-AWS` (ledger storage interaction), `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security and privacy review of the execution ordering and ledger contents; operations review of the restore re-deletion runbook step (SEC-OPS-2); the 35-day-from-execution position is confirmed at the SQ-1 counsel review.
- **Open Decisions**: None — the execution clock, re-authentication, privileged floor, ledger, and restore contract are all fixed (FR-9.4, SEC-DATA-4, SQ-5, 2026-08-03).

**Estimated effort**: 1.5–2 engineer-days. **Estimated changed lines**: 500–900.
**Recommended model**: Claude Opus (`claude-opus-5`) — an irreversible destructive operation whose ordering, transactional integrity, and restore contract are the whole point.
