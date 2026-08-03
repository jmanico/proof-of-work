# [REQ-AUTHZ-060] Admin health-data prohibition and administrative account views

## Metadata

- **ID**: REQ-AUTHZ-060
- **Title**: Admin health-data prohibition and administrative account views
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-10.3, FR-9.12 (both added 2026-08-03); `SECURITY.md` SEC-AUTHZ-9, SEC-DATA-5; threat-model actor table (former admin-capability assumption made normative)

## Requirement

- **Statement**: The system MUST NOT allow an `admin` to read a subscriber's health data (FR-9.12: workout, food, body-weight, and body-measurement log entries; customized plan copies; active plan selections) through the application — the authorization policy module MUST map no admin capability over health-data resource types — while admin administration views MUST expose, and allow listing and searching subscriber accounts by, only the administrative account fields: email address, role, email-verification state, MFA-enabled state, consent state, subscription periods, and consultant engagements.
- **Rationale**: FR-10.3 makes the threat model's admin-capability assumption normative: the compromised-admin actor must have no application path to subscriber health data, while FR-3.5 (subscription grants) and FR-11.5 (engagement creation) still need admins to find and administer accounts. SEC-AUTHZ-9 places the prohibition in the policy module itself, and SEC-DATA-5 requires least-privilege response shapes with no bulk retrieval of subscriber data through any role.
- **Assumptions**: The central typed policy module and its capability mapping exist (REQ-AUTHZ-050); accounts carry exactly one role (REQ-AUTH-010); the FR-9.12 health-data definition is bound platform-wide (REQ-PRIVACY-070).
- **Out of Scope**: The subscription-grant and engagement-creation operations the views serve (REQ-ENTITLE-030, REQ-CONSULT-030); privileged invitation and deprovisioning flows on the People page (REQ-AUTH-140, REQ-AUTH-160, REQ-AUTH-170); admin investigative reads of audit entries, which reference data without containing it (SEC-LOG-5, REQ-AUDIT-010); operational emergencies below the application via break-glass (SEC-OPS-1, REQ-INFRA-050); consultant access to health data within engagements (REQ-CONSULT-010).
- **Design Traceability**: `DESIGN.md` — "People (admin)": accounts are listed and searched by administrative fields only — email, role, verification state, MFA on or off, consent state, subscription summary, and engagements (FR-10.3); no health data appears anywhere in admin chrome.
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application as the sole authority for authorization; single enforcement point; DR-2 (no rule exists only client-side), DR-3 (role from Identity and Session Handling), DR-4.
- **Security Traceability**: SEC-AUTHZ-9, SEC-DATA-5, SEC-AUTHZ-1, SEC-AUTHZ-5, SEC-AUTHZ-6, SEC-AUTHZ-7; SEC-LOG-3 (audit reads value-free); SEC-OPS-1 (the only sanctioned emergency path, below the application).

## Scope

- **Applies To**: Server-Side Application, API, Web Client
- **Components**: REST API Application (policy module, response serialization); Browser Client (People administration views)
- **Interfaces / Operations**: Admin account list and search; admin account detail; every route serving health-data resource types (log entries, plan copies, plan selections), which must be unreachable with an admin actor
- **Actors**: `admin` (constrained); `subscriber` (data subject); compromised admin account (adversary)
- **Preconditions**: An authenticated `admin` session for the administration views
- **Data Classification**: Restricted — the prohibition protects health data; the views expose Confidential account metadata
- **Personal or Regulated Data**: Health Data (protected by the prohibition); Personal Data (administrative account fields exposed by the views)
- **Jurisdiction / Regulatory Scope**: `SECURITY.md` SQ-1 regime set — GDPR/UK GDPR for EU/UK data subjects; CCPA/CPRA, Washington My Health My Data, and the FTC Health Breach Notification Rule for US users; HIPAA not applicable. This requirement implements data-minimization of internal access; section-level statute mapping: TO BE DECIDED (SQ-1 counsel review).

## Security Context

- **Security Objectives**: Confidentiality, Privacy, Authorization
- **Control Layers**: Authorization, Data Protection, Architecture
- **Threat References**: `SECURITY.md` threat-model actor table (compromised admin account), TM-I-3 (bulk retrieval of subscriber records through any role), TM-E-2 (policy gap grants unintended access); CWE-863 (incorrect authorization), CWE-359 (exposure of private personal information to an unauthorized actor)
- **Abuse / Misuse Case**: A compromised admin account attempts to read subscribers' logs, plan copies, or selections directly by identifier, through list or search endpoints, through over-wide response serialization on account views, or through search parameters that filter on health values; an attacker relies on a policy gap that maps an admin capability onto a health-data resource type.
- **Trust Boundary**: Boundary 1 — Browser Client → REST API Application; the prohibition is a property of the server-side policy module and response schemas, not of admin chrome.
- **Untrusted Inputs or Assertions**: Object identifiers, list/search parameters, and field-selection parameters supplied by the admin client; any client claim about purpose of access.
- **Authoritative Enforcement Point**: The single authorization enforcement point evaluating the typed policy module (SEC-AUTHZ-5); Fastify response serialization against declared schemas limits administration-view fields (SEC-DATA-5, `CLAUDE.md`).
- **Independent Verification**: The policy module maps capabilities from role plus relationship over persisted state (SEC-AUTHZ-6); no admin-mapped capability targets a health-data resource type, so no request parameter can create one; deny-overrides with missing-attribute denial (SEC-AUTHZ-7).
- **Zero Trust Relevance**: NIST SP 800-207 — least-privilege, per-request access decisions; administrative role grants no implicit access to subscriber resources. Exact tenet: TO BE DECIDED (SQ-10 assessment).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — per-issue mapping deferred to the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A — no AI-enabled component is involved.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — per `SECURITY.md` SQ-10.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: SQ-1 regime set as above; this control supports data-minimization obligations, but section-level citations are TO BE DECIDED pending the SQ-1 pre-launch counsel review.
- **Other**: `REF-PROMPT-ABAC`, `REF-API-2023`, `REF-ASVS-5` as cited by SEC-AUTHZ-9 context and SEC-DATA-5.
- **Mapping Basis**: SEC-AUTHZ-9 and SEC-DATA-5 name these references; CWE identifiers describe the incorrect-authorization and personal-information-exposure classes this requirement closes.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated `admin`, when they list or search subscriber accounts by email address, role, email-verification state, MFA-enabled state, consent state, subscription periods, or consultant engagements, then matching accounts are returned with only those administrative fields in the response, sufficient to perform FR-3.5 and FR-11.5 administration.
2. **AC-02 — Policy-module prohibition**: Given the policy module's capability mapping, when the authorization test suite enumerates every capability mapped to the `admin` role, then no mapped capability reads any FR-9.12 health-data resource type (log entries of any kind, customized plan copies, active plan selections), and this assertion is a standing test (SEC-AUTHZ-9).
3. **AC-03 — Prohibited behavior (direct access)**: Given an authenticated `admin` and any subscriber's health-data object identifier, when the admin requests it through any route — object fetch, list, search, or export of another account — then the request is denied with no health value disclosed, and the denial is logged (SEC-LOG-4).
4. **AC-04 — Boundary (response and search shape)**: Given the admin account list, search, and detail endpoints, when responses are serialized, then no health-data field appears in any response schema, and search or filter parameters outside the enumerated administrative fields are rejected by schema validation (SEC-INPUT-1, SEC-DATA-5) rather than ignored.
5. **AC-05 — Prohibited behavior (fail-open)**: Given an error or missing attribute during policy evaluation of an admin request touching a health-data resource type, when the decision is computed, then the result is denial (SEC-AUTHZ-7) — under no failure does the admin role gain a health-data read.

## Failure Behavior

- **On Invalid Input**: Reject malformed or non-enumerated search, filter, and field-selection parameters with a schema-validation error (SEC-INPUT-1); no query executes against health-data tables.
- **On Authentication Failure**: Deny per the deny-by-default guard (REQ-AUTHZ-010) with a uniform response (SEC-AUTHN-3).
- **On Authorization Failure**: Deny the operation; account existence MAY be visible to admins through the administrative views (listing is a sanctioned capability), but denial of a health-data request discloses no health value and no object contents.
- **On Security-Decision Failure**: Deny by default; a policy error, unresolvable attribute, or unrecognized resource type denies (SEC-AUTHZ-7).
- **On External Dependency Failure**: N/A — no external dependency; if persisted attribute state cannot be read, the decision denies.
- **On System Error**: Generic error with correlation identifier (SEC-ERR-1); no partial serialization may leak fields outside the declared schema.
- **Logging / Audit**: Authorization denials logged with actor, route, and reason class (SEC-LOG-4); admin reads of administrative account data are personal-data access — audited per FR-9.7's affected-subject model where applicable — and never contain health values (SEC-LOG-3); attempted admin health-data access is a first-order detection signal.
- **Alerting**: Repeated admin-role denials on health-data resource types alert the security lead as SEC-OPS-2 detection inputs (SQ-11 RESOLVED) — this pattern is the compromised-admin actor's signature.

## Test Strategy

- **Unit Tests**: Policy-module tests enumerating admin-mapped capabilities and asserting none targets a health-data resource type; deny-overrides and missing-attribute denial for admin requests; search-parameter allow-list validation.
- **Integration Tests**: Admin list/search/detail flows asserting response bodies contain exactly the enumerated administrative fields; end-to-end FR-3.5 and FR-11.5 administration flows proving the views are sufficient without health data.
- **Security Tests**: BOLA-style probes as an authenticated admin against subscriber log entries, plan copies, and selections by direct identifier, list, and search, asserting denial and zero health-value bytes in responses; response-shape fuzzing for field-selection and expansion parameters; fail-open probe injecting policy-evaluation errors and asserting denial.
- **Compliance Tests / Evidence**: The standing SEC-AUTHZ-9 policy-enumeration test result as data-minimization evidence for the SQ-1 counsel review.
- **Acceptance-Criteria Traceability**: AC-01 — administration-view integration suite; AC-02 — policy-enumeration unit suite; AC-03 — admin BOLA security suite; AC-04 — response-shape and parameter-rejection tests; AC-05 — fail-open denial tests.
- **Coverage Target**: Project coverage threshold is TO BE DECIDED (`CLAUDE.md`); every admin-reachable route serving subscriber data MUST appear in either the administrative-fields assertion or the denial assertion.
- **Required Test Environment**: Vitest; HTTP test client; fixtures for admin, subscriber (with health data across all FR-9.12 types), and consultant identities; the authorization test suite named as a merge-blocking CI gate (SEC-CICD-4).

## Dependencies

- **Upstream Requirements**: REQ-AUTHZ-050 (central typed policy module), REQ-AUTH-010 (exactly one role per account), REQ-PRIVACY-070 (FR-9.12 health-data definition)
- **Downstream Requirements**: REQ-ENTITLE-030 (admin subscription-period administration uses these views), REQ-CONSULT-030 (engagement creation names consultant and subscriber), REQ-AUTH-160 and REQ-AUTH-170 (People-page invitation and deprovisioning flows)
- **External Dependencies**: None
- **Dependency Assumptions**: REQ-AUTHZ-050 delivers a capability mapping that is exhaustively enumerable in tests; REQ-PRIVACY-070 fixes the health-data resource-type set the prohibition quantifies over.
- **Failure Impact**: A single admin-mapped health-data capability, or one over-wide response schema, converts every compromised admin account into a bulk health-data exfiltration channel (TM-I-3) and falsifies the threat model's actor analysis.

## Implementation Notes

- **Constraints**: Node.js/Fastify (`CLAUDE.md`): Fastify response serialization against declared schemas is the delivery mechanism for the administrative-fields-only shape (SEC-DATA-5); the prohibition lives in the typed policy module evaluated at the single preHandler enforcement point (SQ-4 RESOLVED).
- **Prohibited Approaches**: Filtering health fields out of responses after over-fetching; implementing the prohibition as client-side view logic (DR-2); per-endpoint bespoke admin checks outside the policy module (SEC-AUTHZ-5); an admin "support mode" or impersonation path that assumes a subscriber's capabilities; search parameters that accept arbitrary field names.
- **Implementation Guidance**: Express the prohibition structurally — the capability map for `admin` simply contains no entry whose resource type is a health-data type — so the SEC-AUTHZ-9 test is an enumeration, not a probe list; keep the administrative-field set a single shared constant used by both response schemas and search-parameter schemas so the two cannot drift.
- **AI Development Guidance**: `REF-PROMPT-ABAC`, `REF-PROMPT-API`, `REF-PROMPT-QUALITY`; `CLAUDE.md` (Fastify built-ins for serialization and hooks rather than per-endpoint reimplementation).
- **Required Human Review**: Security review of the policy-module mapping and every admin-reachable response schema; privacy review that the administrative-field set matches FR-10.3 exactly.
- **Open Decisions**: None — FR-10.3, FR-9.12, and SEC-AUTHZ-9 fix the behavior; per-issue standards and statute-section mappings await the SQ-10 assessment and SQ-1 counsel review.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 250–500.
**Recommended model**: Claude Opus (`claude-opus-5`) — a structural authorization prohibition whose single gap turns a compromised admin into a bulk health-data exfiltration channel.
