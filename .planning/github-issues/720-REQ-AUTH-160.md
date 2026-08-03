# [REQ-AUTH-160] Vetting record required on privileged invitations

## Metadata

- **ID**: REQ-AUTH-160
- **Title**: Vetting record required on privileged invitations
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: High
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-2.16 (resolves the vetting half of OQ-13 and `SECURITY.md` SQ-12); `SECURITY.md` SEC-AUTHN-9

## Requirement

- **Statement**: The system MUST record, on every `admin` and `consultant` invitation, who vetted the invitee, when, and on what basis, and MUST refuse to issue an invitation without that record.
- **Rationale**: Vetting itself — identity and fitness-credential checks — happens out of band by the operator; the record makes it auditable (FR-2.16), so that every privileged account traces to a named vetting decision and SEC-AUTHN-9's invitation path carries accountability, not just a token.
- **Assumptions**: The invitation mechanism, enrolment token, and role scoping exist (SEC-AUTHN-9; REQ-AUTH-140). The invitee's email address is the one recorded with this vetting record — for an invitee with no existing account, the address is attested by the record and control is proven by redemption of the token (SEC-AUTHN-9, 2026-08-03 wording).
- **Out of Scope**: The invitation and enrolment-token mechanics, delivery, expiry, and first-passkey enrolment (REQ-AUTH-140); the out-of-band vetting process itself — what checks the operator performs is an operator business matter with no system behavior attached (OQ-13); deprovisioning (REQ-AUTH-170).
- **Design Traceability**: `DESIGN.md` — Credentials, account security, and administration → People (admin): "The invitation form requires the vetting record — who vetted the invitee, when, and on what basis — before **Send invitation** is available (FR-2.16), and says the record is kept."
- **Architecture Traceability**: `ARCHITECTURE.md` — Identity and Session Handling (privileged enrolment invitation entity); REST API Application; traceability row FR-2.1–FR-2.18 ("vetting recorded on invitations"); DR-2 (the client-side form gating is convenience only — refusal is server-side).
- **Security Traceability**: SEC-AUTHN-9 ("An invitation MUST NOT be issued without the FR-2.16 vetting record"), SEC-LOG-4 (invitation issuance audit); supports SEC-AUTHN-13's fresh-invitation re-enable path.

## Scope

- **Applies To**: Multiple — API, Server-Side Application, Web Client
- **Components**: REST API Application; Identity and Session Handling; Relational Persistence; Browser Client (People invitation form)
- **Interfaces / Operations**: Privileged-invitation issuance (admin workspace, People); invitation audit reads through the SEC-LOG-5 investigative interface
- **Actors**: `admin` (issuer); invitee (subject of the record); `subscriber` and `consultant` (denied issuers)
- **Preconditions**: An authenticated `admin` session; out-of-band vetting already performed by the operator
- **Data Classification**: Confidential
- **Personal or Regulated Data**: Personal Data — the vetting record names the vetter, the invitee, and the basis; no health data is involved
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA for US persons named in the record; HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED). Section-level mappings TO BE DECIDED (SQ-1 counsel review).

## Security Context

- **Security Objectives**: Accountability, Authenticity, Integrity
- **Control Layers**: Business-Rule Validation, Authorization, Logging and Monitoring, Input Validation
- **Threat References**: `SECURITY.md` TM-S-4 (privileged-account bootstrap), TM-E-1 (privilege escalation via provisioning flows) — this record is the accountability half of their mitigation; CWE-778 (insufficient logging) for the refused-without-record class
- **Abuse / Misuse Case**: A compromised or hostile admin invites an accomplice without any vetting trail, leaving no way to establish after the fact who approved the privileged account, when, or on what basis; or an issuer submits an empty or placeholder record to satisfy a client-side check.
- **Trust Boundary**: Boundary 1 — the vetting-record fields arrive as client input from the admin's browser; boundary 2 — issuance is a privileged authenticated operation.
- **Untrusted Inputs or Assertions**: The submitted vetting-record fields (who, when, basis) — validated for presence and shape server-side; the client's assertion that the form was completed.
- **Authoritative Enforcement Point**: REST API Application invitation-issuance operation — the refusal without a complete vetting record is enforced server-side, never by the form alone (DR-2).
- **Independent Verification**: Issuer identity and `admin` role come from Identity and Session Handling, not from the request (DR-3); the record is stored with the invitation and readable through the audited investigative interface.
- **Zero Trust Relevance**: TO BE DECIDED — not verified against NIST SP 800-207 in this session.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — per-issue mapping awaits the SQ-10 independent pre-launch assessment.
- **OWASP AISVS 1.0**: N/A — no AI-enabled component is involved.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: The SQ-1 regime set governs the personal data in the record (GDPR/UK GDPR; CCPA/CPRA); no spec document states a section-level mapping for this requirement — TO BE DECIDED.
- **Other**: `REF-AUTH`, `REF-ASVS-5` as cited by SEC-AUTHN-9; `REF-LOG` as cited by SEC-LOG-4.
- **Mapping Basis**: SEC-AUTHN-9 and SEC-LOG-4 name these references for the invitation lifecycle this record gates.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an authenticated `admin` and a complete vetting record (who vetted, when, on what basis), when an `admin` or `consultant` invitation is issued, then the invitation is created carrying that record, the record is persisted with the invitation, and issuance produces an audit entry (SEC-LOG-4).
2. **AC-02 — Boundary or failure behavior**: Given a vetting record with any field absent, empty, or whitespace-only, when issuance is attempted, then it is refused with the specific failing field identified, and no invitation or enrolment token is created.
3. **AC-03 — Prohibited behavior**: Given any request path — including one bypassing the Browser Client form — when an invitation issuance omits the vetting record, then no invitation MUST be issued; client-side form gating MUST NOT be the enforcing control, and the record MUST NOT be addable after issuance to legitimize an invitation issued without one.
4. **AC-04 — Auditability**: Given an issued invitation, when an `admin` reads it through the SEC-LOG-5 investigative interface, then the vetting record (who, when, basis) is retrievable alongside the issuance audit entry, and that read is itself audited.

## Failure Behavior

- **On Invalid Input**: Reject against the allow-list schema (SEC-INPUT-1) naming the failing field; no invitation, token, or partial record is created.
- **On Authentication Failure**: Denied by the deny-by-default gate (REQ-AUTHZ-010) with a uniform response.
- **On Authorization Failure**: Non-`admin` actors are denied issuance (SEC-AUTHZ-4-style role restriction via the policy module); the denial does not disclose whether the invitee address is registered (SEC-AUTHN-3).
- **On Security-Decision Failure**: Deny by default — any error validating or persisting the vetting record refuses issuance; the invitation and its record are written atomically or not at all.
- **On External Dependency Failure**: If the mail interface cannot deliver the invitation, the vetting record and refusal semantics are unaffected — delivery behavior belongs to REQ-AUTH-140/REQ-INFRA-060.
- **On System Error**: Generic error with a correlation identifier (SEC-ERR-1); no invitation without its record persists.
- **Logging / Audit**: Audit entry on every issuance carrying acting admin, invitee, role, time, and the vetting record; refused issuances logged with the reason class; no enrolment token value in any log (SEC-LOG-3, SEC-AUTHN-11).
- **Alerting**: Threshold alerts on repeated refused privileged issuance route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-3, SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: Record-completeness validation per field (absent, empty, whitespace-only); atomic create-with-record persistence rule; refusal reason attribution.
- **Integration Tests**: Issue an invitation with a complete record and assert persistence plus the audit entry; direct-API issuance without a record asserting refusal; investigative-interface retrieval of the record with its own audit entry.
- **Security Tests**: Bypass the client form and post issuance requests with missing or forged record fields, asserting server-side refusal (DR-2); attempt issuance as `subscriber` and `consultant`, asserting denial; attempt post-hoc record attachment to a recordless invitation fixture, asserting rejection.
- **Compliance Tests / Evidence**: The stored vetting record and issuance audit trail as SQ-12 accountability evidence.
- **Acceptance-Criteria Traceability**: AC-01 — issuance integration suite; AC-02 — field-validation unit suite; AC-03 — form-bypass and post-hoc security suite; AC-04 — investigative-interface integration test.
- **Coverage Target**: Positive and negative coverage of every validation branch and the refusal path (project threshold TO BE DECIDED, `CLAUDE.md`).
- **Required Test Environment**: Vitest and HTTP test client; `admin`, `consultant`, and `subscriber` fixtures; invitation persistence and audit-store fixtures.

## Dependencies

- **Upstream Requirements**: REQ-AUTH-140 (invitation issuance and enrolment-token mechanics this record gates)
- **Downstream Requirements**: REQ-AUTH-170 (re-enable after deprovisioning happens only via fresh invitations, all of which carry this record)
- **External Dependencies**: None
- **Dependency Assumptions**: REQ-AUTH-140's issuance operation exposes a single server-side path this gate can wrap — no alternate issuance route exists (SEC-AUTHN-9).
- **Failure Impact**: If REQ-AUTH-140 is absent, there is nothing to gate; if this gate is bypassed, privileged accounts can be created with no accountable vetting trail, undoing the SQ-12 resolution.

## Implementation Notes

- **Constraints**: Node.js with Fastify; the vetting record is stored with the privileged enrolment invitation entity (`ARCHITECTURE.md`) and survives invitation expiry so the trail outlives the token.
- **Prohibited Approaches**: Enforcing the record only in the Browser Client form (DR-2); defaulting or auto-filling any record field server-side; accepting the vetting date from the client without shape validation; allowing record mutation after issuance; treating the record as optional metadata rather than an issuance precondition.
- **Implementation Guidance**: Validate who/basis as non-empty bounded text and the vetting time as a past-or-present timestamp via the route JSON Schema (SEC-INPUT-1, Fastify built-ins per `CLAUDE.md`); persist invitation and record in one transaction; surface the record read-only in the admin People detail view per DESIGN.md.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-NODE`, `REF-PROMPT-QUALITY`; `CLAUDE.md` working rules.
- **Required Human Review**: Security review confirming no issuance path skips the gate; product review that form copy matches DESIGN.md ("says the record is kept").
- **Open Decisions**: None — FR-2.16 and SEC-AUTHN-9 fix the behavior; per-issue standards mappings await the SQ-10 assessment.

**Estimated effort**: 0.5–1 engineer-days. **Estimated changed lines**: 150–350.
**Recommended model**: Claude Opus (`claude-opus-5`) — a small gate on the privileged-provisioning path where a bypass or client-only check silently removes the accountability SQ-12 depends on.
