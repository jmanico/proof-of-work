# [REQ-SESSION-050] Cookie session transport and cross-site request forgery protection

## Metadata

- **ID**: REQ-SESSION-050
- **Title**: Cookie session transport and cross-site request forgery protection
- **Version**: 1.1.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `SECURITY.md` SEC-SESSION-5, SEC-HTTP-4; SQ-2

## Requirement

- **Statement**: The session token MUST be transported in an `HttpOnly`, `Secure`, `SameSite` cookie and MUST NOT be readable from JavaScript or appear in a URL, query string, browser history, or client storage; and because that cookie is ambient authority, every state-changing request MUST additionally carry an anti-forgery signal that a cross-site context cannot supply.
- **Rationale**: SEC-SESSION-5 selects cookie transport, which removes the token from script reach and so from the reach of any successful XSS (threat TM-S-5) and from browser storage residue on shared devices (TM-I-9). That choice has a cost SEC-HTTP-4 now states unconditionally: a cookie is sent by the browser on cross-site requests too, so CSRF becomes live (TM-S-6). `SameSite` alone is explicitly recorded as insufficient. The two halves are one decision and are delivered together, because shipping the cookie without the anti-forgery control would trade one vulnerability for another.
- **Assumptions**: One first-party browser client and no documented cross-origin consumer (`SECURITY.md` SQ-6 RESOLVED), so no third-party origin needs a header-based alternative. The SQ-7 addendum realizes the same-origin surface: CloudFront is the single public origin, serving the client from S3 and forwarding the API path prefix to the ALB.
- **Out of Scope**: Session record semantics and resolution (REQ-SESSION-030); logout and revocation (REQ-SESSION-040); token signature and claims (REQ-SESSION-010, REQ-SESSION-020); TLS termination and the wider security header set (REQ-PLATFORM-040); CORS posture (SEC-HTTP-3 — SQ-6 RESOLVED: CORS disabled outright on the first-party same-origin surface); signing key storage (SEC-SESSION-7 — resolved by SQ-7; delivered by REQ-INFRA-030). The `SameSite` value and cookie lifetime are no longer open — SQ-3 fixed them (`SameSite=Lax`, session-scoped non-persistent cookie; see Constraints).
- **Design Traceability**: `DESIGN.md` — Components → Form feedback and errors: a rejected state-changing request must surface as an actionable message rather than a silent failure, and must not leak why it was rejected.
- **Architecture Traceability**: `ARCHITECTURE.md` — trust boundary 1 (Browser Client to REST API Application); Identity and Session Handling; data flow 1. DR-2: the client holds no session authority; it merely carries a credential the server issued.
- **Security Traceability**: SEC-SESSION-5, SEC-HTTP-4; supports SEC-RENDER-4 (no tokens in client storage), SEC-HTTP-1 (TLS), SEC-HTTP-2 (`Cache-Control: no-store` on responses carrying personal data).

## Scope

- **Applies To**: Server-Side Application, Web Client, API
- **Components**: Identity and Session Handling; Browser Client; the public HTTPS boundary
- **Interfaces / Operations**: Session cookie issuance, renewal, and clearing; every state-changing operation at the public boundary
- **Actors**: All authenticated actors; an attacker operating a hostile origin in the victim's browser
- **Preconditions**: A session has been issued
- **Data Classification**: Confidential
- **Personal or Regulated Data**: Personal Data — the session credential
- **Jurisdiction / Regulatory Scope**: Global service, single US primary region with standard lawful cross-border transfer mechanisms (`SECURITY.md` SQ-1 RESOLVED). Regimes: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable.

## Security Context

- **Security Objectives**: Confidentiality, Integrity, Authenticity
- **Control Layers**: Session Management, Architecture, Output Encoding
- **Threat References**: `SECURITY.md` TM-S-5 (token theft via XSS or shared device), TM-S-6 (CSRF against state-changing operations), TM-I-9 (health-data residue in browser storage); CWE-352 (cross-site request forgery), CWE-1004 (sensitive cookie without `HttpOnly`), CWE-614 (sensitive cookie without `Secure`)
- **Abuse / Misuse Case**: A hostile page causes the victim's browser to issue an authenticated state-changing request — deleting log entries, ending a consultant engagement, changing profile data — using the ambient cookie without ever reading it. Separately, an XSS payload attempts to exfiltrate the session and fails because the cookie is unreadable from script.
- **Trust Boundary**: Boundary 1 — the browser client is untrusted, and a request bearing a valid cookie is not by itself evidence of user intent.
- **Untrusted Inputs or Assertions**: The request origin, the `Referer` header, and any client-supplied anti-forgery value until verified server-side. The presence of the cookie proves the browser has a session, not that this application initiated the request.
- **Authoritative Enforcement Point**: The REST API Application, in a pipeline stage covering every state-changing route — not per handler.
- **Independent Verification**: The anti-forgery signal is verified against server-held state, and its absence denies regardless of how valid the cookie is.
- **Zero Trust Relevance**: NIST SP 800-207 — possession of a credential is not sufficient authorization for a request. Exact tenet: TO BE DECIDED.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: GDPR/UK GDPR (EU/UK data subjects); CCPA/CPRA, Washington My Health My Data, FTC HBNR (US users); HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED). Specific article/section mappings: TO BE DECIDED — no source document states one for session transport.
- **Other**: `REF-SESSION`, `REF-PROMPT-JWT`, `REF-PROMPT-VUE`, `REF-ASVS-5`, as named by SEC-SESSION-5 and SEC-HTTP-4.
- **Mapping Basis**: The two rules name these references; the CWE identifiers name the forgery and cookie-attribute classes precisely.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a successful authentication, when the session is issued, then it is set as a session-scoped (non-persistent) cookie carrying `HttpOnly`, `Secure`, and `SameSite=Lax` (SQ-3 RESOLVED, SEC-SESSION-5), and the response body contains no token value.
2. **AC-02 — Boundary or failure behavior**: Given a state-changing request that carries a valid session cookie but no valid anti-forgery signal, when it is submitted, then it is refused, no state changes, and the response does not explain which signal was missing (SEC-ERR-1).
3. **AC-03 — Prohibited behavior**: Given the running client, when its storage, URLs, and history are inspected, then no session token appears in `localStorage`, `sessionStorage`, a query string, a URL path, or browser history, and `document.cookie` does not expose it.
4. **AC-04 — Additional criterion**: Given a cross-origin state-changing request issued from a hostile origin with the victim's cookie present, when it reaches the API, then it is refused (TM-S-6).
5. **AC-05 — Additional criterion**: Given a route inventory, when it is enumerated, then every state-changing route passes through the anti-forgery stage, so a new route cannot silently omit it.
6. **AC-06 — Additional criterion**: Given logout (REQ-SESSION-040), when it completes, then the cookie is cleared with attributes matching those it was set with, so it is genuinely removed rather than orphaned.

## Failure Behavior

- **On Invalid Input**: A malformed or absent anti-forgery value denies the request with a uniform response.
- **On Authentication Failure**: Handled by REQ-SESSION-030; this rule applies only after a session resolves.
- **On Authorization Failure**: N/A — forgery protection precedes and is independent of authorization.
- **On Security-Decision Failure**: If the anti-forgery signal cannot be evaluated, deny. An error in verification MUST NOT permit the request.
- **On External Dependency Failure**: N/A — verification is local to the request and its server-held state.
- **On System Error**: Generic error with a correlation identifier (SEC-ERR-1); no cookie or token value in any response body or log.
- **Logging / Audit**: Log anti-forgery denials with cause class, route, and correlation identifier (SEC-LOG-4). MUST NOT log the cookie value or the anti-forgery value (SEC-LOG-3, SEC-SECRET-1).
- **Alerting**: Threshold alerts on forgery-denial spikes route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-3, SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: Cookie serializer always emits the three attributes; anti-forgery verifier rejects absent, malformed, expired, and foreign-session values, and denies on verification error.
- **Integration Tests**: Authentication sets the cookie correctly (AC-01); a state-changing request without the signal is refused (AC-02); logout clears the cookie with matching attributes (AC-06).
- **Security Tests**: Cross-origin state-change attempt from a hostile origin (AC-04); runtime storage, URL, and history inspection in a real browser (AC-03); a `document.cookie` read asserted not to expose the session; route inventory over every state-changing route (AC-05).
- **Compliance Tests / Evidence**: The route-inventory result and the cross-origin denial transcript, as evidence for SEC-HTTP-4.
- **Acceptance-Criteria Traceability**: AC-01 — cookie attribute assertions; AC-02 — missing-signal suite; AC-03 — Playwright storage inspection; AC-04 — cross-origin suite; AC-05 — route inventory; AC-06 — logout clearing test.
- **Coverage Target**: Every state-changing route covered by the inventory assertion; every cookie attribute asserted on issuance and clearing.
- **Required Test Environment**: A TLS-terminating configuration, a hostile-origin fixture for cross-origin tests, and Playwright for real-browser storage and history inspection; Vitest as the runner. Deployment topology is fixed (`SECURITY.md` SQ-7 RESOLVED): CloudFront is the single public origin, forwarding the API path prefix to the ALB.

## Dependencies

- **Upstream Requirements**: REQ-SESSION-030, REQ-PLATFORM-040, REQ-API-040
- **Downstream Requirements**: REQ-SESSION-040 (cookie clearing on logout); every state-changing operation in the system, since all of them traverse the anti-forgery stage
- **External Dependencies**: None
- **Dependency Assumptions**: TLS is enforced at the edge (REQ-PLATFORM-040), without which `Secure` is unenforceable and the cookie can traverse plaintext.
- **Failure Impact**: Omitting the anti-forgery half converts a defensive transport choice into an attack surface: every state-changing operation in the system becomes reachable from any page the victim visits.

## Implementation Notes

- **Constraints**: Node.js runtime with Fastify; Vue single-page application built by Vite (`CLAUDE.md`). The cookie is `SameSite=Lax` and session-scoped (non-persistent) per SQ-3 (SEC-SESSION-5), recorded as named constants rather than literals scattered through the code. Because the client is a single-page application on the same origin (SQ-6 RESOLVED; CloudFront single public origin per the SQ-7 addendum), no cross-origin exemption is needed.
- **Prohibited Approaches**: Relying on `SameSite` alone, which SEC-HTTP-4 explicitly rejects. Reading the session from JavaScript for any purpose. Returning the token in a response body "for convenience". Applying forgery protection per handler rather than in the pipeline. Exempting a route because it is "internal" — REQ-API-040 already forbids internal routes in production.
- **Implementation Guidance**: Fastify hooks give one enforcement stage for both concerns, consistent with SEC-AUTHZ-5's single-enforcement-point reasoning. Since the client cannot read an `HttpOnly` cookie, the anti-forgery value needs its own delivery path — a separate readable token or a header the server issues — and whichever is chosen, its verification must be server-side against held state, never a comparison of two client-supplied values alone.
- **AI Development Guidance**: `REF-SESSION`, `REF-PROMPT-JWT`, `REF-PROMPT-VUE`, `REF-PROMPT-API`; `CLAUDE.md`.
- **Required Human Review**: Security review of the route inventory covering every state-changing route, and of the anti-forgery verification path for fail-closed behavior.
- **Open Decisions**: None from the specifications — SQ-3, SQ-6, and SQ-7 are all RESOLVED (`SameSite=Lax` session-scoped cookie; CORS disabled; CloudFront single-origin topology). The specific anti-forgery pattern is an implementation choice within this issue, but it MUST satisfy AC-04 against a genuine hostile origin rather than only a same-origin test harness. Standards mappings remain TO BE DECIDED until the SQ-10 pre-launch assessment.

**Estimated effort**: 1.5–2 engineer-days. **Estimated changed lines**: 350–700.
**Recommended model**: Claude Opus (`claude-opus-5`) — two coupled controls where implementing one without the other actively worsens the system's security posture.
