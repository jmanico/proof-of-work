# [REQ-PRIVACY-040] Medical disclaimer acknowledgement before first plan use

## Metadata

- **ID**: REQ-PRIVACY-040
- **Title**: Medical disclaimer acknowledgement before first plan use
- **Version**: 1.1.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: High
- **Requirement Type**: Compliance
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-9.6

## Requirement

- **Statement**: The system MUST display a disclaimer stating that the plans and content are not medical advice, and MUST require the user to acknowledge it before they first use a plan; the acknowledgement MUST be recorded server-side and enforced server-side as a precondition of first plan use.
- **Rationale**: FR-9.6 states the requirement. The product delivers exercise and diet guidance that subscribers act on physically, so the disclaimer is a safety and liability control, and an acknowledgement the client alone tracks would not be evidence of anything.
- **Assumptions**: "First use a plan" means the first operation by which a subscriber selects, customizes, or logs against a plan. `REQUIREMENTS.md` OQ-6 is RESOLVED: "The FR-9.6 disclaimer gates the first plan use, including first selection" — so the trigger set is first plan selection (FR-5.3, FR-6.4), first customization (FR-7.1), and first logging against a plan (FR-8.3).
- **Out of Scope**: The disclaimer's exact wording, which no source document supplies and which requires legal review; health-data consent, which is a separate requirement with separate withdrawal semantics (FR-9.2, REQ-PRIVACY-010; `DESIGN.md` treats the two as separate decisions); the selection operations themselves (REQ-SELECT-010, REQ-SELECT-020) and the customization and logging operations (REQ-CUSTOM-010, REQ-PROGRESS-020) — this issue supplies the precondition they enforce.
- **Design Traceability**: `DESIGN.md` — Product Patterns → Medical disclaimer and health-data consent (OQ-6 RESOLVED): immediately before the subscriber first selects or customizes a plan, a focused interstitial shows a short plain-language summary, a link to the full disclaimer, a secondary Back action, and a primary **I understand — continue** action; no pre-checked checkbox; after acknowledgement, plan pages retain a quiet "Not medical advice · Safety information" link near the Evidence section, and the blocking interstitial does not repeat unless the disclaimer materially changes. Also: Typography ("long-form text is limited to 68ch"); Core Components → Dialogs (focus trap, focus return); Accessibility.
- **Architecture Traceability**: `ARCHITECTURE.md` — Data model expectations ("medical-disclaimer acknowledgement (FR-9.6, versioned)"); REST API Application ("Owns … consent record, disclaimer acknowledgement"); Browser Client open decisions ("The medical disclaimer uses a first-plan acknowledgement interstitial plus a persistent safety link"); FR-9.6 traceability row.
- **Security Traceability**: SEC-TB-1 (the acknowledgement is re-derived server-side), SEC-INPUT-4 (business rules enforced as validation, not UI affordance), SEC-LOG-1.

## Scope

- **Applies To**: Server-Side Application, API, Web Client
- **Components**: REST API Application; Relational Persistence; Browser Client (disclaimer interstitial)
- **Interfaces / Operations**: Disclaimer retrieval; acknowledgement submission; the acknowledgement precondition on first plan use (selection, customization, logging against a plan)
- **Actors**: `subscriber`
- **Preconditions**: Authenticated session
- **Data Classification**: Restricted — the acknowledgement record is personal data tied to health-related activity
- **Personal or Regulated Data**: Personal Data
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED)

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

- **OWASP ASVS 5.0.0**: TO BE DECIDED — per-issue mappings are verified during the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — per-issue mappings are verified during the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **NIST SP 800-207**: N/A
- **Regulatory**: No source document names a statute requiring the disclaimer; `REQUIREMENTS.md` mandates it as product behavior ("a medical disclaimer stating the content is not medical advice must be shown"). The acknowledgement record is personal data governed by the SQ-1 regime set named under Jurisdiction; statute-section precision: TO BE DECIDED (per-issue verification work, `SECURITY.md` SQ-1).
- **Other**: WCAG 2.2 AA as adopted by `DESIGN.md`, for the presentation and focus behavior of the acknowledgement interstitial.
- **Mapping Basis**: FR-9.6 is the normative source. No statutory mapping is asserted because none is documented and `REQUIREMENT_TEMPLATE.md` forbids guessing.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a subscriber who has not acknowledged the disclaimer, when they first attempt to use a plan — select it, customize it, or log against it — then the operation is refused pending acknowledgement, and the disclaimer text stating that the content is not medical advice is retrievable and presented.
2. **AC-02 — Boundary or failure behavior**: Given a subscriber who submits an acknowledgement, when it succeeds, then the acknowledgement is persisted against their account with the time and the disclaimer version it covers (`ARCHITECTURE.md`: the acknowledgement entity is versioned; the interstitial repeats only when the disclaimer materially changes), and subsequent plan-use operations proceed; given the acknowledgement write fails, the operation remains refused and nothing is recorded.
3. **AC-03 — Prohibited behavior**: Given any plan-use request, when it carries a client-supplied acknowledgement flag or relies on browser storage, then that assertion MUST NOT satisfy the precondition, and the acknowledgement MUST NOT be recorded implicitly as a side effect of viewing a plan.
4. **AC-04 — Additional criterion**: Given the disclaimer presentation, when it renders, then it is the focused interstitial `DESIGN.md` specifies — plain-language summary, link to the full disclaimer, secondary Back action, primary **I understand — continue** action, no pre-checked control — its text is constrained to at most 68ch, it is fully operable by keyboard with a visible focus indicator, and focus is managed deliberately when it opens and closes; after acknowledgement, plan pages carry the quiet "Not medical advice · Safety information" link.
5. **AC-05 — Additional criterion**: Given an acknowledgement, when it is recorded, then an audit entry captures the acting account, the action, and the time.

## Failure Behavior

- **On Invalid Input**: A malformed acknowledgement submission is rejected per REQ-API-010; nothing is recorded.
- **On Authentication Failure**: Denied upstream; no acknowledgement recorded.
- **On Authorization Failure**: An acknowledgement submitted on another account's behalf is denied (REQ-AUTHZ-020).
- **On Security-Decision Failure**: If acknowledgement state cannot be resolved, refuse the plan-use operation (fail closed) — an unresolvable state is not treated as acknowledged.
- **On External Dependency Failure**: If persistence is unavailable, refuse plan use rather than proceed unacknowledged.
- **On System Error**: Roll back; generic error with a correlation identifier.
- **Logging / Audit**: Audit entry per AC-05; log refusals with reason class.
- **Alerting**: N/A — pre-acknowledgement refusals are ordinary product flow, not a security event class named by SEC-LOG-4, and no threshold condition attaches to this requirement.

## Test Strategy

- **Unit Tests**: The precondition gate refuses without a persisted acknowledgement, permits with one, and refuses when state is unresolvable; the recorder rejects a third-party submission.
- **Integration Tests**: Plan selection, customization, and logging operations before and after acknowledgement; acknowledgement persisted with time and disclaimer version; audit entry present.
- **Security Tests**: Client-supplied flag ignored; browser-storage-only state does not satisfy the gate; acknowledgement not created implicitly by viewing a plan; cross-account submission denied.
- **Compliance Tests / Evidence**: Retained acknowledgement records and audit entries demonstrating acknowledgement preceded first plan use, as evidence for FR-9.6.
- **Acceptance-Criteria Traceability**: AC-01 — pre-acknowledgement refusal suite across selection, customization, and logging; AC-02 — acknowledgement and failure-path tests; AC-03 — client-assertion negative suite; AC-04 — interstitial presentation and keyboard tests; AC-05 — audit assertion.
- **Coverage Target**: Every plan-use operation — selection, customization, and logging — covered by a pre-acknowledgement refusal test.
- **Required Test Environment**: Subscribers with and without an acknowledgement; seeded published plans. Vitest for API tests; the interstitial's keyboard and focus behavior belongs to the Playwright + axe suite (`DESIGN.md`, Design Verification: keyboard-only completion of the disclaimer flow).

## Dependencies

- **Upstream Requirements**: REQ-AUTHZ-010, REQ-AUTHZ-020, REQ-AUDIT-020, REQ-PLATFORM-030
- **Downstream Requirements**: REQ-SELECT-010, REQ-SELECT-020, REQ-CUSTOM-010, REQ-PROGRESS-020
- **External Dependencies**: None
- **Dependency Assumptions**: None
- **Failure Impact**: Plan guidance reaching a subscriber with no recorded acknowledgement removes the only stated safety caveat on content that people act on physically.

## Implementation Notes

- **Constraints**: PostgreSQL with Drizzle ORM (`CLAUDE.md`); the client is a Vite-built single-page application with `vue-router`. The presentation form is fixed by `DESIGN.md` (OQ-6 RESOLVED): a focused first-plan interstitial plus a persistent quiet safety link — no persistent warning banner. The disclaimer wording is content to be supplied by legal review, not invented here.
- **Prohibited Approaches**: Storing the acknowledgement only in the client; a pre-checked acknowledgement control; treating a page view as acknowledgement; a persistent warning banner in place of the interstitial (`DESIGN.md` reserves banners for current actionable states); bundling the disclaimer acknowledgement with health-data consent (FR-9.2), which is a separate right with separate withdrawal semantics.
- **Implementation Guidance**: Apply the precondition to all plan-use operations: selection (REQ-SELECT-010, REQ-SELECT-020), customization (REQ-CUSTOM-010), and workout logging against a plan (REQ-PROGRESS-020). Note that selection and customization are also health-data writes under FR-9.12 and therefore separately gated by consent (FR-9.2) and email verification (FR-2.11); the disclaimer acknowledgement is a distinct third precondition, not a substitute for either. Version the persisted acknowledgement so a materially changed disclaimer can re-trigger the interstitial without discarding the history of what was acknowledged when.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-VUE`; `CLAUDE.md`.
- **Required Human Review**: Legal review of the disclaimer text (SQ-1 pre-launch counsel review); accessibility review of the interstitial.
- **Open Decisions**: None. `DESIGN.md` OQ-6 (presentation form) and `REQUIREMENTS.md` OQ-6 (plan selection semantics) are both RESOLVED; the disclaimer wording awaits legal review as a content deliverable, not a specification decision.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 250–500.
**Recommended model**: Claude Opus (`claude-opus-5`) — small but safety-relevant, and the server-side enforcement is what distinguishes it from a UI banner.
