# [REQ-AUTH-070] Password credential storage with Argon2id

## Metadata

- **ID**: REQ-AUTH-070
- **Title**: Password credential storage with Argon2id
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-01
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `SECURITY.md` SEC-AUTHN-5; `REQUIREMENTS.md` FR-2.2, FR-2.3

## Requirement

- **Statement**: Passwords MUST be stored only as Argon2id digests with a per-credential salt drawn from a cryptographically secure generator, and the plaintext MUST NOT be persisted, logged, cached, or recoverable from stored state by any means.
- **Rationale**: SEC-AUTHN-5 names Argon2id specifically and prohibits non-memory-hard functions including bcrypt, because a password database reachable through the operational boundary (threat TM-I-8, trust boundary 5) must remain expensive to attack offline. This issue owns the storage and verification primitive; registration (REQ-AUTH-080), authentication (REQ-AUTH-100), and reset (REQ-AUTH-130) all consume it rather than each implementing hashing.
- **Assumptions**: A vetted Argon2id library is used rather than a first-party implementation, per DEP-1, which forbids replacing vetted cryptography with custom code.
- **Out of Scope**: Password policy and breached-password refusal, which belong to registration and reset (REQ-AUTH-080, REQ-AUTH-130); throttling (REQ-AUTH-060); the authentication flow itself (REQ-AUTH-100); concrete Argon2id parameter values, which are `TO BE DECIDED` pending production instance sizing (`SECURITY.md` SQ-7); passkey credential material, which is REQ-AUTH-020 and REQ-AUTH-030.
- **Design Traceability**: N/A — no user-facing surface.
- **Architecture Traceability**: `ARCHITECTURE.md` — Identity and Session Handling owns credential material; Relational Persistence stores it; trust boundary 2 and trust boundary 5, which is where an offline attack on stored digests would originate.
- **Security Traceability**: SEC-AUTHN-5; supports SEC-SECRET-4 (secure generation of salts), SEC-LOG-3 and SEC-SECRET-1 (no credential in logs or source), SEC-OPS-1 (the control that makes operational database access less catastrophic).

## Scope

- **Applies To**: Server-Side Application
- **Components**: Identity and Session Handling; Relational Persistence
- **Interfaces / Operations**: Credential creation, credential verification, and credential replacement
- **Actors**: `subscriber` (the only role that authenticates by password); an attacker holding a stolen credential store; a malicious or careless operator (trust boundary 5)
- **Preconditions**: None
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Personal Data — authentication credentials
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED (`SECURITY.md` SQ-1)

## Security Context

- **Security Objectives**: Confidentiality, Authenticity
- **Control Layers**: Authentication, Data Protection
- **Threat References**: `SECURITY.md` TM-I-8 (operator reads production data directly), TM-I-6 (backups hold recoverable data), TM-S-1 (credential attacks); CWE-916 (use of password hash with insufficient computational effort), CWE-256 (plaintext storage of a password), CWE-759 (one-way hash without a salt)
- **Abuse / Misuse Case**: Someone with database or backup access — an operator, a restored snapshot, an attacker who reached the data tier — takes the credential table offline and recovers passwords, which are commonly reused, giving access to the subscriber's email and thus to every recovery path in the system.
- **Trust Boundary**: Boundary 5 — human and operational access beneath the application, where this control is the last line. Also boundary 2 at verification time.
- **Untrusted Inputs or Assertions**: The submitted password, and any stored value until verified. Length and content of the submitted value must not be permitted to exhaust memory or time (see Prohibited Approaches).
- **Authoritative Enforcement Point**: Identity and Session Handling — a single credential module that is the only code permitted to read or write credential material.
- **Independent Verification**: Verification compares a freshly derived digest against stored state; no reversible transformation of the input is ever retained.
- **Zero Trust Relevance**: N/A — data-at-rest protection rather than a resource-access decision.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: N/A
- **Regulatory**: TO BE DECIDED — blocked by `SECURITY.md` SQ-1.
- **Other**: `REF-63B` (NIST SP 800-63B-4) and `REF-PROMPT-NODE`, as named by SEC-AUTHN-5.
- **Mapping Basis**: SEC-AUTHN-5 names these references; the CWE identifiers name the insufficient-effort, plaintext, and unsalted classes the rule forbids.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a password submitted at credential creation, when it is stored, then the persisted value is an Argon2id digest with a unique per-credential salt, and the correct password verifies against it while any other value does not.
2. **AC-02 — Boundary or failure behavior**: Given a stored credential, when the stored value is inspected directly in the database, then the plaintext is not present and is not derivable by any reversible transformation, including for a password known to the tester.
3. **AC-03 — Prohibited behavior**: Given any code path, when it handles a password, then the plaintext MUST NOT be written to a log, an error response, an audit entry, a cache, a temporary file, or a database column, and MUST NOT be included in any exception message.
4. **AC-04 — Additional criterion**: Given two accounts with an identical password, when both credentials are stored, then the persisted digests differ — proving per-credential salting (CWE-759).
5. **AC-05 — Additional criterion**: Given the credential module, when its parameters are reviewed, then Argon2id memory, iteration, and parallelism values are named constants with a documented tuning basis, not literals at the call site.
6. **AC-06 — Additional criterion**: Given a submitted password of extreme length, when verification runs, then it is rejected by a length bound before hashing, so that hashing cost cannot be driven by attacker-controlled input.

## Failure Behavior

- **On Invalid Input**: A password exceeding the length bound is rejected before any hashing occurs (AC-06), with a generic validation response.
- **On Authentication Failure**: A non-matching digest returns the uniform authentication failure defined by REQ-AUTH-040; this module reports only match or no-match and never why.
- **On Authorization Failure**: N/A
- **On Security-Decision Failure**: If hashing or verification raises, deny. An exception in the credential path MUST NOT be caught and converted into a successful verification, and MUST NOT be silently swallowed — `REF-PROMPT-QUALITY` treats that as a security defect.
- **On External Dependency Failure**: If the credential store is unreadable, deny; MUST NOT fall back to any alternative credential source.
- **On System Error**: Generic error with a correlation identifier (SEC-ERR-1); no credential material in the response or the log.
- **Logging / Audit**: Credential creation and replacement produce an audit entry recording the account, action, and time (SEC-AUTHN-7, SEC-LOG-4). The password, the digest, and the salt MUST NOT appear in any log (SEC-LOG-3, SEC-SECRET-1).
- **Alerting**: TO BE DECIDED — blocked by `SECURITY.md` SQ-3.

## Test Strategy

- **Unit Tests**: Round-trip verification for correct and incorrect passwords; distinct digests for identical passwords (AC-04); length bound enforced before hashing (AC-06); a raised exception yields denial rather than acceptance.
- **Integration Tests**: Credential creation followed by direct database inspection (AC-02); credential replacement invalidating the previous digest.
- **Security Tests**: Assertion that no log, error response, or audit entry contains a known test password (AC-03), run across the creation, verification, and failure paths; parameter review against AC-05; confirmation that no non-Argon2id hashing function is reachable from the credential path.
- **Compliance Tests / Evidence**: The parameter record and its tuning basis, retained as evidence for SEC-AUTHN-5.
- **Acceptance-Criteria Traceability**: AC-01 — round-trip suite; AC-02 — storage inspection; AC-03 — credential-leak scan; AC-04 — salt uniqueness test; AC-05 — code review; AC-06 — length-bound test.
- **Coverage Target**: Every branch of the credential module, including every error path, with positive and negative cases.
- **Required Test Environment**: PostgreSQL with drizzle-kit migrations applied; a known test password for leak assertions; log capture across server output; Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-BUILD-010, REQ-API-030, REQ-AUDIT-040
- **Downstream Requirements**: REQ-AUTH-080 (registration), REQ-AUTH-100 (authentication), REQ-AUTH-130 (password reset), REQ-AUTH-120 (recovery codes are stored with this same function per SEC-AUTHN-11)
- **External Dependencies**: A vetted Argon2id library, subject to DEP-1 … DEP-8. DEP-1 explicitly forbids replacing vetted cryptography with first-party code.
- **Dependency Assumptions**: The library performs constant-time comparison of digests and does not log or retain inputs. This MUST be confirmed during dependency review rather than assumed.
- **Failure Impact**: A weak or misused hashing primitive converts any database or backup exposure into mass credential compromise, and because passwords are reused, into compromise of the email accounts on which every recovery path depends.

## Implementation Notes

- **Constraints**: Node.js runtime with Fastify; PostgreSQL with Drizzle ORM (`CLAUDE.md`). Concrete Argon2id parameters are `TO BE DECIDED` because they depend on production instance sizing, which `SECURITY.md` SQ-7 leaves open; ship named constants with a documented basis and a note that they are provisional.
- **Prohibited Approaches**: bcrypt, scrypt, PBKDF2, or any general-purpose hash, all excluded by SEC-AUTHN-5's naming of Argon2id. A first-party Argon2id implementation (DEP-1). A global pepper hard-coded in source (SEC-SECRET-1). Comparing digests with a non-constant-time equality. Catching and swallowing an exception from the hashing path. Unbounded input length, which turns a deliberately expensive function into a denial-of-service vector.
- **Implementation Guidance**: Keep this a single-purpose module that is the only code touching credential columns, per `REF-PROMPT-QUALITY`'s isolation guidance — it makes the AC-03 leak assertions tractable and gives review one place to look. Because Argon2id is deliberately slow, coordinate with REQ-AUTH-060 so throttling short-circuits before hashing without creating a timing oracle.
- **AI Development Guidance**: `REF-63B`, `REF-PROMPT-NODE`, `REF-PROMPT-QUALITY`, `REF-SECRETS`; `CLAUDE.md`.
- **Required Human Review**: Security review of the credential module in full, and dependency review of the Argon2id library against DEP-3 through DEP-6 and the constant-time assumption.
- **Open Decisions**: Argon2id memory, iteration, and parallelism values (`SECURITY.md` SQ-7 sizing). Whether a pepper held in the secret store is used in addition to the salt is not specified by any source document; it MUST NOT be added unilaterally, since it interacts with key rotation (SEC-SECRET-2) which is itself blocked.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 200–450.
**Recommended model**: Claude Opus (`claude-opus-5`) — small in volume, but the one control standing between an operational data exposure and mass credential compromise, with several silent failure modes.
