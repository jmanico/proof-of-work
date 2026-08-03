# [REQ-AUTH-030] Passkey registration and replacement with re-authentication

## Metadata

- **ID**: REQ-AUTH-030
- **Title**: Passkey registration and replacement with re-authentication
- **Version**: 1.1.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-2.9; `SECURITY.md` SEC-AUTHN-2, SEC-AUTHN-7

## Requirement

- **Statement**: An authenticated `admin` or `consultant` account MUST be able to register a passkey and to register a replacement passkey, and every such registration MUST require fresh re-authentication — no older than 5 minutes when the operation executes, and for these privileged accounts a passkey assertion (SEC-AUTHN-2) — and MUST produce an audit entry.
- **Rationale**: FR-2.9 grants the capability; SEC-AUTHN-7 (Confirmed, extended 2026-08-03) requires fresh re-authentication within the 5-minute named constant and an audit entry for passkey registration or replacement, because a silently added passkey is a persistent account takeover.
- **Assumptions**: The account already holds at least one usable passkey, so it can authenticate per REQ-AUTH-020 before registering another.
- **Out of Scope**: First-passkey enrolment for a newly provisioned privileged account, delivered by REQ-AUTH-140 (SEC-AUTHN-9; threat TM-S-4 closed); recovery when all passkeys are lost, delivered by REQ-AUTH-150 (FR-2.15: fresh invitation, two-passkey and two-admin minimums); subscriber MFA enrolment (REQ-AUTH-110); the two-passkey minimum callout's enforcement, which is FR-2.15 policy delivered by REQ-AUTH-150.
- **Design Traceability**: `DESIGN.md` — Credentials, account security, and administration → Passkeys (Account → Security lists each passkey with label and registration date; registration and replacement require re-authentication; a removal that would leave fewer than two is refused with the remedy) and Re-authentication (the prompt names the operation, honors the 5-minute freshness window, returns focus); Core Components → Actions, Forms and validation.
- **Architecture Traceability**: `ARCHITECTURE.md` — Identity and Session Handling ("passkey registration and verification"; "Owns credential material, MFA enrolment state, passkey registrations, and session state").
- **Security Traceability**: SEC-AUTHN-7, SEC-AUTHN-2, SEC-SECRET-4 (challenge randomness), SEC-LOG-4.

## Scope

- **Applies To**: Server-Side Application, Web Client, API
- **Components**: Identity and Session Handling; Browser Client (account security view)
- **Interfaces / Operations**: Passkey registration; replacement registration
- **Actors**: `admin`, `consultant`
- **Preconditions**: Authenticated session established via passkey (REQ-AUTH-020)
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Personal Data — passkey registration material
- **Jurisdiction / Regulatory Scope**: Global service, single US primary region (`SECURITY.md` SQ-1 RESOLVED): GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; GDPR-grade rights granted to all users; HIPAA not applicable

## Security Context

- **Security Objectives**: Authenticity, Integrity, Accountability
- **Control Layers**: Authentication, Logging and Monitoring
- **Threat References**: `SECURITY.md` TM-S-4 (privileged bootstrap and enrolment), TM-E-1 (privilege escalation), TM-S-5 (session theft used to enrol a new credential); CWE-287 (improper authentication), CWE-306 (missing authentication for critical function)
- **Abuse / Misuse Case**: An attacker holding a stolen session for a privileged account registers their own passkey, gaining durable access that survives session revocation and leaves no trace.
- **Trust Boundary**: Boundary 2 — the registration request arrives over an already-authenticated session, which SEC-AUTHN-7 treats as insufficient on its own.
- **Untrusted Inputs or Assertions**: The attestation or registration response; the claim that the current session's re-authentication falls within the 5-minute freshness window, which is verified server-side.
- **Authoritative Enforcement Point**: Identity and Session Handling — re-authentication is verified server-side before the registration is persisted.
- **Independent Verification**: The registration is bound to the re-authenticated account identity, never to an account identifier supplied in the request (DR-3, SEC-INPUT-3).
- **Zero Trust Relevance**: NIST SP 800-207 — a security-critical state change is re-authorized rather than inherited from an existing session. Exact tenet: TO BE DECIDED (not verified in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: GDPR and UK GDPR (EU/UK data subjects); CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule (US users); HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED). Statute-section mappings: TO BE DECIDED — verified at the SQ-1 pre-launch counsel review.
- **Other**: `REF-WEBAUTHN`, `REF-PASSKEY`, `REF-63B`, `REF-AUTH` as cited by SEC-AUTHN-2 and SEC-AUTHN-7.
- **Mapping Basis**: SEC-AUTHN-7 cites `REF-AUTH`, `REF-63B`, and `REF-ASVS-5`; the WebAuthn and passkey references govern the registration ceremony itself.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated `admin` or `consultant` who completes fresh re-authentication by passkey assertion no more than 5 minutes before the operation executes, when they complete a passkey registration ceremony, then the new passkey is persisted against their account, is immediately usable for authentication, and an audit entry records the actor, the action, and the time.
2. **AC-02 — Boundary or failure behavior**: Given an authenticated privileged account whose most recent re-authentication is absent or older than the 5-minute freshness constant, when a passkey registration is attempted, then it is refused, no registration is persisted, and the refusal is logged.
3. **AC-03 — Prohibited behavior**: Given any registration request, when it is processed, then it MUST NOT bind the passkey to an account identifier taken from the request body, MUST NOT succeed without a verified registration ceremony, and MUST NOT complete without producing an audit entry.
4. **AC-04 — Additional criterion**: Given a replacement registration, when it completes, then the account retains at least one usable passkey at every point in the operation, so the account cannot be locked out by a partially applied replacement.
5. **AC-05 — Additional criterion**: Given a `subscriber` account, when passkey registration is attempted, then the operation is denied — FR-2.9 grants this capability to `admin` and `consultant` accounts only.

## Failure Behavior

- **On Invalid Input**: A malformed registration response is rejected with no state change and a message that names the failure class without exposing internal detail.
- **On Authentication Failure**: If re-authentication fails, refuse the registration with a uniform response and log the failure (SEC-LOG-4).
- **On Authorization Failure**: Deny for accounts whose role is not `admin` or `consultant` (AC-05), and for any attempt to register against another account.
- **On Security-Decision Failure**: Deny if freshness of re-authentication cannot be determined, or if the audit write cannot be performed — the registration MUST NOT complete unaudited (DR-9 reasoning; SEC-AUTHN-7).
- **On External Dependency Failure**: If persistence is unavailable, the registration fails atomically and the previous passkey set is unchanged.
- **On System Error**: Roll back so that neither a partial registration nor a removed prior passkey survives; generic error with a correlation identifier.
- **Logging / Audit**: Audit entry for every registration and replacement with acting account, action, and time (SEC-AUTHN-7, SEC-LOG-4). MUST NOT log attestation material, challenges, or private key material (SEC-LOG-3, SEC-SECRET-1).
- **Alerting**: Threshold alerts on passkey registration against privileged accounts (SEC-LOG-4 events) route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-3, SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: Freshness check refuses a re-authentication older than the 5-minute named constant, including exactly at the boundary; registration binder uses the session identity and ignores a request-supplied account identifier; audit writer is invoked exactly once per successful registration.
- **Integration Tests**: End-to-end registration and replacement for both privileged roles; the new passkey authenticates; the account is never left with zero usable passkeys.
- **Security Tests**: Registration attempted with a valid session but no re-authentication (denied); registration bound to a foreign account identifier (denied); subscriber attempt (denied); a test that a failed audit write aborts the registration; assertion that no attestation or challenge appears in logs.
- **Compliance Tests / Evidence**: Audit records produced by the registration tests, retained as accountability evidence.
- **Acceptance-Criteria Traceability**: AC-01 — registration success suite; AC-02 — re-authentication requirement test; AC-03 — binding and audit negative tests; AC-04 — replacement continuity test; AC-05 — subscriber denial test.
- **Coverage Target**: Both privileged roles, registration and replacement, positive and negative.
- **Required Test Environment**: WebAuthn authenticator simulator; one `admin`, one `consultant`, one `subscriber`; audit log capture. Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-AUTH-010, REQ-AUTH-020, REQ-AUTHZ-010, REQ-AUDIT-010
- **Downstream Requirements**: REQ-AUTH-050
- **External Dependencies**: The WebAuthn server library used in REQ-AUTH-020, subject to DEP-1…DEP-8.
- **Dependency Assumptions**: The library verifies the registration ceremony including challenge freshness, origin, and relying-party identifier.
- **Failure Impact**: An unaudited or un-re-authenticated registration converts a stolen session into permanent privileged access.

## Implementation Notes

- **Constraints**: Node.js runtime with Fastify (`CLAUDE.md`). Freshness is fixed by SEC-AUTHN-7 (2026-08-03): re-authentication no older than 5 minutes when the operation executes, implemented as a named constant. For these privileged accounts, re-authentication is a passkey assertion (SEC-AUTHN-2). Per SEC-AUTHN-12, a passkey registration or replacement terminates all existing sessions for the account; the re-authentication and session-termination interplay is delivered with the session rules (REQ-SESSION-040) and MUST NOT be weakened here.
- **Prohibited Approaches**: Accepting the existing session as sufficient authorization; removing the old passkey before the replacement is confirmed usable; allowing registration for a subscriber "for future use"; completing the operation when the audit write failed.
- **Implementation Guidance**: Perform the audit write inside the same transaction as the registration so AC-03 cannot fail silently.
- **AI Development Guidance**: `REF-WEBAUTHN`, `REF-PASSKEY`, `REF-PROMPT-NODE`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the re-authentication gate and the replacement continuity logic.
- **Open Decisions**: None. The re-authentication freshness window is resolved at 5 minutes (SEC-AUTHN-7, 2026-08-03). First-passkey enrolment (REQ-AUTH-140) and lost-passkey recovery (REQ-AUTH-150) are excluded here; a privileged account that loses all passkeys recovers by fresh invitation under FR-2.15.

**Estimated effort**: 1–2 engineer-days. **Estimated changed lines**: 300–650.
**Recommended model**: Claude Opus (`claude-opus-5`) — credential-lifecycle code with an atomicity requirement and an absolute audit obligation.
