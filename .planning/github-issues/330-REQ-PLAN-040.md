# [REQ-PLAN-040] Plan citation management with URL scheme validation

## Metadata

- **ID**: REQ-PLAN-040
- **Title**: Plan citation management with URL scheme validation
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-4.4, FR-4.6; `SECURITY.md` SEC-EXT-2, SEC-RENDER-3

## Requirement

- **Statement**: An `admin` MUST be able to attach citations to a plan and to remove them, every citation URL MUST be parsed and scheme-checked against an explicit allow-list before it is stored, and the server MUST NOT issue a request to a citation URL.
- **Rationale**: FR-4.4 requires every plan to carry at least one citation to a peer-reviewed source before publication, which requires citations to be managed as first-class records; FR-4.6 requires them to be displayed to the reader. SEC-EXT-2 forbids server-side requests to admin-supplied URLs without an allow-list, and SEC-RENDER-3 requires scheme checking before a URL is bound to a link.
- **Assumptions**: The plan content models exist (REQ-PLAN-010, REQ-PLAN-020) and carry citations as associated records.
- **Out of Scope**: The publication gate that requires at least one citation (REQ-PLAN-050); client-side rendering of citation links (REQ-CATALOG-030); how citations and verification status are presented in the interface (`DESIGN.md` OQ-5, open); any automated verification that a source is genuinely peer-reviewed, which no source document specifies and which would require an external service the system may not have (FR-9.8).
- **Design Traceability**: `DESIGN.md` — Components → Links ("Citation links on plan content are always visibly links, since the evidence behind a plan must be reachable by the reader"); Typography (xs step for citation attribution); `DESIGN.md` OQ-5 leaves the surfacing pattern open.
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application ("Owns … plan citations and verification records"); data flow 2 and 6.
- **Security Traceability**: SEC-EXT-2, SEC-RENDER-3, SEC-INPUT-1, SEC-INPUT-3, SEC-LOG-6 (citation changes are part of the audited edit lifecycle).

## Scope

- **Applies To**: Server-Side Application, API, Web Client
- **Components**: REST API Application; Relational Persistence; Browser Client (admin citation form)
- **Interfaces / Operations**: Add citation; remove citation; retrieve a plan's citations
- **Actors**: `admin` as author; `subscriber` as reader of published plans' citations
- **Preconditions**: An existing plan and an authenticated `admin` session
- **Data Classification**: Internal
- **Personal or Regulated Data**: None
- **Jurisdiction / Regulatory Scope**: N/A

## Security Context

- **Security Objectives**: Integrity, Confidentiality, Safety
- **Control Layers**: Input Validation, Output Encoding, Architecture, Business-Rule Validation
- **Threat References**: `SECURITY.md` TM-T-4 (stored XSS via citation URLs), TM-T-5 (harmful content published under a false evidence claim); CWE-918 (server-side request forgery), CWE-79 (cross-site scripting), CWE-601 (open redirect)
- **Abuse / Misuse Case**: A compromised admin stores a `javascript:` or `data:` citation URL that executes when a subscriber clicks it, or stores an internal address that the server fetches — turning the evidence field into an SSRF or XSS primitive on a page every subscriber reads.
- **Trust Boundary**: Boundary 1 — admin-supplied URLs are untrusted; the system egress boundary for SEC-EXT-2.
- **Untrusted Inputs or Assertions**: The citation URL, title, and any attribution text.
- **Authoritative Enforcement Point**: REST API Application — parse and scheme-check before storage; no outbound client accepts these URLs.
- **Independent Verification**: The scheme check is performed on the parsed URL, not on the raw string, and is re-applied at render time (REQ-CATALOG-030).
- **Zero Trust Relevance**: N/A — input validation and an egress prohibition, not a resource-access decision.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: N/A
- **Regulatory**: N/A
- **Other**: `REF-PC-2024` as cited by SEC-EXT-2; `REF-XSS` and `REF-PROMPT-VUE` as cited by SEC-RENDER-3. RFC 3986 for URL parsing semantics.
- **Mapping Basis**: SEC-EXT-2 and SEC-RENDER-3 name these references; RFC 3986 defines the parsing the scheme check depends on; the CWE identifiers name the SSRF, XSS, and redirect classes.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated `admin` and an existing plan, when they add a citation whose URL parses and whose scheme is on the allow-list, then the citation is persisted against the plan, is retrievable with the plan, and the change is audited as a plan edit (REQ-AUDIT-030).
2. **AC-02 — Boundary or failure behavior**: Given a citation URL that fails to parse, or whose scheme is not on the allow-list — including `javascript:`, `data:`, `vbscript:`, and `file:` — when it is submitted, then it is rejected with the failing field named and nothing is persisted.
3. **AC-03 — Prohibited behavior**: Given any citation URL, when it is added, edited, retrieved, or published, then the server MUST NOT issue a request to it for validation, preview, metadata, or link checking (SEC-EXT-2), and the scheme check MUST NOT be performed by substring matching on the raw string.
4. **AC-04 — Additional criterion**: Given a plan's last remaining citation, when removal is attempted while the plan is published, then the removal is refused, because a published plan must carry at least one citation (FR-4.4).
5. **AC-05 — Additional criterion**: Given a citation URL pointing at an internal, loopback, or cloud metadata address, when it is submitted, then it is stored or rejected according to the scheme allow-list but is never dereferenced, and no internal address is reachable through this field.

## Failure Behavior

- **On Invalid Input**: Reject per REQ-API-010 and the scheme check, naming the failing field; nothing persisted.
- **On Authentication Failure**: Denied upstream; admins authenticate by passkey (REQ-AUTH-020).
- **On Authorization Failure**: Denied for non-admin roles (REQ-AUTHZ-030).
- **On Security-Decision Failure**: If a URL cannot be parsed confidently, reject it (fail closed) rather than store it for the renderer to judge.
- **On External Dependency Failure**: N/A by construction — no external request is made, which is the point of AC-03.
- **On System Error**: Roll back so no citation is persisted without its audit entry.
- **Logging / Audit**: Citation changes are audited as plan edit actions (FR-10.2, REQ-AUDIT-030). Log rejected URLs by failure class; the URL itself is admin-supplied content and may be logged, but never a health value (SEC-LOG-3).
- **Alerting**: TO BE DECIDED — no alerting model exists in the source documents.

## Test Strategy

- **Unit Tests**: URL parser and scheme allow-list accept permitted schemes and reject `javascript:`, `data:`, `vbscript:`, `file:`, scheme-relative, and malformed inputs, including obfuscated and mixed-case variants.
- **Integration Tests**: Add and remove citations on both plan types; retrieve a plan and assert citations are present; assert the audit entry.
- **Security Tests**: SSRF suite asserting no outbound request for internal, loopback, and metadata URLs at every lifecycle point; XSS vector suite over citation URLs verified end to end with REQ-CATALOG-030; last-citation removal on a published plan refused.
- **Compliance Tests / Evidence**: Evidence that published plans carry citations, supporting FR-4.4 alongside REQ-PLAN-050.
- **Acceptance-Criteria Traceability**: AC-01 — add-and-retrieve suite; AC-02 — scheme rejection matrix; AC-03 — SSRF and parsing-method assertions; AC-04 — last-citation removal test; AC-05 — internal-address suite.
- **Coverage Target**: Every permitted and prohibited scheme covered; every citation operation covered positive and negative.
- **Required Test Environment**: An admin identity, seeded published and unpublished plans, and a network interception layer to prove no outbound request occurs. Engine and test framework TO BE DECIDED.

## Dependencies

- **Upstream Requirements**: REQ-PLAN-010, REQ-PLAN-020, REQ-PLAN-030, REQ-AUTHZ-030, REQ-API-010, REQ-PRIVACY-050, REQ-AUDIT-030
- **Downstream Requirements**: REQ-PLAN-050, REQ-CATALOG-030
- **External Dependencies**: None — no citation-checking or metadata service is in scope, and introducing one would cross the FR-9.8 boundary and require an explicit `REQUIREMENTS.md` change (SEC-EXT-1).
- **Dependency Assumptions**: The URL parsing library used is standards-conformant and does not itself perform network resolution.
- **Failure Impact**: This field is the most attacker-shaped surface in admin-authored content: it is a URL, it is rendered as a link to every subscriber, and it is stored by a role the threat model treats as compromisable.

## Implementation Notes

- **Constraints**: RDBMS engine and framework TO BE DECIDED. The scheme allow-list itself is not enumerated by any source document; `https` is the only scheme a peer-reviewed citation plausibly needs, but that choice must be recorded and confirmed rather than assumed to be settled.
- **Prohibited Approaches**: Fetching the URL to validate it, to check for link rot, or to build a preview; substring or regular-expression scheme checks on the raw string; storing an unvalidated URL and relying on the renderer alone; auto-correcting a malformed URL.
- **Implementation Guidance**: Perform the same scheme check at storage and at render (REQ-CATALOG-030), so a record written before a rule change cannot render as an active hostile link. `DESIGN.md` requires citation links to be visibly links; external links must not grant opener access to the application window (SEC-RENDER-3), which REQ-CATALOG-030 implements.
- **AI Development Guidance**: `REF-PC-2024`, `REF-XSS`, `REF-PROMPT-VUE`, `REF-PROMPT-API`; `CLAUDE.md`.
- **Required Human Review**: Security review of the scheme allow-list and the SSRF test coverage.
- **Open Decisions**: The permitted scheme set (not enumerated in any source document); how citations are surfaced in the interface (`DESIGN.md` OQ-5); whether any check that a source is genuinely peer-reviewed is required beyond admin judgement — FR-4.4 requires the citation, not its automated validation.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 250–500.
**Recommended model**: Claude Opus (`claude-opus-5`) — a small feature sitting on top of two high-severity injection classes, where the safe implementation differs subtly from the obvious one.
