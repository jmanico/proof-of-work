# [REQ-AUTH-180] Email-address change

## Metadata

- **ID**: REQ-AUTH-180
- **Title**: Email-address change
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-2.18 (added 2026-08-03: FR-9.5's correction right extends to the registered address); `SECURITY.md` SEC-AUTHN-7, SEC-AUTHN-8, SEC-AUTHN-12, SEC-EXT-3

## Requirement

- **Statement**: The system MUST allow a user to change their registered email address only through a flow that requires fresh re-authentication, verifies control of the new address before it replaces the old — the old address remaining authoritative for recovery until verification completes — sends notice of the change to the previous address, terminates all existing sessions on completion, produces an audit entry, and is refused while an FR-9.11 deletion request is pending.
- **Rationale**: The registered address is both personal data subject to the FR-9.5 correction right and the account's recovery anchor (FR-2.12, FR-9.11); an attacker with a hijacked session who could swap it unverified would capture recovery and silence the pending-deletion safety notices, so the change is gated by re-authentication (SEC-AUTHN-7), verify-new-first (SEC-AUTHN-8), notice-to-old (SEC-EXT-3), and session termination (SEC-AUTHN-12).
- **Assumptions**: Email verification machinery — single-use token, 24-hour lifetime, rate-limited resend, non-enumerating behavior — exists (SEC-AUTHN-8; REQ-AUTH-090). The re-authentication step with its 5-minute freshness window exists (SEC-AUTHN-7). Recovery flows that consume the verified address exist (FR-2.12; REQ-AUTH-130). The pending-deletion state exists (FR-9.11; REQ-PRIVACY-110).
- **Out of Scope**: Initial registration-time verification (REQ-AUTH-090); password reset and other recovery flows that rely on the verified address (REQ-AUTH-130); the out-of-band deletion channel itself (REQ-PRIVACY-110); the mail-interface infrastructure (REQ-INFRA-060); privileged-account recovery, which never uses email possession (SEC-AUTHN-10; REQ-AUTH-150).
- **Design Traceability**: `DESIGN.md` — Credentials, account security, and administration → Email change (FR-2.18): "Lives in Account → Security. The screen states the sequence before starting: the new address is verified first, the old address remains active for recovery until then, a notice goes to the old address, and all sessions end on completion. Starting requires re-authentication. While an out-of-band deletion request is pending, the action is unavailable and says why." Re-authentication prompt pattern with the 5-minute freshness window.
- **Architecture Traceability**: `ARCHITECTURE.md` — Identity and Session Handling (credential material, session state, email-verification token); REST API Application; mail channel through the internal mail interface (SEC-EXT-3); trust boundary 2.
- **Security Traceability**: SEC-AUTHN-7 (fresh re-authentication, 5-minute freshness, audit), SEC-AUTHN-8 (replacement address verified before active; token discipline; non-enumeration per SEC-AUTHN-3), SEC-AUTHN-12 (sessions terminated on completion), SEC-EXT-3 (notice to the old address; no secrets beyond the single-use token), SEC-AUTHN-11 (machine-held verification-token storage), SEC-LOG-4.

## Scope

- **Applies To**: Multiple — API, Server-Side Application, Web Client
- **Components**: REST API Application; Identity and Session Handling; Relational Persistence; Browser Client (Account → Security)
- **Interfaces / Operations**: Change initiation (re-authenticated); verification-token issue and redemption for the new address; notice dispatch to the old address; completion with session termination; refusal while a pending deletion request exists
- **Actors**: Subscriber, consultant, and admin account holders (FR-2.18 says "a user"); attacker holding a hijacked session or a compromised new address (adversary)
- **Preconditions**: An authenticated session; fresh re-authentication no older than 5 minutes at execution (SEC-AUTHN-7); no pending FR-9.11 deletion request
- **Data Classification**: Confidential — the address is personal data and the recovery anchor
- **Personal or Regulated Data**: Personal Data
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED). The flow implements the FR-9.5 correction right for the registered address; section-level mappings TO BE DECIDED.

## Security Context

- **Security Objectives**: Authenticity, Confidentiality, Integrity, Accountability
- **Control Layers**: Authentication, Session Management, Business-Rule Validation, Input Validation, Logging and Monitoring
- **Threat References**: `SECURITY.md` TM-S-2 (recovery-flow abuse), TM-S-3 (address the registrant does not control bound to the account), TM-S-5 (stolen-session abuse); SEC-EXT-3 addendum threats — enumeration through send behavior, token leakage through forwarded or compromised mailboxes; CWE-620 (unverified password change, as the analogous unverified-credential-change class), CWE-613 (insufficient session expiration)
- **Abuse / Misuse Case**: An attacker on a hijacked session swaps the address to their own mailbox to capture password reset and FR-9.11 notices — defeated by fresh re-authentication (including the second factor where enrolled), verify-new-first, notice-to-old, and completion-time session termination. An attacker uses the change endpoint to probe which addresses are registered. A mailbox thief with a pending deletion request re-points the address to suppress cancellation notices — refused while the request is pending.
- **Trust Boundary**: Boundary 2 — the change alters the account's recovery anchor; boundary 1 — the requested new address and the redeemed token arrive as untrusted client input.
- **Untrusted Inputs or Assertions**: The proposed new address; the presented verification token; the session's claim of recency — freshness is checked server-side against the re-authentication record, not a client assertion.
- **Authoritative Enforcement Point**: Identity and Session Handling with the REST API Application — re-authentication freshness, verification state, the pending-deletion refusal, and the atomic address swap are all evaluated server-side.
- **Independent Verification**: Control of the new address is proven by single-use token redemption (SEC-AUTHN-8), never by user assertion; the pending-deletion check reads persisted state (DR-3).
- **Zero Trust Relevance**: TO BE DECIDED — not verified against NIST SP 800-207 in this session.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — per-issue mapping awaits the SQ-10 independent pre-launch assessment.
- **OWASP AISVS 1.0**: N/A — no AI-enabled component is involved.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: GDPR Art. 16-adjacent correction right per FR-9.5 as extended by FR-2.18 — the spec states the correction-right linkage but no article number for it; article-level mapping TO BE DECIDED (SQ-1 counsel review). The SQ-1 regime set applies to the personal data involved.
- **Other**: `REF-AUTH`, `REF-63B`, `REF-SESSION` as cited by SEC-AUTHN-7, SEC-AUTHN-8, and SEC-AUTHN-12.
- **Mapping Basis**: The governing SEC rules name these references; article-level regulatory precision is deferred rather than guessed.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated user with re-authentication no older than 5 minutes and no pending deletion request, when they request a change to a new address and redeem the single-use verification token sent to that address within its 24-hour lifetime, then the new address becomes the registered address, notice of the change is sent to the previous address through the mail interface, all existing sessions for the account are terminated, and an audit entry records the change.
2. **AC-02 — Old address authoritative until verification**: Given a change initiated but the new address not yet verified, when any recovery flow (FR-2.12) or FR-9.11 request runs, then it uses the old address only; the unverified new address grants nothing, and an expired or replaced verification token leaves the old address in place.
3. **AC-03 — Boundary or failure behavior**: Given a change attempt with no re-authentication, re-authentication older than 5 minutes, or — for an MFA-enabled account — re-authentication lacking the second factor (SEC-AUTHN-7), when initiation or execution is attempted, then it is refused with no token issued and no state change.
4. **AC-04 — Pending-deletion refusal**: Given an account with a pending FR-9.11 deletion request, when an email change is attempted, then it is refused, the Browser Client shows the action unavailable with the reason (DESIGN.md), and the refusal is logged.
5. **AC-05 — Prohibited behavior**: Given any step of the flow, when it executes, then the system MUST NOT activate the new address without token redemption, MUST NOT skip the notice to the previous address, MUST NOT leave pre-change sessions alive after completion (SEC-AUTHN-12), and neither token issuance nor any response may reveal whether the new address is already registered (SEC-AUTHN-3, SEC-AUTHN-8).

## Failure Behavior

- **On Invalid Input**: A malformed address or token is rejected against the allow-list schema (SEC-INPUT-1) with a non-descriptive error; no token issued, no address changed.
- **On Authentication Failure**: Missing or stale re-authentication refuses the operation without revealing which factor failed (SEC-AUTHN-3); the deny-by-default gate (REQ-AUTHZ-010) covers unauthenticated requests.
- **On Authorization Failure**: The change applies only to the actor's own account; a request naming another account's address record is denied by owner scoping (SEC-AUTHZ-2).
- **On Security-Decision Failure**: Deny by default — errors in freshness evaluation, verification-state lookup, or the pending-deletion check refuse the change; the swap, notice trigger, and session termination commit atomically or not at all.
- **On External Dependency Failure**: If the mail interface (SEC-EXT-3) cannot send the verification token, the change simply never completes (old address stays authoritative); if the notice to the old address cannot be sent at completion, the failure is retried and surfaced operationally — the notice is part of the flow, not best-effort.
- **On System Error**: Generic error with a correlation identifier (SEC-ERR-1); a partial failure never yields an active-but-unverified address or a completed change without session termination.
- **Logging / Audit**: Audit entries for initiation, token issue/use/failed use (SEC-AUTHN-11), completion, and every refusal — including pending-deletion refusals — with acting account, action, and time; token values and addresses never in logs beyond what SEC-LOG-3 permits; no health values.
- **Alerting**: Threshold alerts on repeated failed re-authentication or token redemption route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-3, SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: Freshness-window evaluation at the 5-minute boundary; state machine (current address, proposed address, verified swap; expired/replaced token leaves state unchanged); pending-deletion refusal predicate; recovery-anchor selection while a change is in flight.
- **Integration Tests**: Full happy path asserting swap, notice to old address, session termination, and audit entry; recovery flow during an in-flight change using the old address only; change refused while an FR-9.11 request is pending (with REQ-PRIVACY-110); resend rate limits per SEC-AUTHN-8 (≤1/min, ≤5/h).
- **Security Tests**: Captured pre-change session replayed after completion, asserting denial (SEC-AUTHN-12); token replay after redemption or replacement, asserting refusal; differential response and timing tests on token issuance for registered versus unregistered new addresses (SEC-AUTHN-3); MFA-enabled account attempting the change with first factor only, asserting refusal (SEC-AUTHN-7); mass-assignment attempt writing the address field directly, asserting it is ignored (SEC-INPUT-3).
- **Compliance Tests / Evidence**: Audit trail of a completed change as FR-9.5 correction-right evidence.
- **Acceptance-Criteria Traceability**: AC-01 — happy-path integration suite; AC-02 — in-flight recovery-anchor tests; AC-03 — freshness and factor unit/security tests; AC-04 — pending-deletion integration test; AC-05 — replay, notice, session-termination, and enumeration security suite.
- **Coverage Target**: Positive and negative coverage of every gate (re-auth, verification, pending-deletion) and every token state transition (project threshold TO BE DECIDED, `CLAUDE.md`).
- **Required Test Environment**: Vitest and HTTP test client; mail-interface test double capturing token and notice sends; controllable clock for freshness and token lifetime; fixtures for MFA-enabled, MFA-disabled, and pending-deletion accounts.

## Dependencies

- **Upstream Requirements**: REQ-AUTH-090 (verification-token machinery and verified-address state), REQ-AUTH-130 (recovery flows whose anchor this change moves), REQ-PRIVACY-110 (pending-deletion state that refuses this change)
- **Downstream Requirements**: None
- **External Dependencies**: Amazon SES, in-account, reached only through the internal mail interface (SEC-EXT-3)
- **Dependency Assumptions**: Verification tokens follow SEC-AUTHN-11's machine-held-token discipline (HMAC-SHA-256 under a Secrets Manager key, single-use, invalidated on replacement); session termination is enforceable via server-side session records (SEC-SESSION-3).
- **Failure Impact**: If verification machinery is unavailable, changes cannot complete and the old address remains authoritative — fail-safe; a defect that activates an unverified address hands the recovery anchor to an attacker-controlled mailbox.

## Implementation Notes

- **Constraints**: Node.js with Fastify; the 5-minute freshness value and the 24-hour token lifetime are named constants (SEC-AUTHN-7, SEC-AUTHN-8; SQ-3); the swap, audit write, and session termination are one transaction boundary.
- **Prohibited Approaches**: Activating the new address on request rather than on token redemption; treating the change as complete while pre-change sessions survive (SEC-AUTHN-12); sending the notice to the new address instead of the previous one; skipping re-authentication because the session is recent — session recency is not re-authentication (SEC-AUTHN-7); revealing address registration through issuance behavior (SEC-AUTHN-3, SEC-EXT-3); accepting the address as a writable field on a general profile-update endpoint (SEC-INPUT-3).
- **Implementation Guidance**: Model the in-flight change as a proposed-address record beside the current address, consumed on redemption — this keeps AC-02's anchor rule a data-shape property rather than conditional logic. Reuse REQ-AUTH-090's token issuance/redemption path with the FR-2.18 replacement-address case SEC-AUTHN-8 now names, rather than a parallel implementation. The Account → Security screen states the four-step sequence up front per DESIGN.md.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-NODE`, `REF-PROMPT-QUALITY`; `CLAUDE.md` working rules.
- **Required Human Review**: Security review of the anchor-selection rule, freshness enforcement, and non-enumeration behavior; privacy review of the notice content (SEC-EXT-3 — no secrets beyond the single-use token, no health data).
- **Open Decisions**: None — FR-2.18 and the governing SEC rules fix the behavior; per-issue standards mappings await the SQ-10 assessment.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 400–750.
**Recommended model**: Claude Opus (`claude-opus-5`) — the flow moves the account's recovery anchor; a verify-order or session-termination defect converts a hijacked session into permanent account takeover.
