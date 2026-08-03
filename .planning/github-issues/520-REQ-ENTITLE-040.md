# [REQ-ENTITLE-040] Record retention across subscription lapse

## Metadata

- **ID**: REQ-ENTITLE-040
- **Title**: Record retention across subscription lapse
- **Version**: 1.0.1
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-3.4; `SECURITY.md` SEC-AUTHZ-8

## Requirement

- **Statement**: The system MUST retain a subscriber's existing progress records and plan customizations unchanged when their subscription lapses — denying access without deleting, altering, or archiving-with-loss any record — and MUST restore access to those same records intact when the subscription becomes active again.
- **Rationale**: FR-3.4 requires retention across lapse and restoration on reactivation; SEC-AUTHZ-8 makes it a security rule that "lapsed entitlement MUST deny access while preserving the underlying records." Payment is out of band in v1 (OQ-1), so lapses caused by administrative timing must never cost a subscriber their health-data history. Deny-not-delete also keeps the data-subject rights over that data (FR-9.3–FR-9.5) meaningful during a lapse.
- **Assumptions**: The entitlement gate exists and denies gated operations during lapse (REQ-ENTITLE-010); reactivation happens via an admin-granted period (REQ-ENTITLE-030).
- **Out of Scope**: The denial behavior and messaging itself (REQ-ENTITLE-010, FR-3.2); deletion at the user's request, which is available regardless of subscription state (FR-9.4, REQ-PRIVACY-030/REQ-PRIVACY-090); export during lapse (FR-9.3 is a data-subject right, not a subscription feature — its availability is defined by the export issue, REQ-PRIVACY-080); consent-withdrawal effects on new writes (FR-9.9); retention periods for audit data (SEC-LOG-5, REQ-INFRA-040).
- **Design Traceability**: `DESIGN.md` — Status, feedback, and loading: "Permission, lapsed-subscription, withdrawn-consent, and ended-engagement states explain the exact reason and the available next step without exposing hidden data"; banners reserved for current actionable states such as lapsed subscription.
- **Architecture Traceability**: `ARCHITECTURE.md` — Relational Persistence ("Must retain progress records and plan customizations across subscription lapse (FR-3.4)"); REST API Application traceability row FR-3.4 (SUPPORTED — REST API Application; Relational Persistence); DR-4 (log entries and plan copies are mutable only by their documented owners — a lapse is not a mutation event).
- **Security Traceability**: SEC-AUTHZ-8 (deny while preserving records); supports SEC-DATA-4 by ensuring lapse is never conflated with deletion; supports FR-9.3–FR-9.5 rights over retained data.

## Scope

- **Applies To**: Server-Side Application, API
- **Components**: REST API Application; Relational Persistence
- **Interfaces / Operations**: Subscription-lapse and reactivation transitions; all storage of workout, food, body-weight, and body-measurement log entries, customized plan copies, and active plan selections
- **Actors**: `subscriber` whose subscription lapses and is later reactivated; `admin` performing the reactivating grant (REQ-ENTITLE-030)
- **Preconditions**: The subscriber previously had an active subscription and recorded progress entries and/or plan customizations
- **Data Classification**: Restricted — the retained records are health data (FR-9.12)
- **Personal or Regulated Data**: Health Data — log entries, plan copies, and active selections (FR-9.12)
- **Jurisdiction / Regulatory Scope**: `SECURITY.md` SQ-1 RESOLVED — GDPR/UK GDPR for EU/UK data subjects; CCPA/CPRA, Washington My Health My Data, and the FTC Health Breach Notification Rule for US users; HIPAA not applicable. Specific sections: TO BE DECIDED.

## Security Context

- **Security Objectives**: Integrity, Availability, Authorization
- **Control Layers**: Business-Rule Validation, Data Protection, Architecture
- **Threat References**: `SECURITY.md` TM-E-4 (the lapse mechanism must not become a destructive side channel); CWE-471 (modification of assumed-immutable data); N/A for attacker-driven classes beyond these — the primary risk is destructive system behavior, not an external attacker
- **Abuse / Misuse Case**: A lapse handler "cleans up" lapsed accounts by deleting or archiving log entries or plan copies; a reactivation restores records from a lossy copy; or the lapse transition mutates active plan selections so that reactivation silently changes what the subscriber had selected.
- **Trust Boundary**: Boundary 3 (REST API Application → Relational Persistence) — the persistence layer must see no destructive operation triggered by an entitlement transition.
- **Untrusted Inputs or Assertions**: None from clients — lapse and reactivation are derived from persisted period records and the clock (FR-3.6); no client input participates in the transition.
- **Authoritative Enforcement Point**: REST API Application — the absence of any lapse-triggered destructive code path, verified by test; Relational Persistence retains all rows across the transition.
- **Independent Verification**: Record equality is asserted around the lapse/reactivation cycle in tests; access denial during lapse is enforced independently by the REQ-ENTITLE-010 gate.
- **Zero Trust Relevance**: N/A — this requirement constrains system behavior over stored data, not resource-access decisions.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — mapping deferred to the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A — no AI-enabled component is involved.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — mapping deferred per SQ-10.
- **NIST SP 800-207**: N/A — see Zero Trust Relevance.
- **Regulatory**: The SQ-1 regime set applies — the retained records are health data over which data-subject rights (FR-9.3–FR-9.5) persist through a lapse. Specific articles/sections: TO BE DECIDED.
- **Other**: `REF-ASVS-5`, `REF-PROMPT-ABAC` as cited by SEC-AUTHZ-8.
- **Mapping Basis**: SEC-AUTHZ-8 names these references and states the deny-while-preserving rule this issue implements.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a subscriber with progress entries, customized plan copies, and active plan selections, when their subscription lapses and is later reactivated by an admin grant, then every record — entry values, units, dates, copy contents, and selections — is identical before the lapse and after reactivation, and gated access to them works again.
2. **AC-02 — Boundary or failure behavior**: Given a subscriber who remains lapsed for an extended time, when any scheduled execution or maintenance path runs (`ARCHITECTURE.md`, scheduled executions of the REST API Application), then no lapsed account's progress records, plan copies, or selections are deleted, mutated, or degraded by reason of the lapse.
3. **AC-03 — Prohibited behavior**: Given a subscription-lapse transition, when it takes effect, then the system MUST NOT delete, overwrite, tombstone, or archive-with-loss any of the subscriber's records, and MUST NOT alter the active plan selections — lapse is exclusively an access denial (SEC-AUTHZ-8), and history is never altered by selection state changes (FR-5.3, FR-6.4).
4. **AC-04 — Rights survive the lapse**: Given a lapsed subscriber, when they exercise deletion under FR-9.4, then deletion proceeds per its own requirement — demonstrating that retention under this issue never blocks the user's own data rights over the retained records.

## Failure Behavior

- **On Invalid Input**: N/A — no client input participates in lapse or reactivation transitions.
- **On Authentication Failure**: N/A — the transition is state-derived, not request-driven; access attempts during lapse are handled by REQ-ENTITLE-010.
- **On Authorization Failure**: N/A — covered by REQ-ENTITLE-010's denial behavior.
- **On Security-Decision Failure**: If entitlement state cannot be determined, deny access (SEC-AUTHZ-7) — never delete or modify records as a fallback.
- **On External Dependency Failure**: N/A — no external dependency exists on this path.
- **On System Error**: Any error during a reactivating grant rolls back per REQ-ENTITLE-030; stored records are untouched by definition because no code path writes them on entitlement transitions.
- **Logging / Audit**: The grant/revocation actions that cause lapse or reactivation are audited by REQ-ENTITLE-030 (FR-3.5); no additional audit event exists for the passive lapse itself. Post-reactivation reads of health data are audited per FR-9.7 by the owning feature issues.
- **Alerting**: N/A — no alert condition of its own; a retention violation is a defect surfaced by tests, and destructive-operation anomalies below the application fall under SEC-LOG-7/SEC-OPS-1 controls (REQ-INFRA-040, REQ-INFRA-050).

## Test Strategy

- **Unit Tests**: Assert no lapse-transition code path issues DELETE or UPDATE against log-entry, plan-copy, or selection tables; the entitlement transition is a pure read of period records (with REQ-ENTITLE-010's active-state function).
- **Integration Tests**: Full lapse cycle — seed records, expire the period, assert denial (via REQ-ENTITLE-010), grant a new period (via REQ-ENTITLE-030), and assert byte-level record equality and restored access (AC-01); long-lapse fixture across scheduled-execution runs asserting no decay (AC-02).
- **Security Tests**: Attempt to trigger destructive behavior through repeated lapse/reactivate cycles; assert selections are unchanged (AC-03); assert FR-9.4 deletion remains available while lapsed (AC-04).
- **Compliance Tests / Evidence**: The record-equality integration result as evidence for the FR-3.4 retention obligation.
- **Acceptance-Criteria Traceability**: AC-01 — lapse-cycle equality suite; AC-02 — scheduled-execution retention suite; AC-03 — destructive-path and selection-integrity assertions; AC-04 — deletion-while-lapsed test.
- **Coverage Target**: Project coverage threshold is 90% line and branch (`CLAUDE.md`, 2026-08-03); the lapse/reactivation cycle MUST be covered end to end with equality assertions.
- **Required Test Environment**: PostgreSQL with migrations applied; a subscriber identity with seeded records of every health-data type (FR-9.12); a controllable clock to force lapse and reactivation; Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-ENTITLE-010
- **Downstream Requirements**: None drafted — the guarantee is consumed implicitly by every progress, customization, and selection issue.
- **External Dependencies**: None
- **Dependency Assumptions**: REQ-ENTITLE-010 denies without touching records; REQ-ENTITLE-030 reactivation is a pure period-row addition.
- **Failure Impact**: A violation destroys subscriber health-data history on an administrative timing event — irreversible data loss with regulatory exposure under the SQ-1 regimes, and a broken FR-3.4 promise that undermines the product's "clear record of effort."

## Implementation Notes

- **Constraints**: Node.js with Fastify; PostgreSQL with Drizzle ORM (`CLAUDE.md`). Largely a property of what is *not* built: entitlement transitions must have no write path to health-data tables, and the tests must pin that.
- **Prohibited Approaches**: Lapse-triggered cleanup jobs, soft-delete flags, or archival moves on subscriber records; clearing active plan selections on lapse (FR-4.9 ends selections on unpublication — lapse has no such effect); conflating lapse with account deletion (FR-9.4) or consent withdrawal (FR-9.9), which have their own distinct semantics.
- **Implementation Guidance**: Because "active" is computed from period records at request time (REQ-ENTITLE-010), the cleanest implementation stores nothing per-record about entitlement at all — then retention is structural and the tests merely guard against future regressions. Verify the scheduled executions named in `ARCHITECTURE.md` (nightly archival, ledger maintenance, FR-9.11 expiry) never select on subscription state for destructive work.
- **AI Development Guidance**: `REF-PROMPT-QUALITY`, `REF-PROMPT-ABAC`; `CLAUDE.md`.
- **Required Human Review**: Review of every scheduled execution and maintenance path for destructive operations keyed on subscription state.
- **Open Decisions**: None.

**Estimated effort**: 0.5–1 engineer-days. **Estimated changed lines**: 150–350.
**Recommended model**: Claude Opus (`claude-opus-5`) — small in code but protecting irreplaceable health-data history; the value is in exhaustively pinning the absence of destructive paths.
