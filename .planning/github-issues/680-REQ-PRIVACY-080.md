# [REQ-PRIVACY-080] Synchronous JSON data export

## Metadata

- **ID**: REQ-PRIVACY-080
- **Title**: Synchronous JSON data export
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Privacy
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-9.3; `SECURITY.md` SEC-DATA-3, SEC-AUTHN-7, SEC-HTTP-5 (export limit), SQ-5 RESOLVED

## Requirement

- **Statement**: The system MUST deliver to an authenticated user, on request and after fresh re-authentication no older than 5 minutes, all of that user's own account, plan, and progress data as a single synchronous JSON download over the authenticated session — storing no export artifact, returning no other actor's records, rate-limited to one export per day per account, and producing exactly one audit entry for the whole operation.
- **Rationale**: FR-9.3 grants the export right in machine-readable form (a GDPR-grade data-subject right under SQ-1). SEC-DATA-3 makes export a sensitive operation requiring SEC-AUTHN-7 fresh re-authentication because export is a one-request bulk read of everything the account holds — the highest-value single response in the system (threat TM-I-7). Synchronous delivery with no stored artifact (SQ-5 RESOLVED) keeps SEC-DATA-6 moot: no second health-data store exists to protect.
- **Assumptions**: Server-side session records (REQ-SESSION-030) make re-authentication freshness verifiable; the health-data classification (REQ-PRIVACY-070) determines which reads the export's audit entry must reference.
- **Out of Scope**: Account deletion (REQ-PRIVACY-090) — `DESIGN.md` keeps export and deletion separate; viewing and correcting personal data (REQ-PRIVACY-030); the out-of-band deletion channel, which never exports (FR-9.11; REQ-PRIVACY-110); the rate-limiting infrastructure itself (REQ-API-050); any stored or asynchronous export artifact (SEC-DATA-6 — moot, retained for a future decision only).
- **Design Traceability**: `DESIGN.md` — Account, privacy, and destructive actions ("Export is described as a JSON download and remains separate from deletion"); Credentials, account security, and administration → Re-authentication (the prompt names the operation — "Confirm it's you to export your data" — honors the 5-minute freshness window, and returns focus to the interrupted operation).
- **Architecture Traceability**: `ARCHITECTURE.md` — data flow 8 (Client → REST API → owner scoping and sensitive-operation authorization with fresh re-authentication → export streamed synchronously → audit entry written); "No background-processing boundary exists for user-facing operations"; REST API Application; DR-9 (the audit write is a dependency of this health-data access path).
- **Security Traceability**: SEC-DATA-3, SEC-AUTHN-7, SEC-HTTP-5 (export once per day per account), SEC-LOG-1, SEC-LOG-3, SEC-AUTHZ-2; SEC-DATA-6 moot by construction.

## Scope

- **Applies To**: Multiple — API, Server-Side Application, Web Client (download presentation)
- **Components**: REST API Application; Identity and Session Handling (re-authentication freshness); Relational Persistence; Browser Client
- **Interfaces / Operations**: The export operation reached from Account → Privacy; the re-authentication step that precedes it
- **Actors**: Any authenticated account exporting its own data — subscriber, consultant, or admin (FR-9.3 says "a user")
- **Preconditions**: Authenticated session; fresh re-authentication no older than 5 minutes at execution (SEC-AUTHN-7)
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED)

## Security Context

- **Security Objectives**: Confidentiality, Privacy, Accountability, Authorization
- **Control Layers**: Authorization, Authentication, Session Management, Data Protection, Logging and Monitoring, Availability
- **Threat References**: `SECURITY.md` TM-I-7 (export as an exfiltration channel), TM-I-1 (BOLA — export of another subscriber's records), TM-S-5 (a stolen session monetized through export), TM-D-1 (export as an expensive endpoint); CWE-639 (authorization bypass through user-controlled key)
- **Abuse / Misuse Case**: An attacker holding a stolen but idle session triggers export to exfiltrate the account's entire history in one response; a malicious subscriber, consultant, or admin manipulates the export scope to receive another subscriber's records; an attacker loops the endpoint as a resource-exhaustion or bulk-exfiltration vector.
- **Trust Boundary**: Boundary 1 (Browser Client → REST API Application) for the request; boundary 2 for the re-authentication assertion, which is verified server-side and never taken from a client claim.
- **Untrusted Inputs or Assertions**: Any request parameter purporting to select the export subject; the claim that re-authentication is fresh — both resolved server-side (DR-3, SEC-INPUT-3).
- **Authoritative Enforcement Point**: REST API Application — export scope is bound to the authenticated account identity from Identity and Session Handling; freshness is verified against the server-side session record before any data is read.
- **Independent Verification**: Queries are constrained by the authenticated owner identity (SEC-AUTHZ-2); the response is assembled only from records whose owner matches — never filtered after retrieval.
- **Zero Trust Relevance**: NIST SP 800-207 — a security-critical bulk access is re-authorized with fresh authentication rather than inherited from an existing session. Exact tenet: TO BE DECIDED (not verified against the publication in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session (SQ-10 pre-launch assessment).
- **OWASP AISVS 1.0**: N/A — no AI component participates in export.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: The SQ-1 regime set — GDPR/UK GDPR (data-portability and access rights motivate FR-9.3), CCPA/CPRA, Washington My Health My Data, FTC HBNR; HIPAA not applicable. Statute-section mappings: TO BE DECIDED (SQ-1 counsel review).
- **Other**: `REF-API-2023`, `REF-ASVS-5` as cited by SEC-DATA-3; `REF-AUTH`, `REF-63B` as cited by SEC-AUTHN-7.
- **Mapping Basis**: SEC-DATA-3 and SEC-AUTHN-7 name these references; CWE-639 describes the scope-manipulation class the ownership binding defeats.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated user whose re-authentication is no older than 5 minutes, when they request export, then the response is a single synchronous JSON download containing all of their own account, plan-copy, selection, and progress-log data, and exactly one audit entry is recorded for the whole operation — acting account, action, affected subject (self), time, and a reference to the export scope (FR-9.7).
2. **AC-02 — Boundary or failure behavior**: Given a user whose re-authentication is absent or older than the 5-minute named constant, when export is requested, then it is refused before any data is read, the refusal is logged, and no partial response or artifact exists; and given a second export request within the same day for the same account, when it arrives, then it is refused under the one-per-day limit (SEC-HTTP-5) with a non-descriptive error.
3. **AC-03 — Prohibited behavior**: Given any actor and any request shape, when export executes, then the response MUST NOT contain another subscriber's records — a consultant with an active engagement and an admin export only their own account data, and a request-supplied subject identifier is ignored (SEC-INPUT-3); no export artifact is stored server-side; and export MUST NOT be reachable through the FR-9.11 out-of-band channel or any path without full authentication.
4. **AC-04 — Additional criterion**: Given an account whose consent is withdrawn, when it requests export with fresh re-authentication, then the export succeeds — FR-9.9 keeps existing records subject to FR-9.3 while blocking only new collection.

## Failure Behavior

- **On Invalid Input**: The export operation binds no client-assignable scope fields; unexpected parameters are rejected or ignored per SEC-INPUT-1 and SEC-INPUT-3 with no data read.
- **On Authentication Failure**: If re-authentication fails or is stale, refuse with a uniform response that does not reveal which factor failed (SEC-AUTHN-3); log the refusal (SEC-LOG-4).
- **On Authorization Failure**: Deny; export is self-scoped by construction, so an attempt to address another subject is denied without disclosing whether that subject exists (REQ-AUTHZ-040 semantics).
- **On Security-Decision Failure**: Deny by default — if freshness, ownership, or the audit write cannot be established, the export does not execute (SEC-AUTHZ-7 discipline; DR-9).
- **On External Dependency Failure**: If Relational Persistence is unavailable mid-assembly, abort with a generic error and correlation identifier; no partial download is presented as complete and no artifact remains.
- **On System Error**: Abort the response, return a generic error with a correlation identifier (SEC-ERR-1), leave no stored artifact, and record the failure server-side.
- **Logging / Audit**: Exactly one audit entry per successful export (FR-9.7); refusals (stale re-auth, rate limit, authorization) logged with reason class and correlation identifier (SEC-LOG-4). Logs MUST NOT contain exported values, tokens, or credentials (SEC-LOG-3).
- **Alerting**: Repeated export attempts hitting the once-per-day limit or repeated stale-re-auth refusals are threshold alerts routed to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-3, SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: Freshness check refuses re-authentication older than 5 minutes, including exactly at the boundary; scope assembly includes only records owned by the authenticated account; a request-supplied subject identifier is ignored; the audit writer is invoked exactly once per export.
- **Integration Tests**: Full export fixture asserting the JSON contains every owned record type (account, plan copies, selections, workout/food/weight/measurement entries, consent state) and nothing owned by another account; second same-day request refused; withdrawn-consent account exports successfully; response delivered synchronously over the session with no artifact persisted.
- **Security Tests**: BOLA suite — subscriber A, an engaged consultant, and an admin each attempt to export subscriber B's data and are denied with no information disclosure (TM-I-1, SEC-DATA-3); stolen-session scenario — a valid session without fresh re-authentication cannot export (TM-S-5); storage inspection after export and after aborted export asserting no artifact (TM-I-7); burst test on the export endpoint (SEC-HTTP-5).
- **Compliance Tests / Evidence**: The export-completeness fixture doubles as evidence for the FR-9.3 data-subject right at the SQ-1 counsel review; audit-entry evidence per export.
- **Acceptance-Criteria Traceability**: AC-01 — full-export integration suite and audit unit test; AC-02 — freshness-boundary and rate-limit tests; AC-03 — BOLA suite, artifact-absence inspection, and unauthenticated-path test; AC-04 — withdrawn-consent export test.
- **Coverage Target**: Project-defined; all authorization, freshness, and audit decision paths covered positive and negative.
- **Required Test Environment**: Vitest and HTTP test client; fixtures for two subscribers with full data sets, an engaged consultant, and an admin; session-record fixture with controllable re-authentication timestamps; storage-inspection hook.

## Dependencies

- **Upstream Requirements**: REQ-AUTHZ-020, REQ-SESSION-030, REQ-AUDIT-020, REQ-API-050, REQ-PRIVACY-070
- **Downstream Requirements**: None — REQ-PRIVACY-090 (deletion) is deliberately independent; `DESIGN.md` separates the flows.
- **External Dependencies**: None
- **Dependency Assumptions**: Session records expose a verifiable re-authentication timestamp; the rate limiter enforces the once-per-day export limit as a named constant (SQ-3).
- **Failure Impact**: A scope or freshness failure turns the single richest endpoint in the system into a bulk health-data exfiltration channel (TM-I-7); a missing audit entry makes that exfiltration unprovable.

## Implementation Notes

- **Constraints**: Node.js runtime with Fastify (`CLAUDE.md`); the 5-minute freshness window and the once-per-day limit are named constants (SEC-AUTHN-7, SQ-3). Delivery is synchronous over the authenticated session — no queue, no stored artifact, no background-processing boundary (SQ-5, ARCHITECTURE.md).
- **Prohibited Approaches**: Storing or caching the export server-side; emailing the export (SEC-EXT-3 carries no health data); accepting a subject identifier from the request; filtering foreign records after retrieval instead of constraining the query (SEC-AUTHZ-2); trusting a client-asserted re-authentication time.
- **Implementation Guidance**: Stream the JSON response using Fastify's serialization against a declared response schema (SEC-DATA-5 delivery mechanism per `CLAUDE.md`), assembling per-entity sections with owner-constrained queries; write the single audit entry within the same request so a failed audit write fails the export (DR-9).
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-NODE`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the scope-binding and freshness checks; privacy review that the export content matches the FR-9.3 right ahead of the SQ-1 counsel review.
- **Open Decisions**: None — synchronous delivery, re-authentication, the audit granularity, and the rate limit are all fixed (SQ-5, SEC-AUTHN-7, FR-9.7, SQ-3).

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 300–600.
**Recommended model**: Claude Opus (`claude-opus-5`) — a bulk health-data read where ownership binding, fail-closed freshness, and audit coupling are security-critical.
