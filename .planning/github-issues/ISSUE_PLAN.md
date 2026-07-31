# ISSUE_PLAN.md — Specification decomposition

- **Scope**: ALL requirements in `REQUIREMENTS.md`
- **Execution mode**: `DRAFT_ONLY` — no GitHub issue was created, and no mutating `gh` command was run
- **Sources**: `REQUIREMENTS.md`, `ARCHITECTURE.md`, `SECURITY.md`, `DESIGN.md`, `REQUIREMENT_TEMPLATE.md`; `CLAUDE.md` followed as agent instruction, not as product specification
- **Produced**: 2026-07-31
- **Result**: 1 epic + 48 leaf issues drafted; 22 areas of scope blocked with no issue body

---

## 1. Open Questions

Each `PQ-*` names the source identifiers it derives from and the scope it blocks. None is resolved here. Two classes are distinguished deliberately:

- **BLOCKING** — affects observable behavior, roles or permissions, auth, API contracts, data semantics, trust boundaries, security controls, failure behavior, or acceptance testing. Scope depending on one is decomposed no further and receives no issue body.
- **NON-BLOCKING FOR DRAFTING** — prevents implementation from starting but does not change what the behavior must be, so acceptance criteria can be written without it.

### Non-blocking for drafting

- **PQ-1 — No technology decision exists.** `CLAUDE.md`, Repository state: language, package manager, Node.js server framework, RDBMS engine, migration tooling, test framework and runner, lint and format tooling, CI platform, local run instructions, and directory layout are all `TO BE DECIDED`. This blocks *starting* every implementation issue but changes none of their acceptance criteria, which are stated at the HTTP, data, and interface level. Every issue records it under Constraints. **No issue may resolve any part of it.**
- **PQ-2 — Brand and presentation identity.** `DESIGN.md` OQ-1 (product name), OQ-2 (brand personality), OQ-3 (dark mode), OQ-9 (imagery policy); `DESIGN.md` Typography (brand typography UNKNOWN). Affects REQ-PLATFORM-010 and the logo's treatment; the documented interim palette and system font stack are sufficient to proceed.

### Blocking

- **PQ-3 — Session model.** `SECURITY.md` SQ-2, SEC-SESSION-3, SEC-SESSION-5, SEC-SESSION-7; `ARCHITECTURE.md` (Authentication and session model, TO BE DECIDED). Token lifetime, refresh strategy, transport (cookie versus header), storage, revocation mechanism, and signing key store are all undecided. Blocks: FR-2.4 logout revocation; SEC-HTTP-4 CSRF applicability; SEC-SESSION-3, SEC-SESSION-5, SEC-SESSION-7. Threats TM-S-5, TM-S-6 remain CONDITIONAL.
- **PQ-4 — Subscription activation and payments.** `REQUIREMENTS.md` OQ-1, OQ-2; `ARCHITECTURE.md` (subscription state, UNKNOWN); `SECURITY.md` SEC-AUTHZ-8. What makes a subscription "active", how it is purchased, renewed, cancelled, and whether a trial or free tier exists. Blocks: FR-3.1, FR-3.2, FR-3.3, FR-3.4, FR-11.1. Threat TM-E-4 CONDITIONAL.
- **PQ-5 — Password policy, account recovery, MFA recovery, lockout.** `REQUIREMENTS.md` OQ-8; `SECURITY.md` SEC-AUTHN-6, TM-D-2 (lockout design must not permit third-party-triggered permanent lockout). Blocks: FR-2.2 registration, all recovery flows. Threat TM-S-2 CONDITIONAL.
- **PQ-6 — MFA factors.** `REQUIREMENTS.md` OQ-9. Blocks: FR-2.5, FR-2.6.
- **PQ-7 — Email verification flow.** `REQUIREMENTS.md` OQ-15; `SECURITY.md` SEC-AUTHN-8, threat TM-S-3. Timing, token expiry, resend behavior. Blocks: FR-2.2 and the email-change path in REQ-PRIVACY-030.
- **PQ-8 — Password hashing algorithm and parameters.** `SECURITY.md` SEC-AUTHN-5 (`TO BE DECIDED`). A security control, so blocking. Blocks: credential storage.
- **PQ-9 — Body measurement fields and unit system.** `REQUIREMENTS.md` OQ-4; `DESIGN.md` OQ-8. Blocks: FR-8.2. Also leaves the unit for body weight and workout load open, which REQ-PROGRESS-010 and REQ-PROGRESS-020 handle by storing units explicitly.
- **PQ-10 — Nutrition data source.** `REQUIREMENTS.md` OQ-5. No external nutrition database is in scope and no first-party catalog is specified. Blocks: FR-8.4, FR-8.5.
- **PQ-11 — Progress history period, granularity, and visualization.** `REQUIREMENTS.md` OQ-7; `DESIGN.md` OQ-4; `ARCHITECTURE.md` Browser Client open decision. Blocks: FR-8.6.
- **PQ-12 — One or many active plans.** `REQUIREMENTS.md` OQ-6. Blocks: FR-5.2, FR-6.3, and the full trigger set for FR-9.6.
- **PQ-13 — Plan verification workflow.** `REQUIREMENTS.md` OQ-10 (re-verification after edit), OQ-16 (dual control); `SECURITY.md` threat TM-T-5. Blocks: the FR-4.5 verification *operation*. The verification *record* and the publication gate that reads it are covered by REQ-PLAN-050.
- **PQ-14 — Consultant capabilities.** `REQUIREMENTS.md` OQ-12; `SECURITY.md` threat TM-E-3, SQ-4. What a consultant may do within an engagement is undefined. Bounds REQ-CONSULT-010, which delivers *who* not *what*.
- **PQ-15 — Consultant onboarding, vetting, and the paid option.** `REQUIREMENTS.md` OQ-13; `SECURITY.md` SQ-12. Blocks: FR-11.1 and engagement creation.
- **PQ-16 — Export and deletion mechanics.** `REQUIREMENTS.md` FR-9.4 (deadline `TO BE DECIDED`); `SECURITY.md` SQ-5, SEC-DATA-4, SEC-DATA-6; `ARCHITECTURE.md` (synchronous versus deferred, TO BE DECIDED). Hard delete versus de-identification, backup and replica treatment, export format and artifact handling. Blocks: FR-9.3, FR-9.4. Threats TM-I-6, TM-I-7, TM-P-1 CONDITIONAL.
- **PQ-17 — Abuse-prevention thresholds.** `SECURITY.md` SQ-3, SEC-AUTHN-6, SEC-HTTP-5. Rate limits, lockout behavior, anti-automation. Blocks: SEC-AUTHN-6, SEC-HTTP-5, and all alerting fields across every issue. Threats TM-S-1, TM-D-1, TM-D-2 CONDITIONAL.
- **PQ-18 — ABAC attribute schema, policy language, and PDP/PEP architecture.** `SECURITY.md` SQ-4, SEC-AUTHZ-5, SEC-AUTHZ-6, SEC-AUTHZ-7. How capabilities map onto the three fixed roles is undefined. Blocks: the central policy engine. The concrete authorization rules (REQ-AUTHZ-010 … 040, REQ-CONSULT-010) are determined independently and are covered.
- **PQ-19 — CI/CD platform, secret store, AWS topology.** `SECURITY.md` SQ-7, SEC-CICD-1, SEC-CICD-3, SEC-CICD-4, SEC-SECRET-2, SEC-SESSION-7; `ARCHITECTURE.md` (Deployment model, TO BE DECIDED). Blocks: pipeline security gates, IaC baseline, secret management, encryption-at-rest configuration, network tiering, egress restriction. Threat TM-T-6 CONDITIONAL.
- **PQ-20 — Audit and log retention, access control, and tamper-evidence.** `SECURITY.md` SQ-8, SQ-13, SEC-LOG-5, SEC-LOG-7, SEC-OPS-1. Blocks: retention policy, tamper-evident audit storage, the operational break-glass model. Threats TM-R-2, TM-I-8, TM-P-4 open.
- **PQ-21 — Governing privacy regime.** `SECURITY.md` SQ-1, SQ-11; `REQUIREMENTS.md` OQ-3. The US-federal/state framing, GDPR/CCPA rights, and HIPAA obligations are recorded as mutually inconsistent. Blocks: every `Regulatory` field, data residency, breach notification, consent wording, and retention periods. Threat TM-P-3 CONDITIONAL. **This is the single most far-reaching open question in the specification.**
- **PQ-22 — Privileged account provisioning and first passkey enrolment.** `SECURITY.md` SQ-12; threats TM-S-4, TM-E-1. Blocks: admin and consultant account creation, role assignment and change, and the bootstrap path REQ-AUTH-030 assumes already happened.
- **PQ-23 — Third-party API exposure and CORS.** `SECURITY.md` SQ-6, SEC-HTTP-3. Blocks: SEC-HTTP-3.
- **PQ-24 — Exact CSP directive set.** `SECURITY.md` SEC-HTTP-2 (`TO BE DECIDED`). Partially blocking: the prohibitions on inline and eval script are normative and are delivered by REQ-PLATFORM-040; the full directive list is not.
- **PQ-25 — UI presentation decisions.** `DESIGN.md` OQ-5 (citation and verification surfacing), OQ-6 (disclaimer presentation and acknowledgement form), OQ-7 (role-distinct treatment and how a subscriber sees consultant access). Partially blocking: REQ-PRIVACY-040 and REQ-CATALOG-010/020 deliver mechanism and enforcement; the presentation form is not settled.
- **PQ-26 — Threat model completion.** `SECURITY.md` SQ-9, SQ-10. The 2026-07-31 model is single-perspective and pre-implementation; ownership, cadence, and cross-functional validation are undecided, and ASVS Level 3 is an unverified target with no gap assessment. Blocks: any conformance claim, and every `OWASP ASVS 5.0.0` mapping field in every issue.

### Conflicts found between documents

Only one, already recorded by the specification itself: the jurisdiction inconsistency (PQ-21 / `SECURITY.md` SQ-1). No other conflict was found between `REQUIREMENTS.md`, `ARCHITECTURE.md`, `SECURITY.md`, and `DESIGN.md`. Two apparent tensions were resolved by reading each document as authoritative in its own domain rather than by choosing between them:

1. `DESIGN.md` requires error messages that "name the specific problem", while SEC-AUTHN-3 and SEC-ERR-1 require generic, non-disclosing failures. Resolved in REQ-API-040 and REQ-AUTH-040 as: field-format validation is specific, anything dependent on account state or internal state is generic. This is a reading of both documents, not a new decision.
2. `REQUIREMENTS.md` OQ-11 leaves accessibility conformance open while `DESIGN.md` targets WCAG 2.2 AA. `ARCHITECTURE.md` explicitly records that `DESIGN.md` resolves OQ-11, so WCAG 2.2 AA is treated as firm.

---

## 2. Coverage matrix — functional requirements

Status values: `COVERED`, `PARTIALLY COVERED`, `BLOCKED`, `OUT OF SCOPE`.

| Requirement | Issue IDs | Boundary | Security rules | Design sections | Status |
|---|---|---|---|---|---|
| FR-1.1 | REQ-PLATFORM-020, REQ-PLATFORM-040 | Browser Client; boundary 1 | SEC-HTTP-1, SEC-HTTP-2 | Layout and Spacing | COVERED |
| FR-1.2 | REQ-PLATFORM-020 | Browser Client | — | Layout and Spacing; Accessibility | COVERED |
| FR-2.1 | REQ-AUTHZ-010, REQ-SESSION-010 | Boundary 2 | SEC-AUTHN-1, SEC-AUTHZ-1 | — | COVERED |
| FR-2.2 | — | Identity | SEC-AUTHN-5, SEC-AUTHN-8 | Components → Inputs | BLOCKED — PQ-5, PQ-7, PQ-8 |
| FR-2.3 | REQ-AUTH-040 | Identity; boundary 2 | SEC-AUTHN-3 | Components → Form feedback | PARTIALLY COVERED — response contract covered; password verification blocked by PQ-8 |
| FR-2.4 | — | Identity | SEC-SESSION-3 | — | BLOCKED — PQ-3 |
| FR-2.5 | — | Identity | SEC-AUTHN-4, SEC-AUTHN-7 | — | BLOCKED — PQ-6 |
| FR-2.6 | — | Identity | SEC-AUTHN-4 | — | BLOCKED — PQ-6 |
| FR-2.7 | REQ-AUTH-010 | Identity; REST API | SEC-AUTHZ-1, SEC-INPUT-3 | — | PARTIALLY COVERED — invariant and resolution covered; assignment lifecycle blocked by PQ-22 |
| FR-2.8 | REQ-AUTH-020 | Identity; boundary 2 | SEC-AUTHN-2 | Components | COVERED |
| FR-2.9 | REQ-AUTH-030 | Identity | SEC-AUTHN-2, SEC-AUTHN-7 | Components → Buttons | PARTIALLY COVERED — registration and replacement covered; first enrolment blocked by PQ-22 |
| FR-3.1 | — | REST API | SEC-AUTHZ-8 | — | BLOCKED — PQ-4 |
| FR-3.2 | — | REST API | SEC-AUTHZ-8 | — | BLOCKED — PQ-4 |
| FR-3.3 | — | REST API | SEC-AUTHZ-8 | — | BLOCKED — PQ-4 |
| FR-3.4 | — | REST API; Persistence | SEC-AUTHZ-8 | — | BLOCKED — PQ-4 |
| FR-4.1 | REQ-PLAN-010, REQ-PLAN-020 | REST API; Persistence | SEC-INPUT-1, SEC-INPUT-5 | Typography; Layout | COVERED |
| FR-4.2 | REQ-AUTHZ-030, REQ-CUSTOM-010 | REST API | SEC-AUTHZ-4 | — | COVERED |
| FR-4.3 | REQ-PLAN-030, REQ-PLAN-050, REQ-PLAN-060 | REST API | SEC-AUTHZ-4, SEC-INPUT-4 | Components | COVERED |
| FR-4.4 | REQ-PLAN-040, REQ-PLAN-050 | REST API; boundary 1 | SEC-INPUT-4, SEC-EXT-2, SEC-RENDER-3 | Components → Links | COVERED |
| FR-4.5 | REQ-PLAN-050, REQ-AUDIT-030 | REST API | SEC-INPUT-4, SEC-LOG-6 | — | PARTIALLY COVERED — record and gate covered; verification workflow blocked by PQ-13 |
| FR-4.6 | REQ-CATALOG-010, REQ-CATALOG-020, REQ-CATALOG-030 | Browser Client | SEC-RENDER-1, SEC-RENDER-3 | Components → Links; Typography | PARTIALLY COVERED — display and safe rendering covered; surfacing pattern open (PQ-25) |
| FR-4.7 | REQ-PLAN-050, REQ-PLAN-060, REQ-CATALOG-010, REQ-CATALOG-020 | REST API | SEC-AUTHZ-1, SEC-DATA-5 | — | COVERED |
| FR-5.1 | REQ-PLAN-010, REQ-CATALOG-010 | REST API; Browser Client | SEC-DATA-5, SEC-RENDER-1 | Typography; Layout | COVERED |
| FR-5.2 | — | REST API | — | — | BLOCKED — PQ-12 |
| FR-6.1 | REQ-CATALOG-020 | REST API; Browser Client | SEC-DATA-5, SEC-RENDER-1 | Typography; Layout | COVERED |
| FR-6.2 | REQ-PLAN-020, REQ-CATALOG-020 | Persistence; Browser Client | SEC-INPUT-1 | Typography | COVERED |
| FR-6.3 | — | REST API | — | — | BLOCKED — PQ-12 |
| FR-7.1 | REQ-CUSTOM-010 | REST API | SEC-INPUT-4, SEC-INPUT-1 | Components → Inputs | COVERED |
| FR-7.2 | REQ-CUSTOM-010 | REST API; Persistence | SEC-INPUT-3, SEC-INPUT-4 | — | COVERED |
| FR-7.3 | REQ-CUSTOM-020 | REST API; Persistence | SEC-AUTHZ-2 | Components → Empty states | COVERED |
| FR-7.4 | REQ-AUTHZ-020, REQ-CUSTOM-020 | REST API | SEC-AUTHZ-2 | — | COVERED |
| FR-7.5 | REQ-CUSTOM-030, REQ-PLAN-030, REQ-PLAN-060 | Persistence | SEC-INPUT-4 | — | COVERED |
| FR-8.1 | REQ-PROGRESS-010 | REST API; Persistence | SEC-DATA-2, SEC-AUTHZ-2, SEC-LOG-1 | Components → Inputs | COVERED |
| FR-8.2 | — | REST API | — | — | BLOCKED — PQ-9 |
| FR-8.3 | REQ-PROGRESS-020 | REST API; Persistence | SEC-DATA-2, SEC-AUTHZ-2, SEC-LOG-1 | Components → Inputs; Typography | COVERED |
| FR-8.4 | — | REST API | — | — | BLOCKED — PQ-10 |
| FR-8.5 | — | REST API; Browser Client | — | — | BLOCKED — PQ-10 |
| FR-8.6 | — | REST API; Browser Client | SEC-DATA-5 | — | BLOCKED — PQ-11 |
| FR-8.7 | REQ-PROGRESS-010, REQ-PROGRESS-020 | REST API | SEC-AUTHZ-2, SEC-LOG-1 | Components → Buttons | PARTIALLY COVERED — covered for weight and workouts; measurement and food entries blocked by PQ-9, PQ-10 |
| FR-8.8 | REQ-PROGRESS-010, REQ-PROGRESS-020 | REST API | SEC-INPUT-1 | — | PARTIALLY COVERED — same limitation as FR-8.7 |
| FR-8.9 | REQ-PROGRESS-030 | Boundary 1; REST API | SEC-INPUT-2, SEC-ERR-1 | Components → Form feedback | COVERED |
| FR-9.1 | REQ-AUTHZ-020, REQ-PRIVACY-060 | REST API | SEC-AUTHZ-2, SEC-DATA-5 | — | COVERED |
| FR-9.2 | REQ-PRIVACY-010 | REST API | SEC-DATA-2 | Components → Inputs | COVERED |
| FR-9.3 | — | REST API; artifact storage | SEC-DATA-3, SEC-DATA-6 | — | BLOCKED — PQ-16 |
| FR-9.4 | — | REST API; Persistence | SEC-DATA-4 | — | BLOCKED — PQ-16 |
| FR-9.5 | REQ-PRIVACY-030 | REST API; Browser Client | SEC-AUTHZ-2, SEC-INPUT-3 | Components → Inputs, Form feedback | PARTIALLY COVERED — email change deferred to PQ-7 |
| FR-9.6 | REQ-PRIVACY-040 | REST API; Browser Client | SEC-TB-1, SEC-INPUT-4 | Layout (72ch); Components | PARTIALLY COVERED — mechanism and enforcement covered; presentation form open (PQ-25), full trigger set open (PQ-12) |
| FR-9.7 | REQ-AUDIT-010, REQ-AUDIT-020 | REST API; Persistence | SEC-LOG-1, SEC-LOG-2, SEC-LOG-3 | — | COVERED |
| FR-9.8 | REQ-PRIVACY-050, REQ-AUDIT-040 | System egress; boundary 4 | SEC-TB-3, SEC-EXT-1, SEC-EXT-2 | — | PARTIALLY COVERED — application-level covered; network egress restriction blocked by PQ-19 |
| FR-9.9 | REQ-PRIVACY-020 | REST API | SEC-DATA-2 | Components → Buttons | COVERED |
| FR-10.1 | REQ-AUTHZ-030 | REST API | SEC-AUTHZ-4 | — | COVERED |
| FR-10.2 | REQ-AUDIT-030 | REST API; Persistence | SEC-LOG-6, SEC-LOG-2 | — | COVERED |
| FR-11.1 | — | REST API | — | — | BLOCKED — PQ-15, PQ-4 |
| FR-11.2 | REQ-CONSULT-010 | REST API | SEC-AUTHZ-3 | — | PARTIALLY COVERED — access bounded; capabilities open (PQ-14) |
| FR-11.3 | REQ-CONSULT-020 | REST API; Identity | SEC-AUTHZ-3, SEC-SESSION-4 | Components → Buttons | COVERED |
| FR-11.4 | REQ-CONSULT-010, REQ-AUDIT-020 | REST API | SEC-LOG-1 | — | COVERED |

**Totals** — 56 functional requirements: 33 `COVERED`, 9 `PARTIALLY COVERED`, 14 `BLOCKED`, 0 `OUT OF SCOPE`, 0 untracked. Every requirement traces to an issue or to a named blocking question.

## 3. Coverage matrix — security rules

| Rule(s) | Issue IDs | Status |
|---|---|---|
| SEC-TB-1 | REQ-API-010, REQ-API-020, REQ-AUTHZ-010, REQ-PRIVACY-040 | COVERED |
| SEC-TB-2 | — | BLOCKED — PQ-19 (network placement) |
| SEC-TB-3 | REQ-PRIVACY-050, REQ-AUDIT-040 | PARTIALLY COVERED — PQ-19 |
| SEC-AUTHN-1 | REQ-AUTHZ-010 | COVERED |
| SEC-AUTHN-2 | REQ-AUTH-020, REQ-AUTH-030 | COVERED |
| SEC-AUTHN-3 | REQ-AUTH-040 | COVERED |
| SEC-AUTHN-4 | — | BLOCKED — PQ-6 |
| SEC-AUTHN-5 | — | BLOCKED — PQ-8 |
| SEC-AUTHN-6 | — | BLOCKED — PQ-17 |
| SEC-AUTHN-7 | REQ-AUTH-030, REQ-AUTH-050 | PARTIALLY COVERED — password and MFA changes blocked by PQ-5, PQ-6 |
| SEC-AUTHN-8 | — | BLOCKED — PQ-7 |
| SEC-SESSION-1, -2 | REQ-SESSION-010 | COVERED |
| SEC-SESSION-3, -4, -5, -7 | REQ-CONSULT-020 (SEC-SESSION-4 for engagements only) | BLOCKED — PQ-3, PQ-19 |
| SEC-SESSION-6 | REQ-SESSION-020 | COVERED |
| SEC-AUTHZ-1, -2 | REQ-AUTHZ-010, REQ-AUTHZ-020 | COVERED |
| SEC-AUTHZ-3 | REQ-CONSULT-010, REQ-CONSULT-020 | COVERED |
| SEC-AUTHZ-4 | REQ-AUTHZ-030 | COVERED |
| SEC-AUTHZ-5, -6, -7 | REQ-AUTHZ-010 (fail-closed only) | BLOCKED — PQ-18 |
| SEC-AUTHZ-8 | — | BLOCKED — PQ-4 |
| SEC-HTTP-1, -2 | REQ-PLATFORM-040 | PARTIALLY COVERED — CSP directives open (PQ-24) |
| SEC-HTTP-3 | — | BLOCKED — PQ-23 |
| SEC-HTTP-4 | — | BLOCKED — PQ-3 (conditional on transport) |
| SEC-HTTP-5 | — | BLOCKED — PQ-17 |
| SEC-HTTP-6 | REQ-API-040 | COVERED |
| SEC-INPUT-1, -3, -6 | REQ-API-010, REQ-API-020 | COVERED |
| SEC-INPUT-2 | REQ-PROGRESS-030 | COVERED |
| SEC-INPUT-4 | REQ-PLAN-050, REQ-CUSTOM-010, REQ-CUSTOM-030 | COVERED |
| SEC-INPUT-5 | REQ-API-030 | COVERED |
| SEC-RENDER-1, -3, -4 | REQ-CATALOG-030 | COVERED |
| SEC-RENDER-2 | REQ-CATALOG-030 (conditional) | TO BE DECIDED — rich-text requirement open |
| SEC-DATA-1 | — | BLOCKED — PQ-19 |
| SEC-DATA-2 | REQ-PRIVACY-010, REQ-PRIVACY-020 | COVERED |
| SEC-DATA-3, -4, -6 | — | BLOCKED — PQ-16 |
| SEC-DATA-5 | REQ-PRIVACY-060 | COVERED |
| SEC-OPS-1 | — | BLOCKED — PQ-20 |
| SEC-SECRET-1, -2, -3, -4 | REQ-AUDIT-040 (partial, for logs) | BLOCKED — PQ-19 |
| SEC-LOG-1, -2 | REQ-AUDIT-010, REQ-AUDIT-020 | COVERED |
| SEC-LOG-3 | REQ-AUDIT-040 | COVERED |
| SEC-LOG-4 | REQ-AUTH-050, REQ-AUTHZ-040 | COVERED |
| SEC-LOG-5, -7 | — | BLOCKED — PQ-20 |
| SEC-LOG-6 | REQ-AUDIT-030 | COVERED |
| SEC-ERR-1 | REQ-API-040 | COVERED |
| SEC-EXT-1, -2 | REQ-PRIVACY-050, REQ-PLAN-040 | COVERED |
| SEC-CICD-1, -2, -3, -4 | — | BLOCKED — PQ-19 |
| SEC-CICD-5 | REQ-PIPE-020 | PARTIALLY COVERED — infrastructure enforcement blocked by PQ-19 |
| DEP-1 … DEP-8 | REQ-PIPE-010 | PARTIALLY COVERED — automated gate blocked by PQ-19 |

## 4. Blocked scope — no issue drafted

| Scope | Requirements | Blocked by |
|---|---|---|
| Registration with email and password | FR-2.2 | PQ-5, PQ-7, PQ-8 |
| Password credential storage and verification | FR-2.3 (verification half) | PQ-8 |
| Email address verification | SEC-AUTHN-8 | PQ-7 |
| Account and MFA recovery; lockout | FR-2.3, OQ-8 | PQ-5 |
| MFA enable, disable, and challenge | FR-2.5, FR-2.6 | PQ-6 |
| Logout and session revocation | FR-2.4 | PQ-3 |
| Token transport, storage, CSRF, key management | SEC-SESSION-5, -7; SEC-HTTP-4 | PQ-3, PQ-19 |
| Anti-automation and rate limiting | SEC-AUTHN-6, SEC-HTTP-5 | PQ-17 |
| Admin and consultant provisioning; role assignment; first passkey | FR-2.7 (lifecycle), FR-2.9 (bootstrap) | PQ-22 |
| Subscription activation, status, entitlement gate, retention across lapse | FR-3.1 – FR-3.4 | PQ-4 |
| Plan selection to follow | FR-5.2, FR-6.3 | PQ-12 |
| Plan verification operation | FR-4.5 (workflow) | PQ-13 |
| Body measurement logging | FR-8.2 | PQ-9 |
| Food logging and target comparison | FR-8.4, FR-8.5 | PQ-10 |
| Progress history retrieval and visualization | FR-8.6 | PQ-11 |
| Data export | FR-9.3 | PQ-16 |
| Account deletion | FR-9.4 | PQ-16 |
| Encryption at rest and backup handling | SEC-DATA-1 | PQ-19 |
| Consultant engagement creation and paid option | FR-11.1 | PQ-15, PQ-4 |
| Consultant capabilities beyond read scope | OQ-12 | PQ-14 |
| CI security gates, Terraform baseline, secret management, network tiering | SEC-CICD-1, -3, -4; SEC-SECRET-2, -3; SEC-TB-2 | PQ-19 |
| Audit retention, tamper-evidence, operational break-glass | SEC-LOG-5, -7; SEC-OPS-1 | PQ-20 |

## 5. Hierarchy

```
REQ-EPIC-001  Implement the specified subscription fitness web application
├─ Platform and delivery
│  ├─ REQ-PLATFORM-010  Design tokens
│  ├─ REQ-PLATFORM-020  Responsive layout and reflow
│  ├─ REQ-PLATFORM-030  Keyboard, focus, reduced motion
│  └─ REQ-PLATFORM-040  TLS and security response headers
├─ API boundary
│  ├─ REQ-API-010  Schema validation
│  ├─ REQ-API-020  Server-controlled field binding
│  ├─ REQ-API-030  Parameterized database access
│  └─ REQ-API-040  Error hygiene and diagnostic exclusion
├─ Authorization
│  ├─ REQ-AUTHZ-010  Deny-by-default authentication gate
│  ├─ REQ-AUTHZ-020  Object-level ownership scoping
│  ├─ REQ-AUTHZ-030  Admin-only plan lifecycle
│  └─ REQ-AUTHZ-040  Denial response and logging
├─ Session
│  ├─ REQ-SESSION-010  JWT verification
│  └─ REQ-SESSION-020  Token claim allow-list
├─ Identity
│  ├─ REQ-AUTH-010  One role per account
│  ├─ REQ-AUTH-020  Passkey authentication for privileged roles
│  ├─ REQ-AUTH-030  Passkey registration and replacement
│  ├─ REQ-AUTH-040  Non-disclosing authentication failures
│  └─ REQ-AUTH-050  Security event logging
├─ Audit
│  ├─ REQ-AUDIT-010  Audit entry model, append-only
│  ├─ REQ-AUDIT-020  Mandatory audit on health-data paths
│  ├─ REQ-AUDIT-030  Admin plan lifecycle audit
│  └─ REQ-AUDIT-040  Log redaction
├─ Privacy and data rights
│  ├─ REQ-PRIVACY-010  Consent capture
│  ├─ REQ-PRIVACY-020  Consent withdrawal
│  ├─ REQ-PRIVACY-030  View and correct personal data
│  ├─ REQ-PRIVACY-040  Medical disclaimer acknowledgement
│  ├─ REQ-PRIVACY-050  No external transmission
│  └─ REQ-PRIVACY-060  Response field minimization
├─ Plan library (admin)
│  ├─ REQ-PLAN-010  Exercise plan model
│  ├─ REQ-PLAN-020  Diet plan model
│  ├─ REQ-PLAN-030  Create and edit
│  ├─ REQ-PLAN-040  Citation management
│  ├─ REQ-PLAN-050  Publication gate
│  └─ REQ-PLAN-060  Unpublication
├─ Catalog (subscriber)
│  ├─ REQ-CATALOG-010  Browse and view exercise plans
│  ├─ REQ-CATALOG-020  Browse and view diet plans
│  └─ REQ-CATALOG-030  Safe rendering
├─ Customization
│  ├─ REQ-CUSTOM-010  Copy-on-customize
│  ├─ REQ-CUSTOM-020  Persist, list, retrieve
│  └─ REQ-CUSTOM-030  Copy stability
├─ Progress
│  ├─ REQ-PROGRESS-010  Body weight logging
│  ├─ REQ-PROGRESS-020  Workout logging
│  └─ REQ-PROGRESS-030  Validation and error reporting
├─ Consultants
│  ├─ REQ-CONSULT-010  Engagement-scoped access
│  └─ REQ-CONSULT-020  Engagement termination revokes access
└─ Build and environments
   ├─ REQ-PIPE-010  Dependency policy
   └─ REQ-PIPE-020  No real health data in non-production
```

Workstream issues were deliberately omitted; the prompt makes them optional and epic-to-leaf keeps the manifest reviewable. The groupings above are organizational only.

## 6. Manifest

Effort is engineer-days; LOC is human-authored changed lines, excluding generated files and lockfiles. Model recommendations use verified identifiers from this session's environment (`claude-opus-5`, `claude-fable-5`). **No OpenAI model is recommended: no official OpenAI model list was retrieved in this session, so any specific GPT identifier would be `NOT VERIFIED`.**

| # | ID | File | Effort | LOC | Depends on | Model |
|---|---|---|---|---|---|---|
| 1 | REQ-PLATFORM-010 | `010-REQ-PLATFORM-010.md` | 0.5–1 | 150–350 | — | Opus |
| 2 | REQ-PLATFORM-020 | `020-REQ-PLATFORM-020.md` | 1–2 | 300–700 | 1 | Opus |
| 3 | REQ-PLATFORM-030 | `030-REQ-PLATFORM-030.md` | 1–2 | 300–650 | 1, 2 | Opus |
| 4 | REQ-PLATFORM-040 | `040-REQ-PLATFORM-040.md` | 0.5–1.5 | 150–400 | — | Opus |
| 5 | REQ-API-010 | `050-REQ-API-010.md` | 1–2 | 400–900 | — | Opus |
| 6 | REQ-API-020 | `060-REQ-API-020.md` | 1–1.5 | 250–600 | 5 | Opus |
| 7 | REQ-API-030 | `070-REQ-API-030.md` | 1–2 | 250–600 | 5 | Opus |
| 8 | REQ-API-040 | `080-REQ-API-040.md` | 0.5–1.5 | 200–450 | 5 | Opus |
| 9 | REQ-SESSION-010 | `130-REQ-SESSION-010.md` | 0.5–1.5 | 150–400 | — | Opus |
| 10 | REQ-SESSION-020 | `140-REQ-SESSION-020.md` | 0.5–1 | 100–250 | 9 | Opus |
| 11 | REQ-AUTHZ-010 | `090-REQ-AUTHZ-010.md` | 0.5–1.5 | 200–450 | 9 | Opus |
| 12 | REQ-AUTHZ-020 | `100-REQ-AUTHZ-020.md` | 1–2 | 300–700 | 11, 7 | Opus |
| 13 | REQ-AUTH-010 | `150-REQ-AUTH-010.md` | 0.5–1 | 100–300 | 6, 7 | Opus |
| 14 | REQ-AUTHZ-030 | `110-REQ-AUTHZ-030.md` | 0.5–1 | 150–350 | 11, 13, 6 | Opus |
| 15 | REQ-AUTHZ-040 | `120-REQ-AUTHZ-040.md` | 0.5–1 | 150–350 | 11, 8 | Opus |
| 16 | REQ-AUDIT-040 | `230-REQ-AUDIT-040.md` | 1–1.5 | 200–450 | — | Opus |
| 17 | REQ-AUDIT-010 | `200-REQ-AUDIT-010.md` | 1–1.5 | 250–500 | 7 | Opus |
| 18 | REQ-AUDIT-020 | `210-REQ-AUDIT-020.md` | 1–2 | 300–650 | 17, 12, 7 | Opus |
| 19 | REQ-AUDIT-030 | `220-REQ-AUDIT-030.md` | 0.5–1 | 150–350 | 17, 14, 13 | Opus |
| 20 | REQ-AUTH-020 | `160-REQ-AUTH-020.md` | 1.5–2 | 400–900 | 13, 9, 10 | Opus |
| 21 | REQ-AUTH-030 | `170-REQ-AUTH-030.md` | 1–2 | 300–650 | 13, 20, 11, 17 | Opus |
| 22 | REQ-AUTH-040 | `180-REQ-AUTH-040.md` | 0.5–1.5 | 150–350 | 8, 20 | Opus |
| 23 | REQ-AUTH-050 | `190-REQ-AUTH-050.md` | 0.5–1.5 | 200–450 | 16, 8 | Opus |
| 24 | REQ-PRIVACY-060 | `290-REQ-PRIVACY-060.md` | 1–1.5 | 250–550 | 12, 5, 4 | Opus |
| 25 | REQ-PRIVACY-050 | `280-REQ-PRIVACY-050.md` | 0.5–1.5 | 150–400 | 16, 4 | Opus |
| 26 | REQ-PRIVACY-010 | `240-REQ-PRIVACY-010.md` | 1–1.5 | 250–500 | 11, 12, 18, 5 | Opus |
| 27 | REQ-PRIVACY-020 | `250-REQ-PRIVACY-020.md` | 0.5–1.5 | 200–400 | 26, 12, 18 | Opus |
| 28 | REQ-PRIVACY-030 | `260-REQ-PRIVACY-030.md` | 1–1.5 | 250–550 | 11, 12, 5, 6, 18, 3 | Opus |
| 29 | REQ-PLAN-010 | `300-REQ-PLAN-010.md` | 1–1.5 | 250–500 | 5, 6, 7 | Fable |
| 30 | REQ-PLAN-020 | `310-REQ-PLAN-020.md` | 1–1.5 | 250–500 | 5, 6, 7 | Fable |
| 31 | REQ-PLAN-030 | `320-REQ-PLAN-030.md` | 1.5–2 | 400–900 | 29, 30, 14, 19, 5, 6, 3 | Opus |
| 32 | REQ-PLAN-040 | `330-REQ-PLAN-040.md` | 1–1.5 | 250–500 | 29, 30, 31, 14, 5, 25, 19 | Opus |
| 33 | REQ-PLAN-050 | `340-REQ-PLAN-050.md` | 1–1.5 | 250–500 | 29–32, 14, 19 | Opus |
| 34 | REQ-CATALOG-030 | `380-REQ-CATALOG-030.md` | 1–1.5 | 200–450 | 1, 3, 4 | Opus |
| 35 | REQ-CATALOG-010 | `360-REQ-CATALOG-010.md` | 1–2 | 350–750 | 29, 33, 11, 24, 2, 3, 34 | Fable |
| 36 | REQ-CATALOG-020 | `370-REQ-CATALOG-020.md` | 1–2 | 350–700 | 30, 33, 11, 24, 2, 3, 34 | Fable |
| 37 | REQ-CUSTOM-030 | `410-REQ-CUSTOM-030.md` | 0.5–1 | 100–300 | 38, 39, 31 | Opus |
| 38 | REQ-CUSTOM-010 | `390-REQ-CUSTOM-010.md` | 1.5–2 | 400–850 | 29, 30, 33, 12, 5, 6, 18, 41, 3 | Opus |
| 39 | REQ-CUSTOM-020 | `400-REQ-CUSTOM-020.md` | 1–1.5 | 250–550 | 38, 12, 18, 24, 34, 3 | Opus |
| 40 | REQ-PLAN-060 | `350-REQ-PLAN-060.md` | 0.5–1 | 150–350 | 33, 14, 19, 37 | Opus |
| 41 | REQ-PRIVACY-040 | `270-REQ-PRIVACY-040.md` | 1–1.5 | 250–500 | 11, 12, 18, 3 | Opus |
| 42 | REQ-PROGRESS-030 | `440-REQ-PROGRESS-030.md` | 0.5–1.5 | 200–450 | 5, 8, 1, 3 | Opus |
| 43 | REQ-PROGRESS-010 | `420-REQ-PROGRESS-010.md` | 1–1.5 | 250–550 | 11, 12, 5, 6, 18, 26, 27, 42, 3 | Opus |
| 44 | REQ-PROGRESS-020 | `430-REQ-PROGRESS-020.md` | 1.5–2 | 400–900 | 43, 42, 12, 18, 26, 27, 41, 38, 35, 2, 3 | Opus |
| 45 | REQ-CONSULT-010 | `450-REQ-CONSULT-010.md` | 1–2 | 300–650 | 13, 20, 11, 12, 18, 24 | Opus |
| 46 | REQ-CONSULT-020 | `460-REQ-CONSULT-020.md` | 0.5–1.5 | 150–400 | 45, 12, 15, 18, 10, 3 | Opus |
| 47 | REQ-PIPE-010 | `470-REQ-PIPE-010.md` | 0.5–1 | 50–200 | — | Opus |
| 48 | REQ-PIPE-020 | `480-REQ-PIPE-020.md` | 0.5–1.5 | 150–400 | 29, 30, 43, 44 | Opus |

**Totals**: 41–68 engineer-days; roughly 11,000–24,000 human-authored lines. Every leaf is within the 0.5–2 day and 1,500-line bounds.

### Model assignment rationale

- **`claude-opus-5` (44 issues)** — the default here because the specification is security-dense: most issues are bounded implementations of an authorization, validation, audit, or privacy control where a plausible-looking implementation that passes the functional test still fails the security requirement (fail-closed behavior, response uniformity, structural audit dependency, copy-versus-reference semantics). These reward careful bounded reasoning rather than autonomy.
- **`claude-fable-5` (4 issues: REQ-PLAN-010, REQ-PLAN-020, REQ-CATALOG-010, REQ-CATALOG-020)** — foundational data models and their paired read surfaces, which span persistence, API, and client, and whose main risk is divergence between the exercise and diet halves and between what is modelled and what several downstream issues assume. Cross-boundary consistency over a longer horizon is the distinguishing need.
- **OpenAI GPT — `NOT VERIFIED`.** No OpenAI model identifier is recommended, because no official OpenAI model list was consulted in this session and `REQUIREMENT_TEMPLATE.md`'s prohibition on unverified identifiers applies equally here. If OpenAI tooling is used, the exact model identifier must be verified against OpenAI's documentation at execution time and recorded in the issue.

## 7. Topological creation order

The manifest's `#` column is the creation order; it is a valid topological sort of the dependency graph (verified: every `Depends on` entry precedes its dependent, and the graph is acyclic). Create the epic first, then leaves 1 through 48 in order, each with `--parent` set to the epic's number.

## 8. Proposed commands — NOT EXECUTED

Run from the repository root. These are recorded for review; `DRAFT_ONLY` mode means none was run.

```sh
# Epic
gh issue create --title "[REQ-EPIC-001] Implement the specified subscription fitness web application" --body-file ".planning/github-issues/000-REQ-EPIC-001.md"

# Leaves, in topological order (add --parent <epic-number> to each)
gh issue create --title "[REQ-PLATFORM-010] Design tokens for color, typography, and spacing" --body-file ".planning/github-issues/010-REQ-PLATFORM-010.md"
gh issue create --title "[REQ-PLATFORM-020] Responsive layout and reflow without loss of function" --body-file ".planning/github-issues/020-REQ-PLATFORM-020.md"
gh issue create --title "[REQ-PLATFORM-030] Keyboard operability, focus management, and reduced motion baseline" --body-file ".planning/github-issues/030-REQ-PLATFORM-030.md"
gh issue create --title "[REQ-PLATFORM-040] TLS enforcement and security response headers" --body-file ".planning/github-issues/040-REQ-PLATFORM-040.md"
gh issue create --title "[REQ-API-010] Allow-list schema validation of all untrusted input" --body-file ".planning/github-issues/050-REQ-API-010.md"
gh issue create --title "[REQ-API-020] Server-controlled field binding (mass-assignment protection)" --body-file ".planning/github-issues/060-REQ-API-020.md"
gh issue create --title "[REQ-API-030] Parameterized database access" --body-file ".planning/github-issues/070-REQ-API-030.md"
gh issue create --title "[REQ-API-040] Error response hygiene and diagnostic endpoint exclusion" --body-file ".planning/github-issues/080-REQ-API-040.md"
gh issue create --title "[REQ-SESSION-010] JWT signature, algorithm, and claim verification" --body-file ".planning/github-issues/130-REQ-SESSION-010.md"
gh issue create --title "[REQ-SESSION-020] Token claim allow-list excluding sensitive data" --body-file ".planning/github-issues/140-REQ-SESSION-020.md"
gh issue create --title "[REQ-AUTHZ-010] Deny-by-default authentication requirement on protected operations" --body-file ".planning/github-issues/090-REQ-AUTHZ-010.md"
gh issue create --title "[REQ-AUTHZ-020] Object-level ownership scoping" --body-file ".planning/github-issues/100-REQ-AUTHZ-020.md"
gh issue create --title "[REQ-AUTH-010] Exactly one role per account" --body-file ".planning/github-issues/150-REQ-AUTH-010.md"
gh issue create --title "[REQ-AUTHZ-030] Admin-only restriction on plan lifecycle operations" --body-file ".planning/github-issues/110-REQ-AUTHZ-030.md"
gh issue create --title "[REQ-AUTHZ-040] Authorization denial response and logging semantics" --body-file ".planning/github-issues/120-REQ-AUTHZ-040.md"
gh issue create --title "[REQ-AUDIT-040] Log redaction of health values, credentials, and tokens" --body-file ".planning/github-issues/230-REQ-AUDIT-040.md"
gh issue create --title "[REQ-AUDIT-010] Audit entry model and append-only enforcement" --body-file ".planning/github-issues/200-REQ-AUDIT-010.md"
gh issue create --title "[REQ-AUDIT-020] Mandatory audit write on every health-data access path" --body-file ".planning/github-issues/210-REQ-AUDIT-020.md"
gh issue create --title "[REQ-AUDIT-030] Admin plan lifecycle audit entries" --body-file ".planning/github-issues/220-REQ-AUDIT-030.md"
gh issue create --title "[REQ-AUTH-020] Passkey authentication for admin and consultant accounts" --body-file ".planning/github-issues/160-REQ-AUTH-020.md"
gh issue create --title "[REQ-AUTH-030] Passkey registration and replacement with re-authentication" --body-file ".planning/github-issues/170-REQ-AUTH-030.md"
gh issue create --title "[REQ-AUTH-040] Non-disclosing authentication failure responses" --body-file ".planning/github-issues/180-REQ-AUTH-040.md"
gh issue create --title "[REQ-AUTH-050] Authentication and account-change security event logging" --body-file ".planning/github-issues/190-REQ-AUTH-050.md"
gh issue create --title "[REQ-PRIVACY-060] Response field minimization" --body-file ".planning/github-issues/290-REQ-PRIVACY-060.md"
gh issue create --title "[REQ-PRIVACY-050] No external transmission of health data" --body-file ".planning/github-issues/280-REQ-PRIVACY-050.md"
gh issue create --title "[REQ-PRIVACY-010] Consent capture before any health-data write" --body-file ".planning/github-issues/240-REQ-PRIVACY-010.md"
gh issue create --title "[REQ-PRIVACY-020] Consent withdrawal blocks new health-data writes" --body-file ".planning/github-issues/250-REQ-PRIVACY-020.md"
gh issue create --title "[REQ-PRIVACY-030] View and correct personal data" --body-file ".planning/github-issues/260-REQ-PRIVACY-030.md"
gh issue create --title "[REQ-PLAN-010] Exercise plan content model" --body-file ".planning/github-issues/300-REQ-PLAN-010.md"
gh issue create --title "[REQ-PLAN-020] Diet plan content model with calorie and macronutrient targets" --body-file ".planning/github-issues/310-REQ-PLAN-020.md"
gh issue create --title "[REQ-PLAN-030] Admin plan creation and editing" --body-file ".planning/github-issues/320-REQ-PLAN-030.md"
gh issue create --title "[REQ-PLAN-040] Plan citation management with URL scheme validation" --body-file ".planning/github-issues/330-REQ-PLAN-040.md"
gh issue create --title "[REQ-PLAN-050] Publication gate requiring citation and verification record" --body-file ".planning/github-issues/340-REQ-PLAN-050.md"
gh issue create --title "[REQ-CATALOG-030] Safe rendering of plan content and citation links" --body-file ".planning/github-issues/380-REQ-CATALOG-030.md"
gh issue create --title "[REQ-CATALOG-010] Browse and view published exercise plans" --body-file ".planning/github-issues/360-REQ-CATALOG-010.md"
gh issue create --title "[REQ-CATALOG-020] Browse and view published diet plans" --body-file ".planning/github-issues/370-REQ-CATALOG-020.md"
gh issue create --title "[REQ-CUSTOM-010] Customize a published plan into a private copy" --body-file ".planning/github-issues/390-REQ-CUSTOM-010.md"
gh issue create --title "[REQ-CUSTOM-020] Persist, list, and retrieve a subscriber's customized plans" --body-file ".planning/github-issues/400-REQ-CUSTOM-020.md"
gh issue create --title "[REQ-CUSTOM-030] Customized copy stability when the source plan changes" --body-file ".planning/github-issues/410-REQ-CUSTOM-030.md"
gh issue create --title "[REQ-PLAN-060] Plan unpublication" --body-file ".planning/github-issues/350-REQ-PLAN-060.md"
gh issue create --title "[REQ-PRIVACY-040] Medical disclaimer acknowledgement before first plan use" --body-file ".planning/github-issues/270-REQ-PRIVACY-040.md"
gh issue create --title "[REQ-PROGRESS-030] Log entry validation and field-level error reporting" --body-file ".planning/github-issues/440-REQ-PROGRESS-030.md"
gh issue create --title "[REQ-PROGRESS-010] Body weight entry logging, editing, deletion, and backdating" --body-file ".planning/github-issues/420-REQ-PROGRESS-010.md"
gh issue create --title "[REQ-PROGRESS-020] Workout completion logging" --body-file ".planning/github-issues/430-REQ-PROGRESS-020.md"
gh issue create --title "[REQ-CONSULT-010] Engagement-scoped consultant access" --body-file ".planning/github-issues/450-REQ-CONSULT-010.md"
gh issue create --title "[REQ-CONSULT-020] Ending an engagement revokes consultant access" --body-file ".planning/github-issues/460-REQ-CONSULT-020.md"
gh issue create --title "[REQ-PIPE-010] Dependency policy and reproducible resolution" --body-file ".planning/github-issues/470-REQ-PIPE-010.md"
gh issue create --title "[REQ-PIPE-020] Non-production environments contain no real health data" --body-file ".planning/github-issues/480-REQ-PIPE-020.md"
```

After creation, replace every `{{ISSUE_URL:<ID>}}` placeholder in `000-REQ-EPIC-001.md` with the created issue URL and update the epic body.

## 9. Notes on drafting decisions

- **`OWASP ASVS 5.0.0` and `NIST SP 800-53 Rev. 5` mappings are `TO BE DECIDED` in every issue.** `REQUIREMENT_TEMPLATE.md` states that only mappings verified against the cited version may be included and that control identifiers must not be guessed. Neither catalog was retrieved in this session, and `SECURITY.md` SQ-10 leaves both the ASVS target and its verifier open. CWE identifiers and the `SECURITY.md` threat-model identifiers are cited where confident.
- **`Regulatory` fields are `TO BE DECIDED` wherever health data is involved**, because PQ-21 records the jurisdiction question as unresolved by the specification itself.
- **`Alerting` fields are `TO BE DECIDED` throughout.** No source document defines an alerting model, and PQ-17 blocks the thresholds any alert would need.
- **No issue resolves an open question.** Where an issue had to proceed alongside one — REQ-PROGRESS-020 logging against an accessible plan while PQ-12 is open, REQ-PLAN-030 leaving verification untouched on edit while PQ-13 is open — the position is labelled provisional in that issue's Open Decisions and named here.
- **No specification file was modified.**
