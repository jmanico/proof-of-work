# [REQ-PRIVACY-020] Consent withdrawal blocks new health-data writes

## Metadata

- **ID**: REQ-PRIVACY-020
- **Title**: Consent withdrawal blocks new health-data writes
- **Version**: 1.1.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: High
- **Requirement Type**: Privacy
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-9.9, FR-9.12; `SECURITY.md` threat TM-P-2

## Requirement

- **Statement**: The system MUST allow a user to withdraw previously given consent to health-data collection, MUST NOT record new health data for that user while consent is withdrawn — no new log entries, plan copies, plan selections, or AI estimation requests (FR-9.12) — and MUST keep existing records subject to the export, deletion, and correction rights in FR-9.3–FR-9.5, with editing and deleting existing entries (FR-8.7) remaining available as exercise of the correction and deletion rights over already-collected data.
- **Rationale**: FR-9.9 states the rule, including the 2026-08-03 clarifications: new collection is enumerated through the FR-9.12 health-data boundary, and correction of existing records is expressly preserved under withdrawal — blocking a user from editing or deleting what was already collected would defeat the rights the withdrawal is meant to serve. The requirement exists because the threat model found that consent was captured (FR-9.2) with no path to withdraw it short of account deletion — LINDDUN Unawareness, recorded as TM-P-2.
- **Assumptions**: Consent capture exists and is checked on every health-data write and estimation request (REQ-PRIVACY-010). The health-data boundary is defined by FR-9.12 (REQ-PRIVACY-070).
- **Out of Scope**: Export (FR-9.3) and deletion (FR-9.4), which execute synchronously (`SECURITY.md` SQ-5 RESOLVED) and are delivered by REQ-PRIVACY-080 and REQ-PRIVACY-090; whether withdrawal should also suspend subscription entitlement, which no source document states; re-consent semantics beyond granting again.
- **Design Traceability**: `DESIGN.md` — Product Patterns → Account, privacy, and destructive actions ("Consent withdrawal explains that new logging stops while existing records remain available under FR-9.3–FR-9.5"); Status, feedback, and loading (withdrawn-consent state explains the exact reason and next step; the withdrawn-consent banner is a reserved actionable state); Core Components → Actions (destructive treatment, explicit verb, confirmation).
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application (FR-9.9 traceability row, SUPPORTED); data flow 4; Relational Persistence (consent record).
- **Security Traceability**: SEC-DATA-2 (consent precondition), SEC-LOG-1 (the change is audited), SEC-AUTHZ-1.

## Scope

- **Applies To**: Server-Side Application, API, Web Client
- **Components**: REST API Application; Relational Persistence; Browser Client (privacy settings view)
- **Interfaces / Operations**: Consent withdrawal; consent re-grant; the consent precondition on new health-data collection (FR-9.12: log entries, plan copies, plan selections, estimation requests); the preserved FR-8.7 edit and delete operations on existing entries
- **Actors**: `subscriber`
- **Preconditions**: Authenticated session; an active consent record exists for the acting subscriber
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable; GDPR-grade rights granted to all users (`SECURITY.md` SQ-1 RESOLVED, `REQUIREMENTS.md` OQ-3 RESOLVED)

## Security Context

- **Security Objectives**: Privacy, Accountability, Authorization
- **Control Layers**: Business-Rule Validation, Authorization, Data Protection
- **Threat References**: `SECURITY.md` TM-P-2 (Unawareness — no withdrawal path), TM-P-3 (Non-compliance); LINDDUN Unawareness; CWE-359 (exposure of private personal information)
- **Abuse / Misuse Case**: A user withdraws consent but health data continues to be recorded — because the check reads a cached session flag, because a consultant path bypasses it, or because the withdrawal only hides the UI. Alternatively, withdrawal is implemented as deletion, destroying records the user still has a right to export.
- **Trust Boundary**: Boundary 1 — the withdrawal state is server-side and cannot be asserted or overridden by the client.
- **Untrusted Inputs or Assertions**: Any client-supplied consent state; a session established before withdrawal.
- **Authoritative Enforcement Point**: The consent check inside the REST API Application's health-data accessor, reading current persisted state on every write.
- **Independent Verification**: Withdrawal takes effect for a session issued before the withdrawal, without waiting for re-authentication.
- **Zero Trust Relevance**: NIST SP 800-207 — policy conditions are re-evaluated per request from authoritative state rather than cached at session establishment. Exact tenet: TO BE DECIDED (not verified in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — mapped only when verified during the independent pre-launch assessment (`SECURITY.md` SQ-10 RESOLVED).
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — mapped only when verified during the independent pre-launch assessment (`SECURITY.md` SQ-10 RESOLVED).
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: The SQ-1 regime set governs (GDPR/UK GDPR for EU/UK data subjects; CCPA/CPRA, Washington My Health My Data, FTC HBNR for US users; HIPAA not applicable); the withdrawal right is required as behavior for all users. Statute-section precision: TO BE DECIDED — per-issue mappings await the SQ-1 pre-launch counsel review.
- **Other**: N/A — no external reference is cited by FR-9.9, which is threat-model-derived.
- **Mapping Basis**: FR-9.9 (with the FR-9.12 boundary) is the normative source and traces to TM-P-2 in `SECURITY.md`. The SQ-1 regime set is named without statute-section citations because no source document states one for withdrawal; section-level precision awaits the SQ-1 counsel review.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a subscriber with active consent, when they withdraw it, then the consent record is marked withdrawn with the time, an audit entry is written, and the response confirms that no new health data will be recorded.
2. **AC-02 — Boundary or failure behavior**: Given a subscriber whose consent is withdrawn, when any new health-data collection is attempted — a new log entry, plan copy, plan selection, or AI estimation request (FR-9.12), by the subscriber, by an engaged consultant, or from a session established before the withdrawal — then it is refused with a message stating that consent is required, and nothing is persisted and no estimation is performed.
3. **AC-03 — Prohibited behavior**: Given a withdrawal, when it is processed, then it MUST NOT delete, alter, or hide existing health records, MUST NOT be satisfiable by a client-side flag, and MUST NOT require the user to delete their account to achieve it.
4. **AC-04 — Additional criterion**: Given a subscriber with withdrawn consent, when they exercise export or correction rights, then those rights still operate on the retained records (FR-9.3–FR-9.5) — in particular, editing and deleting their existing log entries (FR-8.7) succeeds while consent is withdrawn — so withdrawal does not strand the data. An edit corrects an existing entry; it does not create a new one.
5. **AC-05 — Additional criterion**: Given a subscriber who withdrew consent, when they grant consent again, then new health-data writes are permitted from that point, and both the withdrawal and the re-grant are separately audited.

## Failure Behavior

- **On Invalid Input**: A malformed withdrawal request is rejected per REQ-API-010 with no state change.
- **On Authentication Failure**: Denied upstream; consent state unchanged.
- **On Authorization Failure**: A withdrawal submitted on another subscriber's behalf is denied (REQ-AUTHZ-020); withdrawal is a first-person act.
- **On Security-Decision Failure**: If consent state cannot be resolved, refuse the health-data write (fail closed) — an unresolvable state is not treated as consent.
- **On External Dependency Failure**: If persistence is unavailable, the withdrawal fails atomically and health-data writes remain gated by the last known persisted state; if that state cannot be read, writes are refused.
- **On System Error**: Roll back so neither a partial withdrawal nor a post-withdrawal health record survives.
- **Logging / Audit**: Audit entry for withdrawal and for re-grant, recording acting account, action, and time (REQ-AUDIT-020). Log refusals with reason class; MUST NOT log health values (SEC-LOG-3).
- **Alerting**: N/A — the source documents attach no alert condition to withdrawal; refusals are ordinary policy denials logged under SEC-LOG-4, whose threshold alerts route to the security lead as SEC-OPS-2 detection inputs (SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: Consent gate refuses new collection when the record is withdrawn; permits FR-8.7 edits and deletes of existing entries under withdrawal; refuses when state is unresolvable; permits after re-grant; withdrawal handler rejects a third-party request.
- **Integration Tests**: Withdraw, then attempt each new-collection type — log entry, plan copy, plan selection, estimation request — from a pre-existing session, asserting refusal; edit and delete an existing log entry under withdrawal, asserting success; assert existing records remain readable and exportable; re-grant and assert new collection resumes.
- **Security Tests**: Client-supplied consent flag ignored; consultant write against a withdrawn subscriber refused; a session issued before withdrawal does not retain permission; assertion that no records were removed by withdrawal; assertion that an "edit" cannot be used to fabricate a new entry under withdrawal.
- **Compliance Tests / Evidence**: Audit entries for withdrawal and re-grant, plus evidence that retained records remain accessible to the subject, as evidence for FR-9.9.
- **Acceptance-Criteria Traceability**: AC-01 — withdrawal suite; AC-02 — post-withdrawal refusal suite across FR-9.12 collection types and actors; AC-03 — record-preservation and client-flag negative tests; AC-04 — retained-rights test including FR-8.7 edits and deletes under withdrawal; AC-05 — re-grant suite.
- **Coverage Target**: Every FR-9.12 new-collection path covered by a post-withdrawal refusal test; the FR-8.7 edit and delete paths covered by post-withdrawal permission tests.
- **Required Test Environment**: A subscriber with consent granted and seeded health records, an engaged consultant, and a session captured before withdrawal. Runs against PostgreSQL on Vitest.

## Dependencies

- **Upstream Requirements**: REQ-PRIVACY-010, REQ-PRIVACY-070, REQ-AUTHZ-020, REQ-AUDIT-020
- **Downstream Requirements**: REQ-PROGRESS-010, REQ-PROGRESS-020, REQ-CONSULT-010, REQ-FOOD-040, REQ-SELECT-010, REQ-SELECT-020
- **External Dependencies**: None
- **Dependency Assumptions**: The consent check reads current persisted state per request, which REQ-PRIVACY-010 establishes.
- **Failure Impact**: Continuing to record health data after withdrawal is an unlawful-processing failure that later deletion does not cure, and it re-opens TM-P-2.

## Implementation Notes

- **Constraints**: PostgreSQL with Drizzle ORM (`CLAUDE.md`). FR-9.9 explicitly keeps existing records subject to FR-9.3–FR-9.5, so withdrawal MUST NOT be implemented as deletion, and the gate distinguishes new collection (refused) from FR-8.7 correction of existing entries (permitted) — a blanket write block on all health-data operations does not satisfy the amended rule.
- **Prohibited Approaches**: Deleting or anonymizing records on withdrawal; caching consent in the session or token (which SEC-SESSION-6 already forbids for sensitive claims and SEC-SESSION-4's revocation principle argues against generally); hiding the logging UI without enforcing server-side; requiring account deletion as the only withdrawal path.
- **Implementation Guidance**: Model consent as a record with an explicit state and timestamps rather than a boolean, so withdrawal and re-grant are both auditable events and the history is reconstructable.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-VUE`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Privacy and legal review of withdrawal semantics; product review of what the user is told about retained records.
- **Open Decisions**: Whether withdrawal should also affect subscription entitlement or consultant engagements is unspecified in all source documents. The governing regime set is fixed (`SECURITY.md` SQ-1 RESOLVED); whether withdrawal carries additional obligations such as a mandatory erasure offer is confirmed at the SQ-1 pre-launch counsel review.

**Estimated effort**: 0.5–1.5 engineer-days. **Estimated changed lines**: 200–400.
**Recommended model**: Claude Opus (`claude-opus-5`) — a privacy state machine whose failure mode is silent and legally significant.
