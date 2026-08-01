# [REQ-SESSION-030] Server-side session records and per-request resolution

## Metadata

- **ID**: REQ-SESSION-030
- **Title**: Server-side session records and per-request resolution
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-01
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `SECURITY.md` SEC-SESSION-3; SQ-2

## Requirement

- **Statement**: Every authenticated request MUST resolve the session identifier carried in its token against a persisted session record before the request proceeds, and MUST be refused when that record is absent, expired, or invalidated. The token MUST carry the session identifier and no authorization state.
- **Rationale**: SEC-SESSION-3 selects server-side session records as the revocation mechanism, because a self-contained token cannot be withdrawn before it expires. This issue supplies the record and the lookup that every other session behavior depends on: logout (REQ-SESSION-040), engagement termination (REQ-CONSULT-020), and credential-change revocation (SEC-AUTHN-12) are all just invalidations of a record this issue defines. DR-3 already requires role and ownership to be re-read from persisted state on every request, so this adds a lookup to a path that was never stateless.
- **Assumptions**: Token signature, algorithm, and claim verification happen first (REQ-SESSION-010); this issue runs after a token is known to be authentic and before authorization (REQ-AUTHZ-010).
- **Out of Scope**: Token signature and claim verification (REQ-SESSION-010); the claim allow-list (REQ-SESSION-020); logout and the revocation triggers (REQ-SESSION-040); cookie transport and CSRF (REQ-SESSION-050); how a session is first created, which belongs to the authentication issues that issue one; absolute and idle session lifetimes, which are `TO BE DECIDED` under `SECURITY.md` SQ-3; signing key storage (SEC-SESSION-7, blocked by SQ-7).
- **Design Traceability**: N/A — no user-facing surface. A refused session surfaces through the error contract in REQ-API-040.
- **Architecture Traceability**: `ARCHITECTURE.md` — Identity and Session Handling (session state is data it owns); trust boundary 2; data flow 1. The session record is a persisted entity listed under Data model expectations. DR-3: identity and role come from Identity and Session Handling, never from a client assertion.
- **Security Traceability**: SEC-SESSION-3; enables SEC-SESSION-4, SEC-AUTHN-12; supports SEC-AUTHZ-1 by guaranteeing an authenticated subject exists before any authorization decision.

## Scope

- **Applies To**: Server-Side Application, API
- **Components**: Identity and Session Handling; Relational Persistence
- **Interfaces / Operations**: Session record creation, resolution, and invalidation; the request pipeline stage that performs resolution
- **Actors**: All authenticated actors — `subscriber`, `consultant`, `admin`; an attacker replaying a captured or revoked token
- **Preconditions**: A token has been presented and has passed signature and claim verification (REQ-SESSION-010)
- **Data Classification**: Confidential
- **Personal or Regulated Data**: Personal Data — a session record links an account to a device and time
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED (`SECURITY.md` SQ-1)

## Security Context

- **Security Objectives**: Authenticity, Authorization, Accountability
- **Control Layers**: Session Management, Architecture
- **Threat References**: `SECURITY.md` TM-S-5 (theft and replay of a token), TM-I-2 (consultant retains access after engagement ends); CWE-613 (insufficient session expiration), CWE-384 (session fixation)
- **Abuse / Misuse Case**: An attacker holding a token that was valid at capture time continues to use it after logout, after the subscriber changed their password, or after their consultant engagement ended — because nothing between the signature check and the handler ever asked whether the session still exists.
- **Trust Boundary**: Boundary 2 — unauthenticated to authenticated.
- **Untrusted Inputs or Assertions**: The session identifier in the token, until it resolves to a live record. A valid signature proves the token was issued, not that the session it names is still current.
- **Authoritative Enforcement Point**: Identity and Session Handling, in a pipeline stage every protected route traverses — not a per-handler call.
- **Independent Verification**: The record is read from persistence on each request; nothing about the session's continued validity is taken from the token itself.
- **Zero Trust Relevance**: NIST SP 800-207 — per-request access decisions rather than per-session. Exact tenet: TO BE DECIDED (not verified in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: TO BE DECIDED — blocked by `SECURITY.md` SQ-1.
- **Other**: `REF-SESSION` (OWASP Session Management Cheat Sheet) and `REF-PROMPT-JWT`, both named by SEC-SESSION-3.
- **Mapping Basis**: SEC-SESSION-3 names these references directly; the CWE identifiers name the expiration and fixation classes this control addresses.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a token whose session identifier resolves to a live, unexpired record, when any protected route is called, then the request proceeds with the account identity and role read from persisted state.
2. **AC-02 — Boundary or failure behavior**: Given a token that is correctly signed and structurally valid but whose session record is absent, expired, or invalidated, when any protected route is called, then the request is refused with the uniform unauthenticated response (REQ-AUTHZ-040) and no handler executes.
3. **AC-03 — Prohibited behavior**: Given any protected route, when it is invoked, then it MUST NOT be reachable without session resolution having run, and the token payload MUST NOT be the source of role, ownership, entitlement, or session validity.
4. **AC-04 — Additional criterion**: Given a newly issued session, when the record is created, then its identifier comes from a cryptographically secure generator (SEC-SECRET-4) and is not derived from account data, a counter, or a timestamp.
5. **AC-05 — Additional criterion**: Given a route inventory, when it is enumerated in a test, then every protected route passes through the session-resolution stage, so a new route cannot silently omit it.

## Failure Behavior

- **On Invalid Input**: A malformed or absent session identifier is treated as an unauthenticated request; no record lookup result is disclosed.
- **On Authentication Failure**: Deny with the uniform unauthenticated response; the response MUST NOT distinguish "no such session" from "session expired" from "session invalidated" (SEC-AUTHN-3).
- **On Authorization Failure**: N/A — this precedes authorization.
- **On Security-Decision Failure**: If the session store cannot be read, deny. A lookup error MUST NOT be treated as a valid session (SEC-AUTHZ-7 fail-closed reasoning applies).
- **On External Dependency Failure**: If Relational Persistence is unavailable, deny; the system MUST NOT fall back to trusting token contents.
- **On System Error**: Generic error with a correlation identifier (SEC-ERR-1); no session detail in the response.
- **Logging / Audit**: Log session resolution failures with cause class and correlation identifier (SEC-LOG-4). MUST NOT log the session identifier or the token (SEC-LOG-3, SEC-SECRET-1).
- **Alerting**: TO BE DECIDED — thresholds for anomalous resolution-failure rates are blocked by `SECURITY.md` SQ-3.

## Test Strategy

- **Unit Tests**: Resolver returns a live session for a valid identifier; returns denial for absent, expired, and invalidated records; treats a store error as denial rather than success.
- **Integration Tests**: Full request against a protected route with each record state; assertion that identity and role come from persistence and not from token claims.
- **Security Tests**: Replay of a token whose record was invalidated; a forged session identifier; a token bearing elevated role claims asserted to have no effect (AC-03); route inventory assertion (AC-05); identifier entropy review (AC-04).
- **Compliance Tests / Evidence**: The route-inventory result, as evidence that no protected route bypasses session resolution.
- **Acceptance-Criteria Traceability**: AC-01 — happy-path suite; AC-02 — record-state matrix; AC-03 — claim-tampering suite plus route inventory; AC-04 — generator review; AC-05 — route inventory test.
- **Coverage Target**: Every session record state × a representative protected route, positive and negative.
- **Required Test Environment**: PostgreSQL with drizzle-kit migrations applied; identities for all three roles; a token factory able to mint structurally valid tokens for absent sessions; Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-BUILD-010, REQ-SESSION-010, REQ-API-030
- **Downstream Requirements**: REQ-SESSION-040, REQ-SESSION-050, REQ-AUTHZ-010, REQ-CONSULT-020, and every authentication issue that issues a session — REQ-AUTH-020, REQ-AUTH-100, REQ-AUTH-110, REQ-AUTH-120
- **External Dependencies**: None
- **Dependency Assumptions**: The session store is transactionally consistent with the rest of persistence, so an invalidation is visible to the next request rather than eventually.
- **Failure Impact**: Without this, no revocation is possible at all: logout, engagement termination, and credential-change eviction each become a promise the system cannot keep.

## Implementation Notes

- **Constraints**: Node.js runtime with Fastify; PostgreSQL with Drizzle ORM (`CLAUDE.md`). Session lifetimes are `TO BE DECIDED` (SQ-3), so the record MUST carry explicit expiry fields rather than assuming a fixed window, and the values MUST be named constants.
- **Prohibited Approaches**: Caching session validity in the token, in a client-readable field, or in an in-process cache without an invalidation path — any of these reintroduces the revocation delay the mechanism was chosen to eliminate. Resolving the session inside individual route handlers rather than in the pipeline. Distinguishing session-absent from session-expired in the response.
- **Implementation Guidance**: Fastify's `onRequest` hook is the natural home, which also places it before the authorization enforcement point SEC-AUTHZ-5 requires. Keep the resolved session an immutable value for the request's lifetime, per `REF-PROMPT-QUALITY`, so nothing downstream can mutate identity mid-request and create a time-of-check-to-time-of-use gap.
- **AI Development Guidance**: `REF-PROMPT-JWT`, `REF-SESSION`, `REF-PROMPT-NODE`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review that no protected route can reach a handler without resolution, and that no code path treats a store error as a valid session.
- **Open Decisions**: Absolute and idle session lifetimes (`SECURITY.md` SQ-3). Whether the session store is a table or a cache with durable backing is an implementation choice, but it MUST satisfy the consistency assumption above; if a cache is used, the invalidation path is part of this issue, not a later optimization.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 250–500.
**Recommended model**: Claude Opus (`claude-opus-5`) — a small mechanism that every other session behavior rests on, where a fail-open branch or a per-handler bypass silently defeats revocation across the whole system.
