# [REQ-AUTHZ-050] Central typed authorization policy module

## Metadata

- **ID**: REQ-AUTHZ-050
- **Title**: Central typed authorization policy module
- **Version**: 1.0.1
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `SECURITY.md` SEC-AUTHZ-5, SEC-AUTHZ-6, SEC-AUTHZ-7 (SQ-4 RESOLVED); supports SEC-AUTHZ-8, SEC-AUTHZ-9

## Requirement

- **Statement**: The REST API Application MUST evaluate authorization for every protected operation through a single first-party typed policy module — pure functions over a typed attribute context — invoked at the single Fastify preHandler enforcement point, combining policies deny-overrides and producing a denial whenever any required attribute is missing or unresolvable.
- **Rationale**: SEC-AUTHZ-5 requires a single enforcement point that all protected operations traverse, so that per-endpoint bespoke authorization can never be the primary control; SEC-AUTHZ-6 fixes the attribute schema and trust levels; SEC-AUTHZ-7 fixes deny-overrides with missing-attribute denial. `SECURITY.md` SQ-4 resolved the concrete model: a first-party typed policy module with no policy-language dependency (DEP-1, DEP-2), unit-tested as plain functions. Threat TM-E-2 (policy gap or fail-open evaluation) is mitigated by exactly this structure.
- **Assumptions**: The authentication gate (REQ-AUTHZ-010) has already resolved the actor's identity and role from the server-side session record (REQ-SESSION-030) before policy evaluation runs. Persisted state (ownership, publication status, engagement, subscription periods, consent) is readable at decision time.
- **Out of Scope**: The authentication stage of the request pipeline (REQ-AUTHZ-010); the individual rule content delivered by other issues that this module hosts — ownership scoping (REQ-AUTHZ-020), admin-only plan lifecycle restriction (REQ-AUTHZ-030), the admin health-data prohibition views (REQ-AUTHZ-060), engagement scoping (REQ-CONSULT-010), consultant capabilities (REQ-CONSULT-040), and the subscription entitlement gate (REQ-ENTITLE-010). This issue delivers the module, the typed attribute context, the capability map, the combination algorithm, and the enforcement-point wiring those issues plug into.
- **Design Traceability**: N/A — `DESIGN.md` presents authorization outcomes (permission and lapsed-subscription states explain the reason and next step) but does not specify the policy mechanism.
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application ("Single server-side entry point and the sole authority for business rules"); trust boundary 1 (client is untrusted; all authorization decisions re-made server-side); DR-2, DR-3 (identity and role from Identity and Session Handling; ownership from persisted state), DR-6.
- **Security Traceability**: SEC-AUTHZ-5, SEC-AUTHZ-6, SEC-AUTHZ-7; capability-map obligations from SEC-AUTHZ-8 (subscription-active condition) and SEC-AUTHZ-9 (no admin-mapped health-data capability); SEC-TB-1; `SECURITY.md` SQ-4 RESOLVED.

## Scope

- **Applies To**: API, Server-Side Application
- **Components**: REST API Application; Identity and Session Handling (attribute source); Relational Persistence (attribute source)
- **Interfaces / Operations**: Every protected REST route; the policy module's decision interface; the Fastify preHandler enforcement point
- **Actors**: `subscriber`, `consultant`, `admin`; anonymous internet attacker as the excluded actor
- **Preconditions**: Authenticated session resolved by REQ-AUTHZ-010 and REQ-SESSION-030
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED)

## Security Context

- **Security Objectives**: Authorization, Confidentiality, Integrity, Accountability
- **Control Layers**: Authorization, Architecture
- **Threat References**: `SECURITY.md` TM-E-2 (authorization policy gap or fail-open evaluation), TM-I-1 (BOLA), TM-E-3 (consultant capability creep), TM-E-4 (entitlement bypass); CWE-862 (missing authorization), CWE-863 (incorrect authorization), CWE-636 (not failing securely)
- **Abuse / Misuse Case**: An attacker reaches a protected operation through a route whose handler performs bespoke authorization that omits a predicate; or an exception or absent attribute during evaluation resolves as permit; or client-supplied attribute-shaped input (a role field, an owner identifier, an engagement state) influences the decision.
- **Trust Boundary**: Boundary 1 — every request attribute originating from the client is untrusted; decisions consume only attributes from Identity and Session Handling and persisted state.
- **Untrusted Inputs or Assertions**: Any client-supplied role, ownership, entitlement, engagement, or capability assertion; unverified headers; request-body fields shaped like policy attributes.
- **Authoritative Enforcement Point**: The single Fastify preHandler enforcement point in the REST API Application, invoking the first-party typed policy module (SEC-AUTHZ-5; `SECURITY.md` SQ-4).
- **Independent Verification**: Subject attributes come from Identity and Session Handling; resource attributes are read from persisted state at decision time; no attribute is accepted from the request (SEC-AUTHZ-6, DR-3).
- **Zero Trust Relevance**: NIST SP 800-207 — per-request access decisions from verified identity and current resource state. Exact tenet: TO BE DECIDED (not verified against the publication in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — per-issue mappings are verified during the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — per-issue mappings are verified during the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: The policy module is the gate on all health-data access, so the `SECURITY.md` SQ-1 regime set applies — GDPR/UK GDPR for EU/UK data subjects; CCPA/CPRA, Washington My Health My Data, and the FTC Health Breach Notification Rule for US users; HIPAA not applicable. Statute-section precision: TO BE DECIDED (SQ-1 counsel review, SQ-10).
- **Other**: `REF-PROMPT-ABAC` (PDP/PEP/PIP separation, attribute trust levels, policy combination, policy testing), `REF-PC-2024`, `REF-ASVS-5` as cited by SEC-AUTHZ-5–SEC-AUTHZ-7.
- **Mapping Basis**: SEC-AUTHZ-5, SEC-AUTHZ-6, and SEC-AUTHZ-7 name these references; the CWE identifiers name the missing-authorization, incorrect-authorization, and fail-open classes.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated actor whose typed attribute context (subject: account identifier, role, email-verified, MFA-enabled, subscription-active, consent state; resource: type, owner identifier, publication status, engagement; action: named capability) satisfies a policy for the requested capability, when the request reaches the preHandler enforcement point, then the policy module returns permit and the handler executes with that decision recorded in the request context.
2. **AC-02 — Boundary or failure behavior**: Given a decision in which any required attribute is missing or unresolvable (for example, the resource row is absent, the engagement state cannot be read, or persistence errors during attribute resolution), when the policy module evaluates, then the result is a denial — never an error-open permit — and the handler does not execute (SEC-AUTHZ-7).
3. **AC-03 — Prohibited behavior**: Given attribute-shaped client input — a role claim, an owner identifier, a `subscriptionActive` flag, or an engagement state in headers or body — when the request is evaluated, then that input MUST NOT change the decision (SEC-AUTHZ-6); and given the capability map, no capability that reads a subscriber's health data (FR-9.12) may be mapped to the `admin` role (SEC-AUTHZ-9), and no capability over plans, customization, or progress may be granted to a subscriber whose subscription-active attribute is false (SEC-AUTHZ-8).
4. **AC-04 — Additional criterion**: Given the route-inventory test, when it enumerates every registered route, then every protected route traverses the preHandler enforcement point, any route not traversing it appears on the explicitly declared public list from REQ-AUTHZ-010, and the test fails when a new route is registered outside both sets (SEC-AUTHZ-5).
5. **AC-05 — Additional criterion**: Given two applicable policies where one denies and one permits the same request, when the module combines them, then the result is a denial (deny-overrides, SEC-AUTHZ-7); and every policy function is a pure function testable in isolation with no I/O, hidden global state, or clock dependence beyond its input context.

## Failure Behavior

- **On Invalid Input**: N/A at this layer — request-shape validation is REQ-API-010; the policy module consumes only server-sourced attributes.
- **On Authentication Failure**: N/A — handled before policy evaluation by REQ-AUTHZ-010; the module is never invoked without a resolved identity.
- **On Authorization Failure**: Deny the operation with a generic response that does not disclose whether the target resource exists (SEC-AUTHZ-2 posture); the client receives enough structured reason to present the permission state per `DESIGN.md` without re-deriving the rule (DR-2).
- **On Security-Decision Failure**: Deny by default. An exception thrown anywhere inside policy evaluation denies the request (SEC-AUTHZ-7; the `SECURITY.md` CODE_QUALITY_PROMPT resolution's fail-closed discipline).
- **On External Dependency Failure**: If persisted state needed for an attribute cannot be read, the attribute is unresolvable and the decision is a denial; the module MUST NOT substitute defaults for absent attributes.
- **On System Error**: Generic error with a correlation identifier (SEC-ERR-1); no handler execution; no partial state change.
- **Logging / Audit**: Every authorization denial is logged with route, capability, decision reason class, and correlation identifier (SEC-LOG-4); logs never contain health values, credentials, or tokens (SEC-LOG-3). Health-data audit entries themselves are REQ-AUDIT-020's obligation on the paths the module permits.
- **Alerting**: Threshold alerts on denial spikes route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-3, SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: Policy functions tested as plain functions over constructed attribute contexts: permit and deny cases per capability; deny-overrides combination; missing and unresolvable attribute denial; exception-inside-evaluation denial; capability-map assertions that no admin health-data capability exists and that `subscription-active: false` yields no plan, customization, or progress capability.
- **Integration Tests**: End-to-end requests through the preHandler enforcement point asserting the decision governs handler execution; attribute resolution from real persisted state (ownership, engagement, publication status, subscription period rows); attribute-shaped client input injected via headers and body asserting no decision change.
- **Security Tests**: Route-inventory test over the full route table (AC-04); fail-open probes — forced persistence errors during attribute resolution asserting denial; the authorization test suite that SEC-CICD-4 names as a merge-blocking gate runs against this module.
- **Compliance Tests / Evidence**: Policy-module review against the SEC-AUTHZ-6 attribute schema, recorded for the SQ-10 pre-launch assessment.
- **Acceptance-Criteria Traceability**: AC-01 — capability permit unit suite plus integration pass-through; AC-02 — missing-attribute and fault-injection suites; AC-03 — attribute-injection suite and capability-map unit assertions; AC-04 — route-inventory test; AC-05 — deny-overrides and purity unit suite.
- **Coverage Target**: Project coverage threshold is 90% line and branch (`CLAUDE.md`, 2026-08-03); every policy function and every denial path MUST have positive and negative tests regardless.
- **Required Test Environment**: Vitest; HTTP test client against the Fastify app; fixture accounts for all three roles with varied email-verified, MFA, consent, subscription, and engagement states; a route-inventory source.

## Dependencies

- **Upstream Requirements**: REQ-AUTHZ-010, REQ-SESSION-030
- **Downstream Requirements**: REQ-AUTHZ-020, REQ-AUTHZ-030, REQ-AUTHZ-040, REQ-AUTHZ-060, REQ-CONSULT-010, REQ-CONSULT-030, REQ-CONSULT-040, REQ-ENTITLE-010, REQ-ENTITLE-030, and every feature issue whose operations are protected
- **External Dependencies**: None — SQ-4 forbids a policy-language dependency (DEP-1, DEP-2).
- **Dependency Assumptions**: Identity and Session Handling supplies a verified identity and role or an explicit failure, never an ambiguous result; persisted state is transactionally consistent at decision time.
- **Failure Impact**: A gap or fail-open path in this module is TM-E-2 realized — unauthorized access to health data across every protected operation.

## Implementation Notes

- **Constraints**: TypeScript on Node.js with Fastify (`CLAUDE.md`); the enforcement point is the Fastify preHandler lifecycle hook (`SECURITY.md` SQ-4); policy expressed as first-party typed pure functions; attribute sources restricted to Identity and Session Handling and persisted state.
- **Prohibited Approaches**: Per-endpoint bespoke authorization as the primary control (SEC-AUTHZ-5); adopting a policy-language or policy-engine dependency (DEP-1, DEP-2); reading any decision attribute from client input or unverified headers (SEC-AUTHZ-6); permit-overrides or first-applicable combination; defaulting a missing attribute to a permissive value; caching decisions or attributes in the session or token such that revocation waits for expiry (SEC-SESSION-4, SEC-SESSION-6); silent exception swallowing in evaluation paths.
- **Implementation Guidance**: Structure per `REF-PROMPT-ABAC`: the preHandler hook is the PEP, the typed policy module is the PDP, and attribute loaders over Identity and persisted state are the PIPs. Name capabilities as SEC-AUTHZ-6 illustrates (`plan.publish`, `plan_copy.edit`, `log.write`) and derive them from role plus relationship — owner, engaged consultant (FR-11.6), or admin. Keep the capability map a single reviewable declaration so the SEC-AUTHZ-9 review ("no admin-mapped health-data capability exists") is a table inspection, not a code hunt. Immutable attribute context avoids time-of-check-to-time-of-use gaps (the `SECURITY.md` CODE_QUALITY_PROMPT resolution).
- **AI Development Guidance**: `REF-PROMPT-ABAC`, `REF-PROMPT-API`, `REF-PROMPT-QUALITY`; `CLAUDE.md` (Fastify lifecycle hooks as the single authorization enforcement point).
- **Required Human Review**: Security review of the policy module, the capability map, and the enforcement-point wiring; architecture review that no protected route bypasses the hook.
- **Open Decisions**: None — SQ-4 is RESOLVED and fixes the model, the attribute schema, and the enforcement point. The REST API Application's internal module decomposition remains open (`ARCHITECTURE.md`) but does not change this module's responsibility or interface.

**Estimated effort**: 1.5–2 engineer-days. **Estimated changed lines**: 600–1100.
**Recommended model**: Claude Fable (`claude-fable-5`) — a cross-cutting foundation spanning the capability map, attribute loaders, enforcement-point wiring, and the route-inventory test, whose breadth benefits from Fable; every policy function still carries exhaustive positive and negative unit tests.
