# [REQ-CONSULT-020] Ending an engagement revokes consultant access

## Metadata

- **ID**: REQ-CONSULT-020
- **Title**: Ending an engagement revokes consultant access
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-11.3; `SECURITY.md` SEC-AUTHZ-3, SEC-SESSION-4; threat TM-I-2

## Requirement

- **Statement**: A subscriber MUST be able to end a consultant engagement, and that consultant's access to the subscriber's data MUST be revoked immediately — including for a session or token issued before the engagement ended, without waiting for natural expiry.
- **Rationale**: FR-11.3 grants the subscriber the right to end an engagement and requires revocation; SEC-AUTHZ-3 requires denial immediately once the subscriber ends it; SEC-SESSION-4 requires revocation of authorization state to take effect without waiting for token expiry. The threat model records a consultant retaining access via a still-valid JWT (TM-I-2) as high severity, mitigated by the resolved server-side session design (SQ-2; SEC-AUTHZ-3, SEC-SESSION-4).
- **Assumptions**: An engagement exists and access is evaluated per request from persisted state (REQ-CONSULT-010).
- **Out of Scope**: Creating or purchasing an engagement (FR-11.1), blocked by `REQUIREMENTS.md` OQ-13 and OQ-1; whether a consultant may also end an engagement, which no source document states — FR-11.3 grants it to the subscriber; the JWT revocation mechanism for logout generally, resolved by `SECURITY.md` SQ-2 and delivered by REQ-SESSION-040 (SEC-SESSION-3); consultant capabilities (OQ-12).
- **Design Traceability**: `DESIGN.md` — Components → Buttons ("Destructive actions use `error` as the filled color and require explicit confirmation"), Form feedback and errors, Focus states; `DESIGN.md` OQ-7 (how a subscriber sees consultant access) is open.
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application ("consultant engagement scoping (FR-11.2, FR-11.3)"); data flow 7; Identity and Session Handling (revocation of authorization state).
- **Security Traceability**: SEC-AUTHZ-3, SEC-SESSION-4, SEC-LOG-1, SEC-LOG-4.

## Scope

- **Applies To**: Server-Side Application, API, Web Client
- **Components**: REST API Application; Relational Persistence; Browser Client (engagement management view)
- **Interfaces / Operations**: End an engagement; the effect of that state change on every consultant access path
- **Actors**: `subscriber` as the actor ending the engagement; `consultant` as the actor whose access is revoked
- **Preconditions**: Authenticated subscriber session; an active engagement between that subscriber and the consultant
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data — the data whose access is being revoked
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED (`SECURITY.md` SQ-1, `REQUIREMENTS.md` OQ-3)

## Security Context

- **Security Objectives**: Confidentiality, Privacy, Authorization, Accountability
- **Control Layers**: Authorization, Session Management, Logging and Monitoring
- **Threat References**: `SECURITY.md` TM-I-2 (consultant retains access after engagement ends via a still-valid JWT), TM-S-5 (token replay), TM-E-3; CWE-613 (insufficient session expiration), CWE-284 (improper access control)
- **Abuse / Misuse Case**: A subscriber ends an engagement after a dispute, but the consultant's existing token continues to grant access to weight, measurement, and workout history until it expires — the exact scenario SEC-SESSION-4's verification test names.
- **Trust Boundary**: Boundary 1 for the request; boundary 2 for the session whose authority must not outlive the engagement.
- **Untrusted Inputs or Assertions**: The engagement identifier; any engagement state a session or token asserts.
- **Authoritative Enforcement Point**: REST API Application — engagement state is read from persistence on every consultant request, so ending it takes effect on the next request.
- **Independent Verification**: A test captures a consultant token, ends the engagement, and asserts the captured token no longer grants access — SEC-SESSION-4's stated verification.
- **Zero Trust Relevance**: NIST SP 800-207 — authorization is re-evaluated per request from current state rather than granted for a session's lifetime. Exact tenet: TO BE DECIDED (not verified in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: TO BE DECIDED — blocked by `SECURITY.md` SQ-1; withdrawal of third-party access is a data-subject control under the regimes `REQUIREMENTS.md` names.
- **Other**: `REF-PROMPT-JWT`, `REF-PROMPT-ABAC` as cited by SEC-SESSION-4 and SEC-AUTHZ-3.
- **Mapping Basis**: FR-11.3, SEC-AUTHZ-3, and SEC-SESSION-4 are the normative sources and name these references; CWE-613 names the stale-authority class.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a subscriber with an active consultant engagement, when they end it, then the engagement state becomes ended with the time recorded, and the action is logged.
2. **AC-02 — Boundary or failure behavior**: Given a consultant holding a session or token issued while the engagement was active, when they request that subscriber's data after the engagement ends, then the request is denied on the next request without waiting for token expiry, and no data is returned.
3. **AC-03 — Prohibited behavior**: Given the implementation, when engagement state is evaluated, then it MUST NOT be read from a token claim, a session cache, or a client assertion; and a consultant MUST NOT be able to end or reinstate an engagement on the subscriber's behalf.
4. **AC-04 — Additional criterion**: Given an ended engagement, when the consultant requests the subscriber's data, then the response is indistinguishable from that for a subscriber they never had an engagement with (REQ-CONSULT-010 AC-02).
5. **AC-05 — Additional criterion**: Given the subscriber's engagement management view, when ending an engagement is invoked, then it is presented as a destructive action requiring explicit confirmation (`DESIGN.md`, Components → Buttons).

## Failure Behavior

- **On Invalid Input**: Reject a malformed engagement identifier per REQ-API-010; state unchanged.
- **On Authentication Failure**: Denied upstream by REQ-AUTHZ-010.
- **On Authorization Failure**: A request to end an engagement the actor is not party to as the subscriber is denied without disclosing the engagement's existence.
- **On Security-Decision Failure**: If engagement state cannot be resolved, deny consultant access (fail closed) — an unresolvable engagement is not treated as active.
- **On External Dependency Failure**: If persistence is unavailable, the end-engagement operation fails atomically; consultant access evaluation, unable to confirm an active engagement, denies.
- **On System Error**: Roll back; generic error with a correlation identifier.
- **Logging / Audit**: Log the engagement termination as a security-relevant authorization change (SEC-LOG-4) and audit it as an action affecting the subscriber's data access (REQ-AUDIT-020). Post-termination consultant attempts are logged as denials (REQ-AUTHZ-040).
- **Alerting**: TO BE DECIDED — repeated post-termination access attempts by a consultant would be a natural signal; thresholds are blocked by `SECURITY.md` SQ-3.

## Test Strategy

- **Unit Tests**: The engagement resolver returns denied for an ended engagement; the termination handler rejects an actor who is not the subscriber party.
- **Integration Tests**: End an engagement and assert the state change and log record; assert the consultant's subsequent requests across every data type are denied.
- **Security Tests**: SEC-SESSION-4's stated test — capture a consultant token while engaged, end the engagement, assert the captured token no longer grants access to that subscriber's data; attempt termination by the consultant; assert engagement state is not read from any token claim; response-equivalence between "ended engagement" and "never engaged".
- **Compliance Tests / Evidence**: Records of engagement termination and subsequent denials, retained as evidence for FR-11.3.
- **Acceptance-Criteria Traceability**: AC-01 — termination suite; AC-02 — captured-token revocation test; AC-03 — token-claim and actor-authorization negatives; AC-04 — response-equivalence test; AC-05 — confirmation flow test.
- **Coverage Target**: Every consultant-reachable operation asserted denied after termination.
- **Required Test Environment**: One consultant, one subscriber with an active engagement, a captured pre-termination token, seeded health data. Runs against PostgreSQL on Vitest.

## Dependencies

- **Upstream Requirements**: REQ-CONSULT-010, REQ-AUTHZ-020, REQ-AUTHZ-040, REQ-AUDIT-020, REQ-SESSION-020, REQ-PLATFORM-030
- **Downstream Requirements**: None
- **External Dependencies**: None
- **Dependency Assumptions**: Engagement state is evaluated per request from persistence (REQ-CONSULT-010), which is what makes immediate revocation achievable without a token revocation mechanism.
- **Failure Impact**: Access surviving termination is a continuing unauthorized third-party disclosure of health data, initiated by a control the subscriber was told would stop it.

## Implementation Notes

- **Constraints**: PostgreSQL with Drizzle ORM (`CLAUDE.md`). `SECURITY.md` SQ-2 is RESOLVED: sessions are server-side records revoked by invalidating the record (SEC-SESSION-3), and TM-I-2 is MITIGATED BY RULE (SEC-AUTHZ-3, SEC-SESSION-4). This issue additionally never caches engagement state in the session, so revocation of engagement scope never waits on token expiry; logout revocation itself is delivered by REQ-SESSION-040.
- **Prohibited Approaches**: Placing engagement state in a JWT claim; caching it for the session's lifetime; deferring revocation to token expiry; letting a consultant terminate or reinstate an engagement; soft-hiding the subscriber's data in the client while the API still serves it.
- **Implementation Guidance**: Because REQ-CONSULT-010 evaluates the engagement predicate per request, this issue's revocation is a state change plus a test that proves the predicate is genuinely per-request. Keep the ended state rather than deleting the engagement record, so the historical audit trail remains interpretable.
- **AI Development Guidance**: `REF-PROMPT-JWT`, `REF-PROMPT-ABAC`, `REF-PROMPT-API`; `CLAUDE.md`.
- **Required Human Review**: Security review of the revocation path and the captured-token test; privacy review of what the subscriber is told about past access.
- **Open Decisions**: Session lifetimes remain open (`SECURITY.md` SQ-3; SQ-2 is RESOLVED); `REQUIREMENTS.md` OQ-12 (capabilities) and OQ-13 (engagement lifecycle and payment) remain open, so engagement *creation* has no issue.

**Estimated effort**: 0.5–1.5 engineer-days. **Estimated changed lines**: 150–400.
**Recommended model**: Claude Opus (`claude-opus-5`) — small in code, but it is the mitigation for a threat the specification rates as high severity (TM-I-2).
