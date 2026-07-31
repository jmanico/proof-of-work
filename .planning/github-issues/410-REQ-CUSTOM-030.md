# [REQ-CUSTOM-030] Customized copy stability when the source plan changes

## Metadata

- **ID**: REQ-CUSTOM-030
- **Title**: Customized copy stability when the source plan changes
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-7.5; `SECURITY.md` SEC-INPUT-4

## Requirement

- **Statement**: An existing customized copy MUST remain unchanged when the published plan it was derived from is later edited or unpublished, and MUST remain retrievable by its owner regardless of the source plan's state.
- **Rationale**: FR-7.5 states the rule; SEC-INPUT-4 requires it as server-side validation rather than a UI convention. Without it, an admin edit silently rewrites what subscribers believe they are following, and an unpublication destroys their work.
- **Assumptions**: Copies are created as complete independent records (REQ-CUSTOM-010), which is what makes this guarantee structural rather than a runtime check.
- **Out of Scope**: Copy creation and editing (REQ-CUSTOM-010); listing and retrieval (REQ-CUSTOM-020); whether a subscriber is notified that their copy's source changed, which no source document specifies; retention across subscription lapse (FR-3.4), blocked by `REQUIREMENTS.md` OQ-1.
- **Design Traceability**: `DESIGN.md` OQ-5 concerns how verification and provenance are surfaced; whether a copy shows that its source has changed is unspecified and out of scope here.
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application ("copy-on-customize semantics (FR-7.2, FR-7.5)"); Relational Persistence ("preserve customized copies when their source plan changes (FR-7.5)"); DR-4.
- **Security Traceability**: SEC-INPUT-4; supports SEC-AUTHZ-2, SEC-DATA-5.

## Scope

- **Applies To**: Server-Side Application, API
- **Components**: REST API Application; Relational Persistence
- **Interfaces / Operations**: Plan edit and unpublish, viewed from the perspective of derived copies; retrieval of a copy whose source has changed
- **Actors**: `subscriber` as copy owner; `admin` as the actor whose plan changes must not propagate
- **Preconditions**: A customized copy exists, derived from a published plan
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED (`SECURITY.md` SQ-1)

## Security Context

- **Security Objectives**: Integrity, Availability, Privacy, Safety
- **Control Layers**: Business-Rule Validation, Data Protection
- **Threat References**: `SECURITY.md` TM-T-5 (a compromised admin edit that propagated into every derived copy would multiply the harm), TM-E-1; CWE-345 (insufficient verification of data authenticity — a copy that silently reflects upstream change is not the artifact the user saved)
- **Abuse / Misuse Case**: An admin edit or unpublication cascades into subscribers' copies — through a foreign-key reference, a shared child table, or a cascade delete — rewriting or destroying data the subscriber owns and relies on.
- **Trust Boundary**: Boundary 3 — the guarantee is enforced by data model and referential rules within persistence.
- **Untrusted Inputs or Assertions**: None directly; the risk is internal coupling rather than external input.
- **Authoritative Enforcement Point**: Relational Persistence schema plus the REST API Application's copy semantics.
- **Independent Verification**: A test compares full copy content before and after a source edit and a source unpublication.
- **Zero Trust Relevance**: N/A

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: N/A
- **Regulatory**: N/A
- **Other**: `REF-PC-2024`, `REF-ASVS-5` as cited by SEC-INPUT-4.
- **Mapping Basis**: FR-7.5 is the normative source; SEC-INPUT-4 requires server-side enforcement and names these references.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a customized copy derived from a published plan, when an admin edits that plan's title, description, exercises, meals, targets, or citations, then the copy's content is byte-identical to its state before the edit.
2. **AC-02 — Boundary or failure behavior**: Given a customized copy derived from a published plan, when that plan is unpublished, then the copy remains retrievable by its owner with unchanged content, and no cascade removes or hides it.
3. **AC-03 — Prohibited behavior**: Given the data model, when it is reviewed, then a copy MUST NOT store its content as a reference, join, or diff against the published plan, and no referential action — cascade update or cascade delete — may propagate from a plan to a derived copy or its children.
4. **AC-04 — Additional criterion**: Given a copy whose source plan has been unpublished, when the owner retrieves it, then retrieval succeeds without requiring the source plan to be readable, so a copy is never orphaned into inaccessibility.
5. **AC-05 — Additional criterion**: Given a copy, when its provenance is recorded, then the source plan identity may be retained for reference, but the copy's content MUST NOT be resolved through it at read time.

## Failure Behavior

- **On Invalid Input**: N/A — this issue governs an invariant, not an input-bearing operation.
- **On Authentication Failure**: Denied upstream for the retrieval paths involved.
- **On Authorization Failure**: Copy retrieval remains owner-scoped (REQ-AUTHZ-020) regardless of source plan state.
- **On Security-Decision Failure**: If a copy's content cannot be read independently of its source, treat that as a defect and fail the retrieval rather than substituting current plan content.
- **On External Dependency Failure**: If persistence is unavailable, retrieval fails with a generic error; no fallback to source plan content.
- **On System Error**: Roll back; a plan edit that cannot complete without touching copies MUST fail rather than proceed.
- **Logging / Audit**: The admin edit and unpublish actions are audited (REQ-AUDIT-030); copy reads are audited (REQ-AUDIT-020). No additional audit event is introduced here.
- **Alerting**: N/A

## Test Strategy

- **Unit Tests**: The copy read path resolves content from the copy's own records with no query against the plan tables.
- **Integration Tests**: Snapshot a copy, edit its source plan across every content field, and assert byte-identical copy content; repeat for unpublication; delete-adjacent behavior asserted where the schema permits plan removal.
- **Security Tests**: Schema review asserting no cascade update or delete from plans to copies; a test that a copy remains retrievable when its source plan is unreadable by the owner; assertion that no read path falls back to source content.
- **Compliance Tests / Evidence**: Before-and-after content comparisons, retained as evidence for FR-7.5.
- **Acceptance-Criteria Traceability**: AC-01 — edit-propagation suite; AC-02 — unpublication suite; AC-03 — schema and referential-action review; AC-04 — orphaned-source retrieval test; AC-05 — read-path independence test.
- **Coverage Target**: Every plan content field exercised by the edit-propagation suite, for both plan types.
- **Required Test Environment**: A subscriber holding copies of both plan types, an admin able to edit and unpublish, and full content snapshots. Runs against PostgreSQL on Vitest.

## Dependencies

- **Upstream Requirements**: REQ-CUSTOM-010, REQ-CUSTOM-020, REQ-PLAN-030, REQ-PLAN-060
- **Downstream Requirements**: None
- **External Dependencies**: None
- **Dependency Assumptions**: The chosen RDBMS's referential actions are configurable so that no cascade reaches copies.
- **Failure Impact**: A cascade here destroys or rewrites subscriber-owned health data during a routine admin action, and the loss is silent.

## Implementation Notes

- **Constraints**: PostgreSQL with Drizzle ORM and drizzle-kit migrations (`CLAUDE.md`); schema design remains TO BE DECIDED. This issue is largely a schema and data-model guarantee, so it must be settled before REQ-CUSTOM-010 persists its first copy — retrofitting independence after copies exist is a migration, not a change.
- **Prohibited Approaches**: Storing a copy as a plan identifier plus an overrides map; sharing exercise or meal child tables between plans and copies; `ON DELETE CASCADE` or `ON UPDATE CASCADE` from plans to copies; resolving any copy field from the source plan at read time.
- **Implementation Guidance**: Retaining the source plan identifier for provenance is permitted and useful, provided AC-05 holds — nothing is resolved through it. Verify the invariant with the same snapshot comparison used in REQ-PLAN-030 AC-04 and REQ-PLAN-060 AC-02 so the three issues assert one guarantee from three directions.
- **AI Development Guidance**: `REF-PROMPT-QUALITY`, `REF-PROMPT-API`; `CLAUDE.md`.
- **Required Human Review**: Architecture review of the schema and referential actions.
- **Open Decisions**: Whether a subscriber should be told that a copy's source plan changed or was withdrawn is unspecified in all source documents; FR-7.5 requires stability, not notification. FR-3.4 retention across subscription lapse is blocked by `REQUIREMENTS.md` OQ-1.

**Estimated effort**: 0.5–1 engineer-day. **Estimated changed lines**: 100–300.
**Recommended model**: Claude Opus (`claude-opus-5`) — a schema-level invariant whose violation is silent data loss during ordinary admin work.
