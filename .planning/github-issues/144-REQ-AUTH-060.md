# [REQ-AUTH-060] Anti-automation throttling on authentication and recovery paths

## Metadata

- **ID**: REQ-AUTH-060
- **Title**: Anti-automation throttling on authentication and recovery paths
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-01
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `SECURITY.md` SEC-AUTHN-6, SEC-AUTHN-11; threats TM-S-1, TM-D-2

## Requirement

- **Statement**: Every authentication, MFA, passkey, verification, and recovery endpoint MUST apply exponential backoff throttling keyed on both the targeted account and the request source, and the system MUST NOT implement fixed account lockout — no sequence of failed attempts by a third party may render an account permanently unusable.
- **Rationale**: SEC-AUTHN-6 requires anti-automation proportionate to credential-attack risk (threat TM-S-1, credential stuffing). The prohibition on lockout is equally load-bearing: TM-D-2 records that lockout itself is an attack, letting anyone deny a chosen victim access by failing logins on their behalf. Backoff resolves both — it makes guessing economically infeasible while keeping a legitimate user's account recoverable by waiting.
- **Assumptions**: The account identifier presented in a failed attempt may be attacker-chosen, so keying on it alone would let an attacker throttle a victim; keying on source alone would let a distributed attacker bypass throttling. Both keys are required.
- **Out of Scope**: General request rate limiting and body-size limits (SEC-HTTP-5, blocked by `SECURITY.md` SQ-3 and not delivered here); the concrete thresholds, delays, and decay curve, which are `TO BE DECIDED` under SQ-3; the individual endpoints themselves, which are separate issues that consume this control; alerting on attack patterns, blocked by SQ-3.
- **Design Traceability**: `DESIGN.md` — Components → Form feedback and errors. A throttled response must tell the user what to do (wait and retry) without revealing whether the account exists or which factor failed, which sits in tension with the "name the specific problem" guidance and is resolved the same way REQ-AUTH-040 resolves it: format problems are specific, state-dependent outcomes are generic.
- **Architecture Traceability**: `ARCHITECTURE.md` — Identity and Session Handling; trust boundary 2; data flow 1.
- **Security Traceability**: SEC-AUTHN-6; supports SEC-AUTHN-3 (no enumeration through timing or content), SEC-AUTHN-11 (recovery secret use is throttled), SEC-LOG-4 (failures logged with enough context to detect patterns).

## Scope

- **Applies To**: Server-Side Application, API
- **Components**: Identity and Session Handling
- **Interfaces / Operations**: Login, MFA challenge, passkey assertion, email verification and resend, password reset request and completion, recovery-code redemption, privileged enrolment token redemption
- **Actors**: Anonymous attacker performing credential stuffing or password spraying; an attacker attempting to lock out a chosen victim; legitimate users who mistype
- **Preconditions**: None — this applies to unauthenticated paths by definition
- **Data Classification**: Confidential
- **Personal or Regulated Data**: Personal Data — throttling state is keyed on account identifiers
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED (`SECURITY.md` SQ-1)

## Security Context

- **Security Objectives**: Availability, Authenticity, Confidentiality
- **Control Layers**: Authentication, Availability
- **Threat References**: `SECURITY.md` TM-S-1 (credential stuffing and password spraying), TM-D-2 (targeted victim lockout), TM-I-4 (account enumeration), TM-S-2 (recovery abuse); CWE-307 (improper restriction of excessive authentication attempts), CWE-645 (overly restrictive account lockout)
- **Abuse / Misuse Case**: An attacker sprays one common password across many accounts, staying under any per-account limit; or a distributed attacker rotates source addresses to defeat source-only throttling; or an attacker deliberately fails logins against a known victim to lock them out of their own health data.
- **Trust Boundary**: Boundary 2 — unauthenticated to authenticated.
- **Untrusted Inputs or Assertions**: The submitted account identifier, every credential value, and any client-supplied indication of source or identity. Throttling state MUST NOT be keyed on a value the client can freely vary alone.
- **Authoritative Enforcement Point**: Identity and Session Handling, applied before any credential is verified, so that throttling cannot be bypassed by an expensive-verification path.
- **Independent Verification**: Counters are server-side; nothing about prior attempts is read from the request.
- **Zero Trust Relevance**: N/A — availability and abuse prevention rather than a resource-access decision.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: N/A
- **Regulatory**: TO BE DECIDED — blocked by `SECURITY.md` SQ-1.
- **Other**: `REF-AUTH`, `REF-63B`, `REF-PROMPT-API`, `REF-ASVS-5`, as named by SEC-AUTHN-6.
- **Mapping Basis**: SEC-AUTHN-6 names these references; CWE-307 and CWE-645 name the two opposing failure modes this control must satisfy simultaneously.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given repeated failed attempts against one account, when the attempt count rises, then the enforced delay increases and eventually the response is throttled, without the account becoming permanently unusable.
2. **AC-02 — Boundary or failure behavior**: Given a throttled state, when the configured decay period elapses without further failures, then the legitimate account holder can authenticate successfully again with correct credentials.
3. **AC-03 — Prohibited behavior**: Given any number of failed attempts by a third party against a chosen account, when they cease, then the account MUST NOT remain locked, disabled, or require administrative intervention to restore (TM-D-2).
4. **AC-04 — Additional criterion**: Given attempts distributed across many source addresses against one account, and attempts from one source across many accounts, when either pattern occurs, then throttling applies — proving both keys are enforced independently.
5. **AC-05 — Additional criterion**: Given a throttled response, when it is compared with a response for an unknown account, then they are indistinguishable in content, status, and timing (SEC-AUTHN-3, TM-I-4).
6. **AC-06 — Additional criterion**: Given an endpoint inventory, when it is enumerated, then every authentication, MFA, passkey, verification, and recovery endpoint applies the control, so a newly added credential path cannot omit it.

## Failure Behavior

- **On Invalid Input**: Throttling is evaluated before input validity, so a malformed attempt still counts — otherwise malformed requests become a free probing channel.
- **On Authentication Failure**: Increment the counters and return the uniform failure response; the response MUST NOT reveal remaining attempts or the delay's basis.
- **On Authorization Failure**: N/A
- **On Security-Decision Failure**: If throttling state cannot be read or written, deny the attempt. Failing open here would remove the control precisely when the store is under attack-induced load.
- **On External Dependency Failure**: If the state store is unavailable, deny rather than allow unlimited attempts.
- **On System Error**: Generic error with a correlation identifier (SEC-ERR-1).
- **Logging / Audit**: Log each failure with account reference, cause class, source, and correlation identifier, with enough context to detect attack patterns and without any credential value (SEC-LOG-4, SEC-LOG-3).
- **Alerting**: TO BE DECIDED — this control produces the signal an alert would consume, but thresholds and destinations are blocked by `SECURITY.md` SQ-3 and SQ-11.

## Test Strategy

- **Unit Tests**: Backoff function increases delay monotonically and decays over time; counters key independently on account and source; a state-store error yields denial.
- **Integration Tests**: Burst against one account asserting throttled responses (AC-01); recovery after decay (AC-02); distributed and spray patterns (AC-04).
- **Security Tests**: Sustained third-party failures against a victim account followed by a legitimate login, asserting success (AC-03) — this is the regression test for TM-D-2; differential response and timing comparison against an unknown account (AC-05); endpoint inventory assertion (AC-06).
- **Compliance Tests / Evidence**: The endpoint-inventory result and the TM-D-2 regression transcript, as evidence for SEC-AUTHN-6.
- **Acceptance-Criteria Traceability**: AC-01 — burst suite; AC-02 — decay test; AC-03 — lockout-prohibition regression; AC-04 — key-independence matrix; AC-05 — differential timing suite; AC-06 — endpoint inventory.
- **Coverage Target**: Every named endpoint covered by the inventory assertion; both throttling keys covered by an independent test.
- **Required Test Environment**: A registered identity and a known-unregistered identifier; a source-address simulator for distributed patterns; a controllable clock so decay is testable without waiting; latency measurement for AC-05; Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-BUILD-010, REQ-API-040, REQ-AUTH-040
- **Downstream Requirements**: REQ-AUTH-070, REQ-AUTH-080, REQ-AUTH-090, REQ-AUTH-100, REQ-AUTH-110, REQ-AUTH-120, REQ-AUTH-130, REQ-AUTH-140, REQ-AUTH-150, REQ-AUTH-020, REQ-AUTH-030 — every credential-bearing path
- **External Dependencies**: None
- **Dependency Assumptions**: The throttling state store is shared across application instances; per-instance counters would be defeated by load balancing, which makes this a deployment concern interacting with SQ-7.
- **Failure Impact**: Without it, every credential endpoint is an unlimited oracle, and SEC-AUTHN-3's non-enumeration guarantee is unenforceable in practice regardless of how uniform individual responses are.

## Implementation Notes

- **Constraints**: Node.js runtime with Fastify (`CLAUDE.md`). Thresholds, delays, and decay are `TO BE DECIDED` (`SECURITY.md` SQ-3) and MUST be named constants with a documented basis rather than literals — this issue delivers the mechanism and its enforcement surface, not the numbers. The multi-instance assumption above should be confirmed when SQ-7 resolves.
- **Prohibited Approaches**: Fixed account lockout in any form, including "temporary lockout requiring admin unlock" (TM-D-2). Keying solely on source address, which a distributed attacker defeats, or solely on account, which turns the control into the lockout it replaces. Applying throttling after credential verification, which leaves the expensive Argon2id path (REQ-AUTH-070) exposed as a denial-of-service amplifier. Revealing remaining attempts or delay in the response.
- **Implementation Guidance**: Evaluate throttling in a hook that runs before route handlers, consistent with the single-enforcement-point reasoning in SEC-AUTHZ-5. Because Argon2id is deliberately expensive, throttling should short-circuit before hashing; be careful that doing so does not itself become a timing oracle for account existence, which AC-05 tests.
- **AI Development Guidance**: `REF-AUTH`, `REF-63B`, `REF-PROMPT-API`, `REF-PROMPT-NODE`; `CLAUDE.md`.
- **Required Human Review**: Security review of the endpoint inventory and of the interaction between early short-circuit and the timing-uniformity requirement.
- **Open Decisions**: Thresholds, delay curve, and decay period (`SECURITY.md` SQ-3). Whether the state store is shared infrastructure depends on deployment topology (SQ-7). SEC-HTTP-5's general rate limiting remains blocked and is not delivered here.

**Estimated effort**: 1–2 engineer-days. **Estimated changed lines**: 300–650.
**Recommended model**: Claude Opus (`claude-opus-5`) — the control must satisfy two directly opposing requirements at once, and the naive implementation of either one violates the other.
