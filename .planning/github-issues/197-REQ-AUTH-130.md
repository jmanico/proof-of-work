# [REQ-AUTH-130] Password reset

## Metadata

- **ID**: REQ-AUTH-130
- **Title**: Password reset
- **Version**: 1.1.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-2.12; `SECURITY.md` SEC-AUTHN-10, SEC-AUTHN-11, SEC-AUTHN-12

## Requirement

- **Statement**: The system MUST allow a `subscriber` to reset a forgotten password using a single-use, time-limited token sent to their verified email address; completing the reset MUST NOT satisfy, disable, or bypass an enabled second factor, and MUST terminate all existing sessions for that account.
- **Rationale**: Recovery is the path deliberately built to work when the user has lost the thing that proves who they are, which makes it the natural target — threat TM-S-2 rates it high. Two properties keep it from becoming the weakest link. First, SEC-AUTHN-10: a reset restores one factor and no more, so an attacker with mailbox access still cannot pass MFA. Second, SEC-AUTHN-12: the reset evicts every existing session, so a user resetting a password they believe is compromised actually removes the attacker instead of running alongside them.
- **Assumptions**: The address is verified (REQ-AUTH-090); SEC-AUTHN-8 forbids relying on an unverified address in recovery. Session revocation is provided by REQ-SESSION-040.
- **Out of Scope**: MFA recovery when the second factor itself is lost, which is recovery-code redemption in REQ-AUTH-120; privileged account recovery, which is REQ-AUTH-150 and never uses this path; password change by an already-authenticated user, which is a distinct operation not drafted here; throttling (REQ-AUTH-060); the transactional-email channel itself (SEC-EXT-3; REQ-INFRA-060).
- **Design Traceability**: `DESIGN.md` — Product Patterns → Credentials, account security, and administration (Password reset: the request screen always confirms neutrally with "If that address is registered, we sent a message."; the completion screen enforces the 8-minimum/15-encouraged policy with no composition demands, refuses a breached password inline, and states that an enabled second factor still applies and that other sessions end); Core Components → Forms and validation. Policy failures on the new password are format problems and may be specific; whether an address is registered may never be revealed.
- **Architecture Traceability**: `ARCHITECTURE.md` — Identity and Session Handling; Relational Persistence (the reset-token entity); trust boundary 2; data flow 1.
- **Security Traceability**: SEC-AUTHN-10 (recovery may not weaken authentication), SEC-AUTHN-11 (recovery secret handling), SEC-AUTHN-12 (credential change revokes sessions), SEC-AUTHN-3 (no enumeration), SEC-AUTHN-6 (policy applies to the new password), SEC-AUTHN-8 (verified address only).

## Scope

- **Applies To**: Server-Side Application, Web Client, API
- **Components**: Identity and Session Handling; Browser Client; Relational Persistence
- **Interfaces / Operations**: Reset request; reset completion
- **Actors**: `subscriber`; an attacker with mailbox access; an attacker probing for registered addresses
- **Preconditions**: An account exists with a verified email address
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Personal Data — email address, reset token, credential
- **Jurisdiction / Regulatory Scope**: Global service with GDPR as the design ceiling (`SECURITY.md` SQ-1 RESOLVED): GDPR/UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable

## Security Context

- **Security Objectives**: Authenticity, Confidentiality, Accountability
- **Control Layers**: Authentication, Session Management, Business-Rule Validation
- **Threat References**: `SECURITY.md` TM-S-2 (recovery used to bypass MFA), TM-I-4 (enumeration through the request step), TM-S-5 (a surviving session after reset); CWE-640 (weak password recovery mechanism), CWE-620 (unverified password change), CWE-204 (observable response discrepancy)
- **Abuse / Misuse Case**: An attacker who has compromised a mailbox resets the password and expects a session — and is stopped by the MFA requirement. Or an attacker who already holds a live session watches the victim reset their password and retains access because only the credential changed. Or an attacker uses the request step to enumerate registered addresses.
- **Trust Boundary**: Boundary 2 — the reset token is the only thing standing between an anonymous request and a credential change.
- **Untrusted Inputs or Assertions**: The address in the request step, the token, and the new password. The account the reset applies to comes from the token's server-side binding, never from a field in the completion request.
- **Authoritative Enforcement Point**: Identity and Session Handling; the token-to-account binding and the MFA requirement are both server-side.
- **Independent Verification**: The new password is re-evaluated against policy server-side; the MFA requirement is read from persisted enrolment, not from anything carried through the reset flow.
- **Zero Trust Relevance**: NIST SP 800-207 — possession of one credential does not grant full access. Exact tenet: TO BE DECIDED.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — mapping verified only at the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — mapping verified only at the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: GDPR/UK GDPR (EU/UK data subjects); CCPA/CPRA, Washington My Health My Data, FTC Health Breach Notification Rule (US users); HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED). Statute-section precision: TO BE DECIDED pending the SQ-1 pre-launch counsel review.
- **Other**: `REF-AUTH`, `REF-63B`, `REF-SECRETS`, `REF-ASVS-5`, as named by SEC-AUTHN-10 and SEC-AUTHN-11.
- **Mapping Basis**: The cited rules name these references; CWE-640 and CWE-620 name the recovery-mechanism and unverified-change classes precisely.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a verified address and a valid, unexpired reset token, when a policy-conforming new password is submitted, then the credential is replaced, the token is invalidated, and all existing sessions for the account are terminated.
2. **AC-02 — Boundary or failure behavior**: Given a token that is expired, already used, forged, or bound to a different account, when reset completion is attempted, then it is refused, the credential is unchanged, and no session is issued.
3. **AC-03 — Prohibited behavior**: Given an account with MFA enabled, when a reset completes, then the actor MUST still satisfy the second factor to obtain a session — the reset MUST NOT issue a session, disable MFA, consume a recovery code, or mark the second factor satisfied (SEC-AUTHN-10, TM-S-2).
4. **AC-04 — Additional criterion**: Given a reset request, when it is submitted for a registered address and for an unregistered one, then the responses are indistinguishable in content, status, and timing, and both are throttled per REQ-AUTH-060 (SEC-AUTHN-3, TM-I-4).
5. **AC-05 — Additional criterion**: Given a session captured before a reset, when it is replayed afterwards, then it is refused (SEC-AUTHN-12) — this is the property that makes reset an effective response to suspected compromise.
6. **AC-06 — Additional criterion**: Given an account whose address is not verified, when a reset is requested, then the address MUST NOT be used to deliver a token (SEC-AUTHN-8).
7. **AC-07 — Additional criterion**: Given a completed reset, when it finishes, then an audit entry records the account, the action, and the time, without the token or either password (SEC-LOG-4, SEC-LOG-3).

## Failure Behavior

- **On Invalid Input**: A new password failing policy is refused with a specific format-level message, and the token remains valid so the user may retry without requesting a new one.
- **On Authentication Failure**: The reset flow is unauthenticated by design; the token is the only credential, which is why SEC-AUTHN-11's single-use and expiry properties are mandatory.
- **On Authorization Failure**: A token bound to another account is refused as invalid, with no distinguishing detail.
- **On Security-Decision Failure**: If MFA enrolment state cannot be read at completion, refuse the reset rather than complete it — completing while unable to confirm the MFA requirement risks exactly the bypass AC-03 forbids.
- **On External Dependency Failure**: If mail delivery fails, the request step still reports the same uniform outcome (AC-04); the system MUST NOT reveal delivery failure, since that discloses address existence.
- **On System Error**: Generic error with a correlation identifier (SEC-ERR-1); credential replacement and session revocation are transactional together, so a partial reset cannot leave sessions alive against a changed password.
- **Logging / Audit**: Per AC-07. The token MUST NOT appear in logs or in any URL that is logged (SEC-LOG-3, SEC-AUTHN-11).
- **Alerting**: Threshold alerts route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-3, SQ-11 RESOLVED); reset issuance and use are SEC-AUTHN-11 audited events and account-takeover signals feeding that channel.

## Test Strategy

- **Unit Tests**: Token generator uses the secure generator; the store holds only the HMAC-SHA-256 digest under the dedicated secret-store key, never the plaintext token (SEC-AUTHN-11); verifier rejects expired, consumed, forged, and cross-account tokens; policy evaluation runs on the new password; an unreadable MFA state yields refusal.
- **Integration Tests**: Full reset on an account without MFA (AC-01); reset on an account with MFA, asserting the challenge is still required (AC-03); session revocation and replay refusal (AC-05); unverified address producing no delivery (AC-06).
- **Security Tests**: Token replay, expiry, and cross-account suites (AC-02); differential response and timing on the request step (AC-04); log assertion that no token or password value appears; burst test asserting throttling.
- **Compliance Tests / Evidence**: The MFA-still-required transcript, as the primary evidence closing threat TM-S-2 for this path.
- **Acceptance-Criteria Traceability**: AC-01 — reset suite; AC-02 — token negative suite; AC-03 — MFA-preservation suite; AC-04 — differential suite; AC-05 — session replay test; AC-06 — unverified-address test; AC-07 — audit assertions.
- **Coverage Target**: Every token failure mode, plus MFA enabled and disabled, plus verified and unverified address.
- **Required Test Environment**: PostgreSQL with drizzle-kit migrations applied; a `subscriber` with MFA, one without, one unverified, and a known-unregistered address; mail capture; a controllable clock; latency measurement for AC-04; Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-AUTH-070, REQ-AUTH-090, REQ-AUTH-060, REQ-SESSION-040, REQ-AUTH-120, REQ-AUDIT-010, REQ-API-040
- **Downstream Requirements**: None — this is a leaf recovery path.
- **External Dependencies**: The mail delivery mechanism introduced by REQ-AUTH-090 — in-account Amazon SES behind the internal mail interface (SEC-EXT-3; REQ-INFRA-060); no new dependency, and no health data crosses it (FR-9.8, SEC-TB-3, SEC-EXT-3).
- **Dependency Assumptions**: Mail delivery is best-effort; no state depends on delivery confirmation, and failure is not disclosed to the requester.
- **Failure Impact**: A reset that bypasses MFA reduces every MFA-enabled account to the security of its mailbox, which is precisely the failure MFA was enabled to prevent. A reset that leaves sessions alive makes recovery from compromise impossible.

## Implementation Notes

- **Constraints**: Node.js runtime with Fastify; PostgreSQL with Drizzle ORM; Vue single-page application (`CLAUDE.md`). Token lifetime is fixed at 30 minutes (`SECURITY.md` SQ-3 RESOLVED; SEC-AUTHN-11) and MUST be a named constant. Reset tokens are machine-held high-entropy tokens: store only HMAC-SHA-256 digests under a dedicated key from AWS Secrets Manager, permitting indexed single-lookup verification (SEC-AUTHN-11, amended 2026-08-03) — Argon2id is reserved for user-typed secrets. Privileged accounts MUST NOT be reachable by this flow at all — SEC-AUTHN-2 forbids a password path for them, including in recovery, and REQ-AUTH-150 is their only route.
- **Prohibited Approaches**: Issuing a session on reset completion. Treating a successful reset as satisfying MFA, or offering to disable MFA "because the user proved email control" (SEC-AUTHN-10). Any response, status, or timing difference between registered and unregistered addresses. Placing the token in a URL that is logged. Leaving existing sessions alive. Allowing a privileged account to reset a password it is not permitted to use.
- **Implementation Guidance**: Make credential replacement and session revocation one transaction, so no window exists in which the new password is live and old sessions still are. After a successful reset the user should be returned to normal authentication rather than logged in — that is what keeps AC-03 structurally true rather than a check someone remembered.
- **AI Development Guidance**: `REF-AUTH`, `REF-63B`, `REF-SECRETS`, `REF-PROMPT-API`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review confirming the reset path cannot issue a session and cannot alter MFA state, and that privileged accounts are excluded entirely.
- **Open Decisions**: None fixed by this issue — the token lifetime is resolved at 30 minutes (`SECURITY.md` SQ-3). Whether the user is notified by email that a reset occurred is still not specified by any source document (SEC-EXT-3 names the reset token message, not an after-the-fact notice); it is common practice and would help detect takeover, but it MUST be recorded as a decision rather than added silently.

**Estimated effort**: 1.5–2 engineer-days. **Estimated changed lines**: 350–700.
**Recommended model**: Claude Opus (`claude-opus-5`) — the highest-value target in the authentication surface, where the obvious convenience behavior at every step is the insecure one.
