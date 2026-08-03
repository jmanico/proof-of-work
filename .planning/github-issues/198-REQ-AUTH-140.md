# [REQ-AUTH-140] Privileged provisioning by invitation and first passkey enrolment

## Metadata

- **ID**: REQ-AUTH-140
- **Title**: Privileged provisioning by invitation and first passkey enrolment
- **Version**: 1.1.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-2.10, FR-2.7, FR-2.9, FR-2.16 (the issuance-refusal half); `SECURITY.md` SEC-AUTHN-9; threats TM-S-4, TM-E-1

## Requirement

- **Statement**: `admin` and `consultant` accounts MUST be created only by an invitation issued by an existing `admin`, carrying a single-use, short-lived, role-scoped enrolment token delivered to the invitee's email address recorded with the FR-2.16 vetting record — control of that address is proven by redemption of the token; an invitation MUST NOT be issued without that vetting record; the token MUST authorize passkey registration only and MUST NOT yield a session; and the first `admin` MUST be created by a one-time provisioning command that refuses to execute once any `admin` exists.
- **Rationale**: Before this, the plan contained a circular dependency: REQ-AUTH-020 required a registered passkey to authenticate, and REQ-AUTH-030 required an authenticated passkey session to register one, so no privileged account could ever come into being. SEC-AUTHN-9 breaks it by separating the right to enrol from the right to access. The invitation also removes role assignment from the request path entirely, which is what closes threat TM-E-1 — role comes from an admin's deliberate act, never from a field a client can set.
- **Assumptions**: Passkey registration mechanics are provided by REQ-AUTH-030; this issue supplies the authorization to perform a first registration, not the WebAuthn ceremony itself.
- **Out of Scope**: Passkey authentication (REQ-AUTH-020) and replacement registration by an already-authenticated privileged user (REQ-AUTH-030); privileged account minimums and passkey recovery (REQ-AUTH-150); the content, capture, and audit of the vetting record itself (REQ-AUTH-160 — this issue enforces only the refusal to issue without it); deprovisioning (REQ-AUTH-170; `SECURITY.md` SQ-12 RESOLVED via FR-2.16/FR-2.17); consultant engagement and the paid option (REQ-CONSULT-030; `REQUIREMENTS.md` OQ-13 and OQ-1 RESOLVED); throttling (REQ-AUTH-060).
- **Design Traceability**: `DESIGN.md` — Product Patterns → Credentials, account security, and administration (Invitation acceptance: a pre-authentication route at the authentication width showing the lockup, the role being granted, and a single register-first-passkey action; an expired or used token shows a neutral dead end with no account detail) and its administration surfaces (People: the invitation form requires the vetting record — who vetted, when, and on what basis — before **Send invitation** is available). Distinct labelled workspaces per role are fixed by `DESIGN.md` OQ-7 RESOLVED.
- **Architecture Traceability**: `ARCHITECTURE.md` — Identity and Session Handling (registration, passkey registration, role resolution); Relational Persistence (the invitation entity); trust boundary 2; boundary 5 for the bootstrap command, which is an operational action.
- **Security Traceability**: SEC-AUTHN-9; supports SEC-AUTHN-2 (no password path is created for these roles), SEC-INPUT-3 (role is not client-assignable), SEC-AUTHN-11 (enrolment token handling), SEC-LOG-4 (role changes audited), SEC-AUTHZ-4.

## Scope

- **Applies To**: Server-Side Application, Web Client, API
- **Components**: Identity and Session Handling; Relational Persistence; Browser Client (the enrolment view); the operational bootstrap path
- **Interfaces / Operations**: Invitation issuance, acceptance, and expiry; first passkey enrolment; the one-time bootstrap command
- **Actors**: `admin` (issuer); an invited person; an attacker attempting to obtain or forge an invitation, or to escalate their own role
- **Preconditions**: For invitation issuance, an authenticated `admin` session established by passkey, and an FR-2.16 vetting record for the invitee. For bootstrap, no `admin` exists.
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Personal Data — email address, enrolment token, passkey registration material, vetting-record reference
- **Jurisdiction / Regulatory Scope**: Global service with GDPR as the design ceiling (`SECURITY.md` SQ-1 RESOLVED): GDPR/UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable

## Security Context

- **Security Objectives**: Authenticity, Authorization, Accountability
- **Control Layers**: Authentication, Authorization, Business-Rule Validation
- **Threat References**: `SECURITY.md` TM-S-4 (privileged-account bootstrap attackable), TM-E-1 (privilege escalation via role change or provisioning), TM-S-2 (enrolment used as a recovery bypass); CWE-269 (improper privilege management), CWE-1188 (insecure default initialization), CWE-640 (weak recovery mechanism)
- **Abuse / Misuse Case**: An attacker reaches a bootstrap endpoint left enabled in production and creates themselves an admin; or intercepts an invitation and enrols their own passkey against someone else's intended role; or submits `role: admin` during ordinary registration and finds it bound; or replays a consumed invitation to attach a second credential.
- **Trust Boundary**: Boundary 2 for invitation acceptance; boundary 5 for the bootstrap command, which is why it must self-disable rather than rely on deployment discipline.
- **Untrusted Inputs or Assertions**: The enrolment token, the address it was sent to, and every field in an acceptance request. The role comes from the invitation record; nothing in the request may influence it.
- **Authoritative Enforcement Point**: Identity and Session Handling; the invitation record is the sole authority for what role the resulting account holds.
- **Independent Verification**: The token is verified against stored state and its role scope is read from that record, not from the request or the token's decoded contents.
- **Zero Trust Relevance**: NIST SP 800-207 — privilege is granted by an explicit administrative decision, not asserted by the subject. Exact tenet: TO BE DECIDED.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — mapping verified only at the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — mapping verified only at the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: GDPR/UK GDPR (EU/UK data subjects); CCPA/CPRA, Washington My Health My Data, FTC Health Breach Notification Rule (US users); HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED). Statute-section precision: TO BE DECIDED pending the SQ-1 pre-launch counsel review.
- **Other**: `REF-PASSKEY`, `REF-WEBAUTHN`, `REF-AUTH`, `REF-ASVS-5`, as named by SEC-AUTHN-9.
- **Mapping Basis**: SEC-AUTHN-9 names these references; the CWE identifiers name the privilege-management, insecure-initialization, and recovery classes.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated `admin` who issues an invitation scoped to `consultant` or `admin`, when the invited person redeems the token and completes passkey registration, then an account is created with exactly the invited role and at least one registered passkey.
2. **AC-02 — Boundary or failure behavior**: Given an enrolment token that is expired, already redeemed, forged, or scoped to a different address, when it is presented, then it is refused, no account is created, and no role is granted.
3. **AC-03 — Prohibited behavior**: Given an enrolment token, when it is presented, then it MUST NOT yield a session, an authenticated context, or access to any protected resource — it authorizes passkey registration and nothing else (SEC-AUTHN-9).
4. **AC-04 — Additional criterion**: Given any request to any endpoint containing a role field, when it is processed, then the value is ignored, and no self-service path creates an `admin` or `consultant` account (SEC-INPUT-3, TM-E-1).
5. **AC-05 — Additional criterion**: Given the bootstrap command, when any `admin` account already exists, then it refuses to execute; and when it does execute, it creates exactly one `admin` and records the action (TM-S-4).
6. **AC-06 — Additional criterion**: Given an invitation, when it is issued, accepted, expired, or revoked, and whenever a role is assigned or changed, then an audit entry records the acting account, the affected account, the role, and the time (SEC-LOG-4).
7. **AC-07 — Additional criterion**: Given a newly enrolled privileged account, when it subsequently authenticates, then only the passkey path is available and no password credential exists for it (SEC-AUTHN-2, FR-2.8).
8. **AC-08 — Additional criterion**: Given an invitation-issuance request without a recorded FR-2.16 vetting record — who vetted the invitee, when, and on what basis — when it is submitted by any actor including an authenticated `admin`, then issuance is refused, no enrolment token is created, and no message is sent (FR-2.16, SEC-AUTHN-9).

## Failure Behavior

- **On Invalid Input**: A malformed token is refused identically to an invalid one; the response reveals nothing about whether an invitation exists.
- **On Authentication Failure**: Invitation issuance requires an authenticated `admin` session established by passkey; anything less is refused.
- **On Authorization Failure**: A `subscriber` or `consultant` attempting to issue an invitation is refused by role check (REQ-AUTHZ-030); the denial does not confirm the endpoint's existence beyond the standard contract.
- **On Security-Decision Failure**: If the invitation record cannot be read, deny. There is no default role, and an unresolvable scope MUST NOT fall back to the least-privileged role — it must fail.
- **On External Dependency Failure**: If mail delivery fails, the invitation remains unaccepted; the issuing admin can revoke and reissue. No account is created optimistically.
- **On System Error**: Generic error with a correlation identifier (SEC-ERR-1); account creation and passkey registration are transactional together, so no roleless or passkey-less privileged account can persist.
- **Logging / Audit**: Per AC-06. The enrolment token MUST NOT be logged (SEC-LOG-3, SEC-AUTHN-11). The bootstrap execution MUST be recorded even though it runs outside a normal session.
- **Alerting**: Threshold alerts route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-3, SQ-11 RESOLVED); privileged account creation, invitation issuance, and any bootstrap execution are SEC-LOG-4 events feeding that channel.

## Test Strategy

- **Unit Tests**: Token generator uses the secure generator; the store holds only the HMAC-SHA-256 digest under the dedicated secret-store key, never the plaintext token (SEC-AUTHN-11); verifier rejects expired, redeemed, forged, and address-mismatched tokens; role is read from the invitation record and ignores request fields; issuance without a vetting-record reference is refused (AC-08); bootstrap guard returns refusal when an `admin` exists.
- **Integration Tests**: Full invitation-to-enrolment flow for each privileged role (AC-01); transactional rollback leaving no partial account; bootstrap creating the first admin and then refusing (AC-05).
- **Security Tests**: Token replay and cross-address suites (AC-02); attempt to use the token against protected routes (AC-03); mass-assignment probe submitting `role` on registration and on profile update (AC-04); attempt to issue an invitation as each non-admin role; an `admin` issuance attempt with the vetting record absent, asserted refused with no token created and no mail sent (AC-08); confirmation that the enrolled account has no password credential (AC-07); log assertion that no token appears.
- **Compliance Tests / Evidence**: The bootstrap self-disable transcript and the role-assignment audit trail, as evidence closing TM-S-4 and the request-path half of TM-E-1.
- **Acceptance-Criteria Traceability**: AC-01 — enrolment suite; AC-02 — token negative suite; AC-03 — token-scope suite; AC-04 — mass-assignment suite; AC-05 — bootstrap guard test; AC-06 — audit assertions; AC-07 — credential-absence test; AC-08 — vetting-record refusal test.
- **Coverage Target**: Both privileged roles × invitation valid, expired, replayed, and mismatched; issuance with and without the vetting record; plus bootstrap with and without an existing admin.
- **Required Test Environment**: PostgreSQL with drizzle-kit migrations applied; an `admin` with a registered passkey, a `consultant`, and a `subscriber`; a WebAuthn authenticator simulator; mail capture; a controllable clock; audit capture; Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-AUTH-010, REQ-AUTH-020, REQ-AUTH-030, REQ-AUTHZ-030, REQ-AUTH-060, REQ-AUDIT-010, REQ-AUDIT-030, REQ-API-020
- **Downstream Requirements**: REQ-AUTH-150 (privileged minimums and recovery build on this path); REQ-AUTH-160 (the vetting record this issue refuses to issue without); REQ-AUTH-170 (deprovisioned accounts are re-enabled only through this invitation path); REQ-CONSULT-010 and REQ-CONSULT-020, which presuppose consultant accounts exist; every admin operation in the plan library
- **External Dependencies**: The mail delivery mechanism from REQ-AUTH-090 — in-account Amazon SES behind the internal mail interface (SEC-EXT-3; REQ-INFRA-060); the WebAuthn library from REQ-AUTH-020.
- **Dependency Assumptions**: Mail delivery is best-effort, so invitations must be revocable and reissuable rather than assumed delivered.
- **Failure Impact**: Without this, no privileged account can exist at all — the plan library, publication, verification, and consultant access are all unreachable, since every one requires an `admin` or `consultant` that nothing else can create.

## Implementation Notes

- **Constraints**: Node.js runtime with Fastify; PostgreSQL with Drizzle ORM (`CLAUDE.md`). Enrolment token lifetime is fixed at 72 hours (`SECURITY.md` SQ-3 RESOLVED; SEC-AUTHN-11) and MUST be a named constant. Enrolment tokens are machine-held high-entropy tokens: store only HMAC-SHA-256 digests under a dedicated key from AWS Secrets Manager with indexed lookup (SEC-AUTHN-11, amended 2026-08-03). Vetting itself happens out of band (FR-2.16); this issue enforces the issuance refusal, while the record's content and audit belong to REQ-AUTH-160 and deprovisioning to REQ-AUTH-170 (SQ-12 RESOLVED).
- **Prohibited Approaches**: A temporary password for first login, which SEC-AUTHN-2 forbids and REQ-AUTH-020's AC-03 already rules out. A bootstrap path guarded only by configuration or deployment discipline rather than by a self-disabling check — TM-S-4 is precisely the risk of a bootstrap left reachable. Binding role from a request body anywhere in the system. Issuing any session from the enrolment token. Issuing an invitation without the FR-2.16 vetting record, or accepting the record as optional metadata attachable after the fact. Creating the account before passkey registration succeeds, which would leave a privileged account with no way to authenticate.
- **Implementation Guidance**: Treat the invitation record as the single authority for role, and have account creation read from it rather than from anything the enrolling user supplies. Because AC-04 spans the whole system rather than this issue's endpoints, its test belongs to the shared mass-assignment suite from REQ-API-020 and should be extended here rather than duplicated.
- **AI Development Guidance**: `REF-PASSKEY`, `REF-WEBAUTHN`, `REF-AUTH`, `REF-PROMPT-API`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the bootstrap guard and of every path that could set or change a role; architecture review that the enrolment token cannot be widened into a session credential.
- **Open Decisions**: None. The enrolment token lifetime is resolved at 72 hours (`SECURITY.md` SQ-3); vetting and deprovisioning are resolved (SQ-12 via FR-2.16/FR-2.17) and delivered by REQ-AUTH-160 and REQ-AUTH-170; consultant onboarding and the paid option are resolved (`REQUIREMENTS.md` OQ-13, OQ-1) and delivered by REQ-CONSULT-030.

**Estimated effort**: 2 engineer-days. **Estimated changed lines**: 500–950.
**Recommended model**: Claude Opus (`claude-opus-5`) — it creates the system's most privileged accounts, and both the bootstrap and the enrolment token have failure modes that grant full administrative access.
