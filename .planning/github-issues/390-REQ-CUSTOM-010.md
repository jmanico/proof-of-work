# [REQ-CUSTOM-010] Customize a published plan into a private copy

## Metadata

- **ID**: REQ-CUSTOM-010
- **Title**: Customize a published plan into a private copy
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-7.1, FR-7.2, FR-4.2; `SECURITY.md` SEC-INPUT-4

## Requirement

- **Statement**: A subscriber MUST be able to customize a published exercise or diet plan by editing its contents, and the system MUST save the customization as a private copy owned by that subscriber without modifying the published plan.
- **Rationale**: FR-7.1 grants the customization; FR-7.2 requires copy-on-customize semantics and forbids mutating the published plan; SEC-INPUT-4 requires this as server-side validation rather than UI affordance. FR-4.2 forbids subscribers authoring or sharing plans, which bounds what a copy is: private, never publishable.
- **Assumptions**: The plan models and publication state exist (REQ-PLAN-010, REQ-PLAN-020, REQ-PLAN-050) and the subscriber can view the source plan (REQ-CATALOG-010, REQ-CATALOG-020).
- **Out of Scope**: Retrieval and listing of copies (REQ-CUSTOM-020); copy stability when the source plan changes (REQ-CUSTOM-030); cross-subscriber access prevention (REQ-AUTHZ-020, FR-7.4); subscription entitlement (FR-3.1), blocked by `REQUIREMENTS.md` OQ-1; plan selection, blocked by OQ-6.
- **Design Traceability**: `DESIGN.md` — Components → Inputs (persistent visible labels, units adjacent to numeric fields), Buttons (one primary action per view, busy state blocking repeat submission), Form feedback and errors (inline, field-adjacent, focus to first invalid field); Layout and Spacing; Accessibility.
- **Architecture Traceability**: `ARCHITECTURE.md` — data flow 3 ("Client → REST API → entitlement check → copy of published plan written as a subscriber-owned record; published plan untouched"); REST API Application ("copy-on-customize semantics (FR-7.2, FR-7.5)"); DR-4.
- **Security Traceability**: SEC-INPUT-4, SEC-INPUT-3, SEC-AUTHZ-1, SEC-AUTHZ-2, SEC-INPUT-1, SEC-RENDER-1.

## Scope

- **Applies To**: Server-Side Application, API, Web Client
- **Components**: REST API Application; Relational Persistence; Browser Client (customization view)
- **Interfaces / Operations**: Create a customized copy from a published plan; edit an owned copy
- **Actors**: `subscriber`
- **Preconditions**: Authenticated session; a published source plan; the medical disclaimer acknowledged (REQ-PRIVACY-040)
- **Data Classification**: Restricted — a subscriber's customized plan is their own health-related data
- **Personal or Regulated Data**: Health Data — which plan a subscriber follows and how they modified it
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED (`SECURITY.md` SQ-1, `REQUIREMENTS.md` OQ-3)

## Security Context

- **Security Objectives**: Integrity, Confidentiality, Authorization, Privacy
- **Control Layers**: Business-Rule Validation, Authorization, Input Validation
- **Threat References**: `SECURITY.md` TM-I-1 (BOLA on plan copies), TM-T-1 (mass assignment), TM-T-5 (a subscriber-mutated published plan would poison the library for everyone); CWE-915 (mass assignment), CWE-639 (authorization bypass through user-controlled key), CWE-602 (client-side enforcement)
- **Abuse / Misuse Case**: A subscriber's customization writes through to the published plan — by identifier confusion, by a shared update path, or because the copy is stored as a reference rather than a copy — silently altering health guidance for every other subscriber.
- **Trust Boundary**: Boundary 1 — the source plan identifier, the owner identity, and every edited field arrive from an untrusted client.
- **Untrusted Inputs or Assertions**: The source plan identifier; the edited content; any owner or publication field in the payload.
- **Authoritative Enforcement Point**: REST API Application — the copy is created server-side from persisted plan state and bound to the session identity.
- **Independent Verification**: Owner identity comes from the session (DR-3); the published plan is read, never written, in this path.
- **Zero Trust Relevance**: NIST SP 800-207 — per-request authorization of the source plan and the created resource. Exact tenet: TO BE DECIDED (not verified in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: TO BE DECIDED — blocked by `SECURITY.md` SQ-1.
- **Other**: `REF-PC-2024`, `REF-PROMPT-API` as cited by SEC-INPUT-4 and SEC-INPUT-3.
- **Mapping Basis**: FR-7.1, FR-7.2, and SEC-INPUT-4 are the normative sources and name these references.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated subscriber viewing a published plan, when they customize it, then a new record owned by that subscriber is created containing the plan's contents as edited, and the customized copy is retrievable by its owner.
2. **AC-02 — Boundary or failure behavior**: Given a customization request whose edited content violates the plan content model — a non-numeric set count, an absent meal name, a negative calorie target — when it is submitted, then it is rejected with the failing field named, no copy is created, and the client moves focus to the first invalid field.
3. **AC-03 — Prohibited behavior**: Given any customization, when it is processed, then the published source plan MUST NOT be modified in any field, the copy MUST NOT be created with an owner taken from the request body, and the copy MUST NOT be publishable, shareable with another user, or submittable for publication (FR-4.2).
4. **AC-04 — Additional criterion**: Given a customization of an unpublished plan, or of a plan the subscriber cannot see, when it is attempted, then it is refused and the plan's existence is not disclosed (FR-4.7).
5. **AC-05 — Additional criterion**: Given a created or edited copy, when the operation completes, then an audit entry records the acting account, the action, and the affected subject (REQ-AUDIT-020), since a customized plan is the subscriber's health-related data.

## Failure Behavior

- **On Invalid Input**: Reject per REQ-API-010 with field-level detail; no copy created or modified.
- **On Authentication Failure**: Denied upstream by REQ-AUTHZ-010.
- **On Authorization Failure**: Deny; editing another subscriber's copy is refused without disclosing its existence (REQ-AUTHZ-020, FR-7.4).
- **On Security-Decision Failure**: If ownership or the source plan's publication state cannot be resolved, refuse the operation.
- **On External Dependency Failure**: If persistence or audit storage is unavailable, the operation fails atomically and no partial copy is written.
- **On System Error**: Roll back so no partial copy and no orphaned child records survive.
- **Logging / Audit**: Audit entry per AC-05. The copy's content is not written into the entry (SEC-LOG-3).
- **Alerting**: N/A

## Test Strategy

- **Unit Tests**: The copy constructor deep-copies plan contents rather than referencing them; the binding allow-list excludes owner, publication, and verification fields; validation mirrors the plan content model.
- **Integration Tests**: Customize both plan types end to end; assert the copy's content, the owner, and that the source plan is byte-identical before and after; assert the audit entry.
- **Security Tests**: Attempt to modify the published plan through the customization path; mass-assignment of owner and publication fields; customization of an unpublished plan; attempt to edit another subscriber's copy; assertion that no share or publish operation exists for a copy.
- **Compliance Tests / Evidence**: Audit entries for copy creation, retained per FR-9.7.
- **Acceptance-Criteria Traceability**: AC-01 — customization suite; AC-02 — validation and focus tests; AC-03 — source-immutability, mass-assignment, and route-inventory tests; AC-04 — unpublished source test; AC-05 — audit assertion.
- **Coverage Target**: Both plan types; valid and invalid edits; cross-subscriber and unpublished-source negatives.
- **Required Test Environment**: Two subscriber identities, seeded published and unpublished plans of both types, audit capture. Runs against PostgreSQL on Vitest.

## Dependencies

- **Upstream Requirements**: REQ-PLAN-010, REQ-PLAN-020, REQ-PLAN-050, REQ-AUTHZ-020, REQ-API-010, REQ-API-020, REQ-AUDIT-020, REQ-PRIVACY-040, REQ-PLATFORM-030
- **Downstream Requirements**: REQ-CUSTOM-020, REQ-CUSTOM-030, REQ-PROGRESS-020
- **External Dependencies**: None
- **Dependency Assumptions**: Plan contents are modelled as child records that can be deep-copied (REQ-PLAN-010, REQ-PLAN-020).
- **Failure Impact**: A customization that writes through to the published plan is a library-poisoning path available to every subscriber — strictly worse than the compromised-admin threat because it needs no privilege.

## Implementation Notes

- **Constraints**: PostgreSQL with Drizzle ORM (`CLAUDE.md`); client build tooling TO BE DECIDED. Whether a subscriber may hold copies of more than one plan at a time is `REQUIREMENTS.md` OQ-6 and is not decided here; this issue permits creating a copy without asserting how many may be *active*.
- **Prohibited Approaches**: Storing the copy as a reference or a diff against the published plan, which makes FR-7.5 unachievable; a shared update handler for published plans and copies; deriving the owner from the request; adding a share or publish action to copies.
- **Implementation Guidance**: Write the copy as a complete, independent record at creation time — that is what makes REQ-CUSTOM-030's stability guarantee structural rather than a runtime check.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-VUE`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the copy path and the binding allow-list; product review of the customization form.
- **Open Decisions**: `REQUIREMENTS.md` OQ-6 (one or many active plans) and OQ-1 (entitlement gating). Neither blocks copy creation, but both bound the surrounding workflow.

**Estimated effort**: 1.5–2 engineer-days. **Estimated changed lines**: 400–850.
**Recommended model**: Claude Opus (`claude-opus-5`) — the copy-versus-reference decision and the source-immutability guarantee are exactly where a plausible implementation goes wrong.
