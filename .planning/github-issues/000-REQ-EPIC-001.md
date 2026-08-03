# Epic: Implement the specified fitness web application

## Metadata

- **ID**: REQ-EPIC-001
- **Title**: Implement the specified subscription fitness web application
- **Version**: 1.1.1
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Functional
- **Source / Parent**: `REQUIREMENTS.md` (all FR-*), `ARCHITECTURE.md`, `SECURITY.md`, `DESIGN.md`

## Requirement

- **Statement**: The system MUST implement the behavior specified in `REQUIREMENTS.md`, within the component boundaries of `ARCHITECTURE.md`, under the rules of `SECURITY.md`, and to the interface conventions of `DESIGN.md`. This epic is a container; all implementable behavior is decomposed into the child issues listed below and adds no behavior of its own.
- **Rationale**: The repository contains specifications only. A single traceable decomposition is required so that every specified behavior has exactly one implementation owner and every unresolved decision is visible rather than silently resolved during implementation.
- **Assumptions**: The four specification documents are the sole authority. `CLAUDE.md` and `AGENTS.md` are agent instructions, not product specification.
- **Out of Scope**: Any behavior not stated in the four specification documents. Any resolution of a marker that remains open — the `TO BE DECIDED` items in `CLAUDE.md` (runner commands, local development instructions), the `TO BE DECIDED` Identity module-versus-service decision in `ARCHITECTURE.md` (module decomposition was RESOLVED 2026-08-03), the `UNKNOWN` scale targets, and the deferred `REQUIREMENTS.md` OQ-18 payments decision. The coverage threshold (90% line and branch) and branch/PR workflow were fixed in `CLAUDE.md` 2026-08-03.
- **Design Traceability**: `DESIGN.md` in full.
- **Architecture Traceability**: `ARCHITECTURE.md` — Browser Client, REST API Application, Identity and Session Handling, Relational Persistence, AI Inference (in-boundary); trust boundaries 1–6; dependency rules DR-1…DR-9; the scheduled executions of the REST API Application (nightly audit archival, deletion-ledger maintenance, FR-9.11 window expiry).
- **Security Traceability**: `SECURITY.md` in full — SEC-TB-*, SEC-AUTHN-*, SEC-SESSION-*, SEC-AUTHZ-*, SEC-HTTP-*, SEC-INPUT-*, SEC-RENDER-*, SEC-DATA-*, SEC-OPS-*, SEC-SECRET-*, SEC-LOG-*, SEC-ERR-*, SEC-AI-*, SEC-EXT-*, SEC-CICD-*, DEP-1…DEP-8.

## Scope

- **Applies To**: Multiple
- **Components**: Browser Client (Vue.js); REST API Application (Node.js); Identity and Session Handling; Relational Persistence (RDBMS); AI Inference (in-boundary, added 2026-08-01 by the OQ-5 resolution)
- **Interfaces / Operations**: All specified workflows — registration and authentication, subscription entitlement, plan library and publication, plan browse and selection, plan customization, progress logging and review, food logging and AI-assisted estimation, data rights, consultant engagement, administration
- **Actors**: `subscriber`, `consultant`, `admin`, unauthenticated visitor, operator, CI/CD identity
- **Preconditions**: None
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data
- **Jurisdiction / Regulatory Scope**: RESOLVED (`SECURITY.md` SQ-1; `REQUIREMENTS.md` OQ-3) — GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable (no covered-entity or business-associate relationship). GDPR-grade data-subject rights are granted to all users; single US primary region with standard lawful-transfer mechanisms. Counsel review before launch is required.

## Security Context

- **Security Objectives**: Multiple
- **Control Layers**: Architecture, Authentication, Authorization, Session Management, Input Validation, Business-Rule Validation, Output Encoding, Sanitization, Data Protection, Logging and Monitoring, Availability, Supply Chain
- **Threat References**: `SECURITY.md` Threat Model, TM-S-1…TM-P-4 (STRIDE and LINDDUN, 2026-07-31), plus the 2026-08-01 AI-inference addendum and the 2026-08-03 readiness-review addendum
- **Abuse / Misuse Case**: The full adversary set recorded in the threat model — anonymous attacker, malicious subscriber, malicious or compromised consultant, compromised admin, malicious or careless operator, compromised CI/CD identity or dependency.
- **Trust Boundary**: All six boundaries in `ARCHITECTURE.md`, including boundary 6 (REST API Application → AI Inference).
- **Untrusted Inputs or Assertions**: All client-supplied request data, identity, role, ownership, entitlement, and capability assertions; AI model inputs and outputs (SEC-AI-2).
- **Authoritative Enforcement Point**: REST API Application, with Identity and Session Handling for identity and role resolution.
- **Independent Verification**: Every authorization, entitlement, ownership, and validation decision is re-derived server-side from persisted state (SEC-TB-1, DR-2, DR-3).
- **Zero Trust Relevance**: NIST SP 800-207 — per-request access decisions for every resource request. Specific tenet mapping: TO BE DECIDED (not verified against the publication in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED at requirement level — ASVS 5.0.0 Level 3 is the confirmed design target (`SECURITY.md` SQ-10 RESOLVED); per-issue mappings stay `TO BE DECIDED` until verified during the independent third-party assessment before launch, per `REQUIREMENT_TEMPLATE.md`'s no-guessing rule.
- **OWASP AISVS 1.0**: TO BE DECIDED — an AI-enabled component exists (FR-8.12, FR-8.13; OQ-5 RESOLVED 2026-08-01); AISVS mappings are verified with the SQ-10 assessment (SEC-AI-3).
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: The SQ-1 regime set — GDPR/UK GDPR (EU/UK data subjects); CCPA/CPRA, Washington My Health My Data and comparable state consumer-health laws, FTC Health Breach Notification Rule (US users); HIPAA not applicable. Statute-section precision beyond what the specifications state (e.g. GDPR Art. 12(6) in FR-9.11; the 72-hour and 60-day notification clocks in SEC-OPS-2): TO BE DECIDED per issue.
- **Other**: RFC 2119 / RFC 8174 for normative keywords in every child issue.
- **Mapping Basis**: Only mappings verifiable from the cited documents are asserted. The specification documents name ASVS 5.0.0 Level 3 as a design target without a conformance claim; conformance may be asserted only after the SQ-10 independent assessment passes.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given the full decomposition — the 62 filed leaf issues listed below plus the 38 leaves planned in the 2026-08-03 refresh (filed as sub-issues after this branch merges) — when every child issue is closed as Verified, then every one of the 83 functional requirements in `REQUIREMENTS.md` and every SEC-* rule in `SECURITY.md` has passing acceptance tests traceable to at least one child issue, per the `ISSUE_PLAN.md` coverage matrices. Closing only the previously filed subset does not satisfy this criterion.
2. **AC-02 — Boundary or failure behavior**: Given work that depends on a marker that remains open — a `TO BE DECIDED` or `UNKNOWN` in `CLAUDE.md` or `ARCHITECTURE.md`, or the deferred OQ-18 — when implementation is attempted, then the dependent part remains blocked and no implementation resolves the open marker in code.
3. **AC-03 — Prohibited behavior**: Given this epic, when child work is planned or executed, then no behavior outside `REQUIREMENTS.md`, `ARCHITECTURE.md`, `SECURITY.md`, or `DESIGN.md` is introduced, and no specification file is modified as part of implementation.

## Failure Behavior

- **On Invalid Input**: N/A — this issue introduces no runtime behavior.
- **On Authentication Failure**: N/A
- **On Authorization Failure**: N/A
- **On Security-Decision Failure**: N/A — delegated to child issues, each of which fails closed.
- **On External Dependency Failure**: N/A — no runtime third-party integration exists (FR-9.8, SEC-EXT-1); the in-account services (Amazon Bedrock, SEC-AI-1; Amazon SES, SEC-EXT-3) are governed by their child issues.
- **On System Error**: N/A
- **Logging / Audit**: N/A — delegated to REQ-AUDIT-010…040 and the audit-bearing feature issues.
- **Alerting**: N/A at epic level — alert conditions belong to child issues; destinations and process are fixed (`SECURITY.md` SQ-11 RESOLVED: threshold alerts route to the security lead as SEC-OPS-2 detection inputs).

## Test Strategy

- **Unit Tests**: N/A at epic level.
- **Integration Tests**: N/A at epic level.
- **Security Tests**: N/A at epic level.
- **Compliance Tests / Evidence**: `ISSUE_PLAN.md` coverage matrices are the evidence that every FR and every SEC rule maps to a filed or planned issue.
- **Acceptance-Criteria Traceability**: AC-01…AC-03 are verified by review of the coverage matrices and the child issue states.
- **Coverage Target**: 90% line and branch coverage (`CLAUDE.md`, Repository state; fixed 2026-08-03).
- **Required Test Environment**: N/A at epic level; the toolchain is recorded in `CLAUDE.md`, Repository state.

## Dependencies

- **Upstream Requirements**: None
- **Downstream Requirements**: All child issues listed in `ISSUE_PLAN.md`, including the 38 leaves planned 2026-08-03
- **External Dependencies**: None — the system is self-contained (FR-9.8); Bedrock and SES are in-account services (SEC-AI-1, SEC-EXT-3)
- **Dependency Assumptions**: None
- **Failure Impact**: N/A

## Implementation Notes

- **Constraints**: Node.js server runtime and Vue.js client are fixed by `ARCHITECTURE.md`; REST over HTTPS; relational persistence; Terraform-managed AWS deployment; JWT session format (`SECURITY.md`). The toolchain is recorded in `CLAUDE.md`, Repository state: TypeScript with npm workspaces, Fastify, PostgreSQL with Drizzle ORM and drizzle-kit, Vitest, ESLint with Prettier, GitHub Actions, and the `apps/api` / `apps/web` / `packages/shared` / `db/migrations` / `infra` layout. No implementation issue may revisit it; anything still marked open there MUST NOT be chosen inside an implementation issue.
- **Prohibited Approaches**: Client-side enforcement of any business rule (DR-2); resolving an open marker by writing code that assumes one answer (`CLAUDE.md`, Working rules); creating "build the backend", "add security", or "write tests" issues.
- **Implementation Guidance**: Child issues are ordered topologically in `ISSUE_PLAN.md`. Cross-cutting enforcement issues (REQ-AUTHZ-*, REQ-API-*, REQ-AUDIT-*) precede the feature issues that depend on them; the infrastructure leaves (REQ-INFRA-*) deliver the SQ-7 topology those issues deploy onto.
- **AI Development Guidance**: `CLAUDE.md`; `SECURITY.md` prompt imports `REF-PROMPT-NODE`, `REF-PROMPT-VUE`, `REF-PROMPT-JWT`, `REF-PROMPT-API`, `REF-PROMPT-ABAC`, `REF-PROMPT-QUALITY`, `REF-PROMPT-TF-AWS`.
- **Required Human Review**: Security and privacy review before any production release; privacy-counsel review before launch (SQ-1); the post-implementation threat-model re-run validated by product and privacy counsel (SQ-9); the independent third-party ASVS 5.0.0 Level 3 assessment before any conformance claim (SQ-10).
- **Open Decisions**: Runner commands and local development instructions (`CLAUDE.md`; the coverage threshold, branch/PR workflow, and REST API module decomposition were fixed 2026-08-03); the Identity module-versus-service question (`ARCHITECTURE.md`, DR-8); concrete load, latency, and availability targets (`ARCHITECTURE.md`, UNKNOWN); per-issue ASVS 5.0.0 / AISVS 1.0 / NIST SP 800-53 mappings and statute-section precision (SQ-10, SQ-1); `REQUIREMENTS.md` OQ-18 (deferred payments).

## Child Issues

- [ ] https://github.com/jmanico/proof-of-work/issues/60 — Workspace scaffolding and toolchain baseline
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
- [ ] https://github.com/jmanico/proof-of-work/issues/66 — Server-side session records and per-request resolution
- [ ] https://github.com/jmanico/proof-of-work/issues/67 — Logout and session revocation on credential or authorization change
- [ ] https://github.com/jmanico/proof-of-work/issues/68 — Cookie session transport and cross-site request forgery protection
- [ ] https://github.com/jmanico/proof-of-work/issues/69 — Anti-automation throttling on authentication and recovery paths
- [ ] https://github.com/jmanico/proof-of-work/issues/70 — Password credential storage with Argon2id
- [ ] https://github.com/jmanico/proof-of-work/issues/71 — Subscriber registration with email and password
- [ ] https://github.com/jmanico/proof-of-work/issues/72 — Email verification and the health-data write gate
- [ ] https://github.com/jmanico/proof-of-work/issues/73 — Subscriber password authentication
- [ ] https://github.com/jmanico/proof-of-work/issues/74 — MFA enrolment, recovery codes, and disablement
- [ ] https://github.com/jmanico/proof-of-work/issues/75 — MFA challenge and recovery-code redemption
- [ ] https://github.com/jmanico/proof-of-work/issues/76 — Password reset
- [ ] https://github.com/jmanico/proof-of-work/issues/77 — Privileged provisioning by invitation and first passkey enrolment
- [ ] https://github.com/jmanico/proof-of-work/issues/78 — Privileged account minimums and passkey recovery

Planned leaves from the 2026-08-03 refresh, filed as sub-issues after this branch merges (drafts in `.planning/github-issues/`):

- [ ] REQ-ENTITLE-010 — Subscription entitlement gate on plan, customization, and progress access
- [ ] REQ-ENTITLE-020 — View own subscription status
- [ ] REQ-ENTITLE-030 — Admin subscription-period grant, extension, and revocation
- [ ] REQ-ENTITLE-040 — Record retention across subscription lapse
- [ ] REQ-SELECT-010 — Active exercise plan selection
- [ ] REQ-SELECT-020 — Active diet plan selection
- [ ] REQ-SELECT-030 — Unpublication ends active selections
- [ ] REQ-PLAN-070 — One-time plan verification operation
- [ ] REQ-PLAN-080 — Admin exercise catalog management
- [ ] REQ-AUTHZ-060 — Admin health-data prohibition and administrative account views
- [ ] REQ-PROGRESS-040 — Body measurement entry logging
- [ ] REQ-PROGRESS-050 — Per-account unit system and display-only conversion
- [ ] REQ-PROGRESS-060 — Progress history display with trend charts and paired tables
- [ ] REQ-FOOD-010 — Bundled nutrition dataset import, versioning, and search
- [ ] REQ-FOOD-020 — Food log entry with calorie and macronutrient attribution
- [ ] REQ-FOOD-030 — Daily intake versus selected diet plan targets
- [ ] REQ-FOOD-040 — AI-assisted nutrition estimation flow
- [ ] REQ-FOOD-050 — In-boundary inference service configuration
- [ ] REQ-PRIVACY-070 — Health-data definition binding the consent, verification, audit, and admin gates
- [ ] REQ-PRIVACY-080 — Synchronous JSON data export
- [ ] REQ-PRIVACY-090 — Synchronous account deletion with deletion-ledger write
- [ ] REQ-PRIVACY-100 — Audit tombstoning on account deletion
- [ ] REQ-PRIVACY-110 — Out-of-band deletion channel
- [ ] REQ-AUTH-160 — Vetting record required on privileged invitations
- [ ] REQ-AUTH-170 — Privileged deprovisioning
- [ ] REQ-AUTH-180 — Email-address change
- [ ] REQ-AUTHZ-050 — Central typed authorization policy module
- [ ] REQ-CONSULT-030 — Consultant engagement lifecycle by admin action
- [ ] REQ-CONSULT-040 — Consultant capabilities within an active engagement
- [ ] REQ-API-050 — Rate limits, body-size limits, and time-bounded request handling
- [ ] REQ-INFRA-010 — Environment accounts, pipeline identities, and deployment flow
- [ ] REQ-INFRA-020 — Network tiering and encryption at rest and in transit
- [ ] REQ-INFRA-030 — Secrets and signing-key management
- [ ] REQ-INFRA-040 — Audit retention, append-only privileges, and the hash-chained archive
- [ ] REQ-INFRA-050 — Break-glass operational access
- [ ] REQ-INFRA-060 — Transactional email delivery via in-account SES
- [ ] REQ-INFRA-070 — Merge-blocking CI security gates
- [ ] REQ-OPS-010 — Incident-response runbook and readiness

No blocked scope remains: every specified behavior maps to a filed or planned leaf (`ISSUE_PLAN.md`, coverage matrices).
