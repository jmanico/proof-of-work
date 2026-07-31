# [REQ-PRIVACY-010] Consent capture before any health-data write

## Metadata

- **ID**: REQ-PRIVACY-010
- **Title**: Consent capture before any health-data write
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: Critical
- **Requirement Type**: Privacy
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-9.2; `SECURITY.md` SEC-DATA-2

## Requirement

- **Statement**: The system MUST obtain and persist the user's explicit consent to collect and process their health data before any health data is recorded, and MUST refuse to record health data for a user with no active consent record.
- **Rationale**: FR-9.2 requires explicit consent before any health data is recorded; SEC-DATA-2 restates it as a server-side refusal rather than a UI step. Consent is the precondition on which every progress-logging behavior depends.
- **Assumptions**: The consent record is one of the entities the system owns (`ARCHITECTURE.md`, Data model expectations).
- **Out of Scope**: Consent withdrawal (REQ-PRIVACY-020); the medical disclaimer, which is a separate acknowledgement with its own requirement (FR-9.6, REQ-PRIVACY-040); the wording and legal sufficiency of the consent text, which depends on the governing regime and is blocked by `SECURITY.md` SQ-1 and `REQUIREMENTS.md` OQ-3; consent versioning and re-consent on policy change, which no source document specifies.
- **Design Traceability**: `DESIGN.md` — Components → Inputs (persistent visible label, required fields marked in the label not by color), Form feedback and errors, Focus states. `DESIGN.md` OQ-6 concerns the disclaimer's presentation, not consent.
- **Architecture Traceability**: `ARCHITECTURE.md` — REST API Application ("consent capture (FR-9.2)"); data flow 4 ("consent check, owner scoping, field validation"); Relational Persistence (consent record).
- **Security Traceability**: SEC-DATA-2; supports SEC-AUTHZ-1, SEC-LOG-1 (the consent decision itself is health-data-adjacent and audited).

## Scope

- **Applies To**: Server-Side Application, API, Web Client
- **Components**: REST API Application; Relational Persistence; Browser Client (consent view)
- **Interfaces / Operations**: Consent capture; the consent precondition on every health-data write
- **Actors**: `subscriber`
- **Preconditions**: Authenticated session; the actor is the subject of the consent
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data — the consent governs its collection; the consent record itself is personal data
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED — `SECURITY.md` SQ-1 and `REQUIREMENTS.md` OQ-3 leave the governing regime unresolved; FR-9.2 requires the behavior regardless.

## Security Context

- **Security Objectives**: Privacy, Accountability, Authorization
- **Control Layers**: Business-Rule Validation, Data Protection, Authorization
- **Threat References**: `SECURITY.md` TM-P-2 (unawareness — consent captured with no withdrawal path), TM-P-3 (non-compliance); LINDDUN Unawareness and Non-compliance categories as applied in the threat model; CWE-359 (exposure of private personal information)
- **Abuse / Misuse Case**: Health data is recorded for a user who never consented — through a path that skips the check, through a client that assumes consent, or through a consultant acting on a subscriber who has not consented — creating an unlawful processing record that cannot be undone.
- **Trust Boundary**: Boundary 1 — the client's assertion that consent was given is not authoritative.
- **Untrusted Inputs or Assertions**: Any client-supplied consent flag; the assumption that the consent UI was shown.
- **Authoritative Enforcement Point**: REST API Application — every health-data write checks the persisted consent record.
- **Independent Verification**: The check reads persisted consent state, not a request field or a session flag set at login.
- **Zero Trust Relevance**: NIST SP 800-207 — a per-request policy condition evaluated from authoritative state. Exact tenet: TO BE DECIDED (not verified in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: TO BE DECIDED — `REQUIREMENTS.md` asserts GDPR and CCPA data-subject rights and HIPAA obligations while `SECURITY.md` SQ-1 records that these are not consistent; the specific lawful-basis and consent-form obligations cannot be mapped until SQ-1 resolves.
- **Other**: `REF-ASVS-5` as cited by SEC-DATA-2.
- **Mapping Basis**: FR-9.2 and SEC-DATA-2 are the normative sources. No regulatory article is cited because the governing regime is explicitly unresolved and `REQUIREMENT_TEMPLATE.md` forbids guessing.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated subscriber who explicitly grants consent, when the consent is submitted, then a consent record is persisted against that subscriber with the time and the fact of the grant, and subsequent health-data writes are permitted.
2. **AC-02 — Boundary or failure behavior**: Given a subscriber with no consent record, when any health-data write is attempted — by the subscriber or by an engaged consultant — then it is refused, no health data is persisted, and the response states that consent is required.
3. **AC-03 — Prohibited behavior**: Given any request, when it carries a consent flag or asserts that consent was shown, then that assertion MUST NOT satisfy the check; consent MUST NOT be inferred from account creation, subscription, or disclaimer acknowledgement; and consent MUST NOT be recorded as a side effect of a health-data write.
4. **AC-04 — Additional criterion**: Given the consent view, when it is presented, then the consent action is an explicit affirmative act with a persistent visible label and is operable by keyboard with a visible focus indicator (`DESIGN.md`, Components).
5. **AC-05 — Additional criterion**: Given consent is granted or refused, when the event occurs, then an audit entry records the acting account, the action, and the time (REQ-AUDIT-020).

## Failure Behavior

- **On Invalid Input**: A malformed consent submission is rejected per REQ-API-010 with no record created.
- **On Authentication Failure**: Denied upstream; no consent record is created.
- **On Authorization Failure**: A request to grant consent on another subscriber's behalf is denied (REQ-AUTHZ-020); consent is a first-person act.
- **On Security-Decision Failure**: If consent state cannot be determined, refuse the health-data write (fail closed).
- **On External Dependency Failure**: If persistence is unavailable, refuse the write; MUST NOT proceed on an assumption of consent.
- **On System Error**: Roll back so neither a partial consent record nor an unconsented health record survives.
- **Logging / Audit**: Audit entry per AC-05. Log refusals with actor, operation, and reason class; MUST NOT log health values (SEC-LOG-3).
- **Alerting**: N/A

## Test Strategy

- **Unit Tests**: Consent gate permits when an active record exists, refuses when absent, and refuses when the state is unresolvable; consent recorder rejects a third-party grant.
- **Integration Tests**: Write attempts before and after consent for every health-data entity type; consultant write against an unconsented subscriber refused.
- **Security Tests**: Client-supplied consent flag ignored; consent not created implicitly by a health-data write; attempt to grant consent for another subscriber denied.
- **Compliance Tests / Evidence**: Retained consent records and audit entries demonstrating consent precedes the first health-data write, as evidence for FR-9.2.
- **Acceptance-Criteria Traceability**: AC-01 — consent grant suite; AC-02 — pre-consent refusal suite across all entity types; AC-03 — assertion and inference negative tests; AC-04 — client accessibility test; AC-05 — audit assertion.
- **Coverage Target**: Every health-data write path covered by a pre-consent refusal test.
- **Required Test Environment**: Subscribers with and without consent, one consultant with an active engagement, seeded entity fixtures. Engine and test framework TO BE DECIDED.

## Dependencies

- **Upstream Requirements**: REQ-AUTHZ-010, REQ-AUTHZ-020, REQ-AUDIT-020, REQ-API-010
- **Downstream Requirements**: REQ-PRIVACY-020, REQ-PROGRESS-010, REQ-PROGRESS-020, REQ-CONSULT-010
- **External Dependencies**: None
- **Dependency Assumptions**: None
- **Failure Impact**: Recording health data without consent is not correctable after the fact; it is the one failure in this area that cannot be remediated by later deletion alone.

## Implementation Notes

- **Constraints**: RDBMS engine TO BE DECIDED. The consent text's legal sufficiency depends on the unresolved regime (`SECURITY.md` SQ-1) — implement the mechanism and treat the wording as content to be supplied, not invented.
- **Prohibited Approaches**: A pre-ticked or implied consent control; bundling consent into subscription checkout or the medical disclaimer; caching consent in the session so a withdrawal does not take effect (REQ-PRIVACY-020 depends on this check reading current state); creating the record as a side effect of the first log entry.
- **Implementation Guidance**: Place the consent check inside the same health-data accessor that carries the audit dependency (REQ-AUDIT-020), so a new health-data path cannot acquire one obligation without the other.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-VUE`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Privacy and legal review of the consent mechanism and text; accessibility review of the consent view.
- **Open Decisions**: Governing regime (`SECURITY.md` SQ-1, `REQUIREMENTS.md` OQ-3), which determines the required consent wording, granularity, and whether separate consents are needed per processing purpose. Consent versioning and re-consent are unspecified in all source documents.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 250–500.
**Recommended model**: Claude Opus (`claude-opus-5`) — a privacy gate that must hold on every write path and fail closed, with an irreversible failure mode.
