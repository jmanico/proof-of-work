# [REQ-PRIVACY-070] Health-data definition binding the consent, verification, audit, and admin gates

## Metadata

- **ID**: REQ-PRIVACY-070
- **Title**: Health-data definition binding the consent, verification, audit, and admin gates
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Privacy
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-9.12; `SECURITY.md` SEC-AUTHZ-9 (consumer), SEC-AUTHZ-6 (attribute schema)

## Requirement

- **Statement**: The REST API Application MUST classify data through a single shared, typed health-data definition implementing FR-9.12 — workout, food, body-weight, and body-measurement log entries, customized plan copies, and active plan selections are health data; creating or changing any of these is a health-data write, reading any of them is a health-data access, and an AI estimation request (FR-8.12) is collection and processing of health data gated identically; account, subscription, and engagement records are personal data but not health data — and every gate that depends on the health-data boundary (FR-2.11 email verification, FR-9.2 consent, FR-9.7 audit, FR-9.9 withdrawal, FR-10.3 admin prohibition) MUST consume this definition rather than maintaining its own list.
- **Rationale**: FR-9.12 (added 2026-08-03) fixes the health-data boundary that the consent, verification, audit, and admin-prohibition gates depend on. Without one shared definition, each gate re-derives the boundary independently and a new resource type can silently fall outside one gate while inside another — for example, a plan selection that is audited but not consent-gated. A single typed definition makes the boundary a reviewable, testable artifact.
- **Assumptions**: The authorization policy module (REQ-AUTHZ-050; `SECURITY.md` SQ-4 RESOLVED) exists and its resource-type attribute is the natural consumption point for this classification.
- **Out of Scope**: The behavior of each consuming gate — consent capture (REQ-PRIVACY-010), consent withdrawal (REQ-PRIVACY-020), email-verification gating of health-data writes (REQ-AUTH-090), the mandatory audit write (REQ-AUDIT-020), the admin health-data prohibition (REQ-AUTHZ-060), and estimation-request gating (REQ-FOOD-040). This issue delivers only the shared definition those gates consume.
- **Design Traceability**: `DESIGN.md` — Medical disclaimer and health-data consent: the consent screen "names the categories collected", which are the categories this definition fixes. No further design decision is involved; the definition is server-side.
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application ("sole authority for business rules"; owns all listed business objects); Data model expectations entity list (the classification is exhaustive over it); DR-2 (no rule exists only in the client); DR-9 (audit as a dependency of every health-data access path presupposes knowing which paths those are).
- **Security Traceability**: SEC-AUTHZ-6 (resource-type attribute sourced from persisted state), SEC-AUTHZ-7 (unresolvable attribute denies), SEC-AUTHZ-9 (no admin capability maps to health-data reads as defined by FR-9.12); supports SEC-DATA-2, SEC-LOG-1.

## Scope

- **Applies To**: Server-Side Application, API
- **Components**: REST API Application (shared definition and the gates); Identity and Session Handling (FR-2.11 verification state is a consuming gate input)
- **Interfaces / Operations**: Every operation that creates, changes, or reads a log entry, plan copy, or active selection; every AI estimation request; every admin administration view
- **Actors**: Subscriber, consultant, admin; the definition governs how each gate treats them
- **Preconditions**: None — the definition is consulted before any gated operation proceeds
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED)

## Security Context

- **Security Objectives**: Privacy, Confidentiality, Accountability, Authorization
- **Control Layers**: Architecture, Authorization, Business-Rule Validation, Data Protection
- **Threat References**: `SECURITY.md` TM-P-2 (unawareness — consent scope must match what is actually collected), TM-I-5 (health data leaking through paths not recognized as health-bearing), TM-E-2 (policy gap grants unintended access); LINDDUN Nc (non-compliance when a gate misses a health-data category)
- **Abuse / Misuse Case**: A developer adds a new health-bearing resource type wired into only some gates, so writes occur without consent, reads escape audit, or an admin view exposes it; or an attacker targets an endpoint whose resource type was never classified and therefore bypasses the consent and audit gates.
- **Trust Boundary**: Internal to the REST API Application — the definition is the shared boundary all five gates evaluate against; it sits behind trust boundary 1 and is never derived from client input.
- **Untrusted Inputs or Assertions**: Any client-supplied claim about what category a record belongs to. Classification is by server-side resource type only (SEC-AUTHZ-6).
- **Authoritative Enforcement Point**: The shared typed definition inside the REST API Application, consumed by the single authorization enforcement point (SEC-AUTHZ-5) and by each gate.
- **Independent Verification**: Each gate resolves the classification from the shared definition and persisted resource type, never from the request (DR-3, SEC-AUTHZ-6).
- **Zero Trust Relevance**: NIST SP 800-207 — access decisions are made per resource using attributes from trusted sources. Exact tenet: TO BE DECIDED (not verified against the publication in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session (SQ-10 pre-launch assessment).
- **OWASP AISVS 1.0**: TO BE DECIDED — the estimation-request classification touches the AI component; mapping deferred with SQ-10.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: The SQ-1 regime set — GDPR/UK GDPR, CCPA/CPRA, Washington My Health My Data, FTC HBNR; HIPAA not applicable. The definition fixes which records are "consumer health data" for those regimes' purposes. Statute-section mappings: TO BE DECIDED (SQ-1 counsel review).
- **Other**: `REF-PROMPT-ABAC` (declared attribute sources), `REF-PROMPT-QUALITY` (centralized rather than per-endpoint rules) as applied by `SECURITY.md`.
- **Mapping Basis**: SEC-AUTHZ-6 requires every authorization attribute to have a declared source; FR-9.12 declares the health-data boundary those attributes and the privacy gates depend on.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given the shared definition, when any workout, food, body-weight, or body-measurement log entry, customized plan copy, or active plan selection is created, changed, or read, then the operation is classified as a health-data write or access respectively, and the consent (FR-9.2), email-verification (FR-2.11), withdrawal (FR-9.9), and audit (FR-9.7) gates all evaluate against that one classification.
2. **AC-02 — Boundary or failure behavior**: Given an account without recorded consent, with consent withdrawn, or with an unverified email address, when an AI estimation request is submitted, then it is refused as a health-data collection under the same gates even though nothing is stored until confirmation; and given an operation on account, subscription, or engagement records, when it executes, then it is classified as personal data but not health data and the health-data gates do not fire on it (for example, viewing subscription status requires no health-data consent).
3. **AC-03 — Prohibited behavior**: Given any consuming gate or the policy module, when its implementation is inspected and tested, then it MUST NOT contain an independently maintained health-data category list; and given a resource type absent from the classification, when a gated operation on it is attempted, then it MUST NOT be treated as non-health by default — the gate resolves to denial per SEC-AUTHZ-7's missing-attribute rule.
4. **AC-04 — Additional criterion**: Given the policy module (REQ-AUTHZ-050), when capabilities are mapped, then no `admin`-mapped capability reads any type this definition classifies as health data (SEC-AUTHZ-9), and an exhaustiveness check over the `ARCHITECTURE.md` entity list confirms every persisted business-object type carries exactly one classification.

## Failure Behavior

- **On Invalid Input**: N/A — the definition takes no untrusted input; classification operates on server-side resource types only. Malformed requests are rejected upstream per SEC-INPUT-1.
- **On Authentication Failure**: N/A — handled by REQ-AUTHZ-010 before any gate consults the definition.
- **On Authorization Failure**: The consuming gate denies per its own issue (REQ-PRIVACY-010, REQ-AUTH-090, REQ-AUTHZ-060, REQ-AUDIT-020); denial semantics follow REQ-AUTHZ-040.
- **On Security-Decision Failure**: Deny by default — an unresolvable or missing classification produces denial of the gated operation, never a permit (SEC-AUTHZ-7).
- **On External Dependency Failure**: N/A — the definition is in-process with no external dependency.
- **On System Error**: An error while resolving classification denies the gated operation with a generic response and correlation identifier (SEC-ERR-1); no gate proceeds on a partial classification.
- **Logging / Audit**: Gate denials are logged by the consuming gates with route, reason class, and correlation identifier (SEC-LOG-4); classified health-data accesses produce audit entries per REQ-AUDIT-020. Log content never includes health values (SEC-LOG-3).
- **Alerting**: N/A — the definition itself has no alert condition; threshold alerting on gate denials belongs to the consuming gates and routes to the security lead as SEC-OPS-2 detection inputs.

## Test Strategy

- **Unit Tests**: Classification of each enumerated type (four log-entry types, plan copy, active selection) as health data; account, subscription, and engagement records as personal-not-health; create/change classified as write and read as access; estimation request classified as collection; unclassified type resolves to denial, not non-health.
- **Integration Tests**: Each consuming gate (consent, verification, withdrawal, audit, admin prohibition) exercised against every health-data type through real endpoints, asserting the gate fires; subscription-status view exercised asserting the health gates do not fire.
- **Security Tests**: Attribute-injection test asserting a client-supplied category claim never alters classification (SEC-AUTHZ-6); policy-module capability review asserting no admin capability reaches a health-classified type (SEC-AUTHZ-9); static check that no gate module declares its own category list.
- **Compliance Tests / Evidence**: The exhaustiveness check over the `ARCHITECTURE.md` entity list, kept as a failing-by-default test when a new entity lacks classification; evidence for the SQ-1 counsel review that consent-screen categories match the definition.
- **Acceptance-Criteria Traceability**: AC-01 — classification unit suite plus gate integration suite; AC-02 — estimation-gating and personal-data suites; AC-03 — static single-source check and unclassified-type denial test; AC-04 — policy-capability review test and exhaustiveness check.
- **Coverage Target**: Project-defined; 100% of persisted business-object types covered by classification tests, positive and negative.
- **Required Test Environment**: Vitest; fixtures for accounts in each consent/verification state; the policy module test harness from REQ-AUTHZ-050.

## Dependencies

- **Upstream Requirements**: REQ-AUTHZ-050
- **Downstream Requirements**: REQ-PRIVACY-010, REQ-PRIVACY-020, REQ-AUTH-090, REQ-AUDIT-020, REQ-AUTHZ-060, REQ-FOOD-040, REQ-PRIVACY-080, REQ-PRIVACY-090
- **External Dependencies**: None
- **Dependency Assumptions**: The policy module exposes a typed resource attribute the definition can populate, and gates fail closed when it is absent (SEC-AUTHZ-7).
- **Failure Impact**: If the definition is wrong or bypassed, health data is written without consent, read without audit, or exposed to admins — a direct breach of FR-9.2, FR-9.7, and FR-10.3 and a notifiable event under the SQ-1 regimes.

## Implementation Notes

- **Constraints**: TypeScript in the `apps/api` workspace (`CLAUDE.md`). The definition MUST live in `apps/api`: `packages/shared` carries contract shape only, and a classification that exists only in shared or client code violates DR-2.
- **Prohibited Approaches**: Per-gate category lists; classifying from any client-supplied field; defaulting unknown types to non-health; encoding the boundary in database column naming conventions instead of a typed artifact.
- **Implementation Guidance**: Model the classification as a closed discriminated union over resource types with an exhaustiveness-checked mapping, so adding an entity type without classifying it is a compile-time error; expose one predicate pair (health-data write / health-data access) consumed by the Fastify enforcement point and the policy module (SEC-AUTHZ-5, SQ-4).
- **AI Development Guidance**: `REF-PROMPT-ABAC`, `REF-PROMPT-QUALITY`; `CLAUDE.md`. Human review of the classification mapping is mandatory before any gate consumes it.
- **Required Human Review**: Security and privacy review of the classification mapping and its consumption points; the SQ-1 counsel review confirms the boundary against the governing regimes before launch.
- **Open Decisions**: None — FR-9.12 fixes the boundary; per-issue standards mappings await the SQ-10 assessment.

**Estimated effort**: 0.5–1 engineer-days. **Estimated changed lines**: 100–250.
**Recommended model**: Claude Opus (`claude-opus-5`) — a small definitional module on which five privacy-critical gates depend; exhaustiveness and fail-closed semantics are the acceptance bar.
