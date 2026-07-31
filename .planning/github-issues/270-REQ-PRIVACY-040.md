# [REQ-PRIVACY-040] Medical disclaimer acknowledgement before first plan use

## Metadata

- **ID**: REQ-PRIVACY-040
- **Title**: Medical disclaimer acknowledgement before first plan use
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: High
- **Requirement Type**: Compliance
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-9.6

## Requirement

- **Statement**: The system MUST display a disclaimer stating that the plans and content are not medical advice, and MUST require the user to acknowledge it before they first use a plan; the acknowledgement MUST be recorded server-side and enforced server-side as a precondition of first plan use.
- **Rationale**: FR-9.6 states the requirement. The product delivers exercise and diet guidance that subscribers act on physically, so the disclaimer is a safety and liability control, and an acknowledgement the client alone tracks would not be evidence of anything.
- **Assumptions**: "First use a plan" means the first operation by which a subscriber selects, customizes, or logs against a plan — the plan-selection decision itself is blocked by `REQUIREMENTS.md` OQ-6, so the enforced trigger is the first of the unblocked plan-use operations.
- **Out of Scope**: The disclaimer's exact wording, which no source document supplies and which requires legal review; the presentation form — interstitial, checkbox, or persistent banner — which `DESIGN.md` OQ-6 explicitly leaves open; health-data consent, which is a separate requirement (FR-9.2, REQ-PRIVACY-010); plan selection semantics, blocked by OQ-6.
- **Design Traceability**: `DESIGN.md` — Layout and Spacing (max 72ch for long-form reading, "the medical disclaimer" named explicitly); Components → Buttons, Form feedback and errors, Focus states; Accessibility (keyboard operability, focus management when content opens or closes). `DESIGN.md` OQ-6 leaves the presentation and acknowledgement form open.
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application (FR-9.6 traceability row); Browser Client open decision ("Presentation and acknowledgement flow for the medical disclaimer (FR-9.6; DESIGN.md OQ-6)").
- **Security Traceability**: SEC-TB-1 (the acknowledgement is re-derived server-side), SEC-INPUT-4 (business rules enforced as validation, not UI affordance), SEC-LOG-1.

## Scope

- **Applies To**: Server-Side Application, API, Web Client
- **Components**: REST API Application; Relational Persistence; Browser Client (disclaimer view)
- **Interfaces / Operations**: Disclaimer retrieval; acknowledgement submission; the acknowledgement precondition on first plan use
- **Actors**: `subscriber`
- **Preconditions**: Authenticated session
- **Data Classification**: Restricted — the acknowledgement record is personal data tied to health-related activity
- **Personal or Regulated Data**: Personal Data
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED (`SECURITY.md` SQ-1, `REQUIREMENTS.md` OQ-3)

## Security Context

- **Security Objectives**: Safety, Accountability, Integrity
- **Control Layers**: Business-Rule Validation, Logging and Monitoring
- **Threat References**: `SECURITY.md` TM-T-5 (harmful published content — the disclaimer is one of the standing mitigations that content carries a caveat), TM-T-1 (client-asserted state); CWE-602 (client-side enforcement of server-side security)
- **Abuse / Misuse Case**: A subscriber reaches plan content without ever being shown the disclaimer because the client skipped it, or the acknowledgement exists only in browser storage and is lost, spoofed, or never verifiable when it matters.
- **Trust Boundary**: Boundary 1 — the client's claim to have shown and collected the acknowledgement is not authoritative.
- **Untrusted Inputs or Assertions**: Any client-supplied acknowledgement flag; local storage state.
- **Authoritative Enforcement Point**: REST API Application — the precondition is checked on the plan-use operations themselves.
- **Independent Verification**: The acknowledgement is read from a persisted record keyed to the account.
- **Zero Trust Relevance**: N/A — a product precondition, not a resource-access decision.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: N/A
- **Regulatory**: TO BE DECIDED — no source document names a statute requiring the disclaimer; `REQUIREMENTS.md` mandates it as product behavior, and the applicable regime is blocked by `SECURITY.md` SQ-1 and `REQUIREMENTS.md` OQ-3.
- **Other**: WCAG 2.2 AA as adopted by `DESIGN.md`, for the presentation and focus behavior of the acknowledgement.
- **Mapping Basis**: FR-9.6 is the normative source. No regulatory mapping is asserted because none is documented and `REQUIREMENT_TEMPLATE.md` forbids guessing.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a subscriber who has not acknowledged the disclaimer, when they first attempt to use a plan, then the operation is refused pending acknowledgement, and the disclaimer text stating that the content is not medical advice is retrievable and presented.
2. **AC-02 — Boundary or failure behavior**: Given a subscriber who submits an acknowledgement, when it succeeds, then the acknowledgement is persisted against their account with the time, and subsequent plan-use operations proceed; given the acknowledgement write fails, the operation remains refused and nothing is recorded.
3. **AC-03 — Prohibited behavior**: Given any plan-use request, when it carries a client-supplied acknowledgement flag or relies on browser storage, then that assertion MUST NOT satisfy the precondition, and the acknowledgement MUST NOT be recorded implicitly as a side effect of viewing a plan.
4. **AC-04 — Additional criterion**: Given the disclaimer presentation, when it renders, then its text is constrained to at most 72ch for readability, it is fully operable by keyboard with a visible focus indicator, and focus is managed deliberately when it opens and closes (`DESIGN.md`).
5. **AC-05 — Additional criterion**: Given an acknowledgement, when it is recorded, then an audit entry captures the acting account, the action, and the time.

## Failure Behavior

- **On Invalid Input**: A malformed acknowledgement submission is rejected per REQ-API-010; nothing is recorded.
- **On Authentication Failure**: Denied upstream; no acknowledgement recorded.
- **On Authorization Failure**: An acknowledgement submitted on another account's behalf is denied (REQ-AUTHZ-020).
- **On Security-Decision Failure**: If acknowledgement state cannot be resolved, refuse the plan-use operation (fail closed) — an unresolvable state is not treated as acknowledged.
- **On External Dependency Failure**: If persistence is unavailable, refuse plan use rather than proceed unacknowledged.
- **On System Error**: Roll back; generic error with a correlation identifier.
- **Logging / Audit**: Audit entry per AC-05; log refusals with reason class.
- **Alerting**: N/A

## Test Strategy

- **Unit Tests**: The precondition gate refuses without a persisted acknowledgement, permits with one, and refuses when state is unresolvable; the recorder rejects a third-party submission.
- **Integration Tests**: Plan-use operations before and after acknowledgement; acknowledgement persisted with time; audit entry present.
- **Security Tests**: Client-supplied flag ignored; browser-storage-only state does not satisfy the gate; acknowledgement not created implicitly by viewing a plan; cross-account submission denied.
- **Compliance Tests / Evidence**: Retained acknowledgement records and audit entries demonstrating acknowledgement preceded first plan use, as evidence for FR-9.6.
- **Acceptance-Criteria Traceability**: AC-01 — pre-acknowledgement refusal suite; AC-02 — acknowledgement and failure-path tests; AC-03 — client-assertion negative suite; AC-04 — presentation and keyboard tests; AC-05 — audit assertion.
- **Coverage Target**: Every plan-use operation covered by a pre-acknowledgement refusal test.
- **Required Test Environment**: Subscribers with and without an acknowledgement; seeded published plans. Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-AUTHZ-010, REQ-AUTHZ-020, REQ-AUDIT-020, REQ-PLATFORM-030
- **Downstream Requirements**: REQ-CUSTOM-010, REQ-PROGRESS-020
- **External Dependencies**: None
- **Dependency Assumptions**: None
- **Failure Impact**: Plan guidance reaching a subscriber with no recorded acknowledgement removes the only stated safety caveat on content that people act on physically.

## Implementation Notes

- **Constraints**: PostgreSQL with Drizzle ORM (`CLAUDE.md`); client build tooling TO BE DECIDED. `DESIGN.md` OQ-6 leaves the presentation form open, so this issue delivers the mechanism and a presentation that satisfies the documented layout and accessibility rules; the chosen form must be confirmed when OQ-6 resolves. The disclaimer wording is content to be supplied by legal review, not invented here.
- **Prohibited Approaches**: Storing the acknowledgement only in the client; a pre-checked acknowledgement control; treating a page view as acknowledgement; bundling the disclaimer acknowledgement with health-data consent (FR-9.2), which is a separate right with separate withdrawal semantics.
- **Implementation Guidance**: Because plan *selection* is blocked by `REQUIREMENTS.md` OQ-6, apply the precondition to the plan-use operations that do exist — customization (REQ-CUSTOM-010) and workout logging against a plan (REQ-PROGRESS-020) — and extend it to selection when OQ-6 resolves.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-VUE`; `CLAUDE.md`.
- **Required Human Review**: Legal review of the disclaimer text; product and design review once `DESIGN.md` OQ-6 resolves; accessibility review of the presentation.
- **Open Decisions**: `DESIGN.md` OQ-6 (presentation and acknowledgement form) and `REQUIREMENTS.md` OQ-6 (plan selection semantics, which defines the full trigger set). This issue's coverage of FR-9.6 is therefore partial: the mechanism and enforcement are complete, the presentation form and the selection trigger are not.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 250–500.
**Recommended model**: Claude Opus (`claude-opus-5`) — small but safety-relevant, and the server-side enforcement is what distinguishes it from a UI banner.
