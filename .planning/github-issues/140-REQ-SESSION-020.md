# [REQ-SESSION-020] Token claim allow-list excluding sensitive data

## Metadata

- **ID**: REQ-SESSION-020
- **Title**: Token claim allow-list excluding sensitive data
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: High
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `SECURITY.md` SEC-SESSION-6; threats TM-S-5, TM-I-9

## Requirement

- **Statement**: Issued token payloads MUST contain only claims on an approved allow-list, and MUST NOT contain health data, credentials, or sensitive personal data.
- **Rationale**: SEC-SESSION-6 states the rule, on the ground that a signed JWT is not confidential — anyone who obtains the token can read its payload. The threat model records token theft and replay (TM-S-5) and health-data residue on shared devices (TM-I-9); a token carrying health data turns either into a direct disclosure.
- **Assumptions**: Tokens are issued by Identity and Session Handling and verified per REQ-SESSION-010.
- **Out of Scope**: Token lifetime (`SECURITY.md` SQ-3); transport and revocation, resolved by SQ-2 and delivered by REQ-SESSION-030/040/050; which authorization attributes the policy engine consumes (SQ-4) — this issue constrains only what may be carried in the token.
- **Design Traceability**: N/A
- **Architecture Traceability**: `ARCHITECTURE.md` — Identity and Session Handling ("Outputs: Authenticated session context including account identity and role"); Browser Client ("Holds transient view state and session credentials as issued by Identity and Session Handling").
- **Security Traceability**: SEC-SESSION-6; supports SEC-TB-3, SEC-DATA-5, SEC-RENDER-4, SEC-AUTHZ-6 (attributes must have a declared source and trust level).

## Scope

- **Applies To**: Server-Side Application
- **Components**: Identity and Session Handling; REST API Application (as consumer of the session context)
- **Interfaces / Operations**: Token issuance at successful authentication and at any refresh
- **Actors**: `subscriber`, `consultant`, `admin`
- **Preconditions**: Authentication has succeeded
- **Data Classification**: Confidential — the payload is readable by anyone holding the token
- **Personal or Regulated Data**: Personal Data — account identifier and role only; Health Data MUST NOT appear
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED (`SECURITY.md` SQ-1)

## Security Context

- **Security Objectives**: Confidentiality, Privacy
- **Control Layers**: Session Management, Data Protection
- **Threat References**: `SECURITY.md` TM-S-5, TM-I-9, TM-I-5 (health data leaking into non-audited surfaces); CWE-522 (insufficiently protected credentials), CWE-359 (exposure of private personal information)
- **Abuse / Misuse Case**: An attacker who captures a token — through XSS, a shared device, a proxy log, or browser history — decodes the payload and reads health attributes, an email address, or credential material that was placed there for convenience.
- **Trust Boundary**: Boundary 1 — the token is held by the untrusted client.
- **Untrusted Inputs or Assertions**: None on issuance; on consumption, the token is untrusted until verified (REQ-SESSION-010).
- **Authoritative Enforcement Point**: The token issuance function in Identity and Session Handling.
- **Independent Verification**: A test decodes issued tokens and asserts the claim set against the approved allow-list.
- **Zero Trust Relevance**: N/A — this constrains claim content, not a resource-access decision.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: N/A
- **Regulatory**: TO BE DECIDED — blocked by `SECURITY.md` SQ-1; a token carrying health data would engage whichever regime SQ-1 resolves to.
- **Other**: `REF-PROMPT-JWT` as cited by SEC-SESSION-6. RFC 7519 for the registered claims used.
- **Mapping Basis**: SEC-SESSION-6 cites `REF-PROMPT-JWT`; RFC 7519 defines the registered claim names; the CWE identifiers name the exposure classes.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a successful authentication, when a token is issued, then its payload contains exactly the claims on the approved allow-list — the registered claims required by SEC-SESSION-2 plus the account identifier and role — and nothing else.
2. **AC-02 — Boundary or failure behavior**: Given a change that adds a claim not on the allow-list, when the issued-token test runs, then it fails and the change does not merge.
3. **AC-03 — Prohibited behavior**: Given any issued token, when its payload is decoded without the signing key, then it MUST NOT reveal a body weight, measurement, workout, food entry, plan association, email address, password, password hash, MFA secret, or passkey material.
4. **AC-04 — Additional criterion**: Given the allow-list, when it is reviewed, then every claim has a documented purpose and a declared trust level for authorization use (SEC-AUTHZ-6).

## Failure Behavior

- **On Invalid Input**: N/A — issuance follows a successful authentication and consumes no untrusted input.
- **On Authentication Failure**: No token is issued.
- **On Authorization Failure**: N/A
- **On Security-Decision Failure**: If the claim set cannot be assembled from the allow-list, issuance MUST fail rather than emit an ad-hoc payload.
- **On External Dependency Failure**: If signing material is unavailable, no token is issued and authentication returns a generic failure.
- **On System Error**: No token is issued; generic error with a correlation identifier.
- **Logging / Audit**: Log issuance with account identifier, correlation identifier, and time (SEC-LOG-4). MUST NOT log the token itself or any claim value beyond the account identifier (SEC-LOG-3).
- **Alerting**: N/A

## Test Strategy

- **Unit Tests**: Issuance function emits exactly the allow-listed claims for each role; rejects an attempt to add an unlisted claim.
- **Integration Tests**: Authenticate as each role, decode the issued token, and assert the claim set matches the allow-list exactly.
- **Security Tests**: Assertion that no claim value matches any health-data field name or value present in the test fixture; assertion that no credential or passkey material appears.
- **Compliance Tests / Evidence**: The documented allow-list with per-claim purpose and trust level, retained for the SEC-AUTHZ-6 attribute schema work.
- **Acceptance-Criteria Traceability**: AC-01 and AC-02 — issued-token claim-set assertion; AC-03 — sensitive-content scan over decoded payloads; AC-04 — allow-list documentation review.
- **Coverage Target**: Every issuance path and every role covered.
- **Required Test Environment**: One identity per role with seeded health data, so AC-03 can assert absence against real values. Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-SESSION-010
- **Downstream Requirements**: REQ-AUTHZ-010, REQ-AUTHZ-020, REQ-AUTHZ-030, REQ-CONSULT-010
- **External Dependencies**: The JWT library used for issuance, subject to DEP-1…DEP-8.
- **Dependency Assumptions**: The library does not add claims implicitly beyond those the caller supplies and those required by the format.
- **Failure Impact**: A token carrying health data converts any token capture into a health-data breach with no server compromise required.

## Implementation Notes

- **Constraints**: JWT format is fixed (`SECURITY.md`). SQ-2 is RESOLVED: the token carries a session identifier and no authorization state, and role is re-read from persisted state on every request (SEC-SESSION-3, DR-3); that design MUST NOT be revisited here.
- **Prohibited Approaches**: Embedding entitlement, engagement, or health attributes in the token as a performance optimization; treating the payload as confidential because it is signed; adding claims at call sites rather than through the issuance function.
- **Implementation Guidance**: Because SEC-SESSION-4 requires role, subscription, and engagement changes to take effect without waiting for token expiry, keeping volatile authorization state out of the token is also the design the resolved SQ-2 session model requires.
- **AI Development Guidance**: `REF-PROMPT-JWT`, `REF-PROMPT-ABAC`; `CLAUDE.md`.
- **Required Human Review**: Security and privacy review of the claim allow-list.
- **Open Decisions**: `SECURITY.md` SQ-4 remains open (SQ-2 is RESOLVED); this issue's allow-list may gain claims once the authorization attribute schema is settled, and any addition must re-pass AC-03.

**Estimated effort**: 0.5–1 engineer-day. **Estimated changed lines**: 100–250.
**Recommended model**: Claude Opus (`claude-opus-5`) — small and security-sensitive; the failure mode is a silent privacy leak rather than a broken test.
