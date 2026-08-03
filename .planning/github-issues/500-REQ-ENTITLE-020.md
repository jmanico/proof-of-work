# [REQ-ENTITLE-020] View own subscription status

## Metadata

- **ID**: REQ-ENTITLE-020
- **Title**: View own subscription status
- **Version**: 1.0.1
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-3.3; `SECURITY.md` SEC-DATA-5

## Requirement

- **Statement**: The system MUST allow an authenticated user to view the current subscription status of their own account — whether a subscription is active now, and the bounds of the granted periods on the account — and MUST NOT expose any other account's subscription state through this operation.
- **Rationale**: FR-3.3 requires that a user can view their current subscription status. Because payment is out of band in v1 (OQ-1), this view is the subscriber's only way to see when access starts and ends, and D-PRINCIPLE-4 puts subscription state "in plain sight." SEC-DATA-5 bounds the response to the fields the operation requires and forbids bulk retrieval of other subscribers' data.
- **Assumptions**: The subscription-period record and the active-state computation exist (REQ-ENTITLE-010); the status shown is computed by the same bound-comparison rule the entitlement gate uses, so the view and the gate never disagree.
- **Out of Scope**: Granting, extending, or revoking periods (REQ-ENTITLE-030); the entitlement denial itself and its lapsed-state messaging (REQ-ENTITLE-010); admin views of a subscriber's subscription periods, which are FR-10.3 administrative account fields (REQ-AUTHZ-060); payment or renewal representation — payment collection is not represented in v1 (`DESIGN.md`, Information Architecture; FR-3.5, FR-3.6).
- **Design Traceability**: `DESIGN.md` — Account groups "Profile, Units and appearance, Security, Subscription, Consultant access, and Privacy" (Account, privacy, and destructive actions); D-PRINCIPLE-4; Status, feedback, and loading — status chips use factual vocabulary (`Active`, `Inactive`); banners are reserved for current actionable states such as lapsed subscription.
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application (owns the `subscription` entity); Browser Client (renders the status); traceability row FR-3.1–FR-3.3, FR-3.5, FR-3.6; DR-2 (the server supplies the status; the client only presents it).
- **Security Traceability**: SEC-DATA-5 (least-privilege response shape); SEC-AUTHZ-1/SEC-AUTHZ-2 (own-account scoping); supports FR-3.2's requirement that users are told a subscription is required.

## Scope

- **Applies To**: API, Web Client
- **Components**: REST API Application; Relational Persistence; Browser Client
- **Interfaces / Operations**: The subscription-status read operation; the Account → Subscription view
- **Actors**: `subscriber` (primary); `consultant` and `admin` MAY read their own account's status through the same own-account operation — FR-3.3 says "a user"
- **Preconditions**: Valid authenticated session (REQ-AUTHZ-010); the account exists
- **Data Classification**: Confidential — subscription state is treated as sensitive (`SECURITY.md`, Sensitive or regulated data)
- **Personal or Regulated Data**: Personal Data — subscription records are personal data but not health data (FR-9.12)
- **Jurisdiction / Regulatory Scope**: `SECURITY.md` SQ-1 RESOLVED — GDPR/UK GDPR for EU/UK data subjects; CCPA/CPRA, Washington My Health My Data, and the FTC Health Breach Notification Rule for US users; HIPAA not applicable. Specific sections: TO BE DECIDED.

## Security Context

- **Security Objectives**: Confidentiality, Authorization
- **Control Layers**: Authorization, Data Protection
- **Threat References**: `SECURITY.md` TM-I-1 (BOLA/IDOR), TM-I-3 (excessive data exposure / bulk retrieval); CWE-639 (authorization bypass through user-controlled key), CWE-213 (exposure of sensitive information)
- **Abuse / Misuse Case**: An authenticated user substitutes another account's identifier to read that account's subscription state, or the response over-returns grant metadata (such as the acting admin's identity) beyond what the subscriber's view requires.
- **Trust Boundary**: Boundary 1 (Browser Client → REST API Application).
- **Untrusted Inputs or Assertions**: Any client-supplied account identifier — the operation resolves the account from the authenticated session, never from a request parameter.
- **Authoritative Enforcement Point**: REST API Application — own-account scoping at the SEC-AUTHZ-5 enforcement point; response shape bound by a declared serialization schema (SEC-DATA-5; Fastify response serialization per `CLAUDE.md`).
- **Independent Verification**: The account is taken from the session record (REQ-SESSION-030), and the status is computed from persisted periods on the server (DR-3).
- **Zero Trust Relevance**: N/A — this is an own-resource read governed by the generic per-request authorization already required; no distinct SP 800-207 principle applies beyond REQ-AUTHZ-010/REQ-ENTITLE-010.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — mapping deferred to the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A — no AI-enabled component is involved.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — mapping deferred per SQ-10.
- **NIST SP 800-207**: N/A — see Zero Trust Relevance.
- **Regulatory**: The SQ-1 regime set applies to the personal data in the response; this view also supports the transparency interest behind FR-9.5. Specific articles/sections: TO BE DECIDED.
- **Other**: `REF-API-2023`, `REF-PROMPT-API` as cited by SEC-DATA-5.
- **Mapping Basis**: SEC-DATA-5 names these references; the CWE identifiers name the IDOR and over-exposure classes this operation must avoid.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated subscriber with granted periods on their account, when they request their subscription status, then the response states whether the subscription is active now and the bounds of the account's granted periods, computed by the same rule as the REQ-ENTITLE-010 gate.
2. **AC-02 — Boundary or failure behavior**: Given an authenticated user with no granted period, or whose latest period has ended, when they request their status, then the response states the inactive status factually (`DESIGN.md` status vocabulary) with no error, and the operation succeeds even though gated content operations would be denied — the status view itself is not subscription-gated, or FR-3.2's "tell them a subscription is required" could never be satisfied from Account.
3. **AC-03 — Prohibited behavior**: Given any authenticated user, when they attempt to read subscription status for a different account — by identifier substitution or any parameter — then the operation MUST NOT return another account's state; the account is resolved from the session only.
4. **AC-04 — Least-privilege response**: Given the status response, when its shape is asserted in tests, then it contains only the fields this view requires — active state and period bounds — and no unrelated account, grant-audit, or health fields (SEC-DATA-5).

## Failure Behavior

- **On Invalid Input**: The operation takes no client-supplied target identifier; unexpected parameters are rejected by schema validation (SEC-INPUT-1) with no state change.
- **On Authentication Failure**: Uniform unauthenticated denial (REQ-AUTHZ-010, SEC-AUTHN-3).
- **On Authorization Failure**: N/A beyond authentication — the operation is own-account by construction; a cross-account attempt is impossible to express through the interface, and any attempt via crafted parameters fails schema validation.
- **On Security-Decision Failure**: Deny by default (SEC-AUTHZ-7).
- **On External Dependency Failure**: If Relational Persistence is unavailable, return the generic error (SEC-ERR-1); never a fabricated or cached status.
- **On System Error**: Generic message with correlation identifier (SEC-ERR-1); no internal detail.
- **Logging / Audit**: Standard request logging; no FR-9.7 audit entry — subscription records are personal data but not health data (FR-9.12), so this read is not a health-data access. No subscription details in logs beyond what SEC-LOG-4 needs for denials.
- **Alerting**: N/A — a read-only own-account status view has no alert condition of its own; anomalous access patterns are covered by the platform-level SEC-LOG-4 monitoring routed to the security lead (SQ-11).

## Test Strategy

- **Unit Tests**: Status computation over period fixtures — active, ended, future-start, none, multiple periods — asserting agreement with the REQ-ENTITLE-010 bound rule (shared function or property test).
- **Integration Tests**: Authenticated status read returning correct state and bounds; status readable while gated operations are denied (AC-02); response-shape assertion against the declared serialization schema (AC-04).
- **Security Tests**: IDOR attempt with a foreign account identifier in every parameter position, asserting no cross-account disclosure (AC-03); unauthenticated request denied; response over-exposure review.
- **Compliance Tests / Evidence**: N/A — no distinct compliance evidence beyond the tests above.
- **Acceptance-Criteria Traceability**: AC-01 — status integration suite; AC-02 — lapsed/none fixtures suite; AC-03 — IDOR suite; AC-04 — response-shape assertion.
- **Coverage Target**: Project coverage threshold is 90% line and branch (`CLAUDE.md`, 2026-08-03); all status-computation branches and the IDOR negative path MUST be covered.
- **Required Test Environment**: PostgreSQL with migrations applied; two subscriber identities with distinct period fixtures; a controllable clock; Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-ENTITLE-010
- **Downstream Requirements**: None — the Account UI consumes this operation, but no drafted requirement depends on it.
- **External Dependencies**: None
- **Dependency Assumptions**: REQ-ENTITLE-010 exposes its bound-comparison rule as a reusable function so this view cannot drift from the gate.
- **Failure Impact**: Without this view, a subscriber cannot see why access lapsed or when it ends, undermining FR-3.2's messaging and D-PRINCIPLE-4; a scoping defect here leaks another user's subscription state.

## Implementation Notes

- **Constraints**: Node.js with Fastify; response serialization against a declared schema is the SEC-DATA-5 delivery mechanism (`CLAUDE.md`). The Browser Client presents the state in Account → Subscription using the `DESIGN.md` status vocabulary.
- **Prohibited Approaches**: Accepting an account identifier from the client for this operation; computing "active" with logic separate from the REQ-ENTITLE-010 rule; returning grant-audit metadata (acting admin identity) in the subscriber view; gating the status view itself behind an active subscription.
- **Implementation Guidance**: Reuse the pure active-state function from REQ-ENTITLE-010 with the same named bound convention. Keep the response to active state plus period bounds; the admin-facing period management view is a different operation with different authorization (REQ-ENTITLE-030, REQ-AUTHZ-060).
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Review of the response schema for over-exposure and of the own-account scoping.
- **Open Decisions**: None.

**Estimated effort**: 0.5–1 engineer-days. **Estimated changed lines**: 100–250.
**Recommended model**: Claude Opus (`claude-opus-5`) — small surface, but the own-account scoping and response-shape discipline are the point of the issue.
