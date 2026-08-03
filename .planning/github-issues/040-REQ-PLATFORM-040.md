# [REQ-PLATFORM-040] TLS enforcement and security response headers

## Metadata

- **ID**: REQ-PLATFORM-040
- **Title**: TLS enforcement and security response headers
- **Version**: 1.1.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `SECURITY.md` SEC-HTTP-1, SEC-HTTP-2, SEC-HTTP-3; threats TM-T-4, TM-I-9

## Requirement

- **Statement**: The public browser-facing surface MUST serve all traffic over TLS, MUST reject or redirect plaintext HTTP, MUST send HSTS with a two-year max-age and includeSubDomains, and MUST set on every response `X-Content-Type-Options: nosniff`, a restrictive referrer policy, `Cache-Control: no-store` on any response containing health or personal data, and the Content Security Policy fixed by SEC-HTTP-2 (2026-08-03): `default-src 'none'; script-src 'self'; style-src 'self'; img-src 'self' data: blob:; connect-src 'self'; base-uri 'none'; form-action 'self'; frame-ancestors 'none'; object-src 'none'`. No response may carry an `Access-Control-Allow-Origin` header — CORS is disabled outright (SEC-HTTP-3).
- **Rationale**: SEC-HTTP-1 and SEC-HTTP-2 require these headers, and the CSP directive set is now fixed rather than open; the threat model records stored XSS through admin-authored plan content (TM-T-4, closed by the fixed directive set) and health-data residue in browser cache and history on shared devices (TM-I-9) as the threats they answer. SEC-HTTP-3 disables CORS because the REST surface is private to the first-party same-origin client (SQ-6 RESOLVED).
- **Assumptions**: A first-party same-origin browser client is the only consumer (SQ-6 RESOLVED). CloudFront is the single public origin, serving the client from S3 and forwarding the API path prefix to the ALB (SQ-7 addendum, 2026-08-03) — the routing that realizes the same-origin surface.
- **Out of Scope**: Token transport and cookie attributes (SEC-SESSION-5 — `SameSite=Lax` session-scoped cookie, fixed by SQ-3; delivered by REQ-SESSION-050); rate limits, body-size limits, and time-bounded request handling (SEC-HTTP-5 — REQ-API-050, planned 2026-08-03); the CloudFront/ALB Terraform itself (REQ-INFRA-010, REQ-INFRA-020, planned 2026-08-03 — this issue owns the header behavior, not the infrastructure).
- **Design Traceability**: N/A — `DESIGN.md` does not address transport or headers; the `img-src data: blob:` allowance exists for the transient food-photo preview `DESIGN.md` describes (FR-8.12).
- **Architecture Traceability**: `ARCHITECTURE.md` — trust boundary 1 (Browser Client → REST API Application); REST API Application outputs.
- **Security Traceability**: SEC-HTTP-1, SEC-HTTP-2 (directive set fixed 2026-08-03), SEC-HTTP-3 (CORS disabled; folded in per `ISSUE_PLAN.md` PQ-23); supports SEC-RENDER-1 (defense in depth against TM-T-4) and SEC-RENDER-4 (TM-I-9).

## Scope

- **Applies To**: Server-Side Application, API
- **Components**: REST API Application; Browser Client rendering surface
- **Interfaces / Operations**: Every HTTP response on the public surface
- **Actors**: All actors and unauthenticated clients
- **Preconditions**: None
- **Data Classification**: Multiple
- **Personal or Regulated Data**: Health Data — responses carrying it require `Cache-Control: no-store`
- **Jurisdiction / Regulatory Scope**: The SQ-1 regime set (RESOLVED) — GDPR/UK GDPR for EU/UK data subjects; CCPA/CPRA, Washington My Health My Data and comparable state consumer-health laws, and the FTC Health Breach Notification Rule for US users; HIPAA not applicable.

## Security Context

- **Security Objectives**: Confidentiality, Integrity, Privacy
- **Control Layers**: Architecture, Data Protection, Output Encoding
- **Threat References**: `SECURITY.md` TM-T-4 (stored XSS — directive set fixed, note closed 2026-08-03), TM-I-9 (browser residue on shared devices); CWE-319 (cleartext transmission), CWE-524 (use of cache containing sensitive information)
- **Abuse / Misuse Case**: An attacker downgrades a session to plaintext to read health data in transit; injected script executes because no CSP restricts script sources; a subsequent user of a shared device retrieves cached health responses from the browser; a hostile origin scripts credentialed cross-origin requests that permissive CORS headers would allow.
- **Trust Boundary**: Boundary 1 — Browser Client → REST API Application.
- **Untrusted Inputs or Assertions**: The client's protocol choice and any client-supplied cache directives.
- **Authoritative Enforcement Point**: REST API Application and its TLS-terminating edge (CloudFront as the single public origin, SQ-7); header emission MUST NOT be per-endpoint opt-in.
- **Independent Verification**: Headers are asserted on responses regardless of what the client requested.
- **Zero Trust Relevance**: NIST SP 800-207 — all communication is secured regardless of network location. Exact section: TO BE DECIDED (not verified against the publication in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — ASVS 5.0.0 Level 3 is the fixed design target (`SECURITY.md` SQ-10 RESOLVED); per-issue mappings are verified only at the independent pre-launch assessment.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: The SQ-1 regime set applies because responses carry health data; statute-section precision beyond what the specifications state: TO BE DECIDED.
- **Other**: `REF-XSS`, `REF-REST`, `REF-PROMPT-VUE`, `REF-PROMPT-NODE` as cited by SEC-HTTP-1 and SEC-HTTP-2.
- **Mapping Basis**: The rule text of SEC-HTTP-1, SEC-HTTP-2, and SEC-HTTP-3 and their own cited references are the only verified basis; no control catalog identifier is asserted without verification.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given any request to the public surface over TLS, when a response is produced, then it carries HSTS with a two-year max-age and includeSubDomains, `X-Content-Type-Options: nosniff`, a restrictive referrer policy, and exactly the SEC-HTTP-2 Content Security Policy — `default-src 'none'; script-src 'self'; style-src 'self'; img-src 'self' data: blob:; connect-src 'self'; base-uri 'none'; form-action 'self'; frame-ancestors 'none'; object-src 'none'` — whose script sources therefore exclude `unsafe-inline` and `unsafe-eval`.
2. **AC-02 — Boundary or failure behavior**: Given a plaintext HTTP request, when it reaches the edge, then it is rejected or redirected to TLS and no application response body is served over plaintext.
3. **AC-03 — Prohibited behavior**: Given a response whose body contains health or personal data, when it is produced, then it MUST NOT be cacheable — `Cache-Control: no-store` is present — and no endpoint may opt out of the header set.
4. **AC-04 — Additional criterion**: Given the staging environment, when the CSP is introduced, then it runs report-only there before production enforcement (SEC-HTTP-2), violations are collected and reviewable, and the report destination is inside the system boundary (SEC-TB-3).
5. **AC-05 — Additional criterion**: Given a request bearing any `Origin` header, when a response is produced, then it carries no `Access-Control-Allow-Origin` or other permissive CORS header (SEC-HTTP-3 — CORS is disabled outright, with no allow-list to maintain).

## Failure Behavior

- **On Invalid Input**: A malformed or plaintext request is rejected at the edge with no application processing and no sensitive body.
- **On Authentication Failure**: N/A — headers are emitted irrespective of authentication outcome.
- **On Authorization Failure**: N/A — headers are emitted on denial responses too.
- **On Security-Decision Failure**: If the header middleware cannot determine whether a response body contains health or personal data, it MUST apply `no-store` (fail closed).
- **On External Dependency Failure**: N/A
- **On System Error**: Error responses carry the same header set; SEC-ERR-1 governs their bodies.
- **Logging / Audit**: Record CSP violation reports during the report-only staging phase. No health data, token, or credential may appear in a report or its log record (SEC-LOG-3).
- **Alerting**: Threshold alerts route to the security lead as SEC-OPS-2 detection inputs (SQ-11 RESOLVED); a spike of CSP violations after enforcement is a detection signal, not a silent metric.

## Test Strategy

- **Unit Tests**: Header middleware emits the documented set for each response class, and selects `no-store` for health- and personal-data responses.
- **Integration Tests**: Automated header assertions across a representative endpoint per response class; plaintext request test asserting rejection or redirect; cross-origin request test asserting no permissive CORS header (AC-05).
- **Security Tests**: TLS configuration scan; negative test asserting no route can suppress the header set; assertion that the emitted CSP matches the SEC-HTTP-2 directive set exactly, so a drifted or relaxed policy fails.
- **Compliance Tests / Evidence**: Retained header and TLS scan output; the staging report-only review record preceding enforcement.
- **Acceptance-Criteria Traceability**: AC-01 — header assertion suite with exact-CSP match; AC-02 — plaintext test; AC-03 — cache-directive suite plus route-inventory test; AC-04 — staging report-only check; AC-05 — CORS-absence test.
- **Coverage Target**: Every response class and every route exercised by the route-inventory header test.
- **Required Test Environment**: HTTP test client, a TLS-terminating edge configuration, and the staging environment of the SQ-7 topology (separate AWS account, CloudFront single public origin) for the report-only phase.

## Dependencies

- **Upstream Requirements**: None
- **Downstream Requirements**: REQ-CATALOG-030 (CSP is defense in depth for TM-T-4), REQ-PRIVACY-060; REQ-FOOD-040 (planned 2026-08-03 — the transient photo preview relies on the `img-src data: blob:` allowance)
- **External Dependencies**: None
- **Dependency Assumptions**: The TLS-terminating edge is inside the system boundary and does not log request bodies (SEC-LOG-3); CloudFront-to-ALB path routing (SQ-7 addendum) preserves the same-origin surface SEC-HTTP-3 assumes.
- **Failure Impact**: Loss of the header set exposes the client to injection and cache-residue threats; loss of TLS exposes health data in transit.

## Implementation Notes

- **Constraints**: Node.js with Fastify (`CLAUDE.md`); AWS/Terraform deployment with the fixed SQ-7 topology — CloudFront as the single public origin serving the S3 client and forwarding the API path prefix to the ALB. The header set MUST be applied centrally so the route-inventory test can be exhaustive.
- **Prohibited Approaches**: Per-endpoint header opt-in; a CSP permitting `unsafe-inline` or `unsafe-eval`; emitting a directive set other than the one SEC-HTTP-2 fixes — any relaxation (for example `style-src-attr` for chart positioning) requires a documented amendment to that rule first; sending CSP reports to any destination outside the system boundary (SEC-TB-3, FR-9.8); sending any `Access-Control-Allow-Origin` header (SEC-HTTP-3); relying on the client to avoid caching.
- **Implementation Guidance**: Ship exactly the fixed directive set, validated report-only in staging before enforcement. The `data:`/`blob:` image sources exist solely for the transient food-photo preview (FR-8.12); nothing else may depend on them. The no-inline/no-eval floor is normative independent of the directive list.
- **AI Development Guidance**: `REF-PROMPT-NODE`, `REF-PROMPT-VUE`; `CLAUDE.md`.
- **Required Human Review**: Security review of the CSP and TLS configuration, including the staging report-only findings before enforcement.
- **Open Decisions**: None — the CSP directive set is fixed (SEC-HTTP-2, 2026-08-03), CORS posture is fixed (SQ-6 RESOLVED), the topology is fixed (SQ-7 RESOLVED), and alerting destinations are fixed (SQ-11 RESOLVED).

**Estimated effort**: 0.5–1.5 engineer-days. **Estimated changed lines**: 150–400.
**Recommended model**: Claude Opus (`claude-opus-5`) — security-sensitive configuration where an over-permissive default silently defeats the control.
