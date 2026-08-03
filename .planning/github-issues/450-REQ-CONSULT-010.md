# [REQ-CONSULT-010] Engagement-scoped consultant access

## Metadata

- **ID**: REQ-CONSULT-010
- **Title**: Engagement-scoped consultant access
- **Version**: 1.1.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-11.2, FR-11.4; `SECURITY.md` SEC-AUTHZ-3

## Requirement

- **Statement**: A `consultant` MUST NOT be granted access to a subscriber's plans or health data unless an active engagement exists between that consultant and that subscriber, and every consultant access to a subscriber's health data MUST produce an audit entry.
- **Rationale**: FR-11.2 states the access condition; FR-11.4 requires the audit entry; SEC-AUTHZ-3 restates both as a server-side rule with tests for the no-engagement, active-engagement, and ended-engagement cases. The threat model treats a malicious or compromised consultant as a primary adversary.
- **Assumptions**: An engagement record exists linking a consultant to a subscriber with a state. Engagement creation is an `admin` action naming the consultant and the subscriber (FR-11.5; `REQUIREMENTS.md` OQ-13 and OQ-1 RESOLVED), delivered by REQ-CONSULT-030; payment stays out of band in v1 (OQ-18 deferred). This issue enforces the access rule against an engagement that exists.
- **Out of Scope**: Creating and administratively revoking an engagement (FR-11.1, FR-11.5) — REQ-CONSULT-030; what a consultant may *do* within scope, which FR-11.6 fixes as views plus plan-copy edits (`REQUIREMENTS.md` OQ-12 RESOLVED; threat TM-E-3 MITIGATED BY RULE) — REQ-CONSULT-040; ending an engagement (REQ-CONSULT-020); consultant vetting records and deprovisioning (FR-2.16, FR-2.17; `SECURITY.md` SQ-12 RESOLVED) — REQ-AUTH-160 and REQ-AUTH-170.
- **Design Traceability**: `DESIGN.md` OQ-7 RESOLVED — Information Architecture and Navigation: subscribers see consultant access in a Home summary and in Account → Consultant access, both naming the consultant and stating the exact access granted by FR-11.6; consultant pages carry a persistent selected-client context bar stating the scope ("View progress and selected plans; edit plan copies"), and no client data remains visible after an engagement ends. The audit trail from AC-01 is the record those views draw on.
- **Architecture Traceability**: `ARCHITECTURE.md` — data flow 7 ("Consultant client → REST API → active-engagement check → scoped subscriber data → audit entry written"); REST API Application ("consultant engagement scoping (FR-11.2, FR-11.3) before any data access"); DR-4.
- **Security Traceability**: SEC-AUTHZ-3, SEC-AUTHZ-1, SEC-AUTHZ-2, SEC-LOG-1, SEC-DATA-5, SEC-SESSION-4.

## Scope

- **Applies To**: Server-Side Application, API
- **Components**: REST API Application; Relational Persistence
- **Interfaces / Operations**: Every read or modification of a subscriber's plans, plan copies, and log entries performed by a `consultant`
- **Actors**: `consultant`; `subscriber` as the data subject
- **Preconditions**: Authenticated `consultant` session established by passkey (REQ-AUTH-020)
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED, `REQUIREMENTS.md` OQ-3 RESOLVED)

## Security Context

- **Security Objectives**: Confidentiality, Privacy, Authorization, Accountability
- **Control Layers**: Authorization, Logging and Monitoring, Data Protection
- **Threat References**: `SECURITY.md` TM-I-2 (consultant retains access after engagement ends), TM-E-3 (consultant capability creep), TM-I-1 (BOLA), TM-I-3 (bulk retrieval); CWE-862 (missing authorization), CWE-639 (authorization bypass through user-controlled key), CWE-284 (improper access control)
- **Abuse / Misuse Case**: A consultant enumerates subscriber identifiers and reads the health data of subscribers who never engaged them; or reads an engaged subscriber's data through a path that checks the role but not the engagement, leaving no audit trail.
- **Trust Boundary**: Boundary 1 — the subscriber identifier the consultant names is untrusted input.
- **Untrusted Inputs or Assertions**: The target subscriber identifier; any engagement identifier or state in the request.
- **Authoritative Enforcement Point**: REST API Application — the active-engagement predicate is evaluated from persisted state before any data access, and applied in the query alongside owner scoping.
- **Independent Verification**: The engagement is read from persistence on every request rather than cached in the session or token (SEC-SESSION-4, SEC-SESSION-6).
- **Zero Trust Relevance**: NIST SP 800-207 — access is granted per request based on current relationship state, not on a standing grant. Exact tenet: TO BE DECIDED (not verified against the publication in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — per-issue mappings are verified during the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — per-issue mappings are verified during the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: Third-party access to health data is a regulated disclosure under the `SECURITY.md` SQ-1 regime set — GDPR/UK GDPR for EU/UK data subjects; CCPA/CPRA, Washington My Health My Data, and the FTC Health Breach Notification Rule for US users; HIPAA not applicable. Statute-section precision: TO BE DECIDED per-issue (SQ-1 counsel review, SQ-10).
- **Other**: `REF-PROMPT-ABAC`, `REF-ASVS-5` as cited by SEC-AUTHZ-3.
- **Mapping Basis**: FR-11.2, FR-11.4, and SEC-AUTHZ-3 are the normative sources and name these references; the CWE identifiers name the missing-authorization classes.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a `consultant` with an active engagement with subscriber S, when they read S's plans, plan copies, or log entries, then the data is returned scoped to S alone and an audit entry records the consultant as the acting account and S as the affected subject (FR-11.4).
2. **AC-02 — Boundary or failure behavior**: Given a `consultant` with no engagement with subscriber S, when they request S's data by any identifier, then the request is denied, nothing belonging to S is returned, and the response does not disclose whether S or the requested record exists.
3. **AC-03 — Prohibited behavior**: Given any consultant request, when it is processed, then engagement state MUST NOT be taken from the request or from a session claim, a consultant MUST NOT be able to enumerate or bulk-retrieve data across subscribers, and no consultant access to health data may complete without an audit entry.
4. **AC-04 — Additional criterion**: Given a consultant with engagements with several subscribers, when they list accessible subscriber data, then only subjects with an active engagement appear, and each subject's data remains individually scoped.
5. **AC-05 — Additional criterion**: Given a consultant reading a subscriber's data, when the audit entry is written, then it is distinguishable from the subscriber's own access to the same record.

## Failure Behavior

- **On Invalid Input**: Reject malformed subscriber or record identifiers per REQ-API-010.
- **On Authentication Failure**: Denied upstream; consultants authenticate by passkey (REQ-AUTH-020).
- **On Authorization Failure**: Deny with a response indistinguishable from "does not exist" (REQ-AUTHZ-040); the existence of a subscriber MUST NOT be confirmed to an unengaged consultant.
- **On Security-Decision Failure**: If engagement state cannot be resolved, deny (SEC-AUTHZ-7).
- **On External Dependency Failure**: If persistence or audit storage is unavailable, deny rather than return unaudited data (REQ-AUDIT-020).
- **On System Error**: Generic error with a correlation identifier; no partial data.
- **Logging / Audit**: Audit entry for every consultant access to health data (FR-11.4, SEC-LOG-1). Denials logged as security events (REQ-AUTHZ-040). No health values in logs (SEC-LOG-3).
- **Alerting**: Authorization denials and consultant health-data access entries (SEC-LOG-4, SEC-LOG-1) are detection inputs; threshold alerts route to the security lead under SEC-OPS-2 (`SECURITY.md` SQ-3 and SQ-11 RESOLVED). A consultant accessing an unusual number of subjects is a natural signal within that channel.

## Test Strategy

- **Unit Tests**: The engagement resolver returns active only for a persisted active engagement, denies for none, ended, and unresolvable; the scope builder combines engagement scope with owner scope in the query.
- **Integration Tests**: Consultant with an active engagement reads each data type for the engaged subscriber; audit entries asserted and distinguishable from the subscriber's own reads.
- **Security Tests**: The three cases SEC-AUTHZ-3 names — no engagement, active engagement, ended engagement — asserting denial in the first and third; cross-subscriber enumeration attempts; engagement state injected in the request body and headers; bulk-listing attempts across subjects; audit-failure fault injection.
- **Compliance Tests / Evidence**: Consultant-access audit entries, retained as evidence for FR-11.4.
- **Acceptance-Criteria Traceability**: AC-01 — engaged-access suite; AC-02 — unengaged denial suite; AC-03 — injection, enumeration, and audit-dependency tests; AC-04 — multi-engagement scoping test; AC-05 — audit distinguishability assertion.
- **Coverage Target**: Every consultant-reachable operation × the three engagement states.
- **Required Test Environment**: One consultant, two subscribers (one engaged, one not), a third with an ended engagement, seeded health data and copies, audit capture. Runs against PostgreSQL on Vitest.

## Dependencies

- **Upstream Requirements**: REQ-AUTH-010, REQ-AUTH-020, REQ-AUTHZ-010, REQ-AUTHZ-020, REQ-AUDIT-020, REQ-PRIVACY-060, REQ-CONSULT-030
- **Downstream Requirements**: REQ-CONSULT-020, REQ-CONSULT-040
- **External Dependencies**: None
- **Dependency Assumptions**: An engagement record with a resolvable active state exists in persistence, created by the `admin` action FR-11.5 defines (REQ-CONSULT-030).
- **Failure Impact**: An unengaged consultant reading subscriber health data is a third-party disclosure — the failure mode FR-11.2 exists to prevent, and a regulated disclosure under the `SECURITY.md` SQ-1 regime set (GDPR/UK GDPR; CCPA/CPRA, Washington My Health My Data, FTC HBNR).

## Implementation Notes

- **Constraints**: PostgreSQL with Drizzle ORM (`CLAUDE.md`). This issue bounds *who* a consultant may reach; *what* they may do inside scope is fixed by FR-11.6 (`REQUIREMENTS.md` OQ-12 RESOLVED) — views of the engaged subscriber's selected plans, customized plan copies, and progress logs, plus edits of plan copies only, never log-entry writes — and is delivered by REQ-CONSULT-040. `SECURITY.md` records threat TM-E-3 as MITIGATED BY RULE on that basis. Implement read access consistent with the subscriber's own read paths.
- **Prohibited Approaches**: Caching engagement state in the session or token, which would defeat SEC-SESSION-4 and REQ-CONSULT-020; a role check without an engagement check; a consultant-wide listing endpoint that returns data across subjects; treating an ended engagement as active until token expiry.
- **Implementation Guidance**: Compose the engagement predicate with the owner predicate from REQ-AUTHZ-020 in a single scope resolver, so that a consultant's scope is a strict subset of the subject's own scope and cannot exceed it.
- **AI Development Guidance**: `REF-PROMPT-ABAC`, `REF-PROMPT-API`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the engagement predicate; privacy review of what a consultant sees.
- **Open Decisions**: None affecting this issue's behavior — `REQUIREMENTS.md` OQ-12, OQ-13, and OQ-1 and `SECURITY.md` SQ-4 and SQ-12 are RESOLVED; engagement creation is planned as REQ-CONSULT-030 and in-scope capabilities as REQ-CONSULT-040. Self-serve payments stay deferred (`REQUIREMENTS.md` OQ-18). Per-issue standards mappings remain TO BE DECIDED until the SQ-10 pre-launch assessment.

**Estimated effort**: 1–2 engineer-days. **Estimated changed lines**: 300–650.
**Recommended model**: Claude Opus (`claude-opus-5`) — third-party access to health data, where the correct scope is a composition of two predicates and the failure is a regulated disclosure.
