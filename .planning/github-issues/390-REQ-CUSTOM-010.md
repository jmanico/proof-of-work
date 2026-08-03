# [REQ-CUSTOM-010] Customize a published plan into a private copy

## Metadata

- **ID**: REQ-CUSTOM-010
- **Title**: Customize a published plan into a private copy
- **Version**: 1.2.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-7.1, FR-7.2, FR-4.2, FR-5.4, FR-9.12; `SECURITY.md` SEC-INPUT-4, SEC-DATA-2

## Requirement

- **Statement**: A subscriber MUST be able to customize a published exercise or diet plan by editing its contents, and the system MUST save the customization as a private copy owned by that subscriber without modifying the published plan. For an exercise plan, customization selects among FR-5.4 catalog exercises and edits prescriptions (sets, repetitions, order) and MUST NOT introduce movements outside the catalog. Creating or changing a copy is a health-data write (FR-9.12) and MUST be refused without recorded consent and a verified email address.
- **Rationale**: FR-7.1 grants the customization; FR-7.2 requires copy-on-customize semantics and forbids mutating the published plan; SEC-INPUT-4 requires this as server-side validation rather than UI affordance. FR-4.2 forbids subscribers authoring or sharing plans, which bounds what a copy is: private, never publishable. FR-5.4 constrains exercise-plan composition to catalog references so FR-8.14 per-exercise trends keep a stable identity. FR-9.12 classifies plan copies as health data, so the FR-9.2 consent and FR-2.11 email-verification gates apply to copy creation and edits (SEC-DATA-2).
- **Assumptions**: The plan models and publication state exist (REQ-PLAN-010, REQ-PLAN-020, REQ-PLAN-050), the exercise catalog exists (REQ-PLAN-080, FR-5.4), and the subscriber can view the source plan (REQ-CATALOG-010, REQ-CATALOG-020).
- **Out of Scope**: Retrieval and listing of copies (REQ-CUSTOM-020); copy stability when the source plan changes (REQ-CUSTOM-030); cross-subscriber access prevention (REQ-AUTHZ-020, FR-7.4); subscription entitlement (FR-3.1, REQ-ENTITLE-010); selecting a copy as the active plan (FR-5.3, FR-6.4 — REQ-SELECT-010, REQ-SELECT-020); catalog management itself (FR-5.4, REQ-PLAN-080); the health-data definition binding the gates (FR-9.12, REQ-PRIVACY-070).
- **Design Traceability**: `DESIGN.md` — Core Components → Forms and validation (persistent visible labels, unit suffix tied to the account preference per FR-8.10, inline validation, focus to the first invalid field), Core Components → Actions (one primary action per region, busy state preventing repeat submission); Layout, Spacing, and Responsive Behavior; Accessibility; Product Patterns → Medical disclaimer and health-data consent (the disclaimer interstitial gates first customization).
- **Architecture Traceability**: `ARCHITECTURE.md` — data flow 3 ("Client → REST API → entitlement check → copy of published plan written as a subscriber-owned record; published plan untouched"); REST API Application ("copy-on-customize semantics (FR-7.2, FR-7.5)"); DR-4.
- **Security Traceability**: SEC-INPUT-4, SEC-INPUT-3, SEC-AUTHZ-1, SEC-AUTHZ-2, SEC-INPUT-1, SEC-RENDER-1.

## Scope

- **Applies To**: Server-Side Application, API, Web Client
- **Components**: REST API Application; Relational Persistence; Browser Client (customization view)
- **Interfaces / Operations**: Create a customized copy from a published plan; edit an owned copy
- **Actors**: `subscriber`
- **Preconditions**: Authenticated session; a published source plan; the medical disclaimer acknowledged (REQ-PRIVACY-040); recorded, unwithdrawn consent and a verified email address (FR-9.12, FR-9.2, FR-2.11)
- **Data Classification**: Restricted — a customized plan copy is health data (FR-9.12)
- **Personal or Regulated Data**: Health Data — which plan a subscriber follows and how they modified it (FR-9.12)
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, Washington My Health My Data, and the FTC Health Breach Notification Rule for US users; HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED, `REQUIREMENTS.md` OQ-3 RESOLVED)

## Security Context

- **Security Objectives**: Integrity, Confidentiality, Authorization, Privacy
- **Control Layers**: Business-Rule Validation, Authorization, Input Validation
- **Threat References**: `SECURITY.md` TM-I-1 (BOLA on plan copies), TM-T-1 (mass assignment), TM-T-5 (a subscriber-mutated published plan would poison the library for everyone); CWE-915 (mass assignment), CWE-639 (authorization bypass through user-controlled key), CWE-602 (client-side enforcement)
- **Abuse / Misuse Case**: A subscriber's customization writes through to the published plan — by identifier confusion, by a shared update path, or because the copy is stored as a reference rather than a copy — silently altering health guidance for every other subscriber.
- **Trust Boundary**: Boundary 1 — the source plan identifier, the owner identity, and every edited field arrive from an untrusted client.
- **Untrusted Inputs or Assertions**: The source plan identifier; the edited content, including every catalog-entry reference in an exercise-plan copy; any owner or publication field in the payload.
- **Authoritative Enforcement Point**: REST API Application — the copy is created server-side from persisted plan state and bound to the session identity.
- **Independent Verification**: Owner identity comes from the session (DR-3); the published plan is read, never written, in this path.
- **Zero Trust Relevance**: NIST SP 800-207 — per-request authorization of the source plan and the created resource. Exact tenet: TO BE DECIDED (not verified in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — verified during the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — verified during the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: GDPR/UK GDPR (EU/UK data subjects); CCPA/CPRA, Washington My Health My Data, FTC Health Breach Notification Rule (US users); HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED). Statute-section precision: TO BE DECIDED — no source document states sections for this requirement.
- **Other**: `REF-PC-2024`, `REF-PROMPT-API` as cited by SEC-INPUT-4 and SEC-INPUT-3.
- **Mapping Basis**: FR-7.1, FR-7.2, and SEC-INPUT-4 are the normative sources and name these references.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated subscriber viewing a published plan, when they customize it, then a new record owned by that subscriber is created containing the plan's contents as edited, and the customized copy is retrievable by its owner.
2. **AC-02 — Boundary or failure behavior**: Given a customization request whose edited content violates the plan content model — a non-numeric set count, an absent meal name, a negative calorie target — when it is submitted, then it is rejected with the failing field named, no copy is created, and the client moves focus to the first invalid field.
3. **AC-03 — Prohibited behavior**: Given any customization, when it is processed, then the published source plan MUST NOT be modified in any field, the copy MUST NOT be created with an owner taken from the request body, and the copy MUST NOT be publishable, shareable with another user, or submittable for publication (FR-4.2).
4. **AC-04 — Additional criterion**: Given a customization of an unpublished plan, or of a plan the subscriber cannot see, when it is attempted, then it is refused and the plan's existence is not disclosed (FR-4.7).
5. **AC-05 — Additional criterion**: Given a created or edited copy, when the operation completes, then an audit entry records the acting account, the action, the affected subject, and the time (FR-9.7, REQ-AUDIT-020), since a customized plan copy is health data (FR-9.12).
6. **AC-06 — Additional criterion**: Given an exercise-plan customization, when it edits the exercises, then it may select among FR-5.4 catalog entries and edit prescriptions (sets, repetitions, order), and a submission referencing a movement outside the catalog — a nonexistent or free-text exercise — is rejected with the failing field named and no copy created or changed (FR-5.4).
7. **AC-07 — Additional criterion**: Given an account without recorded consent or with an unverified email address (FR-2.11), when a copy create or edit is attempted, then it is refused as a health-data write (FR-9.12, SEC-DATA-2) and nothing is persisted. Given an account with consent withdrawn (FR-9.9), a copy *create* is refused identically, but the owning subscriber editing or deleting an *existing* copy succeeds under FR-9.9's correction carve-out (fixed 2026-08-03); a consultant edit under FR-11.6 is refused while the subscriber's consent is withdrawn.

## Failure Behavior

- **On Invalid Input**: Reject per REQ-API-010 with field-level detail; no copy created or modified.
- **On Authentication Failure**: Denied upstream by REQ-AUTHZ-010.
- **On Authorization Failure**: Deny; editing another subscriber's copy is refused without disclosing its existence (REQ-AUTHZ-020, FR-7.4).
- **On Security-Decision Failure**: If ownership, consent state, email-verification state, or the source plan's publication state cannot be resolved, refuse the operation (fail closed).
- **On External Dependency Failure**: If persistence or audit storage is unavailable, the operation fails atomically and no partial copy is written.
- **On System Error**: Roll back so no partial copy and no orphaned child records survive.
- **Logging / Audit**: Audit entry per AC-05. The copy's content is not written into the entry (SEC-LOG-3). Authorization denials are logged as security events (SEC-LOG-4).
- **Alerting**: Threshold alerts on repeated authorization denials route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-3, SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: The copy constructor deep-copies plan contents rather than referencing them; the binding allow-list excludes owner, publication, and verification fields; validation mirrors the plan content model, including the catalog-reference rule for exercise-plan copies.
- **Integration Tests**: Customize both plan types end to end; assert the copy's content, the owner, and that the source plan is byte-identical before and after; assert the audit entry.
- **Security Tests**: Attempt to modify the published plan through the customization path; mass-assignment of owner and publication fields; customization of an unpublished plan; attempt to edit another subscriber's copy; an exercise-plan copy referencing a non-catalog movement; copy create and edit attempted without consent, with consent withdrawn, and with an unverified email; assertion that no share or publish operation exists for a copy.
- **Compliance Tests / Evidence**: Audit entries for copy creation, retained per FR-9.7.
- **Acceptance-Criteria Traceability**: AC-01 — customization suite; AC-02 — validation and focus tests; AC-03 — source-immutability, mass-assignment, and route-inventory tests; AC-04 — unpublished source test; AC-05 — audit assertion; AC-06 — catalog-constraint tests; AC-07 — consent and email-verification gate tests.
- **Coverage Target**: Both plan types; valid and invalid edits; cross-subscriber, unpublished-source, non-catalog-reference, and gate negatives.
- **Required Test Environment**: Two subscriber identities (one without consent, one with consent withdrawn, one with an unverified email available as variants), seeded published and unpublished plans of both types, a seeded exercise catalog, audit capture. Runs against PostgreSQL on Vitest.

## Dependencies

- **Upstream Requirements**: REQ-PLAN-010, REQ-PLAN-020, REQ-PLAN-050, REQ-PLAN-080, REQ-AUTHZ-020, REQ-API-010, REQ-API-020, REQ-AUDIT-020, REQ-PRIVACY-010, REQ-PRIVACY-020, REQ-PRIVACY-040, REQ-PRIVACY-070, REQ-PLATFORM-030
- **Downstream Requirements**: REQ-CUSTOM-020, REQ-CUSTOM-030, REQ-PROGRESS-020, REQ-SELECT-010, REQ-SELECT-020
- **External Dependencies**: None
- **Dependency Assumptions**: Plan contents are modelled as child records that can be deep-copied (REQ-PLAN-010, REQ-PLAN-020); exercise-plan contents reference FR-5.4 catalog entries (REQ-PLAN-080).
- **Failure Impact**: A customization that writes through to the published plan is a library-poisoning path available to every subscriber — strictly worse than the compromised-admin threat because it needs no privilege.

## Implementation Notes

- **Constraints**: PostgreSQL with Drizzle ORM (`CLAUDE.md`); the client is a Vite-built single-page application with `vue-router`. A subscriber may hold any number of copies; at most one copy or published plan per type is *active* as a selection (FR-5.3, FR-6.4 — REQ-SELECT-010, REQ-SELECT-020), which is out of scope here. Exercise-plan copies compose their exercises by catalog reference (FR-5.4), and prescriptions — sets, repetitions, order — are the copy-owned, editable content.
- **Prohibited Approaches**: Storing the copy as a reference or a diff against the published plan, which makes FR-7.5 unachievable; a shared update handler for published plans and copies; deriving the owner from the request; adding a share or publish action to copies; accepting free-text exercises into an exercise-plan copy.
- **Implementation Guidance**: Write the copy as a complete, independent record at creation time — that is what makes REQ-CUSTOM-030's stability guarantee structural rather than a runtime check. Copying an exercise plan deep-copies the prescriptions while retaining the FR-5.4 catalog references, which stay stable across renames by design.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-VUE`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the copy path, the binding allow-list, and the consent and email-verification gates; product review of the customization form.
- **Open Decisions**: None — `REQUIREMENTS.md` OQ-1 and OQ-6 are RESOLVED (FR-3.5/FR-3.6; FR-5.3/FR-6.4). Entitlement gating is REQ-ENTITLE-010 and active selection is REQ-SELECT-010/REQ-SELECT-020.

**Estimated effort**: 1.5–2 engineer-days. **Estimated changed lines**: 400–850.
**Recommended model**: Claude Opus (`claude-opus-5`) — the copy-versus-reference decision and the source-immutability guarantee are exactly where a plausible implementation goes wrong.
