# [REQ-AUTH-040] Non-disclosing authentication failure responses

## Metadata

- **ID**: REQ-AUTH-040
- **Title**: Non-disclosing authentication failure responses
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: High
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-2.3 (second clause); `SECURITY.md` SEC-AUTHN-3; threat TM-I-4

## Requirement

- **Statement**: Authentication failures MUST NOT reveal which factor was incorrect, and MUST NOT allow account existence to be inferred from response content, status, or timing, across login, registration, and recovery surfaces.
- **Rationale**: FR-2.3 requires rejecting invalid credentials without revealing which factor was wrong; SEC-AUTHN-3 extends that to account enumeration by content, status, or timing. The threat model records enumeration (TM-I-4) as a privacy concern because an account's existence in a health application is itself disclosure.
- **Assumptions**: The credential paths that produce these failures exist. Subscriber password login and registration are blocked by `REQUIREMENTS.md` OQ-8 and OQ-15; privileged passkey authentication exists (REQ-AUTH-020). This issue defines the response contract that every such path must satisfy, including those added later.
- **Out of Scope**: The credential verification itself; anti-automation thresholds and lockout (SEC-AUTHN-6, blocked by `SECURITY.md` SQ-3); the registration and recovery flows themselves, which are blocked.
- **Design Traceability**: `DESIGN.md` — Components → Form feedback and errors. Note the deliberate exception: `DESIGN.md` requires messages that name the specific problem, but for authentication the security rule takes precedence and a generic message is required; specificity remains for field-format validation of the submitted form.
- **Architecture Traceability**: `ARCHITECTURE.md` — Identity and Session Handling ("Rejects invalid credentials without revealing which factor failed (FR-2.3)"); trust boundary 2.
- **Security Traceability**: SEC-AUTHN-3; supports SEC-ERR-1, SEC-LOG-4, SEC-AUTHN-6.

## Scope

- **Applies To**: Server-Side Application, Web Client, API
- **Components**: Identity and Session Handling; Browser Client presentation
- **Interfaces / Operations**: Every authentication, registration, and recovery response that can differ by account existence or by which factor failed
- **Actors**: Anonymous attacker; all account holders
- **Preconditions**: An authentication-related request has been submitted
- **Data Classification**: Confidential — the existence of an account is itself personal data in a health context
- **Personal or Regulated Data**: Personal Data
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED (`SECURITY.md` SQ-1)

## Security Context

- **Security Objectives**: Confidentiality, Privacy
- **Control Layers**: Authentication, Output Encoding, Logging and Monitoring
- **Threat References**: `SECURITY.md` TM-I-4 (account-existence enumeration via response content or timing), TM-S-1 (credential stuffing, which enumeration makes cheaper); CWE-204 (observable response discrepancy), CWE-203 (observable discrepancy), CWE-208 (observable timing discrepancy)
- **Abuse / Misuse Case**: An attacker submits candidate email addresses and distinguishes registered from unregistered accounts by message text, status code, response shape, or response latency — learning who uses a fitness and health service.
- **Trust Boundary**: Boundary 2 — unauthenticated → authenticated.
- **Untrusted Inputs or Assertions**: Submitted identifiers and credentials, including probes designed to elicit differential responses.
- **Authoritative Enforcement Point**: Identity and Session Handling, through a single failure-response path shared by all credential surfaces.
- **Independent Verification**: The response is produced by the shared path regardless of which internal check failed.
- **Zero Trust Relevance**: N/A — this governs disclosure in a failure response, not a resource-access decision.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: N/A
- **Regulatory**: TO BE DECIDED — blocked by `SECURITY.md` SQ-1; account existence is personal data under the regimes REQUIREMENTS.md names.
- **Other**: `REF-AUTH` as cited by SEC-AUTHN-3; `REF-63B`.
- **Mapping Basis**: SEC-AUTHN-3 cites `REF-AUTH` and `REF-ASVS-5`; the CWE identifiers name the observable-discrepancy classes, including the timing variant the rule explicitly covers.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a failed authentication for any reason, when the response is produced, then it carries a single generic message and status that is identical across all failure causes.
2. **AC-02 — Boundary or failure behavior**: Given an authentication attempt against a registered account with a wrong credential, and one against an unregistered identifier, when both are made, then the two responses are identical in status, body, and headers, and their latency distributions are not separable by an observer.
3. **AC-03 — Prohibited behavior**: Given any authentication, registration, or recovery response, when it is inspected, then it MUST NOT indicate which factor failed, MUST NOT state or imply that an account does or does not exist, and MUST NOT vary its shape by account state.
4. **AC-04 — Additional criterion**: Given any of these failures, when it is logged, then the server-side record retains the specific cause and the account identifier where known, so that detection is unaffected by the client-facing uniformity (SEC-LOG-4).

## Failure Behavior

- **On Invalid Input**: Field-format problems in the submitted form (for example a malformed email) may be reported specifically, because they disclose nothing about account state; anything dependent on account state uses the generic path.
- **On Authentication Failure**: The behavior this issue defines — uniform generic response, uniform status, uniform timing characteristics.
- **On Authorization Failure**: N/A
- **On Security-Decision Failure**: If the failure cause cannot be classified, emit the generic response — never a more specific one.
- **On External Dependency Failure**: If credential storage is unavailable, return the generic authentication failure rather than a distinguishable system error that reveals the account lookup succeeded.
- **On System Error**: Generic error with a correlation identifier (SEC-ERR-1), indistinguishable from other failures in what it reveals about account state.
- **Logging / Audit**: Log every authentication failure with cause class, submitted-identifier hash or account identifier where resolvable, source context, and correlation identifier. MUST NOT log the submitted credential (SEC-LOG-3).
- **Alerting**: TO BE DECIDED — blocked by `SECURITY.md` SQ-3; enumeration detection depends on thresholds that are undefined.

## Test Strategy

- **Unit Tests**: Failure formatter returns an identical response object for every cause class; classification errors fall through to the generic response.
- **Integration Tests**: Paired requests — registered versus unregistered identifier, correct versus incorrect credential — asserting byte-identical responses apart from any correlation identifier.
- **Security Tests**: Differential response test across all authentication-related endpoints; timing test comparing latency distributions for registered and unregistered identifiers with a documented tolerance; assertion that a registration attempt on an existing address does not reveal the collision.
- **Compliance Tests / Evidence**: Retained differential and timing test output as evidence for FR-2.3 and SEC-AUTHN-3.
- **Acceptance-Criteria Traceability**: AC-01 and AC-03 — response-uniformity suite; AC-02 — paired differential and timing tests; AC-04 — log-content assertion.
- **Coverage Target**: Every authentication-related endpoint and every failure cause class exercised.
- **Required Test Environment**: A registered identity and a known-unregistered identifier; latency measurement harness. Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-API-040, REQ-AUTH-020
- **Downstream Requirements**: REQ-AUTH-050; every credential path added once `REQUIREMENTS.md` OQ-8, OQ-9, and OQ-15 resolve
- **External Dependencies**: None
- **Dependency Assumptions**: Credential verification returns a classified failure to the shared formatter rather than shaping its own response.
- **Failure Impact**: Enumeration turns a fitness application's user list into a health-related disclosure and makes credential stuffing far cheaper.

## Implementation Notes

- **Constraints**: Node.js runtime with Fastify (`CLAUDE.md`). Registration and recovery flows are blocked, so this issue delivers the contract and applies it to the paths that exist today; it MUST be re-applied to each new credential path.
- **Prohibited Approaches**: "User not found" versus "incorrect password"; different status codes by account state; short-circuiting credential verification when the account does not exist, which creates the timing signal AC-02 forbids; revealing a collision during registration.
- **Implementation Guidance**: Perform equivalent work on the unregistered path — including a dummy verification of comparable cost — so timing does not separate the cases. The same uniformity helper serves REQ-AUTHZ-040's denial responses.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-NODE`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of every authentication-related response path; privacy review of what the client displays.
- **Open Decisions**: `REQUIREMENTS.md` OQ-8 (recovery flows, lockout) and OQ-15 (email verification) are blocked; both add response paths that must satisfy this contract. `SECURITY.md` TM-D-2 warns that lockout design must not permit third-party-triggered permanent lockout, which interacts with this contract and remains undecided.

**Estimated effort**: 0.5–1.5 engineer-days. **Estimated changed lines**: 150–350.
**Recommended model**: Claude Opus (`claude-opus-5`) — subtle disclosure and timing behavior where a naive implementation passes functional tests and still leaks.
