# [REQ-AUDIT-030] Admin plan lifecycle audit entries

## Metadata

- **ID**: REQ-AUDIT-030
- **Title**: Admin plan lifecycle audit entries
- **Version**: 1.1.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: High
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-10.2, FR-5.4; `SECURITY.md` SEC-LOG-6; threats TM-R-1, TM-T-5

## Requirement

- **Statement**: Every admin plan lifecycle action — create, edit, verify, publish, unpublish — and every admin exercise-catalog action — create, edit, retire (FR-5.4) — MUST produce exactly one audit entry recording the acting admin, the action, the affected plan or catalog entry, and the time.
- **Rationale**: FR-10.2 states the plan-lifecycle rule; SEC-LOG-6 restates it; FR-5.4 (added 2026-08-03) requires catalog create, edit, and retire actions to produce audit entries per FR-10.2. The threat model added FR-10.2 because only verification was previously recorded, leaving a hostile admin action repudiable (TM-R-1), and because a compromised admin can publish harmful exercise or diet content that subscribers follow as health guidance (TM-T-5) — accountability is the compensating control now that dual-control verification has been consciously declined (`REQUIREMENTS.md` OQ-16 RESOLVED; TM-T-5 RISK ACCEPTED).
- **Assumptions**: The audit entry model and its append-only semantics exist (REQ-AUDIT-010). The catalog operations themselves are delivered by REQ-PLAN-080; this issue supplies their audit obligation.
- **Out of Scope**: Health-data access auditing (REQ-AUDIT-020); the verification operation itself, resolved as a one-time gate that never re-triggers on edit (`REQUIREMENTS.md` OQ-10 and OQ-16 RESOLVED; FR-4.8 — delivered by REQ-PLAN-070); the catalog management operations (FR-5.4 — delivered by REQ-PLAN-080); retention and the hash-chained archive (`SECURITY.md` SQ-8 RESOLVED — SEC-LOG-5, SEC-LOG-7, delivered by REQ-INFRA-040).
- **Design Traceability**: `DESIGN.md` OQ-5 RESOLVED — subscribers see "Evidence reviewed by Proof of Work" with the date, admins see verifier identity and publication gates; that presentation is separate from this record. The admin exercise catalog (DESIGN.md, Credentials, account security, and administration) shows rename and Retire actions whose accountability this issue records.
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application; Relational Persistence; data flow 6 (admin publication); FR-10.2 traceability row.
- **Security Traceability**: SEC-LOG-6, SEC-LOG-2 (append-only), SEC-AUTHZ-4 (only admins reach these actions; catalog actions are admin-restricted by FR-5.4).

## Scope

- **Applies To**: Server-Side Application, API
- **Components**: REST API Application; Relational Persistence
- **Interfaces / Operations**: Plan create, plan edit, citation change as part of an edit, verify, publish, unpublish; exercise-catalog create, edit (including rename), retire (FR-5.4)
- **Actors**: `admin`
- **Preconditions**: An authorized admin plan lifecycle or exercise-catalog operation is about to execute (REQ-AUTHZ-030)
- **Data Classification**: Internal — plan and catalog content is not personal data; the entry identifies the acting admin
- **Personal or Regulated Data**: Personal Data — the acting admin's account identifier
- **Jurisdiction / Regulatory Scope**: For the acting admin's identifier as personal data, the SQ-1 regime set applies (GDPR/UK GDPR for EU/UK data subjects; CCPA/CPRA for US users; HIPAA not applicable — `SECURITY.md` SQ-1 RESOLVED); plan and catalog content carries no health data

## Security Context

- **Security Objectives**: Accountability, Integrity, Safety
- **Control Layers**: Logging and Monitoring, Business-Rule Validation
- **Threat References**: `SECURITY.md` TM-R-1 (unaudited admin plan lifecycle actions), TM-T-5 (compromised admin publishes harmful content); CWE-778 (insufficient logging), CWE-223 (omission of security-relevant information)
- **Abuse / Misuse Case**: A compromised or hostile admin authors, verifies, and publishes dangerous exercise or diet guidance and later denies it, or quietly edits a published plan with no record of who changed what and when.
- **Trust Boundary**: Boundary 1 for the request; the audit obligation sits on the trusted side.
- **Untrusted Inputs or Assertions**: The acting admin identity MUST come from the verified session, not the request (DR-3).
- **Authoritative Enforcement Point**: The plan lifecycle and exercise-catalog services in the REST API Application, with the audit write inside the same transaction as the state change.
- **Independent Verification**: A test enumerating the five plan lifecycle actions and the three catalog actions asserts one entry each, independently of handler code.
- **Zero Trust Relevance**: N/A

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — mapped only when verified during the independent pre-launch assessment (`SECURITY.md` SQ-10 RESOLVED).
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — mapped only when verified during the independent pre-launch assessment (`SECURITY.md` SQ-10 RESOLVED).
- **NIST SP 800-207**: N/A
- **Regulatory**: The accountability obligation here is product and threat-model driven, not statutory; plan and catalog content is not personal data. The acting admin's identifier is personal data under the SQ-1 regime set (GDPR/UK GDPR; CCPA/CPRA); statute-section precision: TO BE DECIDED — per-issue mappings await the SQ-1 pre-launch counsel review.
- **Other**: `REF-LOG` as cited by SEC-LOG-6.
- **Mapping Basis**: FR-10.2 and SEC-LOG-6 are the normative sources; both trace to threat-model entries TM-R-1 and TM-T-5 recorded in `SECURITY.md`.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an admin performing create, edit, verify, publish, or unpublish on a plan, when the operation succeeds, then exactly one audit entry records the acting admin, which of the five actions occurred, the affected plan, and the time.
2. **AC-02 — Boundary or failure behavior**: Given the audit write fails, when a lifecycle operation is attempted, then the plan state change is not persisted and the operation returns a generic error with a correlation identifier.
3. **AC-03 — Prohibited behavior**: Given any lifecycle operation, when it executes, then it MUST NOT complete without an entry, MUST NOT record an acting admin taken from the request body, and MUST NOT produce an entry for an operation that was denied or that failed its publication gate.
4. **AC-04 — Additional criterion**: Given a sequence of lifecycle actions on one plan, when the entries are read back, then the full history — who created, who edited, who verified, who published, who unpublished, and when — is reconstructable in order.
5. **AC-05 — Additional criterion**: Given a lifecycle inventory test, when it runs, then all five plan lifecycle actions and all three catalog actions are covered and a newly added lifecycle or catalog operation without an audit dependency fails the test.
6. **AC-06 — Additional criterion**: Given an admin performing create, edit (including rename), or retire on an exercise-catalog entry (FR-5.4), when the operation succeeds, then exactly one audit entry records the acting admin, which catalog action occurred, the affected catalog entry by its stable identifier, and the time.

## Failure Behavior

- **On Invalid Input**: Rejected upstream by REQ-API-010; no entry for a rejected request.
- **On Authentication Failure**: Denied upstream; no entry.
- **On Authorization Failure**: Denied per REQ-AUTHZ-030 and logged as a security event; MUST NOT be written as a lifecycle audit entry (AC-03).
- **On Security-Decision Failure**: If the acting admin cannot be resolved, deny the operation rather than write an entry with an unknown actor.
- **On External Dependency Failure**: If audit persistence is unavailable, the lifecycle operation fails closed (AC-02).
- **On System Error**: The entry and the state change share a transaction; a rollback removes both.
- **Logging / Audit**: This issue defines the audit entries. Failures of the audit path are logged as security events (SEC-LOG-4).
- **Alerting**: Threshold alerts route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-11 RESOLVED). Single-admin author-verify-publish is permitted by decision (`REQUIREMENTS.md` OQ-16 RESOLVED — TM-T-5 RISK ACCEPTED), so it is not an alert condition; these entries are the compensating post-hoc review record.

## Test Strategy

- **Unit Tests**: The lifecycle and catalog services invoke the audit writer exactly once per action with the correct action value; refuse to proceed when the writer fails.
- **Integration Tests**: Each of the five plan actions and each of the three catalog actions performed as an admin, asserting one correct entry each; a full lifecycle sequence read back in order; a catalog rename entry carries the entry's stable identifier (FR-5.4).
- **Security Tests**: Denied operations produce no lifecycle entry; a request-supplied admin identifier is ignored; fault-injected audit failure leaves plan state unchanged; a lifecycle inventory test that fails on a bypassing operation.
- **Compliance Tests / Evidence**: The lifecycle audit coverage report, retained as accountability evidence for FR-10.2.
- **Acceptance-Criteria Traceability**: AC-01 — per-action entry assertions; AC-02 — audit-failure fault injection; AC-03 — denial and identity-binding tests; AC-04 — sequence reconstruction test; AC-05 — inventory test; AC-06 — per-catalog-action entry assertions.
- **Coverage Target**: All five plan lifecycle actions and all three catalog actions, positive and negative.
- **Required Test Environment**: Two admin identities (authorship and verification distinguishable in the record even though the verifier MAY be the author — FR-4.8, OQ-16 RESOLVED), seeded draft and published plans, seeded catalog entries, audit storage. Runs against PostgreSQL on Vitest.

## Dependencies

- **Upstream Requirements**: REQ-AUDIT-010, REQ-AUTHZ-030, REQ-AUTH-010
- **Downstream Requirements**: REQ-PLAN-030, REQ-PLAN-040, REQ-PLAN-050, REQ-PLAN-060, REQ-PLAN-070, REQ-PLAN-080
- **External Dependencies**: None
- **Dependency Assumptions**: Audit storage shares the transactional context of plan and catalog storage.
- **Failure Impact**: Without these entries a harmful publication cannot be attributed. Dual control was consciously declined (OQ-16 RESOLVED; TM-T-5 RISK ACCEPTED), so this audit trail is a named compensating control alongside passkey-only admin authentication (SEC-AUTHN-2) and the two-admin minimum (FR-2.15).

## Implementation Notes

- **Constraints**: PostgreSQL with Drizzle ORM (`CLAUDE.md`). The verification workflow is resolved as a one-time gate before first publication that never re-triggers on edit (`REQUIREMENTS.md` OQ-10 and OQ-16 RESOLVED; FR-4.8, delivered by REQ-PLAN-070); recording who verified and when is required by FR-4.5 and FR-10.2. Catalog entries are retire-never-delete with stable identifiers (FR-5.4), so a catalog entry referenced by an audit record always remains resolvable.
- **Prohibited Approaches**: A generic "plan changed" entry that does not distinguish the actions; audit calls placed in handlers rather than in the lifecycle or catalog service; writing the entry outside the state change's transaction; recording the full plan body in the entry; a catalog audit entry that references the entry by mutable name rather than stable identifier.
- **Implementation Guidance**: Record the action as a closed enumeration — the five plan values plus the three catalog values — so AC-04's reconstruction and any later review query are exact.
- **AI Development Guidance**: `REF-LOG`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the action enumeration and the coverage inventory.
- **Open Decisions**: None. `REQUIREMENTS.md` OQ-10 and OQ-16 are RESOLVED (2026-08-01): verification is a one-time gate, the verifier MAY be the author, and the residual TM-T-5 risk is accepted with this audit trail as a named compensating control. The verification operation is delivered by REQ-PLAN-070 and catalog management by REQ-PLAN-080.

**Estimated effort**: 0.5–1 engineer-day. **Estimated changed lines**: 150–350.
**Recommended model**: Claude Opus (`claude-opus-5`) — small and mechanical, but it is the compensating control for the highest safety-impact threat in the model.
