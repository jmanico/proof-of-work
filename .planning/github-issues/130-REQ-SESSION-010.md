# [REQ-SESSION-010] JWT signature, algorithm, and claim verification

## Metadata

- **ID**: REQ-SESSION-010
- **Title**: JWT signature, algorithm, and claim verification
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `SECURITY.md` SEC-SESSION-1, SEC-SESSION-2; threats TM-T-3, TM-S-5

## Requirement

- **Statement**: Identity and Session Handling MUST verify a token's signature before reading any claim, MUST accept only algorithms on a server-side allow-list and reject `alg: none` and any algorithm outside that list, and MUST validate `exp`, `iss`, and `aud` on every token, rejecting a token that is missing any required claim rather than applying a default.
- **Rationale**: SEC-SESSION-1 and SEC-SESSION-2 state these rules. JWT is the selected session format (`SECURITY.md`, Session model), and the threat model rates algorithm and signature confusion (TM-T-3) and token replay (TM-S-5) as high severity.
- **Assumptions**: Tokens are issued by this system; no federated issuer exists (no external integrations, FR-9.8).
- **Out of Scope**: Token lifetime, refresh strategy, transport, storage, and the revocation mechanism, all `TO BE DECIDED` in `SECURITY.md` SQ-2 and SEC-SESSION-3; signing key storage and rotation (SEC-SESSION-7, blocked by SQ-7); claim content (REQ-SESSION-020).
- **Design Traceability**: N/A — `DESIGN.md` does not address session mechanics.
- **Architecture Traceability**: `ARCHITECTURE.md` — Identity and Session Handling ("role resolution supplied to the REST API Application for every request"); trust boundary 2; data flow 1.
- **Security Traceability**: SEC-SESSION-1, SEC-SESSION-2; supports SEC-AUTHN-1, SEC-AUTHZ-1, SEC-TB-1.

## Scope

- **Applies To**: Server-Side Application, API
- **Components**: Identity and Session Handling; REST API Application
- **Interfaces / Operations**: Verification of the session credential on every authenticated request
- **Actors**: `subscriber`, `consultant`, `admin`; anonymous attacker presenting forged tokens
- **Preconditions**: A token has been presented with the request
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Personal Data — the token identifies an account; it MUST NOT carry health data (REQ-SESSION-020)
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED (`SECURITY.md` SQ-1)

## Security Context

- **Security Objectives**: Authenticity, Integrity, Authorization
- **Control Layers**: Session Management, Authentication
- **Threat References**: `SECURITY.md` TM-T-3 (signature/algorithm confusion), TM-S-5 (token theft and replay); CWE-347 (improper verification of cryptographic signature), CWE-345 (insufficient verification of data authenticity)
- **Abuse / Misuse Case**: An attacker submits a token with `alg: none`, an algorithm the server did not intend (HMAC signed with a public key), a tampered payload, or a token missing `exp`, `iss`, or `aud`, and has it accepted as a valid session.
- **Trust Boundary**: Boundary 2 — unauthenticated → authenticated request handling.
- **Untrusted Inputs or Assertions**: The entire token, including its header, its `alg` value, and every claim, until the signature is verified.
- **Authoritative Enforcement Point**: Identity and Session Handling, before any claim is read or acted upon.
- **Independent Verification**: The accepted algorithm comes from server configuration, never from the token header.
- **Zero Trust Relevance**: NIST SP 800-207 — the session credential is re-verified on every request rather than trusted for a period. Exact tenet: TO BE DECIDED (not verified against the publication in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: N/A
- **Other**: `REF-PROMPT-JWT`, `REF-SESSION` as cited by SEC-SESSION-1 and SEC-SESSION-2. RFC 7519 governs the `exp`, `iss`, and `aud` claims named by SEC-SESSION-2.
- **Mapping Basis**: The two rules name these references directly; RFC 7519 defines the registered claims the rule requires. No control catalog identifier is asserted without verification.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a token signed with an allow-listed algorithm, carrying valid `exp`, `iss`, and `aud`, when a request presents it, then the signature is verified before any claim is read and the session resolves to the account and role named by the verified claims.
2. **AC-02 — Boundary or failure behavior**: Given a token with `alg: none`, an algorithm outside the allow-list, an invalid signature, a tampered payload, an expired `exp`, or a mismatched `iss` or `aud`, when a request presents it, then the request is rejected as unauthenticated and no claim value influences any decision.
3. **AC-03 — Prohibited behavior**: Given a token missing `exp`, `iss`, or `aud`, when it is processed, then the token MUST be rejected and a default value MUST NOT be applied; and the algorithm to verify with MUST NOT be selected from the token's own header.
4. **AC-04 — Additional criterion**: Given any verification failure, when the response is produced, then it is indistinguishable across failure causes and discloses neither which check failed nor whether the named account exists (SEC-AUTHN-3).

## Failure Behavior

- **On Invalid Input**: A malformed token is treated as an authentication failure, not a validation error, and produces the uniform unauthenticated response.
- **On Authentication Failure**: Deny; uniform response; no indication of the failing check (AC-04).
- **On Authorization Failure**: N/A — authorization is evaluated only after a session is established.
- **On Security-Decision Failure**: Any error during verification denies the request. An exception MUST NOT be caught and treated as success (SEC-AUTHZ-7, `SECURITY.md` code-quality resolution).
- **On External Dependency Failure**: If signing key material cannot be resolved, deny all requests rather than skip verification.
- **On System Error**: Generic error with a correlation identifier; the token MUST NOT be echoed.
- **Logging / Audit**: Log verification failures with cause class, correlation identifier, and — where the claim set was verified — the account identifier. MUST NOT log the token, its signature, or key material (SEC-LOG-3, SEC-SECRET-1).
- **Alerting**: TO BE DECIDED — no thresholds defined (`SECURITY.md` SQ-3).

## Test Strategy

- **Unit Tests**: Verifier rejects `alg: none`, a non-allow-listed algorithm, an invalid signature, a tampered payload, and each of `exp`, `iss`, `aud` absent, expired, or mismatched; accepts a well-formed token.
- **Integration Tests**: A request bearing each invalid token class is denied at the boundary and reaches no handler.
- **Security Tests**: Algorithm-confusion suite (HMAC token verified against an asymmetric public key); claim-omission suite; assertion via instrumentation or code review that no claim is read before signature verification.
- **Compliance Tests / Evidence**: N/A
- **Acceptance-Criteria Traceability**: AC-01 — happy-path verification test; AC-02 — invalid-token matrix; AC-03 — claim-omission and header-algorithm tests; AC-04 — response-uniformity test.
- **Coverage Target**: Every rejection cause covered by a negative test; positive and negative coverage for all verification branches.
- **Required Test Environment**: Test signing keys distinct from any real key material; Vitest as the runner, with the signing key store TO BE DECIDED (`SECURITY.md` SQ-7).

## Dependencies

- **Upstream Requirements**: None
- **Downstream Requirements**: REQ-AUTHZ-010, REQ-SESSION-020, REQ-AUTH-020, REQ-AUTH-040, REQ-CONSULT-020
- **External Dependencies**: A JWT library, subject to DEP-1…DEP-8 — DEP-1 forbids replacing vetted cryptographic and protocol-parsing functionality with custom code.
- **Dependency Assumptions**: The library allows the accepted algorithm set to be pinned by the caller and does not infer it from the token header.
- **Failure Impact**: A verification flaw grants arbitrary identity and role to any attacker, defeating every authorization control.

## Implementation Notes

- **Constraints**: JWT is fixed as the token format (`SECURITY.md`, Session model). Lifetime, transport, and revocation remain open (SQ-2) and MUST NOT be decided here; this issue covers verification only, so any session it validates is still subject to the unresolved revocation gap in SEC-SESSION-3.
- **Prohibited Approaches**: Decoding claims before verifying the signature; accepting the token's `alg` header as the verification algorithm; catching verification errors and proceeding; logging tokens or key material; writing a bespoke JWT parser (DEP-1).
- **Implementation Guidance**: Keep verification in one single-purpose function so the "signature before claims" ordering is reviewable at a glance (`SECURITY.md` code-quality resolution). A key identifier in the header may select among configured keys, but only from the server's configured set — this anticipates SEC-SESSION-7 without deciding it.
- **AI Development Guidance**: `REF-PROMPT-JWT`, `REF-PROMPT-NODE`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the verification path and the algorithm allow-list.
- **Open Decisions**: `SECURITY.md` SQ-2 (lifetime, transport, revocation) and SEC-SESSION-7 (key store, rotation period) are unresolved. They do not change the verification rules delivered here, but the session model is incomplete until they are closed.

**Estimated effort**: 0.5–1.5 engineer-days. **Estimated changed lines**: 150–400.
**Recommended model**: Claude Opus (`claude-opus-5`) — cryptographic verification logic where the failure mode is total authentication bypass.
