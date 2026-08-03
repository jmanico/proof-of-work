# [REQ-SELECT-030] Unpublication ends active selections

## Metadata

- **ID**: REQ-SELECT-030
- **Title**: Unpublication ends active selections
- **Version**: 1.0.1
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-4.9 (added 2026-08-03), FR-7.5, FR-9.12; `REQUIREMENTS.md` FR-8.5 (no-target state consumer)

## Requirement

- **Statement**: When a plan is unpublished, the system MUST end, server-side and at unpublication time, every active selection (FR-5.3, FR-6.4) that names that published plan, MUST show each affected subscriber an explanatory state prompting selection of a replacement, MUST NOT alter any logged history or any customized copy (FR-7.5), and MUST present the FR-8.5 comparison's no-target state until the affected subscriber selects a new diet plan.
- **Rationale**: FR-4.9 closes the unpublish-while-selected gap: without it, a subscriber's active selection would dangle against content the library no longer shows (FR-4.7). Ending the selection at the unpublication event keeps the selection invariant consistent without touching the subscriber's history or private copies, and the explanatory state tells the subscriber what happened and what to do next rather than failing silently.
- **Assumptions**: The unpublication operation itself — admin-only, audited per FR-10.2/SEC-LOG-6 — is delivered by REQ-PLAN-060; active selections exist per REQ-SELECT-010 and REQ-SELECT-020; a selection that names a subscriber's own customized copy names the copy, not the published plan it was derived from, and is therefore not ended by this requirement.
- **Out of Scope**: The unpublication operation, its role restriction, and its FR-10.2 audit entry (REQ-PLAN-060); selection creation and replacement (REQ-SELECT-010, REQ-SELECT-020); the rendering of the FR-8.5 comparison including its no-target presentation (REQ-FOOD-030); preservation semantics of customized copies themselves (REQ-CUSTOM-030); consultant and admin views of plans.
- **Design Traceability**: `DESIGN.md` — "Status, feedback, and loading": permission and ended states "explain the exact reason and the available next step without exposing hidden data", and the approved chip vocabulary includes `Unpublished`; "Cards, lists, and tables": the selected-state treatment this state replaces; "Progress and target comparison": the FR-8.5 comparison presentation; empty states say what belongs in the region and provide the action that creates it.
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application (owner of plan and selection state; data flow 6, admin publication/unpublication); Relational Persistence; DR-2 (server-side rule, client presents), DR-4 (only the REST API Application mutates the selection), DR-9 (audit write on the health-data modification path).
- **Security Traceability**: SEC-INPUT-4 (business rules enforced server-side as validation); SEC-LOG-1 and FR-9.7 (ending a selection modifies health data per FR-9.12 and is audited); FR-10.3 / SEC-AUTHZ-9 (the admin performing unpublication must not thereby read subscribers' selections); SEC-LOG-6 via REQ-PLAN-060.

## Scope

- **Applies To**: API, Server-Side Application, Web Client
- **Components**: REST API Application; Relational Persistence; Browser Client (explanatory ended state, replacement prompt, no-target comparison state)
- **Interfaces / Operations**: The unpublication operation's selection-ending step; subscriber selection reads; the FR-8.5 comparison read; Home and Plans views of affected subscribers
- **Actors**: Admin (initiates unpublication); affected subscribers (see the ended state); consultant (sees the engaged subscriber's selection state under FR-11.6)
- **Preconditions**: A published plan is actively selected by one or more subscribers; an admin unpublishes it (REQ-PLAN-060)
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data — active plan selections are health data by FR-9.12
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED)

## Security Context

- **Security Objectives**: Integrity, Privacy, Accountability
- **Control Layers**: Business-Rule Validation, Authorization, Data Protection, Logging and Monitoring
- **Threat References**: `SECURITY.md` TM-T-5 (a compromised admin's content actions ripple to subscribers — bounded here to ending selections, never touching history or copies); CWE-459 (incomplete cleanup — the dangling-selection state this requirement prevents)
- **Abuse / Misuse Case**: A compromised or careless admin unpublishes a widely selected plan to disrupt subscribers — the blast radius is bounded to ended selections with an explanatory recovery path, and the action is audited; or the unpublication response is used to learn which subscribers follow a plan, which FR-10.3 prohibits.
- **Trust Boundary**: Boundary 1 for the admin request (handled by REQ-PLAN-060); the selection-ending step runs entirely server-side inside the REST API Application with no client input of its own.
- **Untrusted Inputs or Assertions**: None beyond the unpublication request validated by REQ-PLAN-060 — the set of affected selections is derived exclusively from persisted state.
- **Authoritative Enforcement Point**: REST API Application — the unpublication transaction, which ends the named selections as part of the same server-side operation.
- **Independent Verification**: The affected-selection set is computed from Relational Persistence, never supplied by any client; subscriber-facing state is re-derived on read (DR-2, DR-3).
- **Zero Trust Relevance**: N/A — no new resource-access decision is introduced; access control on the triggering operation is REQ-PLAN-060's, and subscriber reads remain governed by REQ-AUTHZ-020.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — per-issue mapping deferred to the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A — no AI-enabled component is involved.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — per-issue mapping deferred to the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **NIST SP 800-207**: N/A — see Zero Trust Relevance.
- **Regulatory**: GDPR / UK GDPR (EU/UK data subjects), CCPA/CPRA, Washington My Health My Data, FTC Health Breach Notification Rule (US users) — the ended records are health data (FR-9.12) and the modification is audited (FR-9.7). Specific articles/sections: TO BE DECIDED (`SECURITY.md` SQ-1 counsel review).
- **Other**: N/A
- **Mapping Basis**: The regulatory regimes follow from SQ-1 because the selection records modified here are health data; the CWE identifier describes the incomplete-cleanup class the requirement prevents.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given subscribers A and B each have an active selection naming published plan P, when an admin unpublishes P, then both selections are ended within the same server-side unpublication operation, and on their next read A and B each have no active selection of that plan type.
2. **AC-02 — Explanatory state and replacement prompt**: Given subscriber A's selection was ended by AC-01, when A next views Home or Plans, then an explanatory state states the exact reason (the selected plan was unpublished) and the next step, and prompts A to select a replacement per `DESIGN.md`, without exposing any other data.
3. **AC-03 — Diet no-target state**: Given the ended selection was A's diet plan selection, when A views the FR-8.5 daily comparison before selecting a replacement, then the comparison presents its no-target state — logged calories and macronutrients remain visible without targets — and resumes target comparison only after A selects a new diet plan.
4. **AC-04 — Untouched copies, history, and copy-naming selections**: Given subscriber C holds a customized copy derived from P and subscriber D's active selection names D's own copy derived from P, when P is unpublished, then C's and D's copies are byte-for-byte unchanged (FR-7.5), D's selection remains active (it names the copy, not P), and no workout, food, weight, or measurement log entry of any subscriber is created, altered, or deleted.
5. **AC-05 — Audit and prohibited disclosure**: Given the unpublication of P ended one or more selections, when the operation completes, then the health-data modification is audited per FR-9.7 with the acting admin and each affected subscriber as the affected subject, and the unpublication response and every admin view MUST NOT identify to the admin which subscribers had P selected (FR-10.3, SEC-AUTHZ-9).

## Failure Behavior

- **On Invalid Input**: N/A — the selection-ending step takes no client input; the unpublication request is validated by REQ-PLAN-060.
- **On Authentication Failure**: N/A at this layer — governed by REQ-PLAN-060 for the triggering operation and REQ-AUTHZ-010 for subscriber reads.
- **On Authorization Failure**: N/A at this layer — the admin-only restriction on unpublication is REQ-PLAN-060's; subscriber reads of their own state are owner-scoped by REQ-AUTHZ-020.
- **On Security-Decision Failure**: Deny by default — if the affected-selection set cannot be computed, the unpublication transaction fails closed rather than unpublishing while leaving selections dangling.
- **On External Dependency Failure**: N/A — no external dependency.
- **On System Error**: Unpublication and the ending of every affected selection are one transaction: on error, roll back both so the plan is not left unpublished with live selections naming it, and no selection is ended for a plan that remains published; respond per SEC-ERR-1 with a correlation identifier.
- **Logging / Audit**: The FR-10.2/SEC-LOG-6 unpublication audit entry (via REQ-PLAN-060) plus FR-9.7 audit coverage of the selection modifications recording acting account, action, affected subject, and time; no plan content or health values in logs (SEC-LOG-3); the affected-subscriber set never appears in any admin-readable surface.
- **Alerting**: Failures of the selection-ending step (transaction rollback on unpublication) are logged as security-relevant errors and route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-11).

## Test Strategy

- **Unit Tests**: Affected-selection computation selects exactly the active selections naming the published plan (not copy-naming selections, not other plans); explanatory-state derivation for an ended selection; no-target comparison state derivation when the diet selection is absent.
- **Integration Tests**: Unpublish-with-selections flow asserting AC-01 end-to-end against persistence; byte-for-byte pre/post comparison of copies and log entries (AC-04); FR-8.5 read before and after replacement selection (AC-03); transactional rollback test injecting a failure mid-operation and asserting neither unpublication nor selection-ending persisted.
- **Security Tests**: Assert the unpublication response body and admin plan views contain no subscriber identifiers or affected-subscriber enumeration (FR-10.3); assert no client-invocable operation can end another subscriber's selection outside the unpublication path; audit-entry presence and field assertions for the health-data modification.
- **Compliance Tests / Evidence**: FR-9.7 audit coverage of the selection modification retained as evidence for the SQ-1 counsel review; N/A otherwise.
- **Acceptance-Criteria Traceability**: AC-01 — unpublish-with-selections integration test; AC-02 — ended-state UI integration/Playwright test; AC-03 — no-target comparison tests; AC-04 — copy/history invariance suite; AC-05 — audit and admin-disclosure security tests.
- **Coverage Target**: Project threshold 90% line and branch (`CLAUDE.md`, 2026-08-03); the transaction, invariance, and disclosure paths MUST have positive and negative coverage.
- **Required Test Environment**: Vitest and HTTP test client; Playwright for the subscriber-facing ended and no-target states; fixtures with a published plan selected by multiple subscribers, a derived copy, a copy-naming selection, and existing log entries.

## Dependencies

- **Upstream Requirements**: REQ-PLAN-060 (unpublication operation and its audit entry), REQ-SELECT-010 (active exercise selection), REQ-SELECT-020 (active diet selection), REQ-AUDIT-020 (mandatory audit write on health-data modification paths)
- **Downstream Requirements**: REQ-FOOD-030 (renders the FR-8.5 no-target state this requirement triggers)
- **External Dependencies**: None
- **Dependency Assumptions**: REQ-PLAN-060 exposes a single unpublication operation into which this step composes transactionally; selections are stored so that plan-naming and copy-naming selections are distinguishable.
- **Failure Impact**: Without this behavior, unpublication leaves subscribers following invisible content: the FR-8.5 comparison reads targets from a plan the subscriber can no longer view, and the FR-4.7 published-only rule is undermined by dangling references.

## Implementation Notes

- **Constraints**: TypeScript on Fastify; PostgreSQL via Drizzle (`CLAUDE.md`); the selection-ending step MUST execute in the same database transaction as the unpublication state change.
- **Prohibited Approaches**: Ending selections lazily on next subscriber read instead of at unpublication time (FR-4.9 requires the end at unpublication, and lazy repair leaves the FR-8.5 read serving stale targets); cascading any change into customized copies or log entries; deleting the selection row in a way that erases the material needed for the subscriber's explanatory state; returning affected-subscriber identifiers to the admin (FR-10.3); client-side-only handling of the ended state (DR-2).
- **Implementation Guidance**: Record the ended selection with an ended-at marker and reason so the subscriber-facing explanatory state and replacement prompt (DESIGN.md "Status, feedback, and loading") can be rendered from persisted state; reuse the REQ-SELECT-010/020 selection-resolution function so an ended selection uniformly resolves to "none active" for FR-8.5 and workout flows.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the admin-disclosure surface (AC-05) and the transaction boundary; product review of the explanatory-state copy against `DESIGN.md` content-voice rules.
- **Open Decisions**: None — FR-4.9 fixes the behavior; per-issue standards mappings await the SQ-10 assessment.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 250–500.
**Recommended model**: Claude Opus (`claude-opus-5`) — a cross-cutting transactional invariant with privacy-sensitive disclosure constraints and multi-surface state presentation.
