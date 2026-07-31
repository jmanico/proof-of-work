# [REQ-AUDIT-040] Log redaction of health values, credentials, and tokens

## Metadata

- **ID**: REQ-AUDIT-040
- **Title**: Log redaction of health values, credentials, and tokens
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: Critical
- **Requirement Type**: Privacy
- **Source / Parent**: REQ-EPIC-001; `SECURITY.md` SEC-LOG-3, SEC-TB-3; threat TM-I-5

## Requirement

- **Statement**: No logging surface may record health values, credentials, tokens, or full personal records, and no log or diagnostic data may be transmitted to a destination outside the system boundary.
- **Rationale**: SEC-LOG-3 states the content rule and notes that audit entries reference the data accessed rather than copying it; SEC-TB-3 and FR-9.8 forbid transmitting health data to any external service including logging and monitoring destinations. The threat model rates health data leaking into logs, error responses, or external analytics (TM-I-5) as high severity.
- **Assumptions**: A structured logging component exists (REQ-AUTH-050) and all logging flows through it.
- **Out of Scope**: What events are logged (REQ-AUTH-050); the audit entry model (REQ-AUDIT-010); retention and access control for logs (SEC-LOG-5, blocked by `SECURITY.md` SQ-8); tamper-evidence (SEC-LOG-7, blocked by SQ-13).
- **Design Traceability**: `DESIGN.md` — Components → Form feedback and errors, insofar as user-visible messages must not become a second uncontrolled surface for the values this rule protects.
- **Architecture Traceability**: `ARCHITECTURE.md` — "All outbound paths"; the absence of an external-integration boundary by construction; DR-7.
- **Security Traceability**: SEC-LOG-3, SEC-TB-3, SEC-SECRET-1, SEC-ERR-1; supports SEC-DATA-1, SEC-RENDER-4.

## Scope

- **Applies To**: Server-Side Application, API, Web Client
- **Components**: REST API Application; Identity and Session Handling; Browser Client (client-side error reporting); Relational Persistence (audit content)
- **Interfaces / Operations**: Every logging call, error handler, diagnostic output, and any client-side console or reporting path
- **Actors**: All actors indirectly; operators as log readers (trust boundary 5)
- **Preconditions**: None
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED (`SECURITY.md` SQ-1, `REQUIREMENTS.md` OQ-3)

## Security Context

- **Security Objectives**: Confidentiality, Privacy
- **Control Layers**: Logging and Monitoring, Data Protection, Supply Chain
- **Threat References**: `SECURITY.md` TM-I-5 (health data in logs, errors, or external analytics), TM-I-8 (operator reads production data), TM-P-4 (audit trail sensitivity); CWE-532 (insertion of sensitive information into log file), CWE-359 (exposure of private personal information), CWE-200 (exposure of sensitive information to an unauthorized actor)
- **Abuse / Misuse Case**: A request body containing a body weight, a food entry, or a password is logged verbatim during debugging and then read by an operator, retained indefinitely, or shipped to a third-party monitoring service — turning a control-free surface into the easiest path to health data.
- **Trust Boundary**: Boundary 5 (operators read logs without traversing the REST API's enforcement point) and the system egress boundary (SEC-TB-3).
- **Untrusted Inputs or Assertions**: Any object passed to a logging call; any dependency that logs autonomously.
- **Authoritative Enforcement Point**: The shared logging component, which redacts by policy rather than by caller discipline.
- **Independent Verification**: A corpus test scans emitted logs for known seeded health values and credentials.
- **Zero Trust Relevance**: N/A — data minimization in a monitoring surface, not a resource-access decision.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: N/A
- **Regulatory**: TO BE DECIDED — blocked by `SECURITY.md` SQ-1; under any of the regimes `REQUIREMENTS.md` names, health data in logs is in scope for the same obligations as health data in the database.
- **Other**: `REF-LOG`, `REF-PROMPT-NODE` as cited by SEC-LOG-3.
- **Mapping Basis**: SEC-LOG-3 cites `REF-LOG`, `REF-PROMPT-NODE`, and `REF-ASVS-5`; SEC-TB-3 supplies the egress prohibition; the CWE identifiers name the exposure classes.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a logging call carrying an object that includes health, credential, or token fields, when the record is emitted, then those fields are absent or replaced with a redaction marker, and the remaining context — event type, actor, correlation identifier — is preserved.
2. **AC-02 — Boundary or failure behavior**: Given a corpus of logs produced by exercising every logged event with seeded health values and credentials, when the corpus is scanned, then none of the seeded values appears in any record, in any encoding.
3. **AC-03 — Prohibited behavior**: Given the application and infrastructure configuration, when egress is reviewed, then no log, error report, trace, metric, or diagnostic payload is sent to any destination outside the system boundary (SEC-TB-3, FR-9.8), including third-party analytics, error-reporting, and monitoring services.
4. **AC-04 — Additional criterion**: Given a redaction policy, when a new field is added to any request or entity, then the policy defaults to redacting unknown fields in logged payloads rather than emitting them.
5. **AC-05 — Additional criterion**: Given the Browser Client, when an error occurs, then it does not write health data or tokens to the browser console or to any client-side reporting channel (SEC-RENDER-4).

## Failure Behavior

- **On Invalid Input**: N/A — redaction consumes internal objects, not request input directly.
- **On Authentication Failure**: The failure is logged per REQ-AUTH-050 with the submitted credential redacted.
- **On Authorization Failure**: The denial is logged per REQ-AUTHZ-040 with target content redacted.
- **On Security-Decision Failure**: If the redaction policy cannot classify a field, redact it (fail closed, AC-04).
- **On External Dependency Failure**: N/A — no external logging destination is permitted, so none can fail.
- **On System Error**: The fault is logged with a correlation identifier and redacted context; the client sees only the generic error (SEC-ERR-1).
- **Logging / Audit**: This issue governs the content of all logging. A redaction failure detected in test is a security defect, not a cosmetic one.
- **Alerting**: N/A

## Test Strategy

- **Unit Tests**: Redactor removes each protected field class from nested objects and arrays; unknown fields are redacted by default; the redaction marker is not itself confusable with data.
- **Integration Tests**: Exercise every logged event with seeded values and assert absence in the emitted records.
- **Security Tests**: Corpus scan for seeded health values, passwords, tokens, and passkey material across all log surfaces; egress review of application and infrastructure configuration asserting no external destination; client-side console assertion; a test that a raw request body cannot be logged.
- **Compliance Tests / Evidence**: The corpus scan report and the egress review, retained as evidence for FR-9.8 and SEC-TB-3.
- **Acceptance-Criteria Traceability**: AC-01 — redactor unit suite; AC-02 — corpus scan; AC-03 — egress review; AC-04 — unknown-field default test; AC-05 — client console test.
- **Coverage Target**: Every logged event and every protected field class exercised.
- **Required Test Environment**: Seeded health data with distinctive values, log capture across server and client, and access to the deployment configuration for egress review. pino log capture on Vitest; the log sink and deployment topology remain TO BE DECIDED (`SECURITY.md` SQ-7, SQ-8).

## Dependencies

- **Upstream Requirements**: None
- **Downstream Requirements**: REQ-AUTH-050, REQ-AUTHZ-040, REQ-AUDIT-010, REQ-AUDIT-020, REQ-API-040, REQ-PRIVACY-050
- **External Dependencies**: None permitted for log transport (SEC-TB-3).
- **Dependency Assumptions**: No third-party library logs request or entity payloads autonomously; where one does, its logging is disabled or routed through the redactor (DEP-6, DEP-8).
- **Failure Impact**: A logged health value is a disclosure that persists in backups and operator-readable stores, outside every control the REST API enforces.

## Implementation Notes

- **Constraints**: Node.js runtime with Fastify and its pino logger, whose redaction paths are the intended delivery mechanism for SEC-LOG-3; the log sink remains TO BE DECIDED (`SECURITY.md` SQ-8). `SECURITY.md` explicitly anticipates future integrations ("none YET"), so the egress prohibition must be enforced by configuration review, not only by the current absence of integrations.
- **Prohibited Approaches**: Deny-lists of field names as the only mechanism; logging whole request or response bodies at any level including debug; enabling a third-party APM, error-reporting, or session-replay tool; redacting in the sink rather than before emission.
- **Implementation Guidance**: Prefer an allow-list of loggable fields over a deny-list of forbidden ones, which is what makes AC-04 achievable. Audit entries reference data by identifier rather than copying it (REQ-AUDIT-010), so the same discipline applies on both surfaces.
- **AI Development Guidance**: `REF-LOG`, `REF-PROMPT-NODE`, `REF-PROMPT-VUE`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Privacy review of the redaction policy; security review of egress configuration.
- **Open Decisions**: Log retention and access control (`SECURITY.md` SQ-8) and operator access to logs (SQ-13, SEC-OPS-1) remain open; redaction reduces but does not eliminate the exposure those questions cover.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 200–450.
**Recommended model**: Claude Opus (`claude-opus-5`) — a privacy control whose failures are silent and whose correct form (allow-list, fail-closed) is easy to get subtly wrong.
