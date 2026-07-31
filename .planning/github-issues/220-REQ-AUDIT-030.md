# [REQ-AUDIT-030] Admin plan lifecycle audit entries

## Metadata

- **ID**: REQ-AUDIT-030
- **Title**: Admin plan lifecycle audit entries
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: High
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-10.2; `SECURITY.md` SEC-LOG-6; threats TM-R-1, TM-T-5

## Requirement

- **Statement**: Every admin plan lifecycle action — create, edit, verify, publish, unpublish — MUST produce exactly one audit entry recording the acting admin, the action, the affected plan, and the time.
- **Rationale**: FR-10.2 states the rule; SEC-LOG-6 restates it. The threat model added FR-10.2 because only verification was previously recorded, leaving a hostile admin action repudiable (TM-R-1), and because a compromised admin can publish harmful exercise or diet content that subscribers follow as health guidance (TM-T-5) — accountability is the compensating control while dual-control verification remains an open question.
- **Assumptions**: The audit entry model and its append-only semantics exist (REQ-AUDIT-010).
- **Out of Scope**: Health-data access auditing (REQ-AUDIT-020); whether verification requires a second admin, which `REQUIREMENTS.md` OQ-16 leaves open; whether an edit invalidates a prior verification, which OQ-10 leaves open; retention (blocked by `SECURITY.md` SQ-8).
- **Design Traceability**: `DESIGN.md` OQ-5 asks how verification status is surfaced to readers; that presentation is open and separate from this record.
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application; Relational Persistence; data flow 6 (admin publication); FR-10.2 traceability row.
- **Security Traceability**: SEC-LOG-6, SEC-LOG-2 (append-only), SEC-AUTHZ-4 (only admins reach these actions).

## Scope

- **Applies To**: Server-Side Application, API
- **Components**: REST API Application; Relational Persistence
- **Interfaces / Operations**: Plan create, plan edit, citation change as part of an edit, verify, publish, unpublish
- **Actors**: `admin`
- **Preconditions**: An authorized admin plan lifecycle operation is about to execute (REQ-AUTHZ-030)
- **Data Classification**: Internal — plan content is not personal data; the entry identifies the acting admin
- **Personal or Regulated Data**: Personal Data — the acting admin's account identifier
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED (`SECURITY.md` SQ-1)

## Security Context

- **Security Objectives**: Accountability, Integrity, Safety
- **Control Layers**: Logging and Monitoring, Business-Rule Validation
- **Threat References**: `SECURITY.md` TM-R-1 (unaudited admin plan lifecycle actions), TM-T-5 (compromised admin publishes harmful content); CWE-778 (insufficient logging), CWE-223 (omission of security-relevant information)
- **Abuse / Misuse Case**: A compromised or hostile admin authors, verifies, and publishes dangerous exercise or diet guidance and later denies it, or quietly edits a published plan with no record of who changed what and when.
- **Trust Boundary**: Boundary 1 for the request; the audit obligation sits on the trusted side.
- **Untrusted Inputs or Assertions**: The acting admin identity MUST come from the verified session, not the request (DR-3).
- **Authoritative Enforcement Point**: The plan lifecycle service in the REST API Application, with the audit write inside the same transaction as the state change.
- **Independent Verification**: A test enumerating the five lifecycle actions asserts one entry each, independently of handler code.
- **Zero Trust Relevance**: N/A

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: N/A
- **Regulatory**: N/A — plan content is not personal data; the accountability obligation here is product and threat-model driven, not statutory.
- **Other**: `REF-LOG` as cited by SEC-LOG-6.
- **Mapping Basis**: FR-10.2 and SEC-LOG-6 are the normative sources; both trace to threat-model entries TM-R-1 and TM-T-5 recorded in `SECURITY.md`.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an admin performing create, edit, verify, publish, or unpublish on a plan, when the operation succeeds, then exactly one audit entry records the acting admin, which of the five actions occurred, the affected plan, and the time.
2. **AC-02 — Boundary or failure behavior**: Given the audit write fails, when a lifecycle operation is attempted, then the plan state change is not persisted and the operation returns a generic error with a correlation identifier.
3. **AC-03 — Prohibited behavior**: Given any lifecycle operation, when it executes, then it MUST NOT complete without an entry, MUST NOT record an acting admin taken from the request body, and MUST NOT produce an entry for an operation that was denied or that failed its publication gate.
4. **AC-04 — Additional criterion**: Given a sequence of lifecycle actions on one plan, when the entries are read back, then the full history — who created, who edited, who verified, who published, who unpublished, and when — is reconstructable in order.
5. **AC-05 — Additional criterion**: Given a lifecycle inventory test, when it runs, then all five actions are covered and a newly added lifecycle operation without an audit dependency fails the test.

## Failure Behavior

- **On Invalid Input**: Rejected upstream by REQ-API-010; no entry for a rejected request.
- **On Authentication Failure**: Denied upstream; no entry.
- **On Authorization Failure**: Denied per REQ-AUTHZ-030 and logged as a security event; MUST NOT be written as a lifecycle audit entry (AC-03).
- **On Security-Decision Failure**: If the acting admin cannot be resolved, deny the operation rather than write an entry with an unknown actor.
- **On External Dependency Failure**: If audit persistence is unavailable, the lifecycle operation fails closed (AC-02).
- **On System Error**: The entry and the state change share a transaction; a rollback removes both.
- **Logging / Audit**: This issue defines the audit entries. Failures of the audit path are logged as security events (SEC-LOG-4).
- **Alerting**: TO BE DECIDED — publication of a plan by a single admin is a plausible alert candidate given TM-T-5, but no alerting model exists in the source documents and `REQUIREMENTS.md` OQ-16 leaves dual control open.

## Test Strategy

- **Unit Tests**: The lifecycle service invokes the audit writer exactly once per action with the correct action value; refuses to proceed when the writer fails.
- **Integration Tests**: Each of the five actions performed as an admin, asserting one correct entry each; a full lifecycle sequence read back in order.
- **Security Tests**: Denied operations produce no lifecycle entry; a request-supplied admin identifier is ignored; fault-injected audit failure leaves plan state unchanged; a lifecycle inventory test that fails on a bypassing operation.
- **Compliance Tests / Evidence**: The lifecycle audit coverage report, retained as accountability evidence for FR-10.2.
- **Acceptance-Criteria Traceability**: AC-01 — per-action entry assertions; AC-02 — audit-failure fault injection; AC-03 — denial and identity-binding tests; AC-04 — sequence reconstruction test; AC-05 — inventory test.
- **Coverage Target**: All five lifecycle actions, positive and negative.
- **Required Test Environment**: Two admin identities (so authorship and verification can be distinguished for the OQ-16 decision later), seeded draft and published plans, audit storage. Engine and test framework TO BE DECIDED.

## Dependencies

- **Upstream Requirements**: REQ-AUDIT-010, REQ-AUTHZ-030, REQ-AUTH-010
- **Downstream Requirements**: REQ-PLAN-030, REQ-PLAN-040, REQ-PLAN-050, REQ-PLAN-060
- **External Dependencies**: None
- **Dependency Assumptions**: Audit storage shares the transactional context of plan storage.
- **Failure Impact**: Without these entries a harmful publication cannot be attributed, which is the only accountability control standing while OQ-16 (dual-control verification) remains open.

## Implementation Notes

- **Constraints**: RDBMS engine TO BE DECIDED. The verify action is auditable here even though the verification *workflow* is blocked by `REQUIREMENTS.md` OQ-10 and OQ-16 — recording who verified and when is required by FR-4.5 and FR-10.2 independently of those questions.
- **Prohibited Approaches**: A generic "plan changed" entry that does not distinguish the five actions; audit calls placed in handlers rather than in the lifecycle service; writing the entry outside the state change's transaction; recording the full plan body in the entry.
- **Implementation Guidance**: Record the action as a closed enumeration of the five values so AC-04's reconstruction and any later review query are exact.
- **AI Development Guidance**: `REF-LOG`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the action enumeration and the coverage inventory.
- **Open Decisions**: `REQUIREMENTS.md` OQ-16 (dual-control verification) and OQ-10 (re-verification after edit) remain open. Neither changes this issue's obligations; both would add constraints to the verification operation itself, which is blocked.

**Estimated effort**: 0.5–1 engineer-day. **Estimated changed lines**: 150–350.
**Recommended model**: Claude Opus (`claude-opus-5`) — small and mechanical, but it is the compensating control for the highest safety-impact threat in the model.
