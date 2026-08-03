# [REQ-PRIVACY-110] Out-of-band deletion channel

## Metadata

- **ID**: REQ-PRIVACY-110
- **Title**: Out-of-band deletion channel
- **Version**: 1.1.1
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Privacy
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-9.11 (OQ-17 RESOLVED); `SECURITY.md` SEC-AUTHN-10, SEC-EXT-3

## Requirement

- **Statement**: The system MUST accept an account-deletion request submitted out of band from the account's verified email address when the requester cannot authenticate, and MUST execute it only after verifying control of that address by a single-use, time-limited confirmation token (SEC-AUTHN-11 discipline) matched to a registered account — that proof and match are the entire identity check (FR-9.11, fixed 2026-08-03) — recording a pending deletion request, sending notice to the address and surfacing it in any live session, and waiting 14 days during which any authenticated session can cancel — after which FR-9.4 deletion executes; the channel MUST NOT restore access, satisfy or bypass any authentication factor, or disclose any account or health data.
- **Rationale**: FR-2.14's deliberate unrecoverability trade would otherwise extinguish the deletion right for a locked-out user; FR-9.11 preserves that right without weakening authentication (SEC-AUTHN-10), and the 14-day cancellable window prevents a mailbox thief from silently destroying an active user's data (OQ-17).
- **Assumptions**: The account's email address was verified before it became usable in this channel (FR-2.11, SEC-AUTHN-8; REQ-AUTH-090). FR-9.4 synchronous deletion, including the deletion-ledger write, is delivered by REQ-PRIVACY-090. The mail interface exists (SEC-EXT-3; REQ-INFRA-060).
- **Out of Scope**: The deletion execution mechanics themselves — hard delete, tombstoning, deletion ledger, backup horizon (REQ-PRIVACY-090, REQ-PRIVACY-100); in-session authenticated deletion with re-authentication (FR-9.4, SEC-AUTHN-7; REQ-PRIVACY-090); refusal of email change while a request is pending (FR-2.18; REQ-AUTH-180); the scheduled-execution infrastructure shared with archival and ledger maintenance beyond this channel's expiry evaluation.
- **Design Traceability**: `DESIGN.md` — Status, feedback, and loading (`Pending deletion` chip); Medical disclaimer and health-data consent section's banner rule: "The pending-deletion banner names the scheduled execution date and offers **Cancel deletion request** from any authenticated session (FR-9.11)."
- **Architecture Traceability**: `ARCHITECTURE.md` — Primary data flow 9 (*Out-of-band deletion*); pending deletion request entity (Data model expectations); scheduled executions of the REST API Application (FR-9.11 window expiry); Identity and Session Handling (address-control verification); DR-5.
- **Security Traceability**: SEC-AUTHN-10 (no access restoration, no factor bypass), SEC-AUTHN-3 (no enumeration or disclosure), SEC-AUTHN-6 (anti-automation on the unauthenticated endpoint), SEC-EXT-3 (notices through the in-account mail interface), SEC-DATA-4 (execution mechanics), SEC-LOG-4 (logged refusals).

## Scope

- **Applies To**: Multiple — API, Server-Side Application, Web Client
- **Components**: REST API Application (including its scheduled executions); Identity and Session Handling; Relational Persistence; Browser Client (banner and cancel action)
- **Interfaces / Operations**: Unauthenticated deletion-request submission; address-control verification; pending-request creation; notice dispatch through the mail interface; pending-state surfacing in live sessions; authenticated cancellation; scheduled window-expiry evaluation; refusal of unauthenticated export and correction requests
- **Actors**: Locked-out account holder (subscriber, consultant, or admin); mailbox thief (adversary); authenticated account holder (cancellation); anonymous internet attacker
- **Preconditions**: The account exists and its email address is verified; the requester cannot authenticate
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED). FR-9.11 records reliance on GDPR Art. 12(6) for refusing unauthenticated export and correction.

## Security Context

- **Security Objectives**: Privacy, Confidentiality, Authenticity, Accountability
- **Control Layers**: Authentication, Business-Rule Validation, Data Protection, Logging and Monitoring
- **Threat References**: `SECURITY.md` TM-S-2 (recovery flow used to bypass MFA or passkey), TM-I-4 (account-existence enumeration), TM-P-1 (identifiability and the deletion right); SEC-EXT-3's 2026-08-03 addendum threats — enumeration through send behavior and single-use-token leakage through forwarded or compromised mailboxes; CWE-640 (weak password recovery mechanism, as the analogous out-of-band-channel abuse class)
- **Abuse / Misuse Case**: A mailbox thief submits a deletion request to destroy a victim's data — defeated by notice to the address, surfacing in live sessions, and the 14-day cancellable window. An attacker probes the endpoint to enumerate registered addresses, or attempts to convert the channel into account recovery, data disclosure, or an MFA bypass.
- **Trust Boundary**: Boundary 1 (the submission is unauthenticated client input); boundary 2 (the channel deliberately operates below full authentication and must never cross into it).
- **Untrusted Inputs or Assertions**: The submitted email address and account details; the address-control verification token presented back; any claim that the requester is the account holder.
- **Authoritative Enforcement Point**: REST API Application with Identity and Session Handling — address-control verification, account-detail matching, the 14-day clock, and the refusal of export and correction all evaluated server-side; window expiry evaluated by a scheduled execution of the REST API Application.
- **Independent Verification**: Control of the address is proven by round-trip of a single-use verification step to the registered address, never by the requester's assertion; account details are matched against persisted state (DR-3).
- **Zero Trust Relevance**: TO BE DECIDED — not verified against NIST SP 800-207 in this session.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — per-issue mapping awaits the SQ-10 independent pre-launch assessment.
- **OWASP AISVS 1.0**: N/A — no AI-enabled component is involved.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: GDPR Art. 17 right to erasure exercised through this channel — exact article-level mapping beyond what the spec states: TO BE DECIDED (SQ-1 counsel review). GDPR Art. 12(6) is cited by FR-9.11 as the identity-verification allowance under which unauthenticated export (FR-9.3) and correction (FR-9.5) requests are refused; that position is recorded for confirmation at the SQ-1 pre-launch counsel review. CCPA/CPRA and Washington My Health My Data deletion rights apply for US users; section-level mappings TO BE DECIDED.
- **Other**: `REF-AUTH`, `REF-63B` (recovery-channel discipline as cited by SEC-AUTHN-10).
- **Mapping Basis**: FR-9.11 itself names GDPR Art. 12(6); the remaining mappings are deferred per SQ-10 and SQ-1 rather than guessed.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an account with a verified email address, when a deletion request is submitted out of band, control of that address is verified, and the submitted account details match, then a pending deletion request is recorded with a scheduled execution date 14 days out, notice of the pending deletion is sent to the address through the mail interface, and every live session for that account surfaces the pending state.
2. **AC-02 — Cancellation**: Given a pending deletion request within its 14-day window, when any authenticated session for that account cancels it, then the request is cancelled, no deletion occurs, and the cancellation produces an audit entry.
3. **AC-03 — Execution at expiry**: Given a pending deletion request whose 14-day window has elapsed uncancelled, when the scheduled execution evaluates the window, then FR-9.4 deletion executes (REQ-PRIVACY-090) with the FR-9.4 completion clock running from that execution, and an audit entry is written.
4. **AC-04 — Boundary or failure behavior**: Given a submission naming an unregistered address, an address that fails control verification, or account details that do not match, when it is processed, then no pending request is created and the response is indistinguishable from the success-path response — account existence is not inferable from response content, status, or timing (SEC-AUTHN-3) — and the failed attempt is logged.
5. **AC-05 — Prohibited behavior**: Given any state of this channel, when its steps complete or fail, then it MUST NOT issue a session, restore access, satisfy, disable, or bypass any authentication factor (FR-2.13, SEC-AUTHN-10), and MUST NOT disclose any account or health data in any response or notice.
6. **AC-06 — Refused adjacent rights**: Given an out-of-band export (FR-9.3) or correction (FR-9.5) request without full authentication, when it is received, then it is refused under GDPR Art. 12(6) and the refusal is logged.

## Failure Behavior

- **On Invalid Input**: Malformed submissions are rejected against the allow-list schema (SEC-INPUT-1) with a non-descriptive error; no pending request or side effect beyond throttle counters and the attempt log.
- **On Authentication Failure**: N/A by design for submission — the channel exists for requesters who cannot authenticate; cancellation, by contrast, requires an authenticated session and is refused without one.
- **On Authorization Failure**: A cancellation attempt from a session not belonging to the affected account is denied without revealing whether a pending request exists.
- **On Security-Decision Failure**: Deny by default — any error in address-control verification, detail matching, or window evaluation leaves the account undeleted and unmodified; verification errors never create a pending request, and evaluation errors never execute deletion early.
- **On External Dependency Failure**: If the mail interface (SEC-EXT-3) fails, the pending request is not considered noticed — the notice MUST be delivered before the window may expire into execution; send failures are retried through the mail interface and surfaced operationally without revealing address validity externally.
- **On System Error**: Errors return a generic message with a correlation identifier (SEC-ERR-1); a partially processed submission never leaves an unnoticed pending request.
- **Logging / Audit**: Audit entries for request creation, notice dispatch, cancellation, expiry execution, and every refusal — including refused unauthenticated export and correction requests (FR-9.11 "every refusal is logged") — with acting context, action, affected account (tombstoned after deletion per FR-9.10), and time; no health values, no tokens (SEC-LOG-3, SEC-LOG-4).
- **Alerting**: Threshold alerts on submission bursts and repeated failed verification route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-3, SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: Window arithmetic (14 days, expiry boundary at exactly 14 days); detail-matching rules; state machine (pending → cancelled, pending → executed, no other transitions); refusal logic for unauthenticated export and correction.
- **Integration Tests**: Full flow — submit, verify address control, observe pending record and notice, cancel from an authenticated session; full flow uncancelled through scheduled-execution expiry into REQ-PRIVACY-090 deletion; pending state surfaced in a live session; FR-2.18 email change refused while pending (with REQ-AUTH-180).
- **Security Tests**: Differential response and timing tests across registered and unregistered addresses (SEC-AUTHN-3); replay of a consumed verification step; attempts to convert the channel into a session, factor reset, or data disclosure, asserting refusal; burst tests asserting SEC-AUTHN-6 throttling; assertion that no notice or response contains account or health data.
- **Compliance Tests / Evidence**: Logged-refusal evidence for Art. 12(6) refusals; the audit trail for a full request→execution cycle, retained for the SQ-1 counsel review.
- **Acceptance-Criteria Traceability**: AC-01/AC-02/AC-03 — integration flow suite; AC-04 — enumeration and mismatch suite; AC-05 — channel-abuse security suite; AC-06 — refusal unit and log-assertion tests.
- **Coverage Target**: Positive and negative coverage of every state transition and every refusal path (project threshold 90% line and branch, `CLAUDE.md`, 2026-08-03).
- **Required Test Environment**: Vitest and HTTP test client; a mail-interface test double capturing sends; controllable clock for window expiry; fixtures for verified and unverified accounts with and without live sessions.

## Dependencies

- **Upstream Requirements**: REQ-PRIVACY-090 (FR-9.4 deletion execution and ledger write), REQ-INFRA-060 (SEC-EXT-3 mail interface), REQ-AUTH-090 (verified-address state and verification-token discipline)
- **Downstream Requirements**: REQ-AUTH-180 (email change refused while a request is pending)
- **External Dependencies**: Amazon SES, in-account, reached only through the internal mail interface (SEC-EXT-3)
- **Dependency Assumptions**: The mail interface delivers notices without revealing address registration in its send behavior (SEC-EXT-3); the scheduled execution runs as the REST API Application with the same database identity (`ARCHITECTURE.md`, DR-5).
- **Failure Impact**: If deletion execution (REQ-PRIVACY-090) is unavailable at expiry, the request remains pending and execution retries — the window may lengthen, never shorten; if mail delivery fails, no window may expire into execution unnoticed.

## Implementation Notes

- **Constraints**: Node.js with Fastify; expiry evaluation is a scheduled execution of the REST API Application, not a new component (`ARCHITECTURE.md`, 2026-08-03); the FR-9.4 completion clock (35 days including backup expiry) runs from execution after the window, not from submission.
- **Prohibited Approaches**: Issuing any credential, session, or factor reset from this channel (SEC-AUTHN-10); returning different responses for registered versus unregistered addresses (SEC-AUTHN-3); executing deletion before the 14-day window elapses or without notice delivered; including account details or health data in notices (SEC-EXT-3); allowing cancellation from anything but an authenticated session of the affected account; identity-proofing escalation via any external service (OQ-17 — contradicts the self-contained posture).
- **Implementation Guidance**: Model the pending deletion request as the `ARCHITECTURE.md` entity with states pending, cancelled, executed. Follow the SEC-AUTHN-8/SEC-AUTHN-11 machine-held-token discipline for the address-control verification step: cryptographically secure generation (SEC-SECRET-4), single-use, short-lived, stored hashed, throttled under SEC-AUTHN-6. The DESIGN.md banner needs the scheduled execution date in the pending-state payload returned to live sessions.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-NODE`, `REF-PROMPT-QUALITY`; `CLAUDE.md` working rules.
- **Required Human Review**: Security review of the non-enumeration and no-bypass properties; privacy review of notice content and the Art. 12(6) refusal position (flagged for the SQ-1 counsel review).
- **Open Decisions**: None — FR-9.11, SEC-EXT-3, and the scheduled-execution model fix the behavior; per-issue standards mappings await the SQ-10 assessment.

**Estimated effort**: 1.5–2 engineer-days. **Estimated changed lines**: 500–900.
**Recommended model**: Claude Opus (`claude-opus-5`) — an unauthenticated channel touching account destruction, where an enumeration leak, a factor bypass, or a premature execution is a severe privacy failure.
