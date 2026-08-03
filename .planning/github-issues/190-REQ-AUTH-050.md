# [REQ-AUTH-050] Authentication and account-change security event logging

## Metadata

- **ID**: REQ-AUTH-050
- **Title**: Authentication and account-change security event logging
- **Version**: 1.1.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: High
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `SECURITY.md` SEC-LOG-4, SEC-AUTHN-7; threats TM-S-1, TM-S-4, TM-E-1

## Requirement

- **Statement**: Authentication successes and failures, authorization denials, and security-relevant account changes MUST be recorded as structured log events carrying enough context to detect attack patterns — account identifier where known, event type, outcome, cause class, source context, and time — without recording the credentials involved.
- **Rationale**: SEC-LOG-4 states the rule. The threat model relies on it to detect credential stuffing (TM-S-1), privileged enrolment abuse (TM-S-4), and privilege escalation attempts (TM-E-1), none of which are individually blocked by another control.
- **Assumptions**: A structured logging sink exists inside the system boundary.
- **Out of Scope**: Health-data audit entries, which are a separate obligation with separate integrity requirements (REQ-AUDIT-010, REQ-AUDIT-020); log retention, archive storage, and access control (SEC-LOG-5, SEC-LOG-7; `SECURITY.md` SQ-8 RESOLVED — 12-month security logs, 3-year audit entries, hash-chained archive — delivered by REQ-INFRA-040); the application of SQ-3's fixed alert thresholds, which routes to the security lead per SEC-OPS-2 (REQ-OPS-010); log redaction rules, which REQ-AUDIT-040 owns and this issue consumes.
- **Design Traceability**: N/A
- **Architecture Traceability**: `ARCHITECTURE.md` — Identity and Session Handling; REST API Application outputs.
- **Security Traceability**: SEC-LOG-4, SEC-LOG-3 (redaction), SEC-AUTHN-7 (account changes are audited as well as logged), SEC-TB-3 (logs stay inside the boundary).

## Scope

- **Applies To**: Server-Side Application, API
- **Components**: Identity and Session Handling; REST API Application
- **Interfaces / Operations**: Authentication attempts; session issuance and termination; passkey registration and replacement; role changes; authorization denials
- **Actors**: All roles; anonymous attackers
- **Preconditions**: None
- **Data Classification**: Confidential — security logs identify accounts and their activity
- **Personal or Regulated Data**: Personal Data — account identifiers; Health Data MUST NOT appear
- **Jurisdiction / Regulatory Scope**: Global service, single US primary region (`SECURITY.md` SQ-1 RESOLVED): GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; GDPR-grade rights granted to all users; HIPAA not applicable

## Security Context

- **Security Objectives**: Accountability, Integrity, Privacy
- **Control Layers**: Logging and Monitoring
- **Threat References**: `SECURITY.md` TM-S-1, TM-S-4, TM-E-1, TM-I-5 (data leaking into logs); CWE-778 (insufficient logging), CWE-532 (insertion of sensitive information into log file), CWE-117 (improper output neutralization for logs)
- **Abuse / Misuse Case**: A sustained credential attack or a privileged enrolment leaves no detectable trace; or the logging itself becomes the leak, recording passwords, tokens, or health values, or accepting attacker-controlled text that forges log entries.
- **Trust Boundary**: Boundary 2 for the events; boundary 5 for the log store, protected against operator-level tampering by SEC-LOG-7's append-only database privileges and hash-chained Object Lock archive (`SECURITY.md` SQ-8, SQ-13 RESOLVED; delivered by REQ-INFRA-040).
- **Untrusted Inputs or Assertions**: Any request-derived value written into a log record, including submitted identifiers.
- **Authoritative Enforcement Point**: A shared structured logging component invoked by Identity and Session Handling and the REST API Application.
- **Independent Verification**: Log events are emitted by the enforcement paths themselves, not reconstructed from client reports.
- **Zero Trust Relevance**: N/A — this is monitoring, not a resource-access decision.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: N/A
- **Regulatory**: GDPR and UK GDPR (EU/UK data subjects); CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule (US users); HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED). Retention is fixed at 12 months for security logs and 3 years for audit entries (SQ-8 RESOLVED, SEC-LOG-5). Statute-section mappings: TO BE DECIDED — verified at the SQ-1 pre-launch counsel review.
- **Other**: `REF-LOG` as cited by SEC-LOG-4.
- **Mapping Basis**: SEC-LOG-4 cites `REF-LOG` and `REF-ASVS-5`; the CWE identifiers name insufficient logging and the two ways logging itself becomes a defect.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authentication success, an authentication failure, an authorization denial, or a security-relevant account change, when it occurs, then exactly one structured log event is emitted carrying event type, outcome, cause class where applicable, account identifier where known, source context, correlation identifier, and time.
2. **AC-02 — Boundary or failure behavior**: Given the logging sink is unavailable, when a security event occurs, then the underlying operation still completes or fails on its own merits, the failure to log is itself recorded through the fallback path, and the outcome is never converted from deny to allow.
3. **AC-03 — Prohibited behavior**: Given any security log event, when it is inspected, then it MUST NOT contain a password, password hash, token, passkey or attestation material, MFA secret, or any health value (SEC-LOG-3), and MUST NOT be shipped to a destination outside the system boundary (SEC-TB-3).
4. **AC-04 — Additional criterion**: Given a request-derived value written into a log record, when the record is emitted, then the value is encoded so that newline or delimiter characters cannot forge an additional log entry.
5. **AC-05 — Additional criterion**: Given a security-relevant account change — passkey registration or replacement, role change, or an email-address change under FR-2.18 — when it occurs, then it produces both this security log event and the audit entry required by SEC-AUTHN-7.

## Failure Behavior

- **On Invalid Input**: N/A — logging consumes already-classified events, and untrusted values are encoded per AC-04.
- **On Authentication Failure**: The failure is one of the logged events; the client-facing response remains uniform per REQ-AUTH-040.
- **On Authorization Failure**: The denial is logged per REQ-AUTHZ-040, using this component.
- **On Security-Decision Failure**: Never allow an operation because logging failed; and never suppress a denial because the log write errored.
- **On External Dependency Failure**: On sink unavailability, fall back to the local process log and record the degradation.
- **On System Error**: Log the fault with a correlation identifier; the diagnostic content stays server-side (SEC-ERR-1).
- **Logging / Audit**: This issue defines the events and fields. Audit entries for health-data access are separate (REQ-AUDIT-020) and MUST NOT be replaced by these log events.
- **Alerting**: These events are the named detection inputs of the incident-response process: SQ-3 threshold alerts over them route to the security lead per SEC-OPS-2 (`SECURITY.md` SQ-3, SQ-11 RESOLVED; runbook delivered by REQ-OPS-010). The event schema supports those thresholds without changing emitters.

## Test Strategy

- **Unit Tests**: Event builder produces the required field set per event type; encoder neutralizes newline and delimiter characters; redaction drops prohibited fields.
- **Integration Tests**: Each event type triggered end-to-end produces exactly one record with the expected fields; sink-unavailable case exercises the fallback without changing the operation's outcome.
- **Security Tests**: Log-content assertion over a corpus of events confirming no credential, token, or health value appears; log-injection test submitting newline-bearing identifiers; assertion that no log transport targets an external destination.
- **Compliance Tests / Evidence**: Sample records per event type, retained as accountability evidence.
- **Acceptance-Criteria Traceability**: AC-01 — event emission suite; AC-02 — sink-failure test; AC-03 — content assertions plus egress review; AC-04 — injection test; AC-05 — paired log-and-audit test.
- **Coverage Target**: Every event type emitted and asserted; positive and negative coverage of redaction.
- **Required Test Environment**: Log capture; identities for each role; a failing sink fixture. Vitest as the runner, capturing pino output; retention and archive behavior are exercised by REQ-INFRA-040's suite, not here.

## Dependencies

- **Upstream Requirements**: REQ-AUDIT-040 (redaction rules), REQ-API-040 (correlation identifiers)
- **Downstream Requirements**: REQ-AUTH-020, REQ-AUTH-030, REQ-AUTH-040, REQ-AUTHZ-040
- **External Dependencies**: None — logs MUST NOT leave the system boundary while they identify health-data subjects (SEC-TB-3, FR-9.8).
- **Dependency Assumptions**: The logging sink is inside the system boundary and is not a third-party monitoring service.
- **Failure Impact**: Without these events, every credential and privilege attack in the threat model is invisible; with careless implementation, the log becomes the disclosure.

## Implementation Notes

- **Constraints**: Node.js runtime with Fastify and its pino logger, whose redaction paths are the intended delivery mechanism for SEC-LOG-3. Retention, archive storage, and tamper evidence against operator-level actors are resolved (`SECURITY.md` SQ-8, SQ-13; SEC-LOG-5, SEC-LOG-7) and delivered by REQ-INFRA-040, not here.
- **Prohibited Approaches**: Logging the submitted credential "temporarily for debugging"; free-text log lines that cannot be queried by field; shipping security logs to an external analytics or error-reporting service; emitting an event per code path so that one action produces several ambiguous records.
- **Implementation Guidance**: Emit events from the enforcement points themselves so that adding a new credential path without logging is visible in review. Where an account cannot be resolved, log a stable hash of the submitted identifier rather than the identifier itself, so patterns remain detectable without storing unverified personal data.
- **AI Development Guidance**: `REF-LOG`, `REF-PROMPT-NODE`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the event schema; privacy review that no health data or credential can enter a record.
- **Open Decisions**: None. Retention, storage, and access control (`SECURITY.md` SQ-8 RESOLVED; SEC-LOG-5) and tamper evidence below the application (SEC-LOG-7, SQ-13 RESOLVED) are delivered by REQ-INFRA-040; alerting thresholds and routing (SQ-3, SQ-11 RESOLVED) are exercised through the SEC-OPS-2 runbook delivered by REQ-OPS-010.

**Estimated effort**: 0.5–1.5 engineer-days. **Estimated changed lines**: 200–450.
**Recommended model**: Claude Opus (`claude-opus-5`) — a control where the implementation can easily become the vulnerability it is meant to detect.
