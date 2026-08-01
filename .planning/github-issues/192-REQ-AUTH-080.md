# [REQ-AUTH-080] Subscriber registration with email and password

## Metadata

- **ID**: REQ-AUTH-080
- **Title**: Subscriber registration with email and password
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-01
- **Priority**: Critical
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-2.2, FR-2.7; `SECURITY.md` SEC-AUTHN-5, SEC-AUTHN-6

## Requirement

- **Statement**: The system MUST allow a person to register an account with an email address and a password, MUST assign that account the `subscriber` role, and MUST refuse a password that fails the policy in SEC-AUTHN-6 or appears in the known-breached list.
- **Rationale**: FR-2.2 requires registration; FR-2.7 requires exactly one role per account, and registration is the only self-service path that creates one, so it is where the `subscriber` default is fixed. SEC-AUTHN-6 sets the policy — a length floor with no composition rules and no forced rotation, plus refusal of known-breached passwords — because `REF-63B` finds composition rules degrade real-world password choice while breach reuse is the dominant practical attack (threat TM-S-1).
- **Assumptions**: Credential storage is provided by REQ-AUTH-070 and is not reimplemented here. Privileged roles are never created by this path; they come only from REQ-AUTH-140.
- **Out of Scope**: Credential hashing itself (REQ-AUTH-070); email verification and the health-data gate (REQ-AUTH-090); authentication (REQ-AUTH-100); throttling (REQ-AUTH-060); `admin` and `consultant` creation (REQ-AUTH-140); subscription and entitlement, blocked by `REQUIREMENTS.md` OQ-1; the source and refresh cadence of the breached-password list, which is an operational decision noted under Open Decisions.
- **Design Traceability**: `DESIGN.md` — Components → Inputs (persistent visible labels, required fields marked in the label rather than by color) and Form feedback and errors (inline, field-associated, focus moved to the first invalid field). Password-policy failures are format problems and so may be specific; anything revealing whether the address is already registered may not be.
- **Architecture Traceability**: `ARCHITECTURE.md` — Identity and Session Handling (registration); REST API Application; Relational Persistence; trust boundary 2; data flow 1.
- **Security Traceability**: SEC-AUTHN-6 (policy), SEC-AUTHN-5 (storage, via REQ-AUTH-070), SEC-AUTHN-3 (no account enumeration), SEC-INPUT-1 (schema validation), SEC-INPUT-3 (role is not client-assignable).

## Scope

- **Applies To**: Server-Side Application, Web Client, API
- **Components**: Identity and Session Handling; Browser Client (the registration form); Relational Persistence
- **Interfaces / Operations**: Registration
- **Actors**: Anonymous visitor; an attacker probing for registered addresses or attempting mass account creation
- **Preconditions**: None
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Personal Data — email address and credential
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED (`SECURITY.md` SQ-1)

## Security Context

- **Security Objectives**: Confidentiality, Authenticity, Privacy
- **Control Layers**: Authentication, Input Validation, Business-Rule Validation
- **Threat References**: `SECURITY.md` TM-I-4 (account enumeration), TM-S-1 (credential attacks), TM-S-3 (registration with an address the registrant does not control), TM-E-1 (privilege escalation via provisioning); CWE-521 (weak password requirements), CWE-204 (observable response discrepancy), CWE-522 (insufficiently protected credentials)
- **Abuse / Misuse Case**: An attacker submits many addresses and learns which are registered from a differing response, error, or latency; or registers with a role field in the body hoping it is bound; or registers a weak or already-breached password that credential stuffing will find immediately.
- **Trust Boundary**: Boundary 1 and boundary 2 — everything in the request is untrusted, including any role or status field.
- **Untrusted Inputs or Assertions**: Email address, password, and every other field in the body. Role, account status, verification state, and creation time are server-controlled and MUST NOT be bindable (SEC-INPUT-3).
- **Authoritative Enforcement Point**: Identity and Session Handling; policy and role assignment are applied server-side regardless of anything the client sent or displayed.
- **Independent Verification**: Password policy is re-evaluated server-side even though the client validates for usability; the client's judgement is never trusted (DR-2).
- **Zero Trust Relevance**: N/A — account creation rather than a resource-access decision.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: N/A
- **Regulatory**: TO BE DECIDED — blocked by `SECURITY.md` SQ-1.
- **Other**: `REF-63B` for the password policy, `REF-AUTH`, and `REF-INPUT`, as named by SEC-AUTHN-6 and SEC-INPUT-1.
- **Mapping Basis**: SEC-AUTHN-6 adopts `REF-63B` explicitly; the CWE identifiers name the weak-requirements, response-discrepancy, and credential-protection classes.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a well-formed email address and a policy-conforming password, when registration is submitted, then an account is created with the `subscriber` role, the credential is stored per REQ-AUTH-070, and email verification is initiated (REQ-AUTH-090).
2. **AC-02 — Boundary or failure behavior**: Given a password shorter than the minimum or present in the known-breached list, when registration is submitted, then it is refused with a message naming the policy problem, and no account is created.
3. **AC-03 — Prohibited behavior**: Given a registration request containing a role, status, verification, entitlement, or identifier field, when it is processed, then those fields MUST be ignored, and the created account MUST always be `subscriber` (SEC-INPUT-3, TM-E-1).
4. **AC-04 — Additional criterion**: Given an address that is already registered, when registration is submitted, then the response is indistinguishable in content, status, and timing from one for a new address, and no account is duplicated (SEC-AUTHN-3, TM-I-4).
5. **AC-05 — Additional criterion**: Given the registration endpoint, when it is called repeatedly, then throttling applies per REQ-AUTH-060, so it cannot be used as an enumeration or mass-creation oracle.
6. **AC-06 — Additional criterion**: Given a successful registration, when it completes, then an audit entry records the action and time without recording the password (SEC-LOG-4, SEC-LOG-3).

## Failure Behavior

- **On Invalid Input**: Reject with field-level detail for format and policy problems only (`DESIGN.md` Form feedback); never disclose registration state.
- **On Authentication Failure**: N/A — no authentication precedes registration.
- **On Authorization Failure**: N/A
- **On Security-Decision Failure**: If the breached-password list cannot be consulted, refuse the registration rather than accept an unchecked password. A control that silently degrades is worse than a visible failure.
- **On External Dependency Failure**: The breached-password list is locally hosted (SEC-AUTHN-6) precisely so this is not a network dependency; an external lookup MUST NOT be introduced without a SEC-EXT-1 change.
- **On System Error**: Generic error with a correlation identifier (SEC-ERR-1); no partial account is left behind — creation is transactional.
- **Logging / Audit**: Audit entry per AC-06. The password MUST NOT appear in any log, error, or audit record (SEC-LOG-3, SEC-SECRET-1). The email address is personal data and is referenced rather than duplicated across logs.
- **Alerting**: TO BE DECIDED — mass-registration alerting is blocked by `SECURITY.md` SQ-3.

## Test Strategy

- **Unit Tests**: Policy evaluator accepts and rejects at the length boundary; breached-list lookup refuses a known-breached value; role assignment ignores any client-supplied value.
- **Integration Tests**: Successful registration creating exactly one `subscriber` account and initiating verification (AC-01); duplicate-address submission (AC-04); transactional rollback on a mid-operation error.
- **Security Tests**: Mass-assignment probe submitting role, status, and verification fields (AC-03); differential response and timing comparison between registered and unregistered addresses (AC-04); burst registration asserting throttling (AC-05); log assertion that a known test password never appears (AC-06).
- **Compliance Tests / Evidence**: The enumeration-resistance transcript, as evidence for SEC-AUTHN-3.
- **Acceptance-Criteria Traceability**: AC-01 — happy-path suite; AC-02 — policy boundary suite; AC-03 — mass-assignment suite; AC-04 — differential response and timing suite; AC-05 — burst test; AC-06 — audit and log assertions.
- **Coverage Target**: Every policy branch and every server-controlled field covered by a negative test.
- **Required Test Environment**: PostgreSQL with drizzle-kit migrations applied; a seeded registered address and a known-unregistered address; a breached-password fixture list; latency measurement for AC-04; Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-BUILD-010, REQ-AUTH-070, REQ-AUTH-060, REQ-API-010, REQ-API-020, REQ-AUTH-010, REQ-AUDIT-010
- **Downstream Requirements**: REQ-AUTH-090 (verification is initiated here), REQ-AUTH-100 (authentication consumes the credential), REQ-AUTH-130 (reset consumes the same credential path)
- **External Dependencies**: None — the breached-password list is locally hosted by design (SEC-AUTHN-6).
- **Dependency Assumptions**: The breached list is present and current enough to be meaningful; a stale or empty list silently weakens AC-02, so its provenance must be reviewable.
- **Failure Impact**: This is the only self-service path that creates an account. Without it there are no subscribers, and every subscriber-facing issue in the plan is unreachable in a running system.

## Implementation Notes

- **Constraints**: Node.js runtime with Fastify; PostgreSQL with Drizzle ORM; Vue single-page application (`CLAUDE.md`). Subscription entitlement is blocked by `REQUIREMENTS.md` OQ-1, so registration creates an account without any entitlement state; that gap must be stated when the issue is closed. The concrete length minimum is fixed by SEC-AUTHN-6 at 8 with 15 encouraged; anything beyond that is `TO BE DECIDED` under SQ-3.
- **Prohibited Approaches**: Composition rules, forced rotation, or password hints, all excluded by SEC-AUTHN-6. Any response, status code, or latency difference between registered and unregistered addresses. Binding role or status from the request body. Logging the submitted password, including inside a validation error. Accepting registration when the breach list is unavailable.
- **Implementation Guidance**: Fastify's route-level JSON Schema handles shape; the policy and breach checks are business-rule validation and belong behind it. Send the same response for a new and an existing address and initiate the verification email in both cases — for an existing account, an email saying "someone tried to register your address" preserves AC-04 while still being useful to the real owner.
- **AI Development Guidance**: `REF-63B`, `REF-AUTH`, `REF-INPUT`, `REF-PROMPT-API`, `REF-PROMPT-NODE`; `CLAUDE.md`.
- **Required Human Review**: Security review of the enumeration-resistance behavior, including timing, and of the mass-assignment allow-list.
- **Open Decisions**: The source, format, and refresh cadence of the locally hosted breached-password list is not specified by any source document and MUST be recorded as a decision rather than chosen silently. Whether registration is open to the public or gated by a subscription purchase depends on `REQUIREMENTS.md` OQ-1 and is not settled here.

**Estimated effort**: 1.5–2 engineer-days. **Estimated changed lines**: 400–850.
**Recommended model**: Claude Opus (`claude-opus-5`) — the functional path is simple, but enumeration resistance and mass-assignment defence are both easy to implement in a way that passes functional tests and fails the security requirement.
