# [REQ-AUTHZ-030] Admin-only restriction on plan lifecycle operations

## Metadata

- **ID**: REQ-AUTHZ-030
- **Title**: Admin-only restriction on plan lifecycle operations
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-4.2, FR-4.3, FR-10.1; `SECURITY.md` SEC-AUTHZ-4

## Requirement

- **Statement**: Plan authoring, editing, verification, publication, and unpublication MUST be permitted only to accounts with the `admin` role and MUST be denied to `subscriber` and `consultant` accounts, and subscribers MUST NOT be able to author plans, submit plans for publication, or share plans with other users.
- **Rationale**: FR-10.1 restricts these actions to admins; FR-4.2 forbids subscriber authoring, submission, and sharing; FR-4.3 grants the actions to admins; SEC-AUTHZ-4 states the enforcement rule.
- **Assumptions**: Role is resolved from the verified session (REQ-AUTHZ-010) and is server-assigned (REQ-API-020, REQ-AUTH-010).
- **Out of Scope**: The verification workflow itself, which depends on `REQUIREMENTS.md` OQ-10 (re-verification after edit) and OQ-16 (dual control) and is blocked; how admin accounts are provisioned (REQ-AUTH-140, SEC-AUTHN-9; vetting and deprovisioning remain open under `SECURITY.md` SQ-12); customization of a published plan by a subscriber, which is a distinct permitted behavior (REQ-CUSTOM-010).
- **Design Traceability**: `DESIGN.md` OQ-7 asks whether admins get a distinct interface region; that is open and does not affect this server-side rule.
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application ("enforces role rules (FR-2.7, FR-10.1)"); DR-4 ("Published plans are mutable only by admin-role operations").
- **Security Traceability**: SEC-AUTHZ-4; supports SEC-AUTHZ-1, SEC-LOG-6, SEC-INPUT-4.

## Scope

- **Applies To**: API, Server-Side Application
- **Components**: REST API Application
- **Interfaces / Operations**: Plan create, plan edit, citation management, verification, publish, unpublish
- **Actors**: `admin` (permitted), `subscriber` and `consultant` (denied), anonymous (denied upstream)
- **Preconditions**: Authenticated session with a resolved role
- **Data Classification**: Internal — plan content is not personal data, but its integrity carries safety impact
- **Personal or Regulated Data**: None — plan content is admin-authored; the verification record names the acting admin
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED (`SECURITY.md` SQ-1)

## Security Context

- **Security Objectives**: Integrity, Authorization, Safety, Accountability
- **Control Layers**: Authorization, Business-Rule Validation
- **Threat References**: `SECURITY.md` TM-T-5 (compromised admin publishes harmful content), TM-E-1 (privilege escalation), TM-R-1 (unaudited admin actions); CWE-862 (missing authorization), CWE-269 (improper privilege management)
- **Abuse / Misuse Case**: A subscriber or consultant invokes a plan lifecycle endpoint directly, bypassing the UI, to publish or alter plan content that other subscribers will follow as health guidance.
- **Trust Boundary**: Boundary 1 — the client is untrusted, including a client that never renders admin controls.
- **Untrusted Inputs or Assertions**: Any role claim in a request; the assumption that only an admin UI can reach these routes.
- **Authoritative Enforcement Point**: REST API Application, using the session-resolved role.
- **Independent Verification**: Role comes from Identity and Session Handling, not from the request (DR-3).
- **Zero Trust Relevance**: NIST SP 800-207 — privilege is evaluated per request rather than inferred from the interface used. Exact tenet: TO BE DECIDED (not verified in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: N/A
- **Other**: `REF-ASVS-5`, `REF-API-2023` as cited by SEC-AUTHZ-4.
- **Mapping Basis**: SEC-AUTHZ-4 cites these references directly; the CWE identifiers name the missing-authorization and privilege-management classes.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated `admin`, when the admin invokes create, edit, verify, publish, or unpublish on a plan, then the operation proceeds subject to the other gates (REQ-PLAN-050) and emits an audit entry (REQ-AUDIT-030).
2. **AC-02 — Boundary or failure behavior**: Given an authenticated `subscriber` or `consultant`, when they invoke any plan lifecycle operation directly against the API, then the operation is denied, no plan state changes, and the denial is logged.
3. **AC-03 — Prohibited behavior**: Given any actor, when a request supplies a `role` value or an admin-suggesting header, then it MUST NOT influence the decision; and no plan lifecycle route may rely on the client's failure to render a control as its protection.
4. **AC-04 — Additional criterion**: Given a `subscriber`, when they attempt to submit a plan for publication or to share a plan copy with another user, then no such operation exists or it is denied (FR-4.2).

## Failure Behavior

- **On Invalid Input**: Rejected by REQ-API-010 before the role decision.
- **On Authentication Failure**: Denied upstream by REQ-AUTHZ-010.
- **On Authorization Failure**: Deny with no state change. Existence of the plan MAY be disclosed to any authenticated actor only where the plan is published (FR-4.7); unpublished plan existence MUST NOT be disclosed to non-admins.
- **On Security-Decision Failure**: Deny if the role cannot be resolved (SEC-AUTHZ-7).
- **On External Dependency Failure**: Deny on persistence unavailability; never proceed on an unverified role.
- **On System Error**: Roll back; generic error with a correlation identifier.
- **Logging / Audit**: Log denials with actor, role, operation, and plan identifier (SEC-LOG-4). Permitted admin actions emit audit entries per FR-10.2 and SEC-LOG-6 (REQ-AUDIT-030).
- **Alerting**: TO BE DECIDED — no threshold for repeated privileged-operation denials is defined.

## Test Strategy

- **Unit Tests**: Role gate permits `admin` and denies `subscriber` and `consultant` for each lifecycle operation; denies when role is unresolvable.
- **Integration Tests**: Each lifecycle operation invoked as each of the three roles, asserting one allow and two denies, plus unchanged plan state on denial.
- **Security Tests**: Direct-API invocation bypassing the client; role-claim injection in body and headers; assertion that unpublished plans are not enumerable by non-admins.
- **Compliance Tests / Evidence**: N/A
- **Acceptance-Criteria Traceability**: AC-01 and AC-02 — role matrix suite; AC-03 — role-claim injection tests; AC-04 — route inventory assertion that no subscriber submit or share operation exists.
- **Coverage Target**: Every plan lifecycle operation × every role.
- **Required Test Environment**: One identity per role; seeded published and unpublished plans. Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-AUTHZ-010, REQ-AUTH-010, REQ-API-020
- **Downstream Requirements**: REQ-PLAN-030, REQ-PLAN-040, REQ-PLAN-050, REQ-PLAN-060, REQ-AUDIT-030
- **External Dependencies**: None
- **Dependency Assumptions**: Role assignment is server-controlled and immutable from the request path.
- **Failure Impact**: Loss of this control lets any subscriber publish health guidance to all subscribers — the safety threat TM-T-5 without even needing a compromised admin.

## Implementation Notes

- **Constraints**: Node.js runtime with Fastify (`CLAUDE.md`). Admin and consultant accounts authenticate with passkeys (FR-2.8), so this gate composes with REQ-AUTH-020 but does not replace it.
- **Prohibited Approaches**: Hiding admin controls in the client as the enforcement (DR-2); role checks duplicated ad hoc per handler with no shared gate; inferring privilege from the route prefix alone.
- **Implementation Guidance**: Express the role requirement declaratively next to the route so the role matrix test can be generated from the same source.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-ABAC`; `CLAUDE.md`.
- **Required Human Review**: Security review of the role matrix.
- **Open Decisions**: `REQUIREMENTS.md` OQ-16 asks whether verification requires a second admin (dual control). This issue enforces "admin only"; if OQ-16 resolves to dual control, the verification operation gains an additional constraint in the blocked verification issue, not here.

**Estimated effort**: 0.5–1 engineer-day. **Estimated changed lines**: 150–350.
**Recommended model**: Claude Opus (`claude-opus-5`) — small, security-sensitive, and safety-relevant; exhaustive role-matrix coverage matters more than autonomy.
