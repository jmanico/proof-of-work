# [REQ-AUTHZ-040] Authorization denial response and logging semantics

## Metadata

- **ID**: REQ-AUTHZ-040
- **Title**: Authorization denial response and logging semantics
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: High
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `SECURITY.md` SEC-LOG-4, SEC-ERR-1, SEC-AUTHZ-7; threats TM-I-1, TM-I-4, TM-E-2

## Requirement

- **Statement**: Every authorization denial MUST produce a uniform response that does not disclose the existence, ownership, or attributes of the target resource, and MUST be recorded as a structured security log event carrying the acting account, the attempted operation, the target class, and a correlation identifier, without recording health values, credentials, or tokens.
- **Rationale**: SEC-LOG-4 requires authorization denials to be logged with enough context to detect attack patterns without logging credentials; SEC-ERR-1 requires responses free of internal detail; the threat model records enumeration (TM-I-4) and BOLA (TM-I-1) as threats that a leaky denial response assists.
- **Assumptions**: Denials originate from REQ-AUTHZ-010, REQ-AUTHZ-020, REQ-AUTHZ-030, and REQ-CONSULT-010.
- **Out of Scope**: The authorization decisions themselves; audit entries for successful health-data access (REQ-AUDIT-020); log retention and access control (SEC-LOG-5, blocked by `SECURITY.md` SQ-8); alert thresholds (SQ-3).
- **Design Traceability**: `DESIGN.md` — Components → Form feedback and errors, insofar as a denial surfaced in the UI must be presented as text plus icon, never color alone, and announced to assistive technology.
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application outputs ("authorization errors carrying the specific failing … reason"), bounded here by SEC-ERR-1's disclosure limits.
- **Security Traceability**: SEC-LOG-4, SEC-ERR-1, SEC-AUTHZ-7, SEC-LOG-3.

## Scope

- **Applies To**: API, Server-Side Application, Web Client
- **Components**: REST API Application; Browser Client presentation
- **Interfaces / Operations**: Every denial path across all protected operations
- **Actors**: All authenticated roles; anonymous clients
- **Preconditions**: An authorization decision has been evaluated and returned deny
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data — MUST NOT appear in denial responses or logs
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED (`SECURITY.md` SQ-1)

## Security Context

- **Security Objectives**: Confidentiality, Accountability, Privacy
- **Control Layers**: Authorization, Logging and Monitoring
- **Threat References**: `SECURITY.md` TM-I-1, TM-I-4, TM-I-5, TM-E-2; CWE-209 (information exposure through an error message), CWE-778 (insufficient logging)
- **Abuse / Misuse Case**: An attacker distinguishes "exists but forbidden" from "does not exist" to enumerate records or accounts; or mounts a sustained cross-subject probing campaign that leaves no detectable trace.
- **Trust Boundary**: Boundary 1 — Browser Client → REST API Application.
- **Untrusted Inputs or Assertions**: Probing requests designed to elicit differential responses.
- **Authoritative Enforcement Point**: The centralized error handler and the security logging component of the REST API Application.
- **Independent Verification**: The response shape is produced centrally, not by the denying handler.
- **Zero Trust Relevance**: N/A — this governs the reporting of a decision, not the decision.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: N/A
- **Regulatory**: N/A
- **Other**: `REF-LOG`, `REF-ERROR` as cited by SEC-LOG-4 and SEC-ERR-1.
- **Mapping Basis**: The cited rules name these references; the CWE identifiers name the disclosure and insufficient-logging classes.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given any authorization denial, when the response is produced, then it carries a generic denial message and a correlation identifier, and a structured log event records the account identifier, operation, target class, decision reason class, and the same correlation identifier.
2. **AC-02 — Boundary or failure behavior**: Given a request for a record that does not exist and a request for a record owned by another subject, when both are denied, then the two responses are indistinguishable in status, body, and observable timing characteristics.
3. **AC-03 — Prohibited behavior**: Given any denial log or response, when it is inspected, then it MUST NOT contain a health value, credential, token, full personal record, target record content, or internal identifier beyond the correlation identifier.
4. **AC-04 — Additional criterion**: Given a denial surfaced in the client, when it is presented, then it appears as text with an icon rather than color alone and is announced to assistive technology (`DESIGN.md`, Components; SEC-RENDER-4).

## Failure Behavior

- **On Invalid Input**: N/A — invalid input is rejected earlier with a validation response.
- **On Authentication Failure**: A uniform unauthenticated response consistent with SEC-AUTHN-3; no distinction by factor or account existence.
- **On Authorization Failure**: This issue defines that behavior — uniform denial, no existence disclosure, structured log event.
- **On Security-Decision Failure**: An error inside policy evaluation is itself a denial and is logged as such; it MUST NOT be reported to the client as a system error that reveals the policy path.
- **On External Dependency Failure**: If the logging sink is unavailable, the denial still occurs; the logging failure is recorded through the local fallback path and MUST NOT convert the denial into an allow.
- **On System Error**: Generic error, correlation identifier, diagnostics server-side only.
- **Logging / Audit**: Events: authorization denial, authentication failure, security-relevant account change (SEC-LOG-4). Fields: timestamp, account identifier, role, operation, target class, reason class, correlation identifier. Redaction per SEC-LOG-3.
- **Alerting**: TO BE DECIDED — `SECURITY.md` SQ-3 leaves abuse thresholds open, so no alert condition can be specified without inventing one.

## Test Strategy

- **Unit Tests**: Denial formatter produces the uniform shape for each decision reason class; log record builder redacts prohibited fields.
- **Integration Tests**: Denials from each enforcement point (unauthenticated, cross-owner, wrong role, ended engagement) produce the same client-visible shape and a distinct structured log event.
- **Security Tests**: Differential response test between "absent" and "forbidden"; log-content assertion over a corpus of denials confirming no health value, token, or credential appears; a test that policy-evaluation exceptions deny rather than allow.
- **Compliance Tests / Evidence**: Retained sample of denial log records demonstrating the field set, for accountability evidence.
- **Acceptance-Criteria Traceability**: AC-01 — denial formatting and logging suite; AC-02 — differential response test; AC-03 — log and response content assertions; AC-04 — client presentation test.
- **Coverage Target**: Every denial reason class exercised, positive and negative.
- **Required Test Environment**: Identities for each role, log capture, and a second subscriber for cross-owner probes. Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-AUTHZ-010, REQ-API-040
- **Downstream Requirements**: REQ-AUTHZ-020, REQ-AUTHZ-030, REQ-CONSULT-010, REQ-CONSULT-020, REQ-AUDIT-040
- **External Dependencies**: None — logs MUST NOT be shipped outside the system boundary while they may reference health-data subjects (SEC-TB-3).
- **Dependency Assumptions**: A local structured logging sink exists within the system boundary.
- **Failure Impact**: Without uniform denials, ownership scoping still holds but becomes an enumeration oracle; without denial logging, sustained attacks are undetectable.

## Implementation Notes

- **Constraints**: Node.js runtime with Fastify and its pino logger (`CLAUDE.md`); the log sink and retention remain TO BE DECIDED (`SECURITY.md` SQ-7, SQ-8).
- **Prohibited Approaches**: Distinct status codes or messages for "not found" versus "forbidden" on owned records; echoing the requested identifier's associated content; logging the presented token or credential; shipping logs to an external analytics or monitoring destination (SEC-TB-3).
- **Implementation Guidance**: Timing uniformity need only be addressed to the extent it is observable at the response level; `SECURITY.md` SEC-AUTHN-3 already requires timing-insensitive authentication responses, and the same helper can serve both.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the denial shape and the log field set; privacy review that log fields carry no health data.
- **Open Decisions**: Log retention, storage location, and access control are blocked by `SECURITY.md` SQ-8; alerting is blocked by SQ-3.

**Estimated effort**: 0.5–1 engineer-day. **Estimated changed lines**: 150–350.
**Recommended model**: Claude Opus (`claude-opus-5`) — small, security-sensitive, and dependent on getting disclosure boundaries exactly right.
