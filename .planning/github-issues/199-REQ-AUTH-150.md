# [REQ-AUTH-150] Privileged account minimums and passkey recovery

## Metadata

- **ID**: REQ-AUTH-150
- **Title**: Privileged account minimums and passkey recovery
- **Version**: 1.1.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: High
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-2.15; `SECURITY.md` SEC-AUTHN-2, SEC-AUTHN-9, SEC-AUTHN-10

## Requirement

- **Statement**: Every `admin` and `consultant` account MUST hold at least two registered passkeys, the system MUST maintain at least two `admin` accounts, and a privileged account that has lost passkey access MUST be recoverable only by a fresh invitation from another `admin` — no password or email-possession path may restore privileged access.
- **Rationale**: SEC-AUTHN-2 forbids a password fallback for privileged roles including on recovery, which means the ordinary reset path (REQ-AUTH-130) is unavailable to them by design. Something must fill that gap or a lost device permanently removes an administrator. Two passkeys makes single-device loss self-service through the replacement flow FR-2.9 already provides; two admins guarantees someone can always issue the re-invitation. Together they remove the sole-admin dead end without leaning on the SEC-OPS-1 break-glass process (`SECURITY.md` SQ-13 RESOLVED), which grants time-boxed operational data access below the application and never restores account access — total loss of every privileged credential path is unrecoverable by design (SEC-AUTHN-10).
- **Assumptions**: Invitation issuance and enrolment are provided by REQ-AUTH-140; passkey registration and replacement by REQ-AUTH-030. This issue adds the invariants and the recovery route, not new credential mechanics.
- **Out of Scope**: Invitation and first enrolment (REQ-AUTH-140); passkey registration mechanics (REQ-AUTH-030); passkey authentication (REQ-AUTH-020); subscriber recovery (REQ-AUTH-130, REQ-AUTH-120); operational break-glass (SEC-OPS-1; `SECURITY.md` SQ-13 RESOLVED; REQ-INFRA-050 — it reaches production data, never account recovery); the vetting record (REQ-AUTH-160) and privileged deprovisioning (REQ-AUTH-170; SQ-12 RESOLVED via FR-2.16/FR-2.17); the FR-9.4 two-admin floor on admin self-deletion (REQ-PRIVACY-090, which enforces the same invariant on the deletion path).
- **Design Traceability**: `DESIGN.md` — Product Patterns → Credentials, account security, and administration (Passkeys: Account → Security lists each registered passkey with label and registration date; a privileged account below two passkeys sees a persistent `warning-soft` callout prompting registration of the second, which blocks nothing and stays until satisfied; a removal that would leave fewer than two is refused with the reason and the remedy of registering a replacement first). Refusing an action because it would breach a minimum must explain which minimum and how to satisfy it; it discloses nothing sensitive.
- **Architecture Traceability**: `ARCHITECTURE.md` — Identity and Session Handling (passkey registrations, role resolution); Relational Persistence (the invariants are enforced in schema and in application logic); trust boundary 2.
- **Security Traceability**: SEC-AUTHN-2 (no password path, including recovery), SEC-AUTHN-9 (invitation is the only enrolment route), SEC-AUTHN-10 (recovery may not weaken authentication), SEC-AUTHN-7 (re-authentication and audit), SEC-LOG-4.

## Scope

- **Applies To**: Server-Side Application, Web Client, API
- **Components**: Identity and Session Handling; Relational Persistence
- **Interfaces / Operations**: Passkey registration and removal; role change and account deactivation, insofar as either would breach a minimum; privileged recovery by re-invitation
- **Actors**: `admin`, `consultant`; an attacker attempting to strand the system without administrators, or to obtain privileged access through a recovery path
- **Preconditions**: At least one privileged account exists (created by REQ-AUTH-140)
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Personal Data — passkey registrations
- **Jurisdiction / Regulatory Scope**: Global service with GDPR as the design ceiling (`SECURITY.md` SQ-1 RESOLVED): GDPR/UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable

## Security Context

- **Security Objectives**: Availability, Authenticity, Authorization
- **Control Layers**: Authentication, Business-Rule Validation, Availability
- **Threat References**: `SECURITY.md` TM-S-2 (recovery used to bypass passkey), TM-S-4 (privileged bootstrap), TM-D-2 (denial of access), TM-I-8 (operator access below the application); CWE-269 (improper privilege management), CWE-640 (weak recovery mechanism), CWE-1188 (insecure default initialization)
- **Abuse / Misuse Case**: An attacker with brief privileged access removes the other admins, or removes their passkeys, leaving the system with no one able to administer it and forcing an operator into the database — the path SEC-OPS-1 says must be deny-by-default. Conversely, a legitimate admin loses their laptop and, without a second passkey or a second admin, the plan library becomes permanently unadministrable.
- **Trust Boundary**: Boundary 2 for recovery; boundary 5 is what the invariants exist to keep out of scope, by making an operator intervention unnecessary.
- **Untrusted Inputs or Assertions**: Any request that would remove a passkey, change a role, or deactivate an account. The count that determines whether the minimum is breached MUST be read from persisted state at the moment of the change.
- **Authoritative Enforcement Point**: Identity and Session Handling; the invariants are checked server-side and, where expressible, in schema so that a write below the application layer cannot quietly breach them.
- **Independent Verification**: Counts are computed from persistence within the same transaction as the change, so a concurrent removal cannot slip both past the check.
- **Zero Trust Relevance**: N/A — an availability and privilege-management invariant rather than a per-request access decision.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — mapping verified only at the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — mapping verified only at the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **NIST SP 800-207**: N/A
- **Regulatory**: GDPR/UK GDPR (EU/UK data subjects); CCPA/CPRA, Washington My Health My Data, FTC Health Breach Notification Rule (US users); HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED). Statute-section precision: TO BE DECIDED pending the SQ-1 pre-launch counsel review.
- **Other**: `REF-PASSKEY`, `REF-WEBAUTHN`, `REF-AUTH`, as named by SEC-AUTHN-2 and SEC-AUTHN-9.
- **Mapping Basis**: The cited rules name these references; the CWE identifiers name the privilege-management and recovery classes.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a privileged account with two registered passkeys, when one is lost and the holder authenticates with the other, then they can register a replacement without any invitation or administrative involvement (FR-2.9).
2. **AC-02 — Boundary or failure behavior**: Given a privileged account with exactly two passkeys, when a request would remove one and leave fewer than two, then it is refused with a message naming the minimum, unless a replacement is registered in the same operation.
3. **AC-03 — Prohibited behavior**: Given a privileged account that has lost all passkey access, when recovery is attempted, then no password, email-possession, security-question, or support path may restore access — only a fresh invitation from another `admin` under REQ-AUTH-140 (SEC-AUTHN-2, SEC-AUTHN-10).
4. **AC-04 — Additional criterion**: Given exactly two `admin` accounts, when an operation would remove, demote, or deactivate one, then it is refused, so the system can never be left with fewer than two administrators.
5. **AC-05 — Additional criterion**: Given a privileged account created by invitation, when enrolment completes with only one passkey, then the account is prompted to register a second and the outstanding requirement is visible until satisfied.
6. **AC-06 — Additional criterion**: Given concurrent requests that would each individually leave a minimum satisfied but together breach it, when both are processed, then at most one succeeds — the check and the change are atomic.
7. **AC-07 — Additional criterion**: Given any passkey registration or removal, role change, or recovery invitation, when it completes, then an audit entry records the acting account, the affected account, and the time (SEC-AUTHN-7, SEC-LOG-4).

## Failure Behavior

- **On Invalid Input**: A malformed passkey removal request is refused with a validation error that does not enumerate other accounts' credentials.
- **On Authentication Failure**: Passkey registration and removal require fresh re-authentication (SEC-AUTHN-7); a stale session is refused.
- **On Authorization Failure**: A `consultant` attempting to issue a recovery invitation is refused — only an `admin` may (REQ-AUTHZ-030).
- **On Security-Decision Failure**: If the count backing an invariant cannot be computed, refuse the change. Permitting a removal while unable to confirm the minimum is how a system ends up with zero administrators.
- **On External Dependency Failure**: N/A beyond mail delivery for the recovery invitation, which is covered by REQ-AUTH-140.
- **On System Error**: Generic error with a correlation identifier (SEC-ERR-1); the invariant check and the change are one transaction, so a failure never leaves a breached minimum.
- **Logging / Audit**: Per AC-07. A refused change that would have breached a minimum SHOULD also be logged, since repeated attempts may indicate an attacker trying to strand the system (SEC-LOG-4).
- **Alerting**: Threshold alerts route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-3, SQ-11 RESOLVED); dropping to a minimum, and repeated refused changes that would breach one, are SEC-LOG-4 signals feeding that channel.

## Test Strategy

- **Unit Tests**: Minimum checks return refusal at the boundary and permit above it; the atomic check-and-change rejects the second of two concurrent removals; an uncomputable count yields refusal.
- **Integration Tests**: Self-service replacement using the second passkey (AC-01); removal refused at the minimum and permitted when paired with a registration (AC-02); admin demotion and deactivation refused at two admins (AC-04); the outstanding second-passkey prompt after enrolment (AC-05).
- **Security Tests**: Attempt privileged recovery through every non-invitation path — password reset, email possession, support-shaped endpoints — asserting refusal (AC-03); concurrent-removal race (AC-06); attempt as `consultant` to issue a recovery invitation.
- **Compliance Tests / Evidence**: The no-alternate-privileged-recovery transcript, as evidence for SEC-AUTHN-2's explicit extension to recovery paths.
- **Acceptance-Criteria Traceability**: AC-01 — replacement suite; AC-02 — passkey minimum suite; AC-03 — alternate-path denial suite; AC-04 — admin minimum suite; AC-05 — enrolment follow-up test; AC-06 — concurrency test; AC-07 — audit assertions.
- **Coverage Target**: Both privileged roles × at, above, and below each minimum, plus the concurrency case.
- **Required Test Environment**: PostgreSQL with drizzle-kit migrations applied; three `admin` accounts and one `consultant`, each with controllable passkey counts; a WebAuthn authenticator simulator; audit capture; a concurrency harness; Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-AUTH-140, REQ-AUTH-030, REQ-AUTH-020, REQ-AUTH-010, REQ-AUTHZ-030, REQ-AUDIT-010
- **Downstream Requirements**: None — this is a leaf invariant, though it underwrites the availability of every admin-only operation in the plan library.
- **External Dependencies**: The WebAuthn library from REQ-AUTH-020; the mail path from REQ-AUTH-140 for recovery invitations.
- **Dependency Assumptions**: Passkey registrations are countable per account in a single query, so the invariant can be evaluated transactionally rather than by application-level bookkeeping that could drift.
- **Failure Impact**: Without the minimums, one lost device makes the plan library permanently unadministrable, and the only remaining route is direct database access — precisely the operator path SEC-OPS-1 confines to a deny-by-default, two-person, time-boxed break-glass (`SECURITY.md` SQ-13 RESOLVED; REQ-INFRA-050), which reaches data for operational emergencies and never restores account access.

## Implementation Notes

- **Constraints**: Node.js runtime with Fastify; PostgreSQL with Drizzle ORM (`CLAUDE.md`). The minimums are fixed at two by FR-2.15 and are not tunable without a requirement change; express them as named constants so the intent is legible, not so they can be lowered in configuration. The same two-admin floor is enforced by privileged deprovisioning (FR-2.17, SEC-AUTHN-13; REQ-AUTH-170) and by admin account deletion (FR-9.4; REQ-PRIVACY-090); the invariant this issue owns must be the one those paths consult, not a parallel reimplementation.
- **Prohibited Approaches**: Any password, email-possession, security-question, or support-mediated route to privileged access (SEC-AUTHN-2, SEC-AUTHN-10). Enforcing the minimums only in the client. Checking the count and then performing the change in separate transactions, which AC-06 exists to catch. Allowing an admin to remove their own last passkey or demote the second-to-last admin because they are acting on themselves — self-service is not an exemption from the invariant.
- **Implementation Guidance**: Enforce the counts within the same transaction as the mutation, and where PostgreSQL can express the constraint directly, do so — that also protects against a write arriving below the application layer (trust boundary 5). Pair passkey removal with registration as one atomic operation, so a user rotating a device is never briefly below the minimum and never blocked by it.
- **AI Development Guidance**: `REF-PASSKEY`, `REF-WEBAUTHN`, `REF-AUTH`, `REF-PROMPT-QUALITY`, `REF-PROMPT-TF-AWS`; `CLAUDE.md`.
- **Required Human Review**: Security review that no alternate privileged recovery path exists anywhere in the system, and architecture review of the transactional invariant enforcement.
- **Open Decisions**: None. `SECURITY.md` SQ-13 is resolved: the SEC-OPS-1 break-glass (REQ-INFRA-050) is the only path below the application and it grants operational data access, never account recovery — if every privileged credential path is somehow lost, the position is unrecoverable-by-design per SEC-AUTHN-10, not an operator restore. Deprovisioning is resolved (SQ-12; FR-2.17, SEC-AUTHN-13; REQ-AUTH-170) and honors AC-04's floor: a departing admin is replaced by invitation before removal.

**Estimated effort**: 1.5–2 engineer-days. **Estimated changed lines**: 300–650.
**Recommended model**: Claude Opus (`claude-opus-5`) — invariants that must hold transactionally and under concurrency, protecting against both an attacker stranding the system and a legitimate user locking everyone out.
