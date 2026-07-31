# Epic: Implement the specified fitness web application

## Metadata

- **ID**: REQ-EPIC-001
- **Title**: Implement the specified subscription fitness web application
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: Critical
- **Requirement Type**: Functional
- **Source / Parent**: `REQUIREMENTS.md` (all FR-*), `ARCHITECTURE.md`, `SECURITY.md`, `DESIGN.md`

## Requirement

- **Statement**: The system MUST implement the behavior specified in `REQUIREMENTS.md`, within the component boundaries of `ARCHITECTURE.md`, under the rules of `SECURITY.md`, and to the interface conventions of `DESIGN.md`. This epic is a container; all implementable behavior is decomposed into the child issues listed below and adds no behavior of its own.
- **Rationale**: The repository contains specifications only. A single traceable decomposition is required so that every specified behavior has exactly one implementation owner and every unresolved decision is visible rather than silently resolved during implementation.
- **Assumptions**: The four specification documents are the sole authority. `CLAUDE.md` and `AGENTS.md` are agent instructions, not product specification.
- **Out of Scope**: Any behavior not stated in the four specification documents. Any resolution of an `OQ-*`, `SQ-*`, `PQ-*`, `TO BE DECIDED`, or `UNKNOWN` marker.
- **Design Traceability**: `DESIGN.md` in full.
- **Architecture Traceability**: `ARCHITECTURE.md` — Browser Client, REST API Application, Identity and Session Handling, Relational Persistence; trust boundaries 1–5; dependency rules DR-1…DR-9.
- **Security Traceability**: `SECURITY.md` in full — SEC-TB-*, SEC-AUTHN-*, SEC-SESSION-*, SEC-AUTHZ-*, SEC-HTTP-*, SEC-INPUT-*, SEC-RENDER-*, SEC-DATA-*, SEC-OPS-*, SEC-SECRET-*, SEC-LOG-*, SEC-ERR-*, SEC-EXT-*, SEC-CICD-*, DEP-1…DEP-8.

## Scope

- **Applies To**: Multiple
- **Components**: Browser Client (Vue.js); REST API Application (Node.js); Identity and Session Handling; Relational Persistence (RDBMS)
- **Interfaces / Operations**: All specified workflows — registration and authentication, subscription entitlement, plan library and publication, plan browse and selection, plan customization, progress logging and review, data rights, consultant engagement, administration
- **Actors**: `subscriber`, `consultant`, `admin`, unauthenticated visitor, operator, CI/CD identity
- **Preconditions**: None
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED — `SECURITY.md` SQ-1 records an unresolved conflict between the US-federal/state framing, GDPR/CCPA rights, and HIPAA obligations.

## Security Context

- **Security Objectives**: Multiple
- **Control Layers**: Architecture, Authentication, Authorization, Session Management, Input Validation, Business-Rule Validation, Output Encoding, Sanitization, Data Protection, Logging and Monitoring, Availability, Supply Chain
- **Threat References**: `SECURITY.md` Threat Model, TM-S-1…TM-P-4 (STRIDE and LINDDUN, 2026-07-31)
- **Abuse / Misuse Case**: The full adversary set recorded in the threat model — anonymous attacker, malicious subscriber, malicious or compromised consultant, compromised admin, malicious or careless operator, compromised CI/CD identity or dependency.
- **Trust Boundary**: All five boundaries in `ARCHITECTURE.md`.
- **Untrusted Inputs or Assertions**: All client-supplied request data, identity, role, ownership, entitlement, and capability assertions.
- **Authoritative Enforcement Point**: REST API Application, with Identity and Session Handling for identity and role resolution.
- **Independent Verification**: Every authorization, entitlement, ownership, and validation decision is re-derived server-side from persisted state (SEC-TB-1, DR-2, DR-3).
- **Zero Trust Relevance**: NIST SP 800-207 — per-request access decisions for every resource request. Specific tenet mapping: TO BE DECIDED (not verified against the publication in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — ASVS 5.0.0 Level 3 is the stated assurance target (`SECURITY.md`, Security assurance target). No requirement-level mapping has been verified against `REF-ASVS-5`, and `SECURITY.md` SQ-10 leaves both the target and its verifier open.
- **OWASP AISVS 1.0**: N/A — no AI-enabled component is specified.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: TO BE DECIDED — blocked by `SECURITY.md` SQ-1 and `REQUIREMENTS.md` OQ-3.
- **Other**: RFC 2119 / RFC 8174 for normative keywords in every child issue.
- **Mapping Basis**: Only mappings verifiable from the cited documents are asserted. The specification documents name ASVS 5.0.0 as a target without a conformance claim, so no control-level mapping is asserted here.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given the child issues listed below, when every non-blocked child issue is closed as Verified, then every `REQUIREMENTS.md` FR marked `COVERED` in `ISSUE_PLAN.md` has passing acceptance tests traceable to its issue.
2. **AC-02 — Boundary or failure behavior**: Given a child issue whose scope depends on an unresolved `OQ-*`, `SQ-*`, or `PQ-*`, when implementation is attempted, then the issue remains blocked and no implementation resolves the open question in code.
3. **AC-03 — Prohibited behavior**: Given this epic, when child work is planned or executed, then no behavior outside `REQUIREMENTS.md`, `ARCHITECTURE.md`, `SECURITY.md`, or `DESIGN.md` is introduced, and no specification file is modified as part of implementation.

## Failure Behavior

- **On Invalid Input**: N/A — this issue introduces no runtime behavior.
- **On Authentication Failure**: N/A
- **On Authorization Failure**: N/A
- **On Security-Decision Failure**: N/A — delegated to child issues, each of which fails closed.
- **On External Dependency Failure**: N/A — no external integration exists (FR-9.8, SEC-EXT-1).
- **On System Error**: N/A
- **Logging / Audit**: N/A — delegated to REQ-AUDIT-010…040.
- **Alerting**: N/A

## Test Strategy

- **Unit Tests**: N/A at epic level.
- **Integration Tests**: N/A at epic level.
- **Security Tests**: N/A at epic level.
- **Compliance Tests / Evidence**: `ISSUE_PLAN.md` coverage table is the evidence that every FR maps to an issue or to a blocking question.
- **Acceptance-Criteria Traceability**: AC-01…AC-03 are verified by review of the coverage table and the child issue states.
- **Coverage Target**: TO BE DECIDED — Vitest is selected, but no coverage threshold has been set (`CLAUDE.md`, Repository state).
- **Required Test Environment**: N/A at epic level; the toolchain is recorded in `CLAUDE.md`, Repository state.

## Dependencies

- **Upstream Requirements**: None
- **Downstream Requirements**: All child issues listed in `ISSUE_PLAN.md`
- **External Dependencies**: None — the system is self-contained (FR-9.8)
- **Dependency Assumptions**: None
- **Failure Impact**: N/A

## Implementation Notes

- **Constraints**: Node.js server runtime and Vue.js client are fixed by `ARCHITECTURE.md`; REST over HTTPS; relational persistence; Terraform-managed AWS deployment; JWT session format (`SECURITY.md`). The toolchain was selected on 2026-07-31 and is recorded in `CLAUDE.md`, Repository state: TypeScript with npm workspaces, Fastify, PostgreSQL with Drizzle ORM and drizzle-kit, Vitest, ESLint with Prettier, GitHub Actions, and the `apps/api` / `apps/web` / `packages/shared` / `db/migrations` / `infra` layout. No implementation issue may revisit it; anything still marked open there MUST NOT be chosen inside an implementation issue.
- **Prohibited Approaches**: Client-side enforcement of any business rule (DR-2); resolving an open question by writing code that assumes one answer (`CLAUDE.md`, Working rules); creating "build the backend", "add security", or "write tests" issues.
- **Implementation Guidance**: Child issues are ordered topologically in `ISSUE_PLAN.md`. Cross-cutting enforcement issues (REQ-AUTHZ-*, REQ-API-*, REQ-AUDIT-*) precede the feature issues that depend on them.
- **AI Development Guidance**: `CLAUDE.md`; `SECURITY.md` prompt imports `REF-PROMPT-NODE`, `REF-PROMPT-VUE`, `REF-PROMPT-JWT`, `REF-PROMPT-API`, `REF-PROMPT-ABAC`, `REF-PROMPT-QUALITY`, `REF-PROMPT-TF-AWS`.
- **Required Human Review**: Security and privacy review before any production release; product review to close `REQUIREMENTS.md` OQ-*; legal/privacy review to close SQ-1 and OQ-3; architecture review to close SQ-2, SQ-4, SQ-7.
- **Open Decisions**: PQ-1…PQ-14 in `ISSUE_PLAN.md`; all `OQ-*` in `REQUIREMENTS.md` and `DESIGN.md`; all `SQ-*` in `SECURITY.md`.

## Child Issues

- [ ] https://github.com/jmanico/proof-of-work/issues/9 — Design tokens for color, typography, and spacing
- [ ] https://github.com/jmanico/proof-of-work/issues/10 — Responsive layout and reflow without loss of function
- [ ] https://github.com/jmanico/proof-of-work/issues/11 — Keyboard operability, focus management, and reduced motion baseline
- [ ] https://github.com/jmanico/proof-of-work/issues/12 — TLS enforcement and security response headers
- [ ] https://github.com/jmanico/proof-of-work/issues/13 — Allow-list schema validation of all untrusted input
- [ ] https://github.com/jmanico/proof-of-work/issues/14 — Server-controlled field binding (mass-assignment protection)
- [ ] https://github.com/jmanico/proof-of-work/issues/15 — Parameterized database access
- [ ] https://github.com/jmanico/proof-of-work/issues/16 — Error response hygiene and diagnostic endpoint exclusion
- [ ] https://github.com/jmanico/proof-of-work/issues/19 — Deny-by-default authentication requirement on protected operations
- [ ] https://github.com/jmanico/proof-of-work/issues/20 — Object-level ownership scoping
- [ ] https://github.com/jmanico/proof-of-work/issues/22 — Admin-only restriction on plan lifecycle operations
- [ ] https://github.com/jmanico/proof-of-work/issues/23 — Authorization denial response and logging semantics
- [ ] https://github.com/jmanico/proof-of-work/issues/17 — JWT signature, algorithm, and claim verification
- [ ] https://github.com/jmanico/proof-of-work/issues/18 — Token claim allow-list excluding sensitive data
- [ ] https://github.com/jmanico/proof-of-work/issues/21 — Exactly one role per account
- [ ] https://github.com/jmanico/proof-of-work/issues/28 — Passkey authentication for admin and consultant accounts
- [ ] https://github.com/jmanico/proof-of-work/issues/29 — Passkey registration and replacement with re-authentication
- [ ] https://github.com/jmanico/proof-of-work/issues/30 — Non-disclosing authentication failure responses
- [ ] https://github.com/jmanico/proof-of-work/issues/31 — Authentication and account-change security event logging
- [ ] https://github.com/jmanico/proof-of-work/issues/25 — Audit entry model and append-only enforcement
- [ ] https://github.com/jmanico/proof-of-work/issues/26 — Mandatory audit write on every health-data access path
- [ ] https://github.com/jmanico/proof-of-work/issues/27 — Admin plan lifecycle audit entries
- [ ] https://github.com/jmanico/proof-of-work/issues/24 — Log redaction of health values, credentials, and tokens
- [ ] https://github.com/jmanico/proof-of-work/issues/34 — Consent capture before any health-data write
- [ ] https://github.com/jmanico/proof-of-work/issues/35 — Consent withdrawal blocks new health-data writes
- [ ] https://github.com/jmanico/proof-of-work/issues/36 — View and correct personal data
- [ ] https://github.com/jmanico/proof-of-work/issues/49 — Medical disclaimer acknowledgement before first plan use
- [ ] https://github.com/jmanico/proof-of-work/issues/33 — No external transmission of health data
- [ ] https://github.com/jmanico/proof-of-work/issues/32 — Response field minimization
- [ ] https://github.com/jmanico/proof-of-work/issues/37 — Exercise plan content model
- [ ] https://github.com/jmanico/proof-of-work/issues/38 — Diet plan content model with calorie and macronutrient targets
- [ ] https://github.com/jmanico/proof-of-work/issues/39 — Admin plan creation and editing
- [ ] https://github.com/jmanico/proof-of-work/issues/40 — Plan citation management with URL scheme validation
- [ ] https://github.com/jmanico/proof-of-work/issues/41 — Publication gate requiring citation and verification record
- [ ] https://github.com/jmanico/proof-of-work/issues/48 — Plan unpublication
- [ ] https://github.com/jmanico/proof-of-work/issues/43 — Browse and view published exercise plans
- [ ] https://github.com/jmanico/proof-of-work/issues/44 — Browse and view published diet plans
- [ ] https://github.com/jmanico/proof-of-work/issues/42 — Safe rendering of plan content and citation links
- [ ] https://github.com/jmanico/proof-of-work/issues/45 — Customize a published plan into a private copy
- [ ] https://github.com/jmanico/proof-of-work/issues/46 — Persist, list, and retrieve a subscriber's customized plans
- [ ] https://github.com/jmanico/proof-of-work/issues/47 — Customized copy stability when the source plan changes
- [ ] https://github.com/jmanico/proof-of-work/issues/51 — Body weight entry logging, editing, deletion, and backdating
- [ ] https://github.com/jmanico/proof-of-work/issues/52 — Workout completion logging
- [ ] https://github.com/jmanico/proof-of-work/issues/50 — Log entry validation and field-level error reporting
- [ ] https://github.com/jmanico/proof-of-work/issues/53 — Engagement-scoped consultant access
- [ ] https://github.com/jmanico/proof-of-work/issues/54 — Ending an engagement revokes consultant access
- [ ] https://github.com/jmanico/proof-of-work/issues/55 — Dependency policy and reproducible resolution
- [ ] https://github.com/jmanico/proof-of-work/issues/56 — Non-production environments contain no real health data

Blocked scope is listed in `ISSUE_PLAN.md` under Open Questions and Blocked Scope; no child issue exists for it.
