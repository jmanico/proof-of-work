# [REQ-ENTITLE-010] Subscription entitlement gate on plan, customization, and progress access

## Metadata

- **ID**: REQ-ENTITLE-010
- **Title**: Subscription entitlement gate on plan, customization, and progress access
- **Version**: 1.0.1
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-3.1, FR-3.2, FR-3.6 (OQ-1 RESOLVED); `SECURITY.md` SEC-AUTHZ-8; threat TM-E-4

## Requirement

- **Statement**: The REST API Application MUST treat a subscriber's subscription as active exactly when the current time falls within a granted subscription period recorded on that account, MUST provide no other mechanism that activates a subscription, and MUST deny access to exercise plans, diet plans, plan customization, and progress tracking — at the central authorization enforcement point, with a subscription-required denial distinct from an authentication failure — whenever no period is active, with the denial taking effect immediately on lapse rather than at session-token expiry.
- **Rationale**: FR-3.1 requires an active subscription for plan, customization, and progress access; FR-3.2 requires denial for authenticated users without one, telling them a subscription is required; FR-3.6 fixes activation as "now falls within a granted period" and forbids any other activation mechanism, closing threat TM-E-4 (entitlement bypass via the activation mechanism). SEC-AUTHZ-8 requires the check to be server-side, distinct from authentication, and record-preserving; SEC-SESSION-4 requires subscription lapse to take effect without waiting for natural token expiry.
- **Assumptions**: An authenticated subject already exists (REQ-AUTHZ-010, REQ-SESSION-030); the subscription-active attribute is evaluated inside the central typed policy module (REQ-AUTHZ-050), whose attribute schema names `subscription-active` as a subject attribute sourced from persisted state (`SECURITY.md` SQ-4 RESOLVED).
- **Out of Scope**: How periods are granted, extended, or revoked by an admin (REQ-ENTITLE-030); the subscriber's own status view (REQ-ENTITLE-020); retention and restoration of records across lapse (REQ-ENTITLE-040); the paid consultant engagement, which is a separate entitlement recorded the same way (FR-11.5, REQ-CONSULT-030); self-serve payments, deferred by `REQUIREMENTS.md` OQ-18; consent and email-verification gates on health-data writes (FR-9.12, REQ-PRIVACY-010, REQ-AUTH-090), which are distinct conditions this gate does not replace.
- **Design Traceability**: `DESIGN.md`, Status, feedback, and loading — "Permission, lapsed-subscription, withdrawn-consent, and ended-engagement states explain the exact reason and the available next step without exposing hidden data"; banners are reserved for current actionable states such as lapsed subscription (FR-3.2). The structured denial reason this issue returns is what lets the client render that state (DR-2).
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application ("enforces … subscription entitlement (FR-3.1, FR-3.2) … before any data access"); `subscription` entity under Data model expectations; traceability row FR-3.1–FR-3.3, FR-3.5, FR-3.6; DR-2 (server-side rule, structured reason for the client); DR-3 (entitlement never from client claims).
- **Security Traceability**: SEC-AUTHZ-8; SEC-SESSION-4 (lapse effective without waiting for token expiry); SEC-INPUT-3 (subscription state is a server-controlled field, never client-assignable); evaluated at the SEC-AUTHZ-5 enforcement point with SEC-AUTHZ-6/SEC-AUTHZ-7 semantics (deny-overrides, missing attribute denies).

## Scope

- **Applies To**: Server-Side Application, API
- **Components**: REST API Application; Relational Persistence
- **Interfaces / Operations**: Every plan-browse, plan-view, plan-selection, customization, and progress-logging/viewing operation; the subscription-period record and the entitlement evaluation inside the authorization policy module
- **Actors**: `subscriber` (with and without an active period); an attacker attempting entitlement bypass (TM-E-4)
- **Preconditions**: The request carries a valid authenticated session (REQ-AUTHZ-010, REQ-SESSION-030)
- **Data Classification**: Confidential — subscription state is treated as sensitive (`SECURITY.md`, Sensitive or regulated data); the gated resources are Restricted health data (FR-9.12)
- **Personal or Regulated Data**: Personal Data — subscription records are personal data but not health data (FR-9.12); the gate protects access to Health Data
- **Jurisdiction / Regulatory Scope**: `SECURITY.md` SQ-1 RESOLVED — GDPR/UK GDPR for EU/UK data subjects; CCPA/CPRA, Washington My Health My Data, and the FTC Health Breach Notification Rule for US users; HIPAA not applicable. Specific sections: TO BE DECIDED (per-issue statute-section mapping remains verification work under SQ-1).

## Security Context

- **Security Objectives**: Authorization, Integrity, Accountability
- **Control Layers**: Authorization, Business-Rule Validation, Architecture
- **Threat References**: `SECURITY.md` TM-E-4 (subscription entitlement bypass via the activation mechanism), TM-E-2 (policy gap or fail-open evaluation); CWE-862 (missing authorization), CWE-863 (incorrect authorization)
- **Abuse / Misuse Case**: A subscriber whose period ended keeps using a still-valid session to read plans and log progress; or a request body, header, or client-side flag asserts an active subscription and a handler honors it; or an endpoint outside the enforcement point serves gated content without evaluating entitlement.
- **Trust Boundary**: Boundary 1 (Browser Client → REST API Application) — the client's opinion of subscription state is never authoritative.
- **Untrusted Inputs or Assertions**: Any client-supplied subscription, entitlement, or period assertion; token claims (the token carries a session identifier and no authorization state, SEC-SESSION-3).
- **Authoritative Enforcement Point**: The central authorization enforcement point (Fastify lifecycle hook, SEC-AUTHZ-5) evaluating the typed policy module (REQ-AUTHZ-050) with `subscription-active` computed from the persisted period records at request time.
- **Independent Verification**: The active state is derived on every request from persisted granted periods and the server clock — never cached in the token, never taken from the client (DR-3, SEC-SESSION-4).
- **Zero Trust Relevance**: NIST SP 800-207 — access is granted per request from current, server-held state. Exact tenet: TO BE DECIDED (not verified against the publication in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — mapping deferred to the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A — no AI-enabled component is involved.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — mapping deferred per SQ-10.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: The SQ-1 regime set applies because the gate controls access to health data (FR-9.12); specific articles/sections: TO BE DECIDED.
- **Other**: `REF-ASVS-5`, `REF-PROMPT-ABAC` as cited by SEC-AUTHZ-8.
- **Mapping Basis**: SEC-AUTHZ-8 names these references; the CWE identifiers name the missing/incorrect-authorization classes an entitlement bypass falls into.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a subscriber account with a granted period whose bounds contain the current time, when the subscriber requests plan browsing, plan viewing, plan selection, customization, or progress logging or viewing, then the entitlement check passes and the operation proceeds to its remaining authorization and validation checks.
2. **AC-02 — Boundary or failure behavior**: Given a subscriber whose only granted period ended before the current time (or starts after it), when they request any gated operation, then the request is denied with a structured subscription-required reason that is distinct from an authentication failure, no gated data is returned, and the response names the next step per `DESIGN.md` without exposing hidden data.
3. **AC-03 — Prohibited behavior**: Given any request carrying a body field, header, query parameter, or token claim asserting an active subscription, when the request is processed, then that assertion MUST NOT influence the entitlement decision (SEC-INPUT-3, DR-3); and no code path other than the persisted granted-period comparison MUST be able to yield an active state (FR-3.6).
4. **AC-04 — Lapse takes immediate effect**: Given a subscriber with an active session, when their period is revoked or expires, then their very next gated request is denied without waiting for session-token expiry (SEC-SESSION-4).
5. **AC-05 — Boundary instants**: Given a granted period, when a gated request arrives exactly at the period's start bound and exactly at its end bound, then activation is evaluated by the documented bound semantics ("the current time falls within a granted period", FR-3.6) consistently in both the policy module and any status computation, with the chosen inclusive/exclusive convention fixed as a named, tested rule.
6. **AC-06 — Fail closed**: Given the period records cannot be read or the subscription-active attribute cannot be resolved, when a gated request is evaluated, then the result is denial, not permit (SEC-AUTHZ-7).

## Failure Behavior

- **On Invalid Input**: N/A for the gate itself — entitlement is derived from persisted state, not request input; malformed requests are rejected upstream by schema validation (SEC-INPUT-1, REQ-API-010).
- **On Authentication Failure**: Handled upstream by REQ-AUTHZ-010 with the uniform unauthenticated response; this gate runs only for authenticated subjects and its denial is deliberately distinguishable from that response (FR-3.2).
- **On Authorization Failure**: Deny with the structured subscription-required reason; the existence of the gated resource types (the plan library, the subscriber's own records) is not concealed — FR-3.2 requires telling the user a subscription is required — but no gated content is disclosed.
- **On Security-Decision Failure**: Deny by default; a missing or unresolvable subscription attribute denies (SEC-AUTHZ-7).
- **On External Dependency Failure**: If Relational Persistence is unavailable, deny; the system MUST NOT fall back to a cached or token-carried entitlement.
- **On System Error**: Generic error with a correlation identifier (SEC-ERR-1); no gated data and no internal detail in the response.
- **Logging / Audit**: Entitlement denials are authorization denials and are logged with route, actor, reason class, and correlation identifier (SEC-LOG-4); no health values, credentials, or tokens in logs (SEC-LOG-3). Reads that pass the gate produce their health-data audit entries under FR-9.7 in the owning feature issues.
- **Alerting**: Threshold alerts on anomalous denial rates route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-3, SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: Policy-module tests over the `subscription-active` attribute: period containing now, ended, not yet started, no period at all, multiple periods (overlapping and disjoint), boundary instants at both bounds, and attribute-unresolvable → deny.
- **Integration Tests**: Full requests per gated operation class (plan browse/view, selection, customization, progress log write, progress view) as an entitled and an unentitled subscriber; revocation mid-session followed by an immediate request (AC-04); records intact behind the denial (with REQ-ENTITLE-040).
- **Security Tests**: Entitlement-bypass suite — request-body and header assertions of active subscription ignored (AC-03); tampered token claims have no effect; store-failure fault injection asserting denial (AC-06); enumeration that every gated route traverses the enforcement point (SEC-AUTHZ-5).
- **Compliance Tests / Evidence**: The route-inventory result showing gated operations behind the enforcement point; denial-log samples showing SEC-LOG-4 fields.
- **Acceptance-Criteria Traceability**: AC-01/AC-02 — per-operation integration suite; AC-03 — mass-assignment and claim-tampering suite; AC-04 — mid-session revocation test; AC-05 — bound-instant unit tests; AC-06 — fault-injection test.
- **Coverage Target**: Project coverage threshold is 90% line and branch (`CLAUDE.md`, 2026-08-03); every entitlement decision branch and error path MUST have positive and negative tests.
- **Required Test Environment**: PostgreSQL with drizzle-kit migrations applied; subscriber identities with controllable period fixtures; a controllable clock; Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-AUTHZ-010, REQ-SESSION-030, REQ-AUTHZ-050
- **Downstream Requirements**: REQ-ENTITLE-020, REQ-ENTITLE-030, REQ-ENTITLE-040; every plan, selection, customization, food, and progress feature issue that serves gated content (REQ-PLAN-*, REQ-SELECT-*, REQ-CUSTOM-*, REQ-PROGRESS-*, REQ-FOOD-*)
- **External Dependencies**: None — payment collection is out of band in v1 (OQ-1, OQ-18).
- **Dependency Assumptions**: The policy module (REQ-AUTHZ-050) sources the subscription attribute from persisted state per the SQ-4 attribute schema and applies deny-overrides with missing-attribute denial.
- **Failure Impact**: Without this gate, the paid-subscription business model and FR-3.1/FR-3.2 are unenforced, and TM-E-4 becomes a standing bypass; a fail-open branch here exposes plan content and health-data operations to unentitled accounts.

## Implementation Notes

- **Constraints**: Node.js with Fastify; PostgreSQL with Drizzle ORM (`CLAUDE.md`). Entitlement is evaluated at the single SEC-AUTHZ-5 enforcement point, not per handler. The period-bound comparison convention is a named constant/rule shared by the policy module and the status view (REQ-ENTITLE-020) so the two can never disagree.
- **Prohibited Approaches**: Any activation mechanism other than the granted-period comparison (FR-3.6); caching entitlement in the token or client (SEC-SESSION-3, SEC-SESSION-4); accepting subscription state from a request body (SEC-INPUT-3); per-endpoint bespoke entitlement checks (SEC-AUTHZ-5); deleting or hiding records on lapse instead of denying access (SEC-AUTHZ-8, REQ-ENTITLE-040).
- **Implementation Guidance**: Model the subscription as zero-or-more granted period rows per account (start, end, grant metadata) so FR-3.5 grant/extend/revoke (REQ-ENTITLE-030) is row manipulation, and "active" is a pure function of (rows, now) — trivially unit-testable per `REF-PROMPT-QUALITY` and the SQ-4 pure-function policy style. The OQ-18 payments seam is exactly this record: processor events would create and revoke periods without changing this gate.
- **AI Development Guidance**: `REF-PROMPT-ABAC`, `REF-PROMPT-API`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the entitlement decision path, the bound-comparison rule, and the route inventory confirming no gated operation bypasses the enforcement point.
- **Open Decisions**: None — OQ-1, SQ-3, and SQ-4 are resolved; self-serve payments remain deferred (OQ-18) and do not block this issue.

**Estimated effort**: 1.5–2 engineer-days. **Estimated changed lines**: 400–800.
**Recommended model**: Claude Opus (`claude-opus-5`) — a security-enforcing gate where a fail-open branch, a client-honored assertion, or a bound-comparison mismatch silently defeats the product's entitlement model.
