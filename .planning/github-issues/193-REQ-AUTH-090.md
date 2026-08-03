# [REQ-AUTH-090] Email verification and the health-data write gate

## Metadata

- **ID**: REQ-AUTH-090
- **Title**: Email verification and the health-data write gate
- **Version**: 1.1.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-2.11; `SECURITY.md` SEC-AUTHN-8, SEC-AUTHN-11; threat TM-S-3

## Requirement

- **Statement**: The system MUST verify control of a registered email address by a single-use, time-limited token before that account may record any health data (as defined by FR-9.12) or before the address is relied upon in any recovery flow, MUST refuse every health-data write while the address is unverified, and MUST apply the same verification to a replacement address under FR-2.18 before it becomes active (SEC-AUTHN-8).
- **Rationale**: SEC-AUTHN-8 and FR-2.11 exist because health data bound to an address its registrant does not control is both a privacy breach against the real owner and a recovery-hijack path (threat TM-S-3). The account may exist and authenticate before verification — that keeps registration friction low — but the write gate is absolute, because an unverified account is one where the system does not yet know who it is talking to.
- **Assumptions**: Registration initiates verification (REQ-AUTH-080). Recovery flows check verification state before using the address (REQ-AUTH-130).
- **Out of Scope**: Registration itself (REQ-AUTH-080); recovery flows that consume verification state (REQ-AUTH-130); throttling of the resend endpoint, provided by REQ-AUTH-060; the email-change flow itself (`REQUIREMENTS.md` FR-2.18, added 2026-08-03), delivered by REQ-AUTH-180, which consumes this issue's verification of the replacement address; the mail delivery channel — in-account Amazon SES behind an internal DR-7 mail interface (SEC-EXT-3) — delivered by REQ-INFRA-060.
- **Design Traceability**: `DESIGN.md` — Credentials, account security, and administration → Email verification: the `Email unverified` banner links to a screen explaining that logging stays unavailable until the address is verified (FR-2.11), offers a rate-limited resend that confirms non-committally (SEC-AUTHN-8, SEC-AUTHN-3), and the emailed link lands on a confirmation route returning the user to what they were doing.
- **Architecture Traceability**: `ARCHITECTURE.md` — Identity and Session Handling (verification state); REST API Application (enforces the write gate); Relational Persistence (the verification token entity); trust boundary 2; data flow 4, where the gate sits alongside the consent check.
- **Security Traceability**: SEC-AUTHN-8, SEC-AUTHN-11 (token handling), SEC-AUTHN-3 (no enumeration through the resend path), SEC-SECRET-4 (token generation).

## Scope

- **Applies To**: Server-Side Application, Web Client, API
- **Components**: Identity and Session Handling; REST API Application; Browser Client; Relational Persistence
- **Interfaces / Operations**: Verification token issuance, redemption, and resend; the health-data write gate on every logging and consent operation
- **Actors**: `subscriber`; a person whose address was registered by someone else; an attacker harvesting or replaying tokens
- **Preconditions**: An account exists with an unverified email address
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Personal Data — email address and verification token; the gate protects Health Data (FR-9.12)
- **Jurisdiction / Regulatory Scope**: Global service, single US primary region (`SECURITY.md` SQ-1 RESOLVED): GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; GDPR-grade rights granted to all users; HIPAA not applicable

## Security Context

- **Security Objectives**: Authenticity, Privacy, Confidentiality
- **Control Layers**: Authentication, Business-Rule Validation, Data Protection
- **Threat References**: `SECURITY.md` TM-S-3 (registration with an address the registrant does not control), TM-S-2 (recovery abuse), TM-I-4 (enumeration through the resend path); CWE-640 (weak password recovery mechanism), CWE-613 (insufficient session expiration, applied to the token), CWE-204 (observable response discrepancy)
- **Abuse / Misuse Case**: Someone registers using another person's address and logs health data against it, so the real owner later takes over an account already holding a stranger's medical information — or, worse, the stranger retains access to the real person's identity. Separately, an attacker uses the resend endpoint to discover which addresses are registered.
- **Trust Boundary**: Boundary 2 — the claimed address is untrusted until the token proves control of it.
- **Untrusted Inputs or Assertions**: The submitted token, the address in a resend request, and any client-supplied verification state. Verification status MUST be read from persisted state and MUST NOT be bindable (SEC-INPUT-3).
- **Authoritative Enforcement Point**: The REST API Application applies the write gate on every health-data path; the verification state itself is owned by Identity and Session Handling.
- **Independent Verification**: The gate re-reads verification state from persistence on each health-data write rather than trusting a session-cached flag, so that revocation of verification takes effect immediately.
- **Zero Trust Relevance**: NIST SP 800-207 — an authenticated subject is still not authorized for every operation. Exact tenet: TO BE DECIDED.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: GDPR and UK GDPR (EU/UK data subjects); CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule (US users); HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED). Statute-section mappings: TO BE DECIDED — verified at the SQ-1 pre-launch counsel review.
- **Other**: `REF-AUTH`, `REF-63B`, `REF-ASVS-5`, as named by SEC-AUTHN-8.
- **Mapping Basis**: SEC-AUTHN-8 names these references; the CWE identifiers name the recovery-mechanism, token-expiry, and response-discrepancy classes.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an unverified account and a valid, unexpired verification token, when it is redeemed, then the address is marked verified, the token is invalidated, and health-data writes become permitted.
2. **AC-02 — Boundary or failure behavior**: Given an account whose address is unverified, when any health-data write as defined by FR-9.12 is attempted — a workout, food, body-weight, or body-measurement log entry, a customized plan copy, an active plan selection, an AI estimation request (FR-8.12), or consent to health-data processing — then it is refused with a message naming verification as the requirement, and nothing is persisted.
3. **AC-03 — Prohibited behavior**: Given a verification token, when it is replayed after use, presented after expiry, presented for a different account, or forged, then it MUST be refused, and no path may mark an address verified without a valid token.
4. **AC-04 — Additional criterion**: Given a resend request, when it is submitted for a registered address and for an unregistered one, then the responses are indistinguishable in content, status, and timing, and resends are limited to at most once per minute and five per hour per address (SEC-AUTHN-8; `SECURITY.md` SQ-3 RESOLVED), enforced through REQ-AUTH-060 (SEC-AUTHN-3, TM-I-4).
5. **AC-05 — Additional criterion**: Given a new token issued by resend, when it is created, then any previously issued token for that address is invalidated (SEC-AUTHN-11).
6. **AC-06 — Additional criterion**: Given a route inventory of health-data write paths, when it is enumerated, then every one applies the gate, so a newly added logging endpoint cannot omit it.

## Failure Behavior

- **On Invalid Input**: A malformed token is refused with the same response as an invalid one; no detail distinguishes the cases.
- **On Authentication Failure**: Redemption does not require an authenticated session, so that a user can verify from any device; it therefore relies entirely on token secrecy, which is why SEC-AUTHN-11's single-use and expiry properties are mandatory here.
- **On Authorization Failure**: A health-data write on an unverified account is refused as a precondition failure, not as an authorization failure, and the message says so — the user can fix it.
- **On Security-Decision Failure**: If verification state cannot be read, refuse the health-data write. Failing open would defeat the entire control.
- **On External Dependency Failure**: If mail delivery fails, the account remains unverified and the resend path remains available; the system MUST NOT mark an address verified because delivery could not be confirmed.
- **On System Error**: Generic error with a correlation identifier (SEC-ERR-1); the token MUST NOT appear in the response.
- **Logging / Audit**: Audit the transition to verified with account, action, and time (SEC-LOG-4). The token MUST NOT appear in any log, URL recorded in a log, or error response (SEC-LOG-3, SEC-AUTHN-11).
- **Alerting**: Threshold alerts on resend bursts and token-failure patterns (SEC-LOG-4 events; limits fixed by SQ-3) route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-3, SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: Token generator uses the secure generator and produces single-use values; redemption rejects expired, consumed, foreign-account, and forged tokens; issuing a replacement invalidates the predecessor.
- **Integration Tests**: Redemption transitions state and unblocks writes (AC-01); every health-data write path refused while unverified (AC-02); resend invalidating the prior token (AC-05).
- **Security Tests**: Token replay, cross-account, and expiry suites (AC-03); differential response and timing on resend (AC-04); route inventory over health-data write paths (AC-06); log assertion that no token value is emitted; storage inspection asserting tokens persist only as HMAC-SHA-256 digests, never plaintext (SEC-AUTHN-11).
- **Compliance Tests / Evidence**: The health-data write-gate inventory, as evidence for SEC-AUTHN-8 and as an input to the TM-S-3 closure.
- **Acceptance-Criteria Traceability**: AC-01 — redemption suite; AC-02 — write-gate matrix across every logging path; AC-03 — token negative suite; AC-04 — resend differential suite; AC-05 — replacement test; AC-06 — route inventory.
- **Coverage Target**: Every health-data write path covered by the gate assertion; every token failure mode covered by a negative test.
- **Required Test Environment**: PostgreSQL with drizzle-kit migrations applied; a verified account, an unverified account, and a known-unregistered address; a mail capture fixture; a controllable clock for expiry; Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-BUILD-010, REQ-AUTH-080, REQ-AUTH-060, REQ-API-010, REQ-AUDIT-010, REQ-AUTHZ-010
- **Downstream Requirements**: REQ-PRIVACY-010 (consent capture is a health-data write and sits behind this gate), REQ-PROGRESS-010, REQ-PROGRESS-020, REQ-AUTH-130 (recovery depends on a verified address), REQ-AUTH-180 (the FR-2.18 replacement address is verified through this mechanism before it becomes active)
- **External Dependencies**: The transactional mail channel: in-account Amazon SES reached through an internally defined mail interface and listed as a named egress endpoint (SEC-EXT-3, SEC-CICD-3; `SECURITY.md` SQ-7 addendum), delivered by REQ-INFRA-060. It carries no health data and no secret beyond the single-use token the message exists to deliver, and its send behavior is non-enumerating (SEC-EXT-3).
- **Dependency Assumptions**: Mail delivery is best-effort and may be delayed or fail silently, which is why resend exists and why no state depends on delivery confirmation.
- **Failure Impact**: Without the gate, health data can be bound to an address its registrant does not control, which is a privacy breach against an uninvolved third party and a standing recovery-hijack path.

## Implementation Notes

- **Constraints**: Node.js runtime with Fastify; PostgreSQL with Drizzle ORM; Vue single-page application (`CLAUDE.md`). Token lifetime and resend limits are fixed (`SECURITY.md` SQ-3 RESOLVED; SEC-AUTHN-8): 24-hour token lifetime, resend at most once per minute and five per hour per address, all as named constants. Verification tokens are machine-held high-entropy tokens and are stored only as HMAC-SHA-256 hashes under a dedicated key from AWS Secrets Manager, permitting indexed single-lookup verification (SEC-AUTHN-11, 2026-08-03) — not with the Argon2id function. The mail transport is decided — in-account SES behind an internal DR-7 mail interface (SEC-EXT-3, REQ-INFRA-060) — so this logic calls the interface and never a vendor client directly.
- **Prohibited Approaches**: Marking an address verified on any signal other than token redemption — a link click without a token, a delivery receipt, or a support action. Placing the token in a URL that is then logged (SEC-LOG-3). Caching verification state in the session, which would let a revoked verification keep working. Differing responses between registered and unregistered addresses on resend. Treating the gate as a client-side affordance rather than a server-side refusal (DR-2).
- **Implementation Guidance**: Make the gate a dependency of the health-data write path rather than a check each endpoint remembers, in the same spirit DR-9 applies to audit writes — AC-06's inventory is the test that this was done structurally. The consent record (REQ-PRIVACY-010) is itself a health-data write and belongs behind the gate, which is worth confirming explicitly during implementation.
- **AI Development Guidance**: `REF-AUTH`, `REF-63B`, `REF-PROMPT-NODE`, `REF-PROMPT-API`; `CLAUDE.md`.
- **Required Human Review**: Security review of the write-gate inventory and of the token lifecycle, particularly replacement invalidation.
- **Open Decisions**: None. Token lifetime and resend limits are fixed (`SECURITY.md` SQ-3 RESOLVED) and the mail transport is decided (SEC-EXT-3, SQ-7 addendum; REQ-INFRA-060). Whether an unverified account should expire after some period is not specified by any source document and MUST NOT be invented; if desired it is a new requirement.

**Estimated effort**: 1.5–2 engineer-days. **Estimated changed lines**: 350–700.
**Recommended model**: Claude Opus (`claude-opus-5`) — the gate must be structural rather than remembered, and the token lifecycle has several replay and enumeration failure modes that functional tests would not surface.
