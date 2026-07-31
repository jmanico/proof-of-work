# [REQ-AUTHZ-010] Deny-by-default authentication requirement on protected operations

## Metadata

- **ID**: REQ-AUTHZ-010
- **Title**: Deny-by-default authentication requirement on protected operations
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-2.1; `SECURITY.md` SEC-AUTHN-1, SEC-AUTHZ-1

## Requirement

- **Statement**: The REST API Application MUST deny every request that lacks a valid authenticated session by default, and MUST require an explicit, reviewed declaration for each route intended to be reachable without authentication.
- **Rationale**: FR-2.1 requires an account for all plan, customization, and progress functionality and denial of unauthenticated access; SEC-AUTHN-1 requires denial by default rather than per-endpoint allow-listing, so that a newly added route is protected unless someone deliberately opens it.
- **Assumptions**: Session validity is determined by Identity and Session Handling (REQ-SESSION-010).
- **Out of Scope**: What an authenticated actor may then do — ownership scoping (REQ-AUTHZ-020), role restriction (REQ-AUTHZ-030), engagement scoping (REQ-CONSULT-010), and subscription entitlement (blocked by `REQUIREMENTS.md` OQ-1); the central ABAC policy architecture (SEC-AUTHZ-5, SEC-AUTHZ-6, blocked by `SECURITY.md` SQ-4).
- **Design Traceability**: N/A — `DESIGN.md` does not specify unauthenticated views.
- **Architecture Traceability**: `ARCHITECTURE.md` — trust boundary 2 (unauthenticated → authenticated); REST API Application ("Authenticates every request via Identity and Session Handling"); DR-3.
- **Security Traceability**: SEC-AUTHN-1, SEC-AUTHZ-1; supports SEC-TB-1.

## Scope

- **Applies To**: API, Server-Side Application
- **Components**: REST API Application; Identity and Session Handling
- **Interfaces / Operations**: Every REST route
- **Actors**: Anonymous internet attacker; all authenticated roles
- **Preconditions**: None
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED (`SECURITY.md` SQ-1)

## Security Context

- **Security Objectives**: Authorization, Confidentiality, Authenticity
- **Control Layers**: Authentication, Authorization, Architecture
- **Threat References**: `SECURITY.md` TM-I-1 (unauthorized reads), TM-E-2 (policy gap or fail-open evaluation); CWE-306 (missing authentication for critical function), CWE-862 (missing authorization)
- **Abuse / Misuse Case**: An anonymous attacker reaches a route that was added without an authentication guard, or an error inside session validation results in the request proceeding as authenticated.
- **Trust Boundary**: Boundary 2 — unauthenticated → authenticated request handling.
- **Untrusted Inputs or Assertions**: Any session credential presented by the client, and any header purporting to assert identity.
- **Authoritative Enforcement Point**: A single request-pipeline guard in the REST API Application, invoked before any handler.
- **Independent Verification**: Identity is resolved from the verified session by Identity and Session Handling, never from a client claim (DR-3).
- **Zero Trust Relevance**: NIST SP 800-207 — every resource request is authenticated and authorized before access. Exact tenet: TO BE DECIDED (not verified against the publication in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: N/A
- **Other**: `REF-AUTH`, `REF-PROMPT-API` as cited by SEC-AUTHN-1.
- **Mapping Basis**: SEC-AUTHN-1 names these references; the CWE identifiers describe the missing-authentication and missing-authorization classes.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a request with a valid authenticated session, when it targets a protected route, then the guard resolves the actor's account identity and role and passes them to the handler as the authoritative context.
2. **AC-02 — Boundary or failure behavior**: Given a request with no session, an expired session, or an unverifiable session credential, when it targets any route not explicitly declared public, then it is denied without executing the handler and without disclosing whether the target resource exists.
3. **AC-03 — Prohibited behavior**: Given a newly added route with no declaration, when it is requested without a session, then it MUST NOT be reachable; and an exception raised inside session resolution MUST NOT result in the request being treated as authenticated.
4. **AC-04 — Additional criterion**: Given the route inventory test, when it runs, then every route is either protected or listed in an explicit public-route declaration, and the public list is reviewable in one place.

## Failure Behavior

- **On Invalid Input**: N/A — malformed session credentials are an authentication failure, not an input-validation failure.
- **On Authentication Failure**: Deny with a uniform response that does not reveal which factor failed or whether an account exists (SEC-AUTHN-3).
- **On Authorization Failure**: N/A at this layer — subsequent authorization is covered by REQ-AUTHZ-020, REQ-AUTHZ-030, and REQ-CONSULT-010.
- **On Security-Decision Failure**: Deny by default. Any error resolving the session denies the request (SEC-AUTHZ-7).
- **On External Dependency Failure**: If session state cannot be resolved because a dependency is unavailable, deny; MUST NOT degrade to allowing the request.
- **On System Error**: Generic error with a correlation identifier (SEC-ERR-1); no handler execution.
- **Logging / Audit**: Log every denial with route, reason class, correlation identifier, and — where known — the account identifier; never the presented credential or token (SEC-LOG-3, SEC-LOG-4).
- **Alerting**: TO BE DECIDED — thresholds for repeated denials are undefined (`SECURITY.md` SQ-3).

## Test Strategy

- **Unit Tests**: Guard denies on absent, malformed, expired, and unverifiable sessions; guard propagates identity and role on success; guard denies when session resolution throws.
- **Integration Tests**: Automated enumeration of every route unauthenticated, asserting denial for all but the declared public set.
- **Security Tests**: Attempt to reach protected routes with a forged or omitted credential; assert no information disclosure difference between "resource absent" and "not authenticated".
- **Compliance Tests / Evidence**: The public-route declaration and its review record.
- **Acceptance-Criteria Traceability**: AC-01 — guard unit suite; AC-02 — unauthenticated enumeration suite; AC-03 — fail-closed exception test and a new-route fixture; AC-04 — route inventory test.
- **Coverage Target**: 100% of routes exercised by the unauthenticated enumeration test.
- **Required Test Environment**: HTTP test client and a route inventory source; test framework TO BE DECIDED.

## Dependencies

- **Upstream Requirements**: REQ-SESSION-010
- **Downstream Requirements**: REQ-AUTHZ-020, REQ-AUTHZ-030, REQ-AUTHZ-040, REQ-CONSULT-010, and every feature issue
- **External Dependencies**: None
- **Dependency Assumptions**: Identity and Session Handling returns a verified identity and role or an explicit failure — never an ambiguous result.
- **Failure Impact**: An unguarded route exposes health data to anonymous requests.

## Implementation Notes

- **Constraints**: Node.js runtime and server framework TO BE DECIDED. The guard MUST be part of the request pipeline, not an opt-in decorator, so that omission cannot silently open a route.
- **Prohibited Approaches**: Per-endpoint opt-in protection; deriving identity from a request header or body (DR-3); treating an unverified token payload as identity; catching and ignoring session-resolution errors.
- **Implementation Guidance**: `SECURITY.md` SEC-AUTHZ-5 requires a single enforcement point for authorization policy; this guard is the natural site for it once SQ-4 resolves, so structure it to accept a policy decision component later without moving the enforcement point.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-ABAC`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the guard and of every entry in the public-route declaration.
- **Open Decisions**: The ABAC attribute schema, policy language, and PDP/PEP architecture remain open (`SECURITY.md` SQ-4); this issue delivers the authentication gate only and does not resolve them.

**Estimated effort**: 0.5–1.5 engineer-days. **Estimated changed lines**: 200–450.
**Recommended model**: Claude Opus (`claude-opus-5`) — a small but security-critical control where fail-closed behavior and exhaustive route coverage are the acceptance bar.
