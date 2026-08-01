# [REQ-PLATFORM-040] TLS enforcement and security response headers

## Metadata

- **ID**: REQ-PLATFORM-040
- **Title**: TLS enforcement and security response headers
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `SECURITY.md` SEC-HTTP-1, SEC-HTTP-2; threat TM-T-4, TM-I-9

## Requirement

- **Statement**: The public browser-facing surface MUST serve all traffic over TLS, MUST reject or redirect plaintext HTTP, MUST send HSTS, and MUST set on every response a Content Security Policy that forbids inline and eval-based script execution, `X-Content-Type-Options: nosniff`, a restrictive referrer policy, and `Cache-Control: no-store` on any response containing health or personal data.
- **Rationale**: SEC-HTTP-1 and SEC-HTTP-2 require these headers; the threat model records stored XSS through admin-authored plan content (TM-T-4) and health-data residue in browser cache and history on shared devices (TM-I-9) as the threats they answer.
- **Assumptions**: A first-party browser client is the only consumer (SEC-HTTP-3 note; `SECURITY.md` SQ-6 leaves third-party exposure open).
- **Out of Scope**: The exact CSP directive set, which `SECURITY.md` SEC-HTTP-2 marks `TO BE DECIDED`; CORS posture (SEC-HTTP-3, blocked by SQ-6); token transport and cookie attributes (SEC-SESSION-5, delivered by REQ-SESSION-050; the `SameSite` value and cookie lifetime remain open under SQ-3); rate limiting (SEC-HTTP-5, blocked by SQ-3).
- **Design Traceability**: N/A — `DESIGN.md` does not address transport or headers.
- **Architecture Traceability**: `ARCHITECTURE.md` — trust boundary 1 (Browser Client → REST API Application); REST API Application outputs.
- **Security Traceability**: SEC-HTTP-1, SEC-HTTP-2; supports SEC-RENDER-1 (defense in depth against TM-T-4) and SEC-RENDER-4 (TM-I-9).

## Scope

- **Applies To**: Server-Side Application, API
- **Components**: REST API Application; Browser Client rendering surface
- **Interfaces / Operations**: Every HTTP response on the public surface
- **Actors**: All actors and unauthenticated clients
- **Preconditions**: None
- **Data Classification**: Multiple
- **Personal or Regulated Data**: Health Data — responses carrying it require `Cache-Control: no-store`
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED (`SECURITY.md` SQ-1)

## Security Context

- **Security Objectives**: Confidentiality, Integrity, Privacy
- **Control Layers**: Architecture, Data Protection, Output Encoding
- **Threat References**: `SECURITY.md` TM-T-4 (stored XSS), TM-I-9 (browser residue on shared devices); CWE-319 (cleartext transmission), CWE-524 (use of cache containing sensitive information)
- **Abuse / Misuse Case**: An attacker downgrades a session to plaintext to read health data in transit; injected script executes because no CSP restricts script sources; a subsequent user of a shared device retrieves cached health responses from the browser.
- **Trust Boundary**: Boundary 1 — Browser Client → REST API Application.
- **Untrusted Inputs or Assertions**: The client's protocol choice and any client-supplied cache directives.
- **Authoritative Enforcement Point**: REST API Application and its TLS-terminating edge; header emission MUST NOT be per-endpoint opt-in.
- **Independent Verification**: Headers are asserted on responses regardless of what the client requested.
- **Zero Trust Relevance**: NIST SP 800-207 — all communication is secured regardless of network location. Exact section: TO BE DECIDED (not verified against the publication in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: TO BE DECIDED — blocked by `SECURITY.md` SQ-1.
- **Other**: `REF-XSS`, `REF-REST`, `REF-PROMPT-VUE`, `REF-PROMPT-NODE` as cited by SEC-HTTP-1 and SEC-HTTP-2.
- **Mapping Basis**: The rule text of SEC-HTTP-1 and SEC-HTTP-2 and their own cited references are the only verified basis; no control catalog identifier is asserted without verification.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given any request to the public surface over TLS, when a response is produced, then it carries HSTS, `X-Content-Type-Options: nosniff`, a restrictive referrer policy, and a Content Security Policy whose script sources exclude `unsafe-inline` and `unsafe-eval`.
2. **AC-02 — Boundary or failure behavior**: Given a plaintext HTTP request, when it reaches the edge, then it is rejected or redirected to TLS and no application response body is served over plaintext.
3. **AC-03 — Prohibited behavior**: Given a response whose body contains health or personal data, when it is produced, then it MUST NOT be cacheable — `Cache-Control: no-store` is present — and no endpoint may opt out of the header set.
4. **AC-04 — Additional criterion**: Given a pre-production environment, when CSP violation reporting is enabled, then violations are collected and reviewable, and the report destination is inside the system boundary (SEC-TB-3).

## Failure Behavior

- **On Invalid Input**: A malformed or plaintext request is rejected at the edge with no application processing and no sensitive body.
- **On Authentication Failure**: N/A — headers are emitted irrespective of authentication outcome.
- **On Authorization Failure**: N/A — headers are emitted on denial responses too.
- **On Security-Decision Failure**: If the header middleware cannot determine whether a response body contains health or personal data, it MUST apply `no-store` (fail closed).
- **On External Dependency Failure**: N/A
- **On System Error**: Error responses carry the same header set; SEC-ERR-1 governs their bodies.
- **Logging / Audit**: Record CSP violation reports in pre-production. No health data, token, or credential may appear in a report or its log record (SEC-LOG-3).
- **Alerting**: TO BE DECIDED — no alerting destination or threshold is defined in any source document.

## Test Strategy

- **Unit Tests**: Header middleware emits the documented set for each response class, and selects `no-store` for health- and personal-data responses.
- **Integration Tests**: Automated header assertions across a representative endpoint per response class; plaintext request test asserting rejection or redirect.
- **Security Tests**: TLS configuration scan; negative test asserting no route can suppress the header set; assertion that CSP forbids inline and eval script.
- **Compliance Tests / Evidence**: Retained header and TLS scan output.
- **Acceptance-Criteria Traceability**: AC-01 — header assertion suite; AC-02 — plaintext test; AC-03 — cache-directive suite plus route-inventory test; AC-04 — pre-production CSP report check.
- **Coverage Target**: Every response class and every route exercised by the route-inventory header test.
- **Required Test Environment**: HTTP test client, a TLS-terminating edge configuration, and a pre-production environment — deployment topology TO BE DECIDED (`SECURITY.md` SQ-7).

## Dependencies

- **Upstream Requirements**: None
- **Downstream Requirements**: REQ-CATALOG-030 (CSP is defense in depth for TM-T-4), REQ-PRIVACY-060
- **External Dependencies**: None
- **Dependency Assumptions**: The TLS-terminating edge is inside the system boundary and does not log request bodies (SEC-LOG-3).
- **Failure Impact**: Loss of the header set exposes the client to injection and cache-residue threats; loss of TLS exposes health data in transit.

## Implementation Notes

- **Constraints**: Node.js with Fastify (`CLAUDE.md`); AWS/Terraform deployment with topology TO BE DECIDED (`SECURITY.md` SQ-7). The header set MUST be applied centrally so the route-inventory test can be exhaustive.
- **Prohibited Approaches**: Per-endpoint header opt-in; a CSP permitting `unsafe-inline` or `unsafe-eval`; sending CSP reports to any destination outside the system boundary (SEC-TB-3, FR-9.8); relying on the client to avoid caching.
- **Implementation Guidance**: Ship a policy that satisfies the stated prohibitions now and record the full directive list as an open decision until SEC-HTTP-2 is resolved; the prohibition on inline and eval script is itself normative and not open.
- **AI Development Guidance**: `REF-PROMPT-NODE`, `REF-PROMPT-VUE`; `CLAUDE.md`.
- **Required Human Review**: Security review of the CSP and TLS configuration.
- **Open Decisions**: The exact CSP directive set (SEC-HTTP-2, `TO BE DECIDED`) — this issue delivers the prohibitions that are normative and leaves the full directive list open, so its coverage of SEC-HTTP-2 is partial. Alerting destination and thresholds are undefined system-wide.

**Estimated effort**: 0.5–1.5 engineer-days. **Estimated changed lines**: 150–400.
**Recommended model**: Claude Opus (`claude-opus-5`) — security-sensitive configuration where an over-permissive default silently defeats the control.
