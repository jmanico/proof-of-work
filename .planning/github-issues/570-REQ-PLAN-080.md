# [REQ-PLAN-080] Admin exercise catalog management

## Metadata

- **ID**: REQ-PLAN-080
- **Title**: Admin exercise catalog management
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-5.4 (added 2026-08-03), FR-10.2; `DESIGN.md` "Exercise catalog (admin Plans)"

## Requirement

- **Statement**: The system MUST maintain an admin-curated exercise catalog in which each exercise is a first-class record with a stable identifier and a name, where create, edit, and retire actions are restricted to `admin` accounts and each produces an audit entry, renaming an entry never changes its identifier, retiring an entry removes it from new plan composition but never from existing plans, copies, logged history, or charts, and deleting an entry referenced by any plan, plan copy, or log entry is refused.
- **Rationale**: FR-5.4 fixes the exercise catalog as the stable identity backbone for FR-8.14 per-exercise trends: plans and customized copies compose exercises by catalog reference, logged exercises reference the catalog entry and snapshot the name (FR-8.3), so a rename or retirement must never rewrite or orphan history. Admin-only curation with FR-10.2 audit keeps content changes attributable.
- **Assumptions**: Admin-only restriction on content administration is enforced at the single enforcement point (REQ-AUTHZ-030); the audit pipeline for admin plan-content lifecycle actions exists (REQ-AUDIT-030).
- **Out of Scope**: Composing plans from catalog entries (REQ-PLAN-030) and customization's catalog-only picker (FR-5.4's composition half, REQ-CUSTOM-010); workout logging's catalog reference and name snapshot (FR-8.3, REQ-PROGRESS-020); per-exercise trend charts (FR-8.14); exercise demonstration diagrams (`DESIGN.md` Imagery); any subscriber-facing catalog browsing beyond what plan views expose.
- **Design Traceability**: `DESIGN.md` — "Exercise catalog (admin Plans)": a searchable list with rename and **Retire** actions; retiring removes an entry from new plan composition and never from history or charts; retired entries carry the `Retired` chip (approved chip vocabulary, "Status, feedback, and loading"). Admin workspace navigation places the catalog under Plans.
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application (owns plan-library business objects; DR-4 single owning component); Relational Persistence (referential integrity and ownership in schema); entity list includes the exercise catalog entry.
- **Security Traceability**: SEC-AUTHZ-4 (admin-only content administration), SEC-INPUT-1, SEC-INPUT-3, SEC-LOG-6/FR-10.2 (audit), SEC-INPUT-4 (business rules enforced server-side).

## Scope

- **Applies To**: Server-Side Application, API, Web Client
- **Components**: REST API Application; Relational Persistence; Browser Client (admin catalog list, rename, retire, `Retired` chip)
- **Interfaces / Operations**: Catalog entry create, rename/edit, retire, delete (unreferenced only), list and search for admin composition pickers
- **Actors**: `admin` (permitted); `subscriber` and `consultant` (denied catalog mutation)
- **Preconditions**: An authenticated `admin` session for every mutation
- **Data Classification**: Internal — catalog content is published, non-personal reference data
- **Personal or Regulated Data**: Personal Data — the acting admin's identity in audit entries only; catalog entries themselves contain no personal data
- **Jurisdiction / Regulatory Scope**: `SECURITY.md` SQ-1 regime set applies to the platform generally; catalog records carry no health or personal data, so no data-subject obligation attaches to the entries themselves

## Security Context

- **Security Objectives**: Integrity, Accountability, Authorization
- **Control Layers**: Authorization, Input Validation, Business-Rule Validation, Logging and Monitoring
- **Threat References**: `SECURITY.md` TM-T-5 context (admin-curated content integrity), TM-R-1 (unaudited admin lifecycle actions); CWE-862 (missing authorization), CWE-471 (modification of assumed-immutable data — the stable identifier)
- **Abuse / Misuse Case**: A subscriber or consultant attempts to create or alter catalog entries to change what plans prescribe; a client supplies a chosen identifier or mutates an identifier through rename; a delete of a referenced entry orphans plan contents and logged history, silently corrupting FR-8.14 trends.
- **Trust Boundary**: Boundary 1 — Browser Client → REST API Application; all catalog rules are enforced server-side (DR-2).
- **Untrusted Inputs or Assertions**: Entry names and search terms (validated per SEC-INPUT-1, rendered per SEC-RENDER-1); any client-supplied identifier value on create or rename (identifiers are server-controlled, SEC-INPUT-3).
- **Authoritative Enforcement Point**: REST API Application behind the single authorization enforcement point (SEC-AUTHZ-5); reference checks against persisted state gate deletion.
- **Independent Verification**: Role resolved from Identity and Session Handling and persisted state (DR-3); the referenced-entry check reads persistence directly, never a client assertion.
- **Zero Trust Relevance**: NIST SP 800-207 — catalog mutations are authorized per request against server-held role state. Exact tenet: TO BE DECIDED (SQ-10 assessment).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — per-issue mapping deferred to the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A — no AI-enabled component is involved.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — per `SECURITY.md` SQ-10.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: N/A — catalog entries contain no personal or health data; the platform-level SQ-1 regime set is unaffected by this requirement.
- **Other**: `REF-LOG`, `REF-ASVS-5` as cited by SEC-LOG-6; `REF-INPUT` for name validation.
- **Mapping Basis**: SEC-LOG-6 names the audit references for admin content lifecycle actions; CWE identifiers describe the missing-authorization and identifier-mutation classes.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated `admin`, when they create a catalog entry with a valid name, then the entry receives a server-generated stable identifier, appears in catalog list and search, and exactly one audit entry records the acting admin, the action, the affected entry, and the time (FR-5.4, FR-10.2).
2. **AC-02 — Rename keeps identity**: Given an existing catalog entry referenced by plans, copies, and workout log entries, when an `admin` renames it, then its identifier is unchanged, every referencing record still resolves to the same entry, logged history continues to show the name snapshot taken at log time (FR-8.3), and an audit entry is written.
3. **AC-03 — Retire hides from new composition only**: Given a catalog entry referenced by existing plans, copies, and logs, when an `admin` retires it, then it no longer appears among the entries offered for new plan composition or customization selection, while every existing plan, copy, log entry, and per-exercise trend that references it is unaffected, its state is exposed so the client can render the `Retired` chip, and an audit entry is written.
4. **AC-04 — Delete refused once referenced**: Given a catalog entry referenced by any plan, plan copy, or log entry, when an `admin` attempts to delete it, then the deletion is refused with an error identifying that the entry is in use, and no referencing record changes; an unreferenced entry MAY be deleted, with an audit entry.
5. **AC-05 — Prohibited behavior**: Given an authenticated `subscriber` or `consultant`, when they attempt any catalog create, rename, retire, or delete, then the operation is denied (SEC-AUTHZ-4); and given any request supplying an identifier value on create or rename, when the operation processes it, then the supplied identifier is ignored (SEC-INPUT-3).

## Failure Behavior

- **On Invalid Input**: Reject an absent, empty, or over-length name, or a malformed entry identifier, with a schema-validation error naming the invalid field (SEC-INPUT-1); no state change occurs.
- **On Authentication Failure**: Deny per the deny-by-default guard (REQ-AUTHZ-010) with a uniform response (SEC-AUTHN-3).
- **On Authorization Failure**: Deny for non-admin roles at the single enforcement point; catalog entries are non-sensitive reference data, so existence disclosure to authenticated actors is acceptable, but the denial response follows the standard shape.
- **On Security-Decision Failure**: Deny by default; an error inside policy evaluation denies (SEC-AUTHZ-7).
- **On External Dependency Failure**: N/A — no external dependency.
- **On System Error**: Mutation and its audit entry commit atomically (DR-9); on failure, roll back with a generic error and correlation identifier (SEC-ERR-1). The referenced-entry check and the delete MUST be race-safe: enforce the reference restriction in the database (foreign-key constraints), not only as an application pre-check.
- **Logging / Audit**: One audit entry per create, edit/rename, retire, and delete (acting admin, action, entry, time — FR-5.4, FR-10.2); denials logged per SEC-LOG-4; no free-text content duplication into logs beyond the entry reference (SEC-LOG-3).
- **Alerting**: Threshold alerts on repeated authorization denials route to the security lead as SEC-OPS-2 detection inputs (SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: Name validation rules; identifier immutability across rename; retire-state transition; referenced/unreferenced delete decision; server-controlled identifier stripping.
- **Integration Tests**: Create→reference-from-plan→rename flow asserting stable resolution and preserved log-time name snapshots; retire flow asserting exclusion from composition pickers while existing plans, copies, logs, and trend queries are unchanged; delete-refusal against each referencing type (plan, copy, log entry) enforced by database constraint under concurrent reference creation; audit-entry emission per action.
- **Security Tests**: Catalog mutations as subscriber and consultant asserting denial; mass-assignment test supplying identifiers on create and rename; stored-XSS probe through the entry name asserting neutral rendering (SEC-RENDER-1).
- **Compliance Tests / Evidence**: Audit-entry field assertions per lifecycle action as FR-10.2 evidence.
- **Acceptance-Criteria Traceability**: AC-01 — create integration suite; AC-02 — rename identity suite; AC-03 — retire suite; AC-04 — delete-refusal suite; AC-05 — role-denial and mass-assignment security tests.
- **Coverage Target**: Project coverage threshold is TO BE DECIDED (`CLAUDE.md`); all mutation paths and denials MUST have positive and negative tests.
- **Required Test Environment**: Vitest; PostgreSQL with the catalog schema and foreign-key constraints; fixtures for admin, subscriber, and consultant identities plus referencing plans, copies, and log entries; Playwright for the admin catalog list and `Retired` chip.

## Dependencies

- **Upstream Requirements**: REQ-AUTHZ-030 (admin-only restriction), REQ-AUDIT-030 (admin lifecycle audit entries)
- **Downstream Requirements**: REQ-PLAN-030 (plans compose by catalog reference), REQ-CUSTOM-010 (customization selects among catalog exercises), REQ-PROGRESS-020 (logged exercises reference catalog entries, FR-8.3), REQ-PROGRESS-060 (per-exercise trends keyed by catalog entry, FR-8.14)
- **External Dependencies**: None
- **Dependency Assumptions**: REQ-AUDIT-030 provides an atomic audit write; downstream issues enforce their own catalog-reference validity at composition and log time.
- **Failure Impact**: An unstable or deletable-while-referenced identifier silently severs plans, copies, and logged history from their exercises, corrupting FR-8.14 trend continuity across plan switches and renames.

## Implementation Notes

- **Constraints**: Node.js/Fastify, PostgreSQL with Drizzle ORM (`CLAUDE.md`); the delete restriction MUST be backed by database referential integrity, not application checks alone; identifiers are server-generated (SEC-SECRET-4 applies if random identifiers are used).
- **Prohibited Approaches**: Client-supplied or mutable identifiers; hard-deleting or renaming through raw content duplication (copying entries instead of referencing them); implementing retirement as deletion plus tombstone; enforcing admin-only rules client-side (DR-2); cascade-deleting referencing records.
- **Implementation Guidance**: Model `retired` as a state column read by composition pickers, with list/search endpoints exposing it for the `Retired` chip; keep the name-snapshot obligation on the logging path (FR-8.3, REQ-PROGRESS-020) rather than versioning catalog names here.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-NODE`, `REF-PROMPT-QUALITY`; `CLAUDE.md` working rules.
- **Required Human Review**: Security review of the authorization and identifier handling; product review that retirement semantics match FR-5.4 and the DESIGN.md catalog pattern.
- **Open Decisions**: None — FR-5.4 fixes the behavior; per-issue standards mappings await the SQ-10 assessment.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 300–600.
**Recommended model**: Claude Opus (`claude-opus-5`) — referential-integrity and identity-stability rules that all future exercise history depends on.
