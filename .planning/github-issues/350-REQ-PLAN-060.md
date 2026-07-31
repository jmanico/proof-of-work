# [REQ-PLAN-060] Plan unpublication

## Metadata

- **ID**: REQ-PLAN-060
- **Title**: Plan unpublication
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-4.3, FR-4.7, FR-7.5, FR-10.2

## Requirement

- **Statement**: An `admin` MUST be able to unpublish a plan, an unpublished plan MUST cease to be visible to subscribers, existing subscriber customized copies derived from it MUST remain unchanged, and the action MUST produce an audit entry.
- **Rationale**: FR-4.3 grants unpublication to admins; FR-4.7 restricts subscriber visibility to published plans; FR-7.5 requires an existing customized copy to remain unchanged when its source plan is unpublished; FR-10.2 requires the action to be audited. Unpublication is the corrective control when a published plan turns out to be wrong or harmful, which makes it the counterpart to the TM-T-5 threat.
- **Assumptions**: The publication gate and publication state exist (REQ-PLAN-050).
- **Out of Scope**: Deleting a plan, which no requirement grants; whether unpublication should notify subscribers who follow the plan, which no source document specifies; plan selection semantics, blocked by `REQUIREMENTS.md` OQ-6, which would determine what "following an unpublished plan" means.
- **Design Traceability**: `DESIGN.md` — Components → Buttons ("Destructive actions use `error` as the filled color and require explicit confirmation"), Form feedback and errors, Focus states.
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application ("Enforces publication gates (FR-4.4, FR-4.5, FR-4.7), copy-on-customize semantics (FR-7.2, FR-7.5)"); data flow 6.
- **Security Traceability**: SEC-AUTHZ-4, SEC-INPUT-3, SEC-INPUT-4, SEC-LOG-6.

## Scope

- **Applies To**: Server-Side Application, API, Web Client
- **Components**: REST API Application; Relational Persistence; Browser Client (admin plan view)
- **Interfaces / Operations**: Unpublish a plan
- **Actors**: `admin`; `subscriber` as the audience whose visibility changes
- **Preconditions**: An existing published plan and an authenticated `admin` session
- **Data Classification**: Internal
- **Personal or Regulated Data**: Personal Data — the acting admin's identifier in the audit entry
- **Jurisdiction / Regulatory Scope**: N/A

## Security Context

- **Security Objectives**: Integrity, Safety, Authorization, Accountability, Availability
- **Control Layers**: Authorization, Business-Rule Validation, Logging and Monitoring
- **Threat References**: `SECURITY.md` TM-T-5 (harmful published content — unpublication is the remediation path), TM-R-1 (unaudited lifecycle actions), TM-E-1; CWE-862 (missing authorization), CWE-284 (improper access control)
- **Abuse / Misuse Case**: A non-admin unpublishes plans as a denial-of-service against the library; or unpublication silently mutates or deletes subscribers' customized copies, destroying data FR-7.5 protects; or an unpublished plan remains reachable through a direct identifier.
- **Trust Boundary**: Boundary 1.
- **Untrusted Inputs or Assertions**: The plan identifier; any publication state in the payload.
- **Authoritative Enforcement Point**: REST API Application — role gate plus persisted state change.
- **Independent Verification**: Subscriber-facing reads filter on persisted publication state, so visibility follows the state rather than a cache the client holds.
- **Zero Trust Relevance**: N/A

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: N/A
- **Regulatory**: N/A
- **Other**: `REF-LOG` for the audit obligation; `REF-ASVS-5` as cited by SEC-AUTHZ-4.
- **Mapping Basis**: FR-4.3, FR-4.7, FR-7.5, and FR-10.2 are the normative sources; the references are those `SECURITY.md` cites for the authorization and audit rules applied here.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a published plan, when an `admin` unpublishes it, then its publication state becomes unpublished, it no longer appears in subscriber-facing listings or retrievals, and exactly one audit entry records the unpublish action, the acting admin, the plan, and the time.
2. **AC-02 — Boundary or failure behavior**: Given a subscriber holding a customized copy derived from the plan, when the plan is unpublished, then the copy remains retrievable and byte-identical to its state before the unpublication (FR-7.5).
3. **AC-03 — Prohibited behavior**: Given an unpublish request, when it is made by a `subscriber` or `consultant`, then it MUST be denied and the plan MUST remain published; and unpublication MUST NOT delete, alter, or hide any subscriber's customized copy or log entries.
4. **AC-04 — Additional criterion**: Given an unpublished plan, when a subscriber requests it directly by identifier, then the response is indistinguishable from that for a plan that does not exist (FR-4.7, REQ-AUTHZ-040).
5. **AC-05 — Additional criterion**: Given the admin interface, when unpublication is invoked, then it requires explicit confirmation and is presented as a destructive action (`DESIGN.md`, Components → Buttons).

## Failure Behavior

- **On Invalid Input**: Reject per REQ-API-010; state unchanged.
- **On Authentication Failure**: Denied upstream; admins authenticate by passkey (REQ-AUTH-020).
- **On Authorization Failure**: Denied for non-admin roles (REQ-AUTHZ-030), with no disclosure of the plan's state beyond what FR-4.7 permits.
- **On Security-Decision Failure**: If the role or the plan's current state cannot be resolved, refuse the operation.
- **On External Dependency Failure**: If persistence or audit storage is unavailable, the operation fails atomically and the plan's state is unchanged.
- **On System Error**: Roll back the state change and its audit entry together.
- **Logging / Audit**: One audit entry per successful unpublish (REQ-AUDIT-030). Denials logged as security events.
- **Alerting**: TO BE DECIDED — no alerting model exists in the source documents, though unpublication of a widely followed plan is operationally significant.

## Test Strategy

- **Unit Tests**: The unpublish service transitions state only from published, is idempotent or explicitly refuses on an already-unpublished plan, and invokes the audit writer exactly once.
- **Integration Tests**: Unpublish end to end; assert subscriber listing and direct retrieval both stop returning it; assert customized copies are unchanged by comparing full content before and after.
- **Security Tests**: Role matrix asserting denial for `subscriber` and `consultant`; direct-identifier retrieval after unpublication returning an indistinguishable response; assertion that no customized copy or log entry row was modified or removed.
- **Compliance Tests / Evidence**: Unpublish audit entries, retained as accountability evidence for FR-10.2.
- **Acceptance-Criteria Traceability**: AC-01 — unpublish suite; AC-02 — copy-immutability comparison; AC-03 — role matrix and data-preservation assertions; AC-04 — response-equivalence test; AC-05 — confirmation flow test.
- **Coverage Target**: Both plan types; all three roles; published and already-unpublished starting states.
- **Required Test Environment**: An admin, a subscriber holding a customized copy of the target plan, and a consultant. Runs against PostgreSQL on Vitest.

## Dependencies

- **Upstream Requirements**: REQ-PLAN-050, REQ-AUTHZ-030, REQ-AUDIT-030, REQ-CUSTOM-030
- **Downstream Requirements**: REQ-CATALOG-010, REQ-CATALOG-020
- **External Dependencies**: None
- **Dependency Assumptions**: Customized copies are independent records rather than references to the published plan, which REQ-CUSTOM-010 and REQ-CUSTOM-030 establish; without that, AC-02 is unachievable.
- **Failure Impact**: If unpublication cascades into customized copies, an admin's corrective action destroys subscriber data that FR-7.5 explicitly protects.

## Implementation Notes

- **Constraints**: PostgreSQL with Drizzle ORM and Fastify (`CLAUDE.md`). Whether a subscriber currently following an unpublished plan retains access to it through their own copy is answered by FR-7.5 for copies; for plan *selection* it depends on `REQUIREMENTS.md` OQ-6 and is out of scope.
- **Prohibited Approaches**: Deleting the plan instead of unpublishing it; cascading the state change to derived copies; soft-deleting copies; relying on the client to stop displaying an unpublished plan.
- **Implementation Guidance**: Treat unpublication as a state transition on the plan alone. The subscriber-facing filter established in REQ-PLAN-050 is what makes visibility follow automatically; do not add a second filtering mechanism.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-VUE`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the role gate; product review of what subscribers see when a plan they follow is withdrawn.
- **Open Decisions**: Whether subscribers should be notified of unpublication is unspecified in all source documents. Plan selection semantics (`REQUIREMENTS.md` OQ-6) remain open and would extend this issue's scope when resolved.

**Estimated effort**: 0.5–1 engineer-day. **Estimated changed lines**: 150–350.
**Recommended model**: Claude Opus (`claude-opus-5`) — small, but the data-preservation guarantee in AC-02 is the kind of thing a cascade delete quietly breaks.
