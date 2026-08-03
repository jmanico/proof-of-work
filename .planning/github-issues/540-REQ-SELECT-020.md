# [REQ-SELECT-020] Active diet plan selection

## Metadata

- **ID**: REQ-SELECT-020
- **Title**: Active diet plan selection
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-6.3, FR-6.4, FR-9.12 (OQ-6 RESOLVED); `SECURITY.md` SEC-AUTHZ-2, SEC-INPUT-3

## Requirement

- **Statement**: The system MUST maintain at most one active diet plan selection per subscriber, where the selection names either a published diet plan or one of that subscriber's own customized diet plan copies; selecting another MUST replace the current selection, neither selection nor replacement may alter any logged history, and the FR-8.5 target comparison MUST read the currently selected diet plan.
- **Rationale**: FR-6.3 grants the subscriber the ability to select a diet plan to follow; FR-6.4 fixes the cardinality (at most one active diet selection), the permitted targets, replacement semantics, the history invariant, and names this selection as the source of the FR-8.5 daily target comparison. FR-9.12 classifies an active plan selection as health data, so creating or replacing one is a health-data write requiring prior consent and a verified email address, audited per FR-9.7.
- **Assumptions**: Published diet plans with calorie and protein/carbohydrate/fat targets exist (FR-6.2, REQ-CATALOG-020); customized copies exist as subscriber-owned records (REQ-CUSTOM-010); the consent record (REQ-PRIVACY-010), email-verification gate (REQ-AUTH-090), medical-disclaimer acknowledgement (REQ-PRIVACY-040), subscription entitlement gate (REQ-ENTITLE-010), and the mandatory audit-write path (REQ-AUDIT-020) are enforced by their own requirements and are traversed by this operation.
- **Out of Scope**: Exercise plan selection (REQ-SELECT-010); server-side ending of selections at unpublication (FR-4.9, REQ-SELECT-030); rendering of the FR-8.5 comparison itself, including its no-target state (REQ-FOOD-030); food logging (REQ-FOOD-020); plan browsing (REQ-CATALOG-020); creating or editing customized copies (REQ-CUSTOM-010, REQ-CUSTOM-020).
- **Design Traceability**: `DESIGN.md` — "Cards, lists, and tables": selected plan cards use a leading check, "Selected" text, and `brand-soft`, never color alone; "Medical disclaimer and health-data consent": the FR-9.6 interstitial appears immediately before the first selection or customization; "Progress and target comparison": the daily comparison text and indicator draw their targets from this selection; "Design Verification": keyboard-only completion of plan selection.
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application (owner of all business objects; enforces owner scoping, entitlement, consent capture, and audit writes); Relational Persistence; data flow 4 (health-data write) and data flow 5 (progress viewing reads logged intake against the selected plan's targets); DR-2, DR-3, DR-4, DR-9.
- **Security Traceability**: SEC-AUTHZ-1, SEC-AUTHZ-2 (object-level authorization on the named plan or copy), SEC-INPUT-3 (owner and server-controlled fields never client-assignable), SEC-DATA-2 (consent precedes the write), SEC-AUTHN-8 (verified email precedes the write), SEC-LOG-1 (audit entry).

## Scope

- **Applies To**: API, Server-Side Application, Web Client
- **Components**: REST API Application; Relational Persistence; Browser Client (selection UI and "Selected" state display)
- **Interfaces / Operations**: Create/replace active diet plan selection; read current selection; the plan-detail and plan-list selection actions; the selection read performed by the FR-8.5 comparison
- **Actors**: Subscriber (owner); consultant (may view the engaged subscriber's selected plans under FR-11.6 — enforced by REQ-CONSULT-010/REQ-CONSULT-040, not here); admin (prohibited from reading selections by FR-10.3)
- **Preconditions**: Authenticated subscriber session; active subscription (FR-3.1); recorded health-data consent not withdrawn (FR-9.2, FR-9.9); verified email address (FR-2.11); medical-disclaimer acknowledgement on first plan use (FR-9.6)
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data — an active plan selection is health data by FR-9.12
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED)

## Security Context

- **Security Objectives**: Authorization, Integrity, Confidentiality, Privacy, Accountability
- **Control Layers**: Authorization, Input Validation, Business-Rule Validation, Data Protection, Logging and Monitoring
- **Threat References**: `SECURITY.md` TM-I-1 (BOLA/IDOR — selecting or reading via another subscriber's copy identifier), TM-T-1 (tampered payloads, mass assignment of server-controlled fields); CWE-639 (authorization bypass through user-controlled key), CWE-915 (improperly controlled modification of dynamically-determined object attributes)
- **Abuse / Misuse Case**: A malicious subscriber submits another subscriber's customized-copy identifier or an unpublished plan identifier to follow content they are not entitled to, to probe for the existence of other subscribers' copies, or to write a selection for another account; or a subscriber without consent or email verification attempts to create the health-data record.
- **Trust Boundary**: Boundary 1 — Browser Client → REST API Application; the plan or copy identifier and all request fields arrive untrusted.
- **Untrusted Inputs or Assertions**: The named plan or copy identifier; any client-supplied owner, subscriber, or selection-state field; any client assertion of consent, verification, entitlement, or disclaimer acknowledgement.
- **Authoritative Enforcement Point**: REST API Application — the single authorization enforcement point (SEC-AUTHZ-5) plus the selection write path, which resolves the target's publication status or ownership from persisted state before writing.
- **Independent Verification**: Ownership and publication status come from Relational Persistence; identity and role come from Identity and Session Handling (DR-3); consent, verification, and entitlement attributes come from persisted state per SEC-AUTHZ-6 — never from the client.
- **Zero Trust Relevance**: NIST SP 800-207 — access to the selection resource is decided per request from verified identity and persisted attributes, not client assertions. Exact tenet: TO BE DECIDED (not verified against the publication in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — per-issue mapping deferred to the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A — no AI-enabled component is involved in plan selection.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — per-issue mapping deferred to the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: GDPR / UK GDPR (EU/UK data subjects), CCPA/CPRA, Washington My Health My Data, FTC Health Breach Notification Rule (US users) — the selection is health data (FR-9.12) collected under the FR-9.2 consent. Specific articles/sections: TO BE DECIDED (`SECURITY.md` SQ-1 counsel review).
- **Other**: `REF-API-2023` (broken object level authorization), `REF-PROMPT-API`, `REF-ASVS-5` as cited by SEC-AUTHZ-2 and SEC-INPUT-3.
- **Mapping Basis**: SEC-AUTHZ-2 and SEC-INPUT-3 name these references; the CWE identifiers describe the user-controlled-key and mass-assignment classes this requirement defends against.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a subscriber with an active subscription, recorded consent, a verified email address, and prior disclaimer acknowledgement, when they select a published diet plan, then the selection is stored naming that plan, it is that subscriber's only active diet plan selection, the response reflects the selected state, and exactly one audit entry records the acting account, the action, the affected subject, and the time (FR-9.7).
2. **AC-02 — Replacement and comparison source**: Given the same subscriber already has an active diet plan selection, when they select one of their own customized diet plan copies, then the previous selection is replaced, exactly one active diet plan selection exists afterwards, no log entry is created, altered, or deleted by the operation, and a subsequent FR-8.5 comparison read resolves the newly selected plan's calorie and protein/carbohydrate/fat targets.
3. **AC-03 — Boundary or failure behavior**: Given a selection request naming an unpublished plan, a nonexistent identifier, or another subscriber's customized copy, when it is processed, then it is denied with no selection change, the denial does not disclose whether the identifier exists (SEC-AUTHZ-2), and the denial is logged per SEC-LOG-4.
4. **AC-04 — Gate failures**: Given a subscriber whose account lacks recorded consent, has withdrawn consent (FR-9.9), or has an unverified email address (FR-2.11), when they attempt to select any diet plan, then the write is refused as a health-data write (FR-9.12), no selection is stored, and the response states the exact reason and next step per `DESIGN.md` without exposing other data.
5. **AC-05 — Prohibited behavior**: Given any selection request, when the request body carries an owner identifier, another account's identifier, or any server-controlled selection field, then those fields MUST NOT take effect (SEC-INPUT-3); the operation MUST NOT create a second concurrent active diet selection for any account; and the operation MUST NOT modify the published plan or any plan copy.

## Failure Behavior

- **On Invalid Input**: Reject requests failing the route's allow-list schema (SEC-INPUT-1) with a structured validation error naming the failing field; no selection state changes.
- **On Authentication Failure**: Deny per the deny-by-default guard (REQ-AUTHZ-010) with the uniform response of SEC-AUTHN-3.
- **On Authorization Failure**: Deny the operation; the response MUST NOT reveal whether the named plan or copy exists when the actor is not entitled to know (SEC-AUTHZ-2); lapsed subscription returns the subscription-required state (FR-3.2).
- **On Security-Decision Failure**: Deny by default — a missing or unresolvable consent, verification, entitlement, or ownership attribute denies the write (SEC-AUTHZ-7).
- **On External Dependency Failure**: N/A — no external dependency; if Relational Persistence is unavailable the request fails closed with no partial state.
- **On System Error**: The selection replacement and its audit entry are written transactionally; on error, roll back so the prior selection remains intact; respond with a generic message plus correlation identifier (SEC-ERR-1).
- **Logging / Audit**: One FR-9.7 audit entry per selection read or write (acting account, action, affected subject, time); authorization denials and gate refusals logged per SEC-LOG-4; no plan content, targets, or health values in logs (SEC-LOG-3).
- **Alerting**: Repeated authorization denials and gate-refusal patterns are SEC-LOG-4 detection inputs; threshold alerts route to the security lead per SEC-OPS-2 (`SECURITY.md` SQ-3, SQ-11).

## Test Strategy

- **Unit Tests**: Selection cardinality rule (at most one active, replacement semantics); target-validity rule (published diet plan or own copy only); gate evaluation for consent, withdrawal, verification, and entitlement each independently denying; allow-list field binding ignoring server-controlled fields; the selection-resolution function used by the FR-8.5 comparison returning the current selection's targets.
- **Integration Tests**: End-to-end select and replace against persistence asserting exactly one active selection and untouched log-entry rows (byte-for-byte comparison before/after); FR-8.5 comparison read reflecting the replacement; audit entry emitted with the four FR-9.7 fields; transactional rollback on injected write failure.
- **Security Tests**: BOLA suite — subscriber A selects subscriber B's copy identifier, asserting denial with no existence disclosure; mass-assignment suite submitting owner and selection-state fields; unpublished-plan selection attempts; unauthenticated and lapsed-subscription attempts.
- **Compliance Tests / Evidence**: Test evidence that no selection write occurs without recorded consent and a verified email (FR-9.2, FR-2.11, FR-9.12), retained for the SQ-1 counsel review.
- **Acceptance-Criteria Traceability**: AC-01 — happy-path integration test; AC-02 — replacement, history-invariance, and comparison-source integration tests; AC-03 — BOLA/boundary security suite; AC-04 — gate-failure unit and integration tests; AC-05 — mass-assignment and cardinality suites.
- **Coverage Target**: Project threshold TO BE DECIDED (`CLAUDE.md`); all authorization, gate, and error paths MUST have positive and negative coverage.
- **Required Test Environment**: Vitest and HTTP test client; fixtures for two subscribers with published diet plans carrying FR-6.2 targets, own and foreign customized copies, and accounts in each gate state (unconsented, withdrawn, unverified, lapsed).

## Dependencies

- **Upstream Requirements**: REQ-CATALOG-020 (published diet plans browsable with targets), REQ-CUSTOM-010 (customized copies exist and are owned), REQ-PRIVACY-010 (consent capture), REQ-PRIVACY-040 (disclaimer acknowledgement gates first plan use), REQ-AUTH-090 (email-verification write gate), REQ-ENTITLE-010 (subscription entitlement gate), REQ-AUDIT-020 (mandatory audit write on health-data paths)
- **Downstream Requirements**: REQ-SELECT-030 (unpublication ends selections), REQ-FOOD-030 (daily intake versus the selected plan's targets), REQ-CONSULT-040 (consultant views selected plans)
- **External Dependencies**: None
- **Dependency Assumptions**: The authorization enforcement point supplies verified subject attributes (consent, verification, subscription) from persisted state per SEC-AUTHZ-6; the audit path is a dependency of this write per DR-9, not an optional call.
- **Failure Impact**: If gate or ownership checks are missing, a subscriber can bind health data without consent or follow another subscriber's private copy; if replacement is non-atomic or cardinality unenforced, the FR-8.5 comparison has no single authoritative target source and its output becomes ambiguous.

## Implementation Notes

- **Constraints**: TypeScript on Fastify with route-level JSON Schema validation and the single preHandler enforcement point (`CLAUDE.md`); selection storage in PostgreSQL via Drizzle with a schema-level uniqueness guarantee of at most one active diet selection per subscriber.
- **Prohibited Approaches**: Client-supplied owner or selection-state fields (SEC-INPUT-3); filtering foreign copies after retrieval instead of constraining the query (SEC-AUTHZ-2); deleting or rewriting log entries as part of replacement; enforcing cardinality only in application code without a persistence-level constraint; skipping the audit write on any path (DR-9); duplicating target values into the comparison instead of reading the current selection.
- **Implementation Guidance**: Share the selection mechanism with REQ-SELECT-010 keyed on (subscriber, plan-type) so both invariants are one structural rule; expose one selection-resolution function that REQ-FOOD-030 consumes, so "the currently selected diet plan" has a single reader.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-ABAC`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the target-validity query and the gate ordering; privacy review that the selection write is gated identically to other FR-9.12 health-data writes.
- **Open Decisions**: None — OQ-6, the FR-6.2 macronutrient set, and the FR-9.12 health-data boundary are resolved; per-issue standards mappings await the SQ-10 assessment.

**Estimated effort**: 0.5–1 engineer-days. **Estimated changed lines**: 200–400.
**Recommended model**: Claude Opus (`claude-opus-5`) — same security-critical gate and ownership surface as REQ-SELECT-010, largely shared machinery plus the FR-8.5 source-of-truth hook.
