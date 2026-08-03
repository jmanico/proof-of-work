# [REQ-PRIVACY-030] View and correct personal data

## Metadata

- **ID**: REQ-PRIVACY-030
- **Title**: View and correct personal data
- **Version**: 1.1.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: High
- **Requirement Type**: Privacy
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-9.5

## Requirement

- **Statement**: The system MUST allow a user to view the personal data held about them and to correct it, scoped strictly to their own account.
- **Rationale**: FR-9.5 states the right. GDPR-grade data-subject rights are granted to all users regardless of jurisdiction (`SECURITY.md` SQ-1 RESOLVED, Applicable privacy or regulatory obligations), and correction is one of the mandated behaviors.
- **Assumptions**: "Personal data held about them" means the account profile fields the system stores. Health log entries are corrected through their own edit operations (FR-8.7, REQ-PROGRESS-010, REQ-PROGRESS-020), not through this surface.
- **Out of Scope**: Data export (FR-9.3 — delivered by REQ-PRIVACY-080) and deletion (FR-9.4 — delivered by REQ-PRIVACY-090); correction of health log entries, owned by the progress issues; changing the email address, which is its own requirement — FR-2.18, delivered by REQ-AUTH-180 with re-authentication, verify-new-first sequencing, notice to the old address, and session termination (SEC-AUTHN-7, SEC-AUTHN-8, SEC-AUTHN-12) — and is excluded here; role, subscription state, and any server-controlled field, which are not user-correctable (SEC-INPUT-3).
- **Design Traceability**: `DESIGN.md` — Core Components → Forms and validation (persistent visible labels, required fields marked in text, inline errors with a summary linking each invalid field and focus moved to the first); Accessibility (focus, announcements); Product Patterns → Account, privacy, and destructive actions (Account groups Profile, Units and appearance, Security, Subscription, Consultant access, and Privacy); Layout, Spacing, and Responsive Behavior (40rem single-column form, 68ch prose).
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application (FR-9.5 traceability row); Browser Client; data flow 8 (owner scoping and sensitive-operation authorization).
- **Security Traceability**: SEC-AUTHZ-1, SEC-AUTHZ-2 (owner scoping), SEC-INPUT-1, SEC-INPUT-3 (server-controlled fields not settable), SEC-DATA-5 (least-privilege response), SEC-LOG-1 (access to health-adjacent data is audited).

## Scope

- **Applies To**: Server-Side Application, API, Web Client
- **Components**: REST API Application; Relational Persistence; Browser Client (account profile view)
- **Interfaces / Operations**: Retrieve own personal data; update own correctable personal data
- **Actors**: `subscriber`, `consultant`, `admin` — each for their own account
- **Preconditions**: Authenticated session
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Personal Data
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable; GDPR-grade rights granted to all users (`SECURITY.md` SQ-1 RESOLVED, `REQUIREMENTS.md` OQ-3 RESOLVED)

## Security Context

- **Security Objectives**: Privacy, Integrity, Authorization, Confidentiality
- **Control Layers**: Authorization, Input Validation, Business-Rule Validation, Data Protection
- **Threat References**: `SECURITY.md` TM-I-1 (BOLA), TM-I-3 (excessive data exposure), TM-E-1 (privilege escalation through a profile update), TM-T-1 (mass assignment); CWE-639 (authorization bypass through user-controlled key), CWE-915 (mass assignment)
- **Abuse / Misuse Case**: An actor reads or corrects another account's profile by substituting an identifier, or escalates privilege by submitting `role` or subscription state through the correction endpoint.
- **Trust Boundary**: Boundary 1 — the target account identity comes from the session, never the request.
- **Untrusted Inputs or Assertions**: The submitted field set, including any server-controlled field; any account identifier in the request.
- **Authoritative Enforcement Point**: REST API Application — owner scoping and the correctable-field allow-list.
- **Independent Verification**: The record retrieved and updated is resolved from the session identity (REQ-AUTHZ-020).
- **Zero Trust Relevance**: NIST SP 800-207 — per-request authorization of a resource identified by the subject rather than the request. Exact tenet: TO BE DECIDED (not verified in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — mapped only when verified during the independent pre-launch assessment (`SECURITY.md` SQ-10 RESOLVED).
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — mapped only when verified during the independent pre-launch assessment (`SECURITY.md` SQ-10 RESOLVED).
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: The SQ-1 regime set governs (GDPR/UK GDPR for EU/UK data subjects; CCPA/CPRA, Washington My Health My Data, FTC HBNR for US users; HIPAA not applicable), with GDPR-grade rights granted to all users; the correction behavior is required regardless of jurisdiction. Statute-section precision: TO BE DECIDED — per-issue mappings await the SQ-1 pre-launch counsel review.
- **Other**: `REF-API-2023`, `REF-PROMPT-API` for object-level authorization and excessive data exposure.
- **Mapping Basis**: FR-9.5 is the normative source; the API references are those `SECURITY.md` cites for the authorization and response-shaping rules this issue depends on.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated actor, when they request their personal data, then the response contains the personal data held about their own account and nothing belonging to another account.
2. **AC-02 — Boundary or failure behavior**: Given a correction request with an invalid value for a correctable field, when it is submitted, then it is rejected, the response names the specific failing field and the rule violated, no field is changed, and the client moves focus to the first invalid field (`DESIGN.md`).
3. **AC-03 — Prohibited behavior**: Given a correction request, when it is processed, then it MUST NOT change `role`, subscription state, engagement state, consent state, audit fields, or any other server-controlled field, and MUST NOT act on an account identifier supplied in the request.
4. **AC-04 — Additional criterion**: Given a successful correction, when it completes, then the change is persisted, an audit entry records the acting account, the action, and the time, and the confirmation is announced to assistive technology without stealing focus (`DESIGN.md`).
5. **AC-05 — Additional criterion**: Given the personal-data view, when it renders, then it presents every correctable field with a persistent visible label and marks required fields in the label rather than by color alone.

## Failure Behavior

- **On Invalid Input**: Reject per REQ-API-010 with field-level detail; no partial update is persisted.
- **On Authentication Failure**: Denied upstream by REQ-AUTHZ-010.
- **On Authorization Failure**: Deny; a request naming another account is refused without disclosing whether that account exists (REQ-AUTHZ-020, REQ-AUTHZ-040).
- **On Security-Decision Failure**: Deny if the subject cannot be resolved from the session.
- **On External Dependency Failure**: If persistence is unavailable, the read returns a generic error and the correction fails atomically.
- **On System Error**: Roll back so no partial correction survives; generic error with a correlation identifier.
- **Logging / Audit**: Audit entry per AC-04. Log the field names changed but MUST NOT log the values, which are personal data (SEC-LOG-3).
- **Alerting**: N/A — the source documents attach no alert condition to profile correction; denials are ordinary authorization events logged under SEC-LOG-4, whose threshold alerts route to the security lead as SEC-OPS-2 detection inputs (SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: Correctable-field allow-list rejects each server-controlled field; validation rules per correctable field; the response builder returns only own-account fields.
- **Integration Tests**: Read and correct own data end to end; assert the audit entry; assert an omitted field is left unchanged rather than reset.
- **Security Tests**: Cross-account read and correction attempts denied; mass-assignment submission of `role`, subscription state, and audit fields ignored or rejected; response-shape assertion that no other subject's data or internal field is present.
- **Compliance Tests / Evidence**: Evidence that a data subject can view and correct their own record, retained for FR-9.5.
- **Acceptance-Criteria Traceability**: AC-01 — own-data read suite; AC-02 — validation and focus-management tests; AC-03 — mass-assignment and cross-account negative suite; AC-04 — audit and announcement tests; AC-05 — form accessibility test.
- **Coverage Target**: Every correctable field positive and negative; every server-controlled field covered by a mass-assignment test.
- **Required Test Environment**: Two accounts of the same role plus one of each other role; Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-AUTHZ-010, REQ-AUTHZ-020, REQ-API-010, REQ-API-020, REQ-AUDIT-020, REQ-PLATFORM-030
- **Downstream Requirements**: None
- **External Dependencies**: None
- **Dependency Assumptions**: None
- **Failure Impact**: A correction endpoint without field binding is a direct privilege-escalation path; without owner scoping it is a profile-disclosure path.

## Implementation Notes

- **Constraints**: PostgreSQL with Drizzle ORM (`CLAUDE.md`); the client is a Vite-built single-page application with `vue-router`. Email address changes are excluded because the address is also the recovery anchor: FR-2.18 (added 2026-08-03) gives the change its own flow — fresh re-authentication, verification of the new address before it replaces the old, notice to the previous address, session termination, audit, and refusal while an FR-9.11 deletion request is pending — delivered by REQ-AUTH-180, with initial verification delivered by REQ-AUTH-090 (SEC-AUTHN-8).
- **Prohibited Approaches**: A generic profile-update endpoint that binds the whole body; returning the full account record including credential, role, and audit fields; treating correction as sufficient to satisfy export (FR-9.3, REQ-PRIVACY-080) or deletion (FR-9.4, REQ-PRIVACY-090), which are separate requirements; routing an email-address change through this surface instead of REQ-AUTH-180.
- **Implementation Guidance**: Derive the correctable-field allow-list from the same declaration used by REQ-API-020 so a new account field cannot become silently correctable.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-VUE`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Privacy review of which fields are exposed and correctable; security review of the binding allow-list.
- **Open Decisions**: Which fields constitute "the personal data held about them" is not enumerated by any source document beyond the account entity; the set delivered here is reviewed at the SQ-1 pre-launch counsel review (`SECURITY.md` SQ-1 RESOLVED fixes the regime set). Email change is resolved: FR-2.18, delivered by REQ-AUTH-180.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 250–550.
**Recommended model**: Claude Opus (`claude-opus-5`) — a user-facing surface that is simultaneously a mass-assignment and BOLA target.
