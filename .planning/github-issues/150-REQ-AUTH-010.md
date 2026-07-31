# [REQ-AUTH-010] Exactly one role per account

## Metadata

- **ID**: REQ-AUTH-010
- **Title**: Exactly one role per account
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-2.7; `SECURITY.md` Users / actors / roles ("Exactly one role per account")

## Requirement

- **Statement**: The system MUST assign every account exactly one of the roles `subscriber`, `consultant`, or `admin`, MUST reject any account state carrying zero roles or more than one role, and MUST resolve that role from persisted state on every request.
- **Rationale**: FR-2.7 states the rule; `SECURITY.md` restates it as an invariant of the authorization model. Every role-dependent control — passkey requirement (FR-2.8), admin-only plan lifecycle (FR-10.1), consultant engagement scoping (FR-11.2) — assumes a single unambiguous role.
- **Assumptions**: Role values are exactly the three named; no additional role exists in any source document.
- **Out of Scope**: How accounts are provisioned and how roles are assigned or changed, which `SECURITY.md` SQ-12 leaves open; the authorization decisions that consume the role (REQ-AUTHZ-030, REQ-CONSULT-010).
- **Design Traceability**: `DESIGN.md` OQ-7 asks whether roles get distinct visual treatment; that is open and does not affect this invariant.
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application ("enforces role rules (FR-2.7, FR-10.1)"); Identity and Session Handling ("Reads user account identity and role"); Relational Persistence ("Enforces referential integrity and ownership relationships in schema"); DR-3.
- **Security Traceability**: SEC-AUTHZ-1, SEC-AUTHZ-4, SEC-INPUT-3 (role is never client-assignable), SEC-AUTHZ-6 (declared attribute source and trust level).

## Scope

- **Applies To**: Server-Side Application, API
- **Components**: Relational Persistence (schema invariant); Identity and Session Handling (resolution); REST API Application (consumption)
- **Interfaces / Operations**: Account persistence; role resolution during session establishment and per request
- **Actors**: `subscriber`, `consultant`, `admin`
- **Preconditions**: An account record exists
- **Data Classification**: Confidential
- **Personal or Regulated Data**: Personal Data
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED (`SECURITY.md` SQ-1)

## Security Context

- **Security Objectives**: Authorization, Integrity, Accountability
- **Control Layers**: Authorization, Business-Rule Validation, Architecture
- **Threat References**: `SECURITY.md` TM-E-1 (privilege escalation via role change or provisioning); CWE-269 (improper privilege management), CWE-266 (incorrect privilege assignment)
- **Abuse / Misuse Case**: An account acquires a second role, or an ambiguous role state resolves differently in two code paths, so that an actor passes an admin check in one place and a subscriber check in another.
- **Trust Boundary**: Boundary 2 — role is resolved on the authenticated side and never accepted from the client.
- **Untrusted Inputs or Assertions**: Any role value present in a request body, header, or token claim that has not been verified against persisted state.
- **Authoritative Enforcement Point**: Relational Persistence for the invariant; Identity and Session Handling for resolution.
- **Independent Verification**: The role used in an authorization decision is read from persisted account state, not from a client assertion (DR-3).
- **Zero Trust Relevance**: NIST SP 800-207 — subject attributes come from an authoritative source rather than the requester. Exact tenet: TO BE DECIDED (not verified in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: N/A
- **Other**: `REF-PROMPT-ABAC` for attribute sourcing and trust level, as cited by SEC-AUTHZ-6.
- **Mapping Basis**: FR-2.7 is the normative source; `REF-PROMPT-ABAC` is cited by the security rule that governs attribute trust. No control identifier is asserted without verification.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an account record, when it is persisted, then it carries exactly one role value drawn from `subscriber`, `consultant`, `admin`, and that value is what role resolution returns for every request by that account.
2. **AC-02 — Boundary or failure behavior**: Given an attempt to persist an account with no role, two roles, or a role value outside the three permitted, when the write occurs, then it is rejected by a schema or constraint violation and no account record is created or updated.
3. **AC-03 — Prohibited behavior**: Given a request containing a role value in its body, headers, or an unverified token, when any authorization decision is made, then that value MUST NOT influence the decision, and role MUST NOT be inferred from the route, the client build, or the presence of a passkey.
4. **AC-04 — Additional criterion**: Given an account whose role cannot be resolved, when a request is made, then the request is denied rather than treated as the least-privileged role.

## Failure Behavior

- **On Invalid Input**: A role value supplied in a request is ignored per REQ-API-020; a write carrying it is rejected or the field stripped, with no state change.
- **On Authentication Failure**: N/A — role resolution follows successful authentication.
- **On Authorization Failure**: N/A — this issue supplies the attribute, it does not make the decision.
- **On Security-Decision Failure**: Unresolvable role denies the request (AC-04); it MUST NOT default to `subscriber`.
- **On External Dependency Failure**: If persistence is unavailable, deny; MUST NOT proceed on a cached or assumed role.
- **On System Error**: Roll back; generic error with a correlation identifier.
- **Logging / Audit**: A role change is a security-relevant account change and MUST be logged (SEC-LOG-4) and, once role-change flows are defined, audited. Log the account identifier and the role values, which are not sensitive personal data on their own.
- **Alerting**: TO BE DECIDED — role-change alerting depends on the provisioning model (`SECURITY.md` SQ-12).

## Test Strategy

- **Unit Tests**: Role resolver returns the persisted value; raises rather than defaults when the value is absent or unrecognized.
- **Integration Tests**: Schema-level rejection of zero-role, multi-role, and unknown-role writes; resolution consistency across two consecutive requests.
- **Security Tests**: Role-injection attempts in body, query, header, and an unverified token, asserting no decision change; a test that an unresolvable role denies rather than downgrades.
- **Compliance Tests / Evidence**: N/A
- **Acceptance-Criteria Traceability**: AC-01 — resolution suite; AC-02 — constraint violation suite; AC-03 — role-injection suite; AC-04 — unresolvable-role denial test.
- **Coverage Target**: All three role values and every invalid role state exercised.
- **Required Test Environment**: One account per role plus fixtures for the invalid states. Runs against PostgreSQL on Vitest.

## Dependencies

- **Upstream Requirements**: REQ-API-020, REQ-API-030
- **Downstream Requirements**: REQ-AUTHZ-010, REQ-AUTHZ-030, REQ-AUTH-020, REQ-CONSULT-010, REQ-AUDIT-030
- **External Dependencies**: None
- **Dependency Assumptions**: The chosen RDBMS supports a check or enumerated-type constraint sufficient to enforce AC-02 in schema.
- **Failure Impact**: An ambiguous role makes every role-dependent control unreliable, including the admin-only plan lifecycle that carries safety impact (TM-T-5).

## Implementation Notes

- **Constraints**: PostgreSQL with Drizzle ORM and drizzle-kit migrations (`CLAUDE.md`). The invariant MUST hold in schema, not only in application code, so that operator-level writes (trust boundary 5) cannot create an ambiguous account.
- **Prohibited Approaches**: A nullable role column with an application-side default; a role list or bitmask; deriving role from group membership or from which credential type was used; permitting a request to carry role as an optimization.
- **Implementation Guidance**: Represent role as an immutable value in the session context so that a time-of-check-to-time-of-use gap cannot open mid-request (`SECURITY.md` code-quality resolution).
- **AI Development Guidance**: `REF-PROMPT-ABAC`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the schema constraint and the resolver.
- **Open Decisions**: `SECURITY.md` SQ-12 — how accounts are provisioned and how a role is assigned, changed, or removed. This issue guarantees the invariant and the resolution path; the lifecycle that sets the value is blocked and has no issue.

**Estimated effort**: 0.5–1 engineer-day. **Estimated changed lines**: 100–300.
**Recommended model**: Claude Opus (`claude-opus-5`) — a small invariant that every authorization control depends on; fail-closed resolution is the subtle part.
