# ISSUE_PLAN.md — Specification decomposition

- **Scope**: ALL requirements in `REQUIREMENTS.md`
- **Execution mode**: originally `DRAFT_ONLY`; the epic was subsequently filed as GitHub issue #8 and the first 62 leaves as issues #9–#56, #60, and #66–#78, each linked as a sub-issue of the epic; the 38 leaves added 2026-08-03 are drafted and await filing after this branch merges
- **Sources**: `REQUIREMENTS.md`, `ARCHITECTURE.md`, `SECURITY.md`, `DESIGN.md`, `REQUIREMENT_TEMPLATE.md`; `CLAUDE.md` followed as agent instruction, not as product specification
- **Produced**: 2026-07-31
- **Updated**: 2026-08-03 — reflects the 2026-08-03 specification amendments: five new functional requirements (FR-2.18, FR-4.9, FR-5.4, FR-9.12, FR-10.3), bringing the total to 83, and three new security rules (SEC-AUTHZ-9, SEC-EXT-3, SEC-INPUT-7), plus the amended FRs and rules recorded in the documents themselves
- **Result**: 1 epic + 62 previously filed leaves (epic #8; leaves #9–#56, #60, #66–#78) + 38 new leaves drafted 2026-08-03 awaiting filing — 100 leaves total; scope is still ALL requirements; no blocked scope remains (section 4 is empty as of 2026-08-03)

---

## 1. Open Questions

Each `PQ-*` names the source identifiers it derives from and the scope it blocks. None is resolved here. Two classes are distinguished deliberately:

- **BLOCKING** — affects observable behavior, roles or permissions, auth, API contracts, data semantics, trust boundaries, security controls, failure behavior, or acceptance testing. Scope depending on one is decomposed no further and receives no issue body.
- **NON-BLOCKING FOR DRAFTING** — prevents implementation from starting but does not change what the behavior must be, so acceptance criteria can be written without it.

### Non-blocking for drafting

- **PQ-1 — Technology stack. RESOLVED.** Recorded in `CLAUDE.md`, Repository state: TypeScript with npm workspaces; Fastify; PostgreSQL with Drizzle ORM and drizzle-kit; Vitest; ESLint flat config with Prettier; GitHub Actions; and the `apps/api`, `apps/web`, `packages/shared`, `db/migrations`, `infra` layout. As predicted, no acceptance criterion changed — every issue's criteria were written at the HTTP, data, and interface level and stand unaltered. The client half: Vite, a single-page application with `vue-router`, plain CSS custom properties with scoped single-file-component styles, and Playwright with axe-core for end-to-end and WCAG 2.2 AA testing. Still open within the original scope of this question, unchanged as of 2026-08-03: the concrete runner commands and local run instructions (no scaffolding exists yet) and the code-coverage threshold.
- **PQ-2 — Brand and presentation identity. RESOLVED** (`DESIGN.md`, 2026-08-01; extended 2026-08-03). The product is **Proof of Work** with the Proof mark and live-text wordmark (OQ-1), a disciplined, evidence-led personality (OQ-2), light and dark themes with a `System` initial preference (OQ-3), a no-photography imagery policy with admin-approved neutral vector diagrams (OQ-9), and a system font stack in place of the formerly UNKNOWN brand typography. `DESIGN.md` closes with no open design questions; the 2026-08-03 amendment added the credentials, account security, and administration patterns consumed by the auth and admin drafts. REQ-PLATFORM-010 and the logo treatment are fully specified.

### Blocking

- **PQ-3 — Session model. RESOLVED.** A signed token carrying a session identifier and no authorization state, in an `HttpOnly`, `Secure`, `SameSite` cookie, resolved against a server-side session record on every request and revoked by invalidating that record. No refresh rotation is needed because revocation is immediate. SEC-HTTP-4 now applies unconditionally, and TM-S-5 and TM-S-6 are no longer CONDITIONAL. Since resolved: session lifetimes and the `SameSite` value are fixed (SQ-3 — subscriber 24 h absolute / 2 h idle, privileged 12 h / 30 min, `SameSite=Lax`), and signing keys live in AWS Secrets Manager with `kid`-based 90-day rotation (SQ-7, SEC-SESSION-7).
- **PQ-4 — Subscription activation and payments. RESOLVED** (`REQUIREMENTS.md` OQ-1, OQ-2, 2026-08-01). Subscriptions are admin-granted periods (FR-3.5), active exactly while the current time falls within a granted period (FR-3.6), audited, with no trial or free tier and payment out of band; the paid consultant option is recorded by admin action the same way (FR-11.5). Self-serve payments are deferred to OQ-18 behind the period-record seam. Drafted 2026-08-03: REQ-ENTITLE-010 … 040 (entitlement gate, status view, admin periods, retention across lapse) and REQ-CONSULT-030 (FR-11.1, FR-11.5); threat TM-E-4 is MITIGATED BY RULE.
- **PQ-5 — Password policy, lockout, and recovery. RESOLVED.** `REF-63B`-aligned policy — 8-character minimum with 15 or more encouraged, no composition rules, no forced rotation, known-breached passwords refused from a locally hosted list — and exponential backoff throttling instead of account lockout, which closes TM-D-2 by making third-party-triggered lockout impossible by construction. FR-2.2 registration is unblocked. Recovery: password reset by single-use token to the verified address, which never bypasses an enabled second factor and terminates all sessions (FR-2.12); single-use recovery codes at MFA enrolment, with no admin-assisted MFA reset (FR-2.13); permanent unrecoverability on total factor loss, disclosed before enrolment (FR-2.14); and privileged passkey recovery only by fresh invitation, with a two-passkey and two-admin minimum (FR-2.15). Threat TM-S-2 is closed. The cost once recorded as `REQUIREMENTS.md` OQ-17 has since resolved (2026-08-01): the deletion right survives lockout through the FR-9.11 out-of-band channel (drafted as REQ-PRIVACY-110), while export and correction without full authentication are refused under GDPR Art. 12(6); SQ-1 is resolved, with that position recorded for the pre-launch counsel review.
- **PQ-6 — MFA factors. RESOLVED.** TOTP (RFC 6238) or a passkey, user-enabled and optional per FR-2.5. SMS and email codes excluded — `REF-63B` treats SMS as a restricted authenticator, and an email code reduces the second factor to email-account security. FR-2.5 and FR-2.6 are unblocked; the lost-device path is handled by the recovery flows resolved in PQ-5 (single-use recovery codes, FR-2.13; SQ-3 has since fixed 10 recovery codes and the token lifetimes).
- **PQ-7 — Email verification flow. RESOLVED.** An account may be created and may authenticate before verification, but every health-data write is refused until control is proven, and the address cannot be relied on for recovery until then. Single-use short-lived token, invalidated on use or replacement, rate-limited resend, no enumeration in the response. Recorded as `REQUIREMENTS.md` FR-2.11 and `SECURITY.md` SEC-AUTHN-8; threat TM-S-3 is closed. SQ-3 has since fixed the parameters: 24-hour token lifetime, resend at most once per minute and five per hour per address; SEC-AUTHN-8 now also covers the FR-2.18 replacement address (2026-08-03).
- **PQ-8 — Password hashing. RESOLVED.** Argon2id with a per-credential salt from a cryptographically secure generator; bcrypt and non-memory-hard functions are prohibited outright. Parameters must be named constants with a documented tuning basis; SQ-7 has since fixed them — 64 MiB memory, 3 iterations, parallelism 1, on the 1 vCPU / 2 GB Fargate basis. Credential storage is unblocked. (The 2026-08-03 SEC-AUTHN-11 amendment splits recovery-secret hashing: Argon2id only for user-typed MFA recovery codes; machine-held high-entropy tokens use HMAC-SHA-256 under a Secrets Manager key with indexed lookup.)
- **PQ-9 — Body measurement fields and unit system. RESOLVED** (`REQUIREMENTS.md` OQ-4, 2026-08-01; `DESIGN.md` OQ-8 unit half). Fields: waist, chest, hips, upper arm, thigh, body-fat % (FR-8.2, since amended: all fields optional per entry, at least one required, single value per field). Per-account metric/imperial preference covering measurements, weight, and workout load, with every record storing value plus unit and display-only conversion (FR-8.10) — REQ-PROGRESS-010/020's explicit-unit storage is confirmed as the permanent design, now anchored to the account preference. Drafted 2026-08-03: REQ-PROGRESS-040 (FR-8.2) and REQ-PROGRESS-050 (FR-8.10). `DESIGN.md` OQ-8 has since resolved: English/LTR v1 with expansion-tolerant, logical-property-ready components; further localization and active RTL deferred beyond v1.
- **PQ-10 — Nutrition data source. RESOLVED** (`REQUIREMENTS.md` OQ-5, 2026-08-01). A bundled nutrition dataset imported at build time (initially USDA FoodData Central; FR-8.11) plus in-boundary AI estimation from a description or transient photo, confirmed by the subscriber before saving (FR-8.12, FR-8.13; SEC-AI-1–SEC-AI-3; ARCHITECTURE.md component 5 and trust boundary 6). Drafted 2026-08-03: REQ-FOOD-010 … 050 (FR-8.11, FR-8.4, FR-8.5, FR-8.12, FR-8.13). The model service is Amazon Bedrock in-account (PQ-19 RESOLVED), now pinned by model identifier and version, changed only via the SEC-CICD-2 reviewed path (SEC-AI-1, 2026-08-03); photo uploads are content-validated per SEC-INPUT-7 (added 2026-08-03).
- **PQ-11 — Progress history period, granularity, and visualization. RESOLVED** (`REQUIREMENTS.md` OQ-7, `DESIGN.md` OQ-4, 2026-08-01). Entry-level history for the account's lifetime; trend charts over 4-week/3-month/1-year/all-time ranges for weight, each measurement field, and per-exercise load, each paired with an accessible data table (FR-8.14). Drafted 2026-08-03 as REQ-PROGRESS-060, with per-exercise trends keyed to the FR-5.4 catalog entry and top-set weight per session as the charted value (FR-8.14 as amended).
- **PQ-12 — One or many active plans. RESOLVED** (`REQUIREMENTS.md` OQ-6, 2026-08-01). One active plan of each type (FR-5.3, FR-6.4): a selection names a published plan or the subscriber's own copy, replacement never alters logged history, and FR-8.5 reads the currently selected diet plan. Drafted 2026-08-03: REQ-SELECT-010 (FR-5.2, FR-5.3) and REQ-SELECT-020 (FR-6.3, FR-6.4), with the FR-4.9 unpublication-ends-selections semantics (added 2026-08-03) in REQ-SELECT-030. The FR-9.6 trigger set is complete (first use includes first selection). REQ-PROGRESS-020's provisional position stands, now anchored by the FR-5.4 catalog: logged exercises reference the catalog entry and snapshot the name, so backdated entries survive plan switches.
- **PQ-13 — Plan verification workflow. RESOLVED** (`REQUIREMENTS.md` OQ-10 and OQ-16, 2026-08-01). One-time single-admin verification before first publication; the author may verify; edits never re-trigger verification but may not leave a published plan citation-less (FR-4.8). Threat TM-T-5 is RISK ACCEPTED with named compensating controls. The verification *operation* is drafted 2026-08-03 as REQ-PLAN-070; the record and publication gate remain covered by REQ-PLAN-050.
- **PQ-14 — Consultant capabilities. RESOLVED** (`REQUIREMENTS.md` OQ-12, 2026-08-01). Views of the engaged subscriber's plans, copies, and logs, plus edits of that subscriber's plan copies — nothing else, all audited (FR-11.6). No log writes, no messaging. Threat TM-E-3 is MITIGATED BY RULE. REQ-CONSULT-010 keeps delivering *who*; FR-11.6's *what* is drafted 2026-08-03 as REQ-CONSULT-040.
- **PQ-15 — Consultant onboarding, vetting, and the paid option. RESOLVED** (`REQUIREMENTS.md` OQ-13, `SECURITY.md` SQ-12, 2026-08-01). Consultants are vetted out of band by the operator with the vetting record required on every invitation (FR-2.16); privileged accounts are deprovisioned by an audited admin action with the two-admin floor (FR-2.17, SEC-AUTHN-13); the paid option and engagement creation were already settled (PQ-4, FR-11.5). Drafted 2026-08-03: REQ-AUTH-160 (FR-2.16), REQ-AUTH-170 (FR-2.17), and REQ-CONSULT-030 (FR-11.1, FR-11.5).
- **PQ-16 — Export and deletion mechanics. RESOLVED** (`SECURITY.md` SQ-5, 2026-08-01). Synchronous JSON export with no stored artifact (SEC-DATA-6 moot); synchronous hard delete with tombstoned audit identifiers (FR-9.10), a 35-day backup horizon with restore re-deletion, and full completion within 35 days of execution (FR-9.4; clock corrected 2026-08-03). No background-processing boundary. Threats TM-I-6, TM-I-7, TM-P-1 all MITIGATED BY RULE. Drafted 2026-08-03: REQ-PRIVACY-080 (FR-9.3), REQ-PRIVACY-090 (FR-9.4), REQ-PRIVACY-100 (FR-9.10), and REQ-PRIVACY-110 (FR-9.11, resolved by OQ-17), with the keyed tombstone derivation and deletion ledger from the amended SEC-DATA-4.
- **PQ-17 — Abuse-prevention thresholds. RESOLVED** (`SECURITY.md` SQ-3, 2026-08-01). The standard parameter set is recorded as named constants in SEC-AUTHN-6/-8/-11, SEC-SESSION-3/-5, SEC-HTTP-1/-5, and SEC-AI-2: sessions 24 h/2 h (subscriber) and 12 h/30 min (privileged) with `SameSite=Lax`; reset tokens 30 min, invitations 72 h, 10 recovery codes; backoff 3-failures/1 s-doubling/15-min cap; auth 10/min, API 120/min, AI 50/day, export 1/day, bodies 1 MB/10 MB. Threats TM-S-1, TM-S-5, TM-D-1 all MITIGATED BY RULE. SQ-11 has since resolved the alerting half: threshold alerts route to the security lead as SEC-OPS-2 detection inputs, and every draft's `Alerting` field now says so (2026-08-03 refresh). Enforcement is drafted as REQ-API-050.
- **PQ-18 — ABAC attribute schema, policy language, and PDP/PEP architecture. RESOLVED** (`SECURITY.md` SQ-4, 2026-08-01). A first-party typed policy module at the single Fastify preHandler enforcement point; deny-overrides with missing-attribute denial; typed attribute schema sourced from Identity and persisted state only; capabilities named per action and mapped from role plus relationship. No policy-language dependency. The central policy module is drafted 2026-08-03 as REQ-AUTHZ-050, now also carrying the SEC-AUTHZ-9 admin health-data prohibition (added 2026-08-03); REQ-AUTHZ-010 … 040 and REQ-CONSULT-010 remain covered as drafted.
- **PQ-19 — CI/CD platform, secret store, AWS topology. RESOLVED** (`SECURITY.md` SQ-7, 2026-08-01). Three AWS accounts (dev/staging/production); ECS Fargate API, S3 + CloudFront client, RDS PostgreSQL Multi-AZ, Bedrock in-account for AI, Secrets Manager for secrets and signing keys (`kid` rotation, 90 days); VPC tiering with restricted NAT egress; GitHub Actions with per-environment OIDC roles, staging auto / production manual; SEC-CICD-4 gate set fixed (lint+typecheck, Vitest, Playwright+axe, osv-scanner, gitleaks, checkov, authorization suite); Argon2id 64 MiB/3/1 on the 1 vCPU/2 GB basis. Threat TM-T-6 MITIGATED BY RULE. Infrastructure scope is drafted 2026-08-03: REQ-INFRA-010 … 070 (accounts and pipeline, network and encryption, secrets, audit archive, break-glass, SES email, CI gates). The SQ-7 addendum (2026-08-03) added in-account Amazon SES to the named-egress set (SEC-EXT-3) and fixed CloudFront as the single public origin forwarding the API path prefix to the ALB — the routing that realizes SQ-6's same-origin surface.
- **PQ-20 — Audit and log retention, access control, and tamper-evidence. RESOLVED** (`SECURITY.md` SQ-8 and SQ-13, 2026-08-01). Security logs 12 months; audit entries 3 years with admin-only audited access; Postgres append-only privileges plus nightly hash-chained batches to S3 Object Lock (governance mode) in a log-archive account — batches pseudonymized at batch time via the SEC-DATA-4 keyed derivation (2026-08-03); break-glass by a dedicated IAM role with second-admin approval, a 4-hour box, session logging, and 7-day review. Threats TM-R-2, TM-I-8, TM-P-4 all MITIGATED BY RULE. Drafted 2026-08-03: REQ-INFRA-040 (SEC-LOG-5/-7) and REQ-INFRA-050 (SEC-OPS-1), with the SEC-OPS-2 incident-response runbook as REQ-OPS-010.
- **PQ-21 — Governing privacy regime. RESOLVED** (`SECURITY.md` SQ-1, `REQUIREMENTS.md` OQ-3, 2026-08-01). Global service with GDPR as the design ceiling: GDPR/UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws, and the FTC Health Breach Notification Rule for US users; HIPAA not applicable (no covered-entity or business-associate relationship). GDPR-grade rights for all users; data residency single US primary region with standard transfer mechanisms. Per-issue `Regulatory` fields were filled at the 2026-08-03 refresh with that regime set (statute-section precision still needs per-issue verification and stays `TO BE DECIDED` except where a specification document states a section); breach notification has since resolved (SQ-11, SEC-OPS-2 — threat TM-P-3 MITIGATED BY RULE) and retention with SQ-8; counsel review before launch required.
- **PQ-22 — Privileged account provisioning and first passkey enrolment. RESOLVED.** Invitation from an existing `admin` carrying a single-use, short-lived, role-scoped enrolment token to a verified address; that token authorizes passkey registration only and never yields a session. The first `admin` comes from a one-time provisioning command that refuses to run once any `admin` exists. Role is fixed by the invitation and never settable from a request body. Recorded as `REQUIREMENTS.md` FR-2.10 and `SECURITY.md` SEC-AUTHN-9. This breaks the REQ-AUTH-020 / REQ-AUTH-030 circular dependency: enrolment no longer presupposes a passkey session. Threats TM-S-4 and the request-path half of TM-E-1 are closed. SQ-12 has since resolved the remainder: vetting is recorded on every invitation (FR-2.16, drafted as REQ-AUTH-160) and deprovisioning is an audited admin action with the two-admin floor (FR-2.17, drafted as REQ-AUTH-170).
- **PQ-23 — Third-party API exposure and CORS. RESOLVED** (`SECURITY.md` SQ-6, 2026-08-01). The REST surface is first-party only; CORS is disabled outright with no allow-list to maintain (SEC-HTTP-3 Confirmed). The no-CORS assertion folded into the REQ-PLATFORM-040 header work at the 2026-08-03 refresh; the SQ-7 addendum realizes the same-origin surface with CloudFront as the single public origin.
- **PQ-24 — Exact CSP directive set. RESOLVED** (2026-08-03; `SECURITY.md` SEC-HTTP-2 Confirmed). The directive set is fixed: `default-src 'none'; script-src 'self'; style-src 'self'; img-src 'self' data: blob:; connect-src 'self'; base-uri 'none'; form-action 'self'; frame-ancestors 'none'; object-src 'none'` — the `data:`/`blob:` image sources exist for the transient food-photo preview (FR-8.12). The policy runs report-only in staging before enforcement, and any relaxation requires a documented amendment to SEC-HTTP-2. Delivered by REQ-PLATFORM-040 (draft refreshed 2026-08-03); threat TM-T-4's open CSP note is closed.
- **PQ-25 — UI presentation decisions. RESOLVED** (`DESIGN.md` OQ-5, OQ-6, OQ-7, 2026-08-01; extended 2026-08-03). Citation surfacing is inline numbered references connected to an in-page Evidence section, with "Evidence reviewed by Proof of Work" plus the date for subscribers and verifier identity for admins (OQ-5). The medical disclaimer is a focused first-plan interstitial plus a persistent quiet safety link, with health-data consent as a separate explicit-checkbox flow (OQ-6). Roles get distinct labelled workspaces, with consultant access visible on Home, in Account, and as edit provenance on affected copies (OQ-7). The 2026-08-03 amendment added the credentials, account security, and administration patterns (sign-in second factor, verification, reset, MFA enrolment, re-authentication prompt, email change, passkeys, invitation acceptance, admin People/Access/catalog views). REQ-PRIVACY-040 and REQ-CATALOG-010/020 were refreshed accordingly.
- **PQ-26 — Threat model completion. RESOLVED** (`SECURITY.md` SQ-9 and SQ-10, 2026-08-01). The security lead owns the model with event-driven plus annual refresh and a product/counsel-validated post-implementation re-run before launch; ASVS 5.0.0 Level 3 is confirmed as the design target with conformance only via an independent pre-launch assessment. Per-issue `OWASP ASVS 5.0.0` mapping fields stay `TO BE DECIDED` until verified in that assessment — deliberately, per the template's no-guessing rule (AISVS 1.0 likewise for the AI-component issues).

### Conflicts found between documents

Only one, already recorded by the specification itself: the jurisdiction inconsistency (PQ-21 / `SECURITY.md` SQ-1), since resolved (2026-08-01). No other conflict was found between `REQUIREMENTS.md`, `ARCHITECTURE.md`, `SECURITY.md`, and `DESIGN.md`. Two apparent tensions were resolved by reading each document as authoritative in its own domain rather than by choosing between them:

1. `DESIGN.md` requires error messages that "name the specific problem", while SEC-AUTHN-3 and SEC-ERR-1 require generic, non-disclosing failures. Resolved in REQ-API-040 and REQ-AUTH-040 as: field-format validation is specific, anything dependent on account state or internal state is generic. This is a reading of both documents, not a new decision.
2. `REQUIREMENTS.md` OQ-11 originally left accessibility conformance open while `DESIGN.md` targets WCAG 2.2 AA. `ARCHITECTURE.md` explicitly records that `DESIGN.md` resolves OQ-11 — and OQ-11 is now marked RESOLVED — so WCAG 2.2 AA is treated as firm.

---

## 2. Coverage matrix — functional requirements

Status values: `COVERED`, `PARTIALLY COVERED`, `BLOCKED`, `UNBLOCKED — AWAITING DRAFT` (blocking question resolved; issue not yet drafted), `OUT OF SCOPE`.

| Requirement | Issue IDs | Boundary | Security rules | Design sections | Status |
|---|---|---|---|---|---|
| FR-1.1 | REQ-PLATFORM-020, REQ-PLATFORM-040 | Browser Client; boundary 1 | SEC-HTTP-1, SEC-HTTP-2 | Layout and Spacing | COVERED |
| FR-1.2 | REQ-PLATFORM-020 | Browser Client | — | Layout and Spacing; Accessibility | COVERED |
| FR-2.1 | REQ-AUTHZ-010, REQ-SESSION-010 | Boundary 2 | SEC-AUTHN-1, SEC-AUTHZ-1 | — | COVERED |
| FR-2.2 | REQ-AUTH-080, REQ-AUTH-070 | Identity | SEC-AUTHN-5, SEC-AUTHN-6 | Components → Inputs | COVERED |
| FR-2.3 | REQ-AUTH-040, REQ-AUTH-100, REQ-AUTH-070 | Identity; boundary 2 | SEC-AUTHN-3, SEC-AUTHN-5 | Components → Form feedback | COVERED |
| FR-2.4 | REQ-SESSION-040, REQ-SESSION-030 | Identity | SEC-SESSION-3, SEC-SESSION-5 | — | COVERED |
| FR-2.5 | REQ-AUTH-110 | Identity | SEC-AUTHN-4, SEC-AUTHN-7 | Components → Buttons | COVERED |
| FR-2.6 | REQ-AUTH-120 | Identity | SEC-AUTHN-4 | — | COVERED |
| FR-2.7 | REQ-AUTH-010, REQ-AUTH-140 | Identity; REST API | SEC-AUTHZ-1, SEC-INPUT-3, SEC-AUTHN-9 | — | COVERED |
| FR-2.8 | REQ-AUTH-020 | Identity; boundary 2 | SEC-AUTHN-2 | Components | COVERED |
| FR-2.9 | REQ-AUTH-030, REQ-AUTH-140, REQ-AUTH-150 | Identity | SEC-AUTHN-2, SEC-AUTHN-7, SEC-AUTHN-9 | Components → Buttons | COVERED |
| FR-2.10 | REQ-AUTH-140 | Identity | SEC-AUTHN-9 | — | COVERED |
| FR-2.11 | REQ-AUTH-090 | Identity; REST API | SEC-AUTHN-8 | Components → Empty states | COVERED |
| FR-2.12 | REQ-AUTH-130 | Identity | SEC-AUTHN-10, SEC-AUTHN-11, SEC-AUTHN-12 | Components → Inputs | COVERED |
| FR-2.13 | REQ-AUTH-110, REQ-AUTH-120 | Identity | SEC-AUTHN-10, SEC-AUTHN-11 | — | COVERED |
| FR-2.14 | REQ-AUTH-110 | Browser Client | SEC-AUTHN-10 | Components → Form feedback | COVERED |
| FR-2.15 | REQ-AUTH-150 | Identity | SEC-AUTHN-2, SEC-AUTHN-9, SEC-AUTHN-10 | — | COVERED |
| FR-3.1 | REQ-ENTITLE-010 | REST API | SEC-AUTHZ-8 | Components → Status, feedback | COVERED (drafted 2026-08-03) |
| FR-3.2 | REQ-ENTITLE-010 | REST API | SEC-AUTHZ-8 | Components → Status, feedback | COVERED (drafted 2026-08-03) |
| FR-3.3 | REQ-ENTITLE-020 | REST API | SEC-AUTHZ-8, SEC-DATA-5 | — | COVERED (drafted 2026-08-03) |
| FR-3.4 | REQ-ENTITLE-040 | REST API; Persistence | SEC-AUTHZ-8 | — | COVERED (drafted 2026-08-03) |
| FR-3.5 | REQ-ENTITLE-030 | REST API; Persistence | SEC-AUTHZ-1, SEC-AUTHZ-8, SEC-INPUT-3 | — | COVERED (drafted 2026-08-03) |
| FR-3.6 | REQ-ENTITLE-010 | REST API | SEC-AUTHZ-8, SEC-INPUT-3 | — | COVERED (drafted 2026-08-03) |
| FR-4.1 | REQ-PLAN-010, REQ-PLAN-020 | REST API; Persistence | SEC-INPUT-1, SEC-INPUT-5 | Typography; Layout | COVERED |
| FR-4.2 | REQ-AUTHZ-030, REQ-CUSTOM-010 | REST API | SEC-AUTHZ-4 | — | COVERED |
| FR-4.3 | REQ-PLAN-030, REQ-PLAN-050, REQ-PLAN-060 | REST API | SEC-AUTHZ-4, SEC-INPUT-4 | Components | COVERED |
| FR-4.4 | REQ-PLAN-040, REQ-PLAN-050 | REST API; boundary 1 | SEC-INPUT-4, SEC-EXT-2, SEC-RENDER-3 | Components → Links | COVERED |
| FR-4.5 | REQ-PLAN-050, REQ-AUDIT-030, REQ-PLAN-070 | REST API | SEC-INPUT-4, SEC-LOG-6 | — | COVERED — record and gate covered; the verification operation drafted 2026-08-03 (REQ-PLAN-070) |
| FR-4.8 | REQ-PLAN-070 | REST API | SEC-INPUT-4, SEC-AUTHZ-4 | — | COVERED (drafted 2026-08-03) |
| FR-4.9 | REQ-SELECT-030 | REST API; Persistence | SEC-INPUT-4, SEC-LOG-1, SEC-LOG-6, SEC-AUTHZ-9 | Components → Status, feedback, and loading | COVERED (FR added and drafted 2026-08-03) |
| FR-4.6 | REQ-CATALOG-010, REQ-CATALOG-020, REQ-CATALOG-030 | Browser Client | SEC-RENDER-1, SEC-RENDER-3 | Product Patterns → Plans, citations, and review state | COVERED — display, safe rendering, and the citation-surfacing pattern (`DESIGN.md` OQ-5; PQ-25 RESOLVED) |
| FR-4.7 | REQ-PLAN-050, REQ-PLAN-060, REQ-CATALOG-010, REQ-CATALOG-020 | REST API | SEC-AUTHZ-1, SEC-DATA-5 | — | COVERED |
| FR-5.1 | REQ-PLAN-010, REQ-CATALOG-010 | REST API; Browser Client | SEC-DATA-5, SEC-RENDER-1 | Typography; Layout | COVERED |
| FR-5.2 | REQ-SELECT-010 | REST API | SEC-AUTHZ-2 | — | COVERED (drafted 2026-08-03) |
| FR-5.3 | REQ-SELECT-010 | REST API; Persistence | SEC-AUTHZ-2, SEC-INPUT-3 | — | COVERED (drafted 2026-08-03) |
| FR-5.4 | REQ-PLAN-080 | REST API; Persistence | SEC-AUTHZ-4, SEC-INPUT-1, SEC-INPUT-3, SEC-LOG-6 | Product Patterns → Exercise catalog; Components → Status chips (`Retired`) | COVERED (FR added and drafted 2026-08-03) |
| FR-6.1 | REQ-CATALOG-020 | REST API; Browser Client | SEC-DATA-5, SEC-RENDER-1 | Typography; Layout | COVERED |
| FR-6.2 | REQ-PLAN-020, REQ-CATALOG-020 | Persistence; Browser Client | SEC-INPUT-1 | Typography | COVERED — macronutrients fixed 2026-08-03 as protein, carbohydrate, fat (grams); drafts refreshed |
| FR-6.3 | REQ-SELECT-020 | REST API | SEC-AUTHZ-2 | — | COVERED (drafted 2026-08-03) |
| FR-6.4 | REQ-SELECT-020 | REST API; Persistence | SEC-AUTHZ-2, SEC-INPUT-3 | — | COVERED (drafted 2026-08-03) |
| FR-7.1 | REQ-CUSTOM-010 | REST API | SEC-INPUT-4, SEC-INPUT-1 | Components → Inputs | COVERED |
| FR-7.2 | REQ-CUSTOM-010 | REST API; Persistence | SEC-INPUT-3, SEC-INPUT-4 | — | COVERED |
| FR-7.3 | REQ-CUSTOM-020 | REST API; Persistence | SEC-AUTHZ-2 | Components → Empty states | COVERED |
| FR-7.4 | REQ-AUTHZ-020, REQ-CUSTOM-020 | REST API | SEC-AUTHZ-2 | — | COVERED |
| FR-7.5 | REQ-CUSTOM-030, REQ-PLAN-030, REQ-PLAN-060 | Persistence | SEC-INPUT-4 | — | COVERED |
| FR-8.1 | REQ-PROGRESS-010 | REST API; Persistence | SEC-DATA-2, SEC-AUTHZ-2, SEC-LOG-1 | Components → Inputs | COVERED |
| FR-8.2 | REQ-PROGRESS-040 | REST API | SEC-DATA-2, SEC-AUTHZ-2, SEC-LOG-1, SEC-INPUT-2 | Components → Inputs | COVERED (drafted 2026-08-03; fields all-optional with at least one value, as amended) |
| FR-8.10 | REQ-PROGRESS-050 | REST API; Persistence | SEC-INPUT-1, SEC-INPUT-3 | Typography (tabular figures) | COVERED (drafted 2026-08-03) |
| FR-8.11 | REQ-FOOD-010 | REST API; Persistence | SEC-INPUT-1, SEC-EXT-1; DEP-5, DEP-7 | — | COVERED (drafted 2026-08-03) |
| FR-8.12 | REQ-FOOD-040 | REST API; AI Inference; Browser Client | SEC-AI-2, SEC-AI-3, SEC-INPUT-1, SEC-INPUT-2, SEC-INPUT-7 | Components → Inputs, Form feedback; Product Patterns → Logging and AI-assisted food estimates | COVERED (drafted 2026-08-03) |
| FR-8.13 | REQ-FOOD-050 | AI Inference; boundary 6 | SEC-AI-1, SEC-AI-3, SEC-TB-3 | — | COVERED (drafted 2026-08-03) |
| FR-8.3 | REQ-PROGRESS-020 | REST API; Persistence | SEC-DATA-2, SEC-AUTHZ-2, SEC-LOG-1 | Components → Inputs; Typography | COVERED — logged exercises reference the FR-5.4 catalog entry and snapshot the name (amended 2026-08-03; draft refreshed) |
| FR-8.4 | REQ-FOOD-020 | REST API; AI Inference | SEC-DATA-2, SEC-AUTHZ-2, SEC-LOG-1 | Components → Inputs | COVERED (drafted 2026-08-03) |
| FR-8.5 | REQ-FOOD-030 | REST API; Browser Client | SEC-DATA-5 | Typography; Product Patterns → Progress and target comparison | COVERED (drafted 2026-08-03) |
| FR-8.6 | REQ-PROGRESS-060 | REST API; Browser Client | SEC-DATA-5 | — | COVERED (drafted 2026-08-03) |
| FR-8.14 | REQ-PROGRESS-060 | REST API; Browser Client | SEC-DATA-5, SEC-AUTHZ-2 | Accessibility; Typography | COVERED (drafted 2026-08-03; per-exercise trends keyed by catalog entry, top-set weight per session) |
| FR-8.7 | REQ-PROGRESS-010, REQ-PROGRESS-020, REQ-PROGRESS-040, REQ-FOOD-020 | REST API | SEC-AUTHZ-2, SEC-LOG-1 | Components → Buttons | COVERED — all four entry types drafted (measurement and food added 2026-08-03) |
| FR-8.8 | REQ-PROGRESS-010, REQ-PROGRESS-020, REQ-PROGRESS-040, REQ-FOOD-020 | REST API | SEC-INPUT-1 | — | COVERED — all four entry types drafted; future-dated entries rejected (amended 2026-08-03) |
| FR-8.9 | REQ-PROGRESS-030 | Boundary 1; REST API | SEC-INPUT-2, SEC-ERR-1 | Components → Form feedback | COVERED — future-dated and out-of-range rejection classes with named-constant plausibility ranges added 2026-08-03; draft refreshed |
| FR-9.1 | REQ-AUTHZ-020, REQ-PRIVACY-060 | REST API | SEC-AUTHZ-2, SEC-DATA-5 | — | COVERED |
| FR-9.2 | REQ-PRIVACY-010 | REST API | SEC-DATA-2 | Components → Inputs | COVERED — consent covers estimation acceptance (amended 2026-08-03; draft refreshed) |
| FR-9.3 | REQ-PRIVACY-080 | REST API | SEC-DATA-3, SEC-AUTHN-7 | — | COVERED (drafted 2026-08-03; fresh re-authentication and one audit entry per export, as amended) |
| FR-9.4 | REQ-PRIVACY-090 | REST API; Persistence | SEC-DATA-4, SEC-AUTHN-7 | Product Patterns → Account, privacy, and destructive actions | COVERED (drafted 2026-08-03) |
| FR-9.10 | REQ-PRIVACY-100 | REST API; Persistence | SEC-DATA-4, SEC-LOG-2, SEC-LOG-3 | — | COVERED (drafted 2026-08-03; keyed HMAC-SHA-256 tombstone derivation, as amended) |
| FR-9.11 | REQ-PRIVACY-110 | REST API; Identity | SEC-AUTHN-3, SEC-AUTHN-10, SEC-DATA-4, SEC-EXT-3 | Components → Form feedback; Status chips (`Pending deletion`) | COVERED (drafted 2026-08-03) |
| FR-9.12 | REQ-PRIVACY-070 | REST API | SEC-AUTHZ-9, SEC-AUTHZ-6, SEC-AUTHZ-7 | — | COVERED (FR added and drafted 2026-08-03; classification consumed by the selection, progress, food, and admin-prohibition drafts) |
| FR-9.5 | REQ-PRIVACY-030, REQ-AUTH-180 | REST API; Browser Client | SEC-AUTHZ-2, SEC-INPUT-3 | Components → Inputs, Form feedback | COVERED — email-address change drafted 2026-08-03 (REQ-AUTH-180; FR-2.18) |
| FR-9.6 | REQ-PRIVACY-040 | REST API; Browser Client | SEC-TB-1, SEC-INPUT-4 | Product Patterns → Medical disclaimer and health-data consent | COVERED — mechanism, enforcement, and presentation (`DESIGN.md` OQ-6; PQ-25 RESOLVED); trigger set complete (PQ-12 RESOLVED) |
| FR-9.7 | REQ-AUDIT-010, REQ-AUDIT-020 | REST API; Persistence | SEC-LOG-1, SEC-LOG-2, SEC-LOG-3 | — | COVERED — affected subject, audited self-reads, one entry per request (amended 2026-08-03; drafts refreshed) |
| FR-9.8 | REQ-PRIVACY-050, REQ-AUDIT-040, REQ-INFRA-020 | System egress; boundary 4 | SEC-TB-3, SEC-EXT-1, SEC-EXT-2 | — | COVERED — application level plus the network egress restriction drafted 2026-08-03 (REQ-INFRA-020) |
| FR-9.9 | REQ-PRIVACY-020 | REST API | SEC-DATA-2 | Components → Buttons | COVERED — edit/delete of existing entries stays available under withdrawal (amended 2026-08-03; draft refreshed) |
| FR-10.1 | REQ-AUTHZ-030 | REST API | SEC-AUTHZ-4 | — | COVERED |
| FR-10.2 | REQ-AUDIT-030 | REST API; Persistence | SEC-LOG-6, SEC-LOG-2 | — | COVERED |
| FR-10.3 | REQ-AUTHZ-060 | REST API | SEC-AUTHZ-9, SEC-DATA-5 | Product Patterns → People (admin) | COVERED (FR added and drafted 2026-08-03) |
| FR-2.16 | REQ-AUTH-160 | Identity | SEC-AUTHN-9, SEC-LOG-4 | Product Patterns → People (admin) | COVERED (drafted 2026-08-03) |
| FR-2.17 | REQ-AUTH-170 | Identity; REST API | SEC-AUTHN-13, SEC-SESSION-4, SEC-LOG-4 | Product Patterns → People (admin) | COVERED (drafted 2026-08-03) |
| FR-2.18 | REQ-AUTH-180 | Identity | SEC-AUTHN-7, SEC-AUTHN-8, SEC-AUTHN-12, SEC-EXT-3 | Product Patterns → Credentials, account security, and administration | COVERED (FR added and drafted 2026-08-03) |
| FR-11.1 | REQ-CONSULT-030 | REST API | SEC-AUTHZ-3 | — | COVERED (drafted 2026-08-03; reworded 2026-08-03 for testability — engagement by admin action, state visible to the subscriber) |
| FR-11.5 | REQ-CONSULT-030 | REST API; Persistence | SEC-AUTHZ-3, SEC-INPUT-3 | — | COVERED (drafted 2026-08-03) |
| FR-11.2 | REQ-CONSULT-010 | REST API | SEC-AUTHZ-3 | — | COVERED — access bounded; capabilities delivered by REQ-CONSULT-040 (PQ-14 RESOLVED) |
| FR-11.6 | REQ-CONSULT-040 | REST API | SEC-AUTHZ-3, SEC-AUTHZ-2, SEC-LOG-1 | Information Architecture (consultant context bar; edit provenance) | COVERED (drafted 2026-08-03) |
| FR-11.3 | REQ-CONSULT-020 | REST API; Identity | SEC-AUTHZ-3, SEC-SESSION-4 | Components → Buttons | COVERED |
| FR-11.4 | REQ-CONSULT-010, REQ-AUDIT-020 | REST API | SEC-LOG-1 | — | COVERED |

**Totals** — 83 functional requirements: 83 `COVERED`, 0 `PARTIALLY COVERED`, 0 `UNBLOCKED — AWAITING DRAFT`, 0 `BLOCKED`, 0 `OUT OF SCOPE`, 0 untracked. Counts re-tallied from the matrix rows on 2026-08-03, after the five new FRs (FR-2.18, FR-4.9, FR-5.4, FR-9.12, FR-10.3) were added and the 38 new drafts absorbed every `UNBLOCKED — AWAITING DRAFT` and `PARTIALLY COVERED` remainder. Every requirement traces to at least one drafted issue.

## 3. Coverage matrix — security rules

| Rule(s) | Issue IDs | Status |
|---|---|---|
| SEC-TB-1 | REQ-API-010, REQ-API-020, REQ-AUTHZ-010, REQ-PRIVACY-040 | COVERED |
| SEC-TB-2 | REQ-INFRA-020 | COVERED — isolated subnets reachable only via DR-5 sanctioned paths (drafted 2026-08-03) |
| SEC-TB-3 | REQ-PRIVACY-050, REQ-AUDIT-040, REQ-INFRA-020 | COVERED — application level plus the network egress restriction (drafted 2026-08-03) |
| SEC-AUTHN-1 | REQ-AUTHZ-010 | COVERED |
| SEC-AUTHN-2 | REQ-AUTH-020, REQ-AUTH-030 | COVERED |
| SEC-AUTHN-3 | REQ-AUTH-040, REQ-INFRA-060, REQ-PRIVACY-110 | COVERED — non-enumerating send/refusal behavior extended to email delivery and the out-of-band channel (2026-08-03) |
| SEC-AUTHN-4 | REQ-AUTH-120, REQ-AUTH-100 | COVERED |
| SEC-AUTHN-5 | REQ-AUTH-070 | COVERED |
| SEC-AUTHN-6 | REQ-AUTH-060, REQ-AUTH-080 | COVERED — thresholds fixed (PQ-17 RESOLVED); the breached-password list is a versioned build/deploy-time artifact under DEP-5/DEP-7 (amended 2026-08-03; drafts refreshed) |
| SEC-AUTHN-7 | REQ-AUTH-030, REQ-AUTH-050, REQ-AUTH-110, REQ-AUTH-150, REQ-AUTH-180, REQ-PRIVACY-080, REQ-PRIVACY-090 | COVERED — now Confirmed and extended 2026-08-03 to email change, export, and deletion with the 5-minute freshness constant |
| SEC-AUTHN-10, -11, -12 | REQ-AUTH-130, REQ-AUTH-110, REQ-AUTH-120, REQ-AUTH-150, REQ-SESSION-040, REQ-AUTH-180, REQ-PRIVACY-110, REQ-INFRA-030 | COVERED — SEC-AUTHN-11's 2026-08-03 hash split (Argon2id for typed recovery codes; HMAC-SHA-256 under a Secrets Manager key for machine-held tokens) rides the auth drafts and REQ-INFRA-030 |
| SEC-AUTHN-8 | REQ-AUTH-090, REQ-AUTH-180 | COVERED — extended 2026-08-03 to the FR-2.18 replacement address |
| SEC-AUTHN-13 | REQ-AUTH-170 | COVERED — row added 2026-08-03; the rule existed since the SQ-12 resolution |
| SEC-SESSION-1, -2 | REQ-SESSION-010 | COVERED |
| SEC-SESSION-3, -4, -5 | REQ-SESSION-030, REQ-SESSION-040, REQ-SESSION-050, REQ-CONSULT-020 | COVERED |
| SEC-SESSION-7 | REQ-INFRA-030 | COVERED — Secrets Manager, `kid`-based 90-day rotation (drafted 2026-08-03) |
| SEC-AUTHN-9 | REQ-AUTH-140, REQ-AUTH-150, REQ-AUTH-160 | COVERED — vetting-record precondition drafted 2026-08-03; wording fixed 2026-08-03 (enrolment token as the only first-passkey path) |
| SEC-SESSION-6 | REQ-SESSION-020 | COVERED |
| SEC-AUTHZ-1, -2 | REQ-AUTHZ-010, REQ-AUTHZ-020 | COVERED |
| SEC-AUTHZ-3 | REQ-CONSULT-010, REQ-CONSULT-020, REQ-CONSULT-030, REQ-CONSULT-040 | COVERED — lifecycle and capability halves drafted 2026-08-03 |
| SEC-AUTHZ-4 | REQ-AUTHZ-030, REQ-PLAN-070, REQ-PLAN-080 | COVERED |
| SEC-AUTHZ-5, -6, -7 | REQ-AUTHZ-010 (fail-closed only), REQ-AUTHZ-050 | COVERED — central typed policy module drafted 2026-08-03 (PQ-18 RESOLVED) |
| SEC-AUTHZ-8 | REQ-ENTITLE-010, REQ-ENTITLE-030, REQ-ENTITLE-040 | COVERED — admin-granted periods gated at the enforcement point (drafted 2026-08-03) |
| SEC-AUTHZ-9 | REQ-AUTHZ-060, REQ-AUTHZ-050, REQ-PRIVACY-070 | COVERED — rule added 2026-08-03 with FR-10.3; structural prohibition in the policy module |
| SEC-HTTP-1, -2 | REQ-PLATFORM-040 | COVERED — CSP directive set fixed (PQ-24 RESOLVED 2026-08-03); draft refreshed with the full set, report-only in staging before enforcement |
| SEC-HTTP-3 | REQ-PLATFORM-040 | COVERED — CORS disabled outright (PQ-23 RESOLVED); folded in at the 2026-08-03 refresh |
| SEC-HTTP-4 | REQ-SESSION-050 | COVERED |
| SEC-HTTP-5 | REQ-API-050 | COVERED — now Confirmed with the SQ-3 limits; drafted 2026-08-03 |
| SEC-HTTP-6 | REQ-API-040 | COVERED |
| SEC-INPUT-1, -3, -6 | REQ-API-010, REQ-API-020 | COVERED |
| SEC-INPUT-2 | REQ-PROGRESS-030, REQ-PROGRESS-040, REQ-FOOD-020 | COVERED — measurement and food validation drafted 2026-08-03 |
| SEC-INPUT-4 | REQ-PLAN-050, REQ-CUSTOM-010, REQ-CUSTOM-030, REQ-PLAN-070, REQ-SELECT-030 | COVERED |
| SEC-INPUT-5 | REQ-API-030; REQ-BUILD-010 (wires the dynamic-query-construction static analysis SEC-INPUT-5 names as its verification method) | COVERED |
| SEC-INPUT-7 | REQ-FOOD-040 | COVERED — rule added 2026-08-03: magic-byte format checks, 12-megapixel decoded cap, decompression-anomaly rejection, EXIF/GPS stripping before inference |
| SEC-RENDER-1, -3, -4 | REQ-CATALOG-030; REQ-BUILD-010 (wires the `vue/no-v-html` lint rule SEC-RENDER-1 names as its verification method) | COVERED |
| SEC-RENDER-2 | REQ-CATALOG-030 | COVERED — Confirmed: v1 plan content is structured plain text with no arbitrary-HTML or rich-text rendering path; the sanitizer clause activates only on a future design change |
| SEC-DATA-1 | REQ-INFRA-020 | COVERED — KMS on storage, backups, snapshots, and replicas; TLS including database connections (drafted 2026-08-03) |
| SEC-DATA-2 | REQ-PRIVACY-010, REQ-PRIVACY-020 | COVERED — extended 2026-08-03 to refuse estimation without consent; the gate reaches the selection, progress, and food drafts through the FR-9.12 classification (REQ-PRIVACY-070) |
| SEC-DATA-3, -4 | REQ-PRIVACY-080, REQ-PRIVACY-090, REQ-PRIVACY-100 | COVERED — synchronous export and deletion with the keyed tombstone derivation and deletion ledger (drafted 2026-08-03); SEC-DATA-6 remains MOOT (no stored artifact) |
| SEC-DATA-5 | REQ-PRIVACY-060 | COVERED |
| SEC-OPS-1 | REQ-INFRA-050 | COVERED — deny-by-default two-person break-glass (drafted 2026-08-03) |
| SEC-OPS-2 | REQ-OPS-010 | COVERED — row added 2026-08-03; IR runbook with statutory clocks and the deletion-ledger restore step |
| SEC-AI-1, -2, -3 | REQ-FOOD-050 (SEC-AI-1), REQ-FOOD-040 (SEC-AI-2, -3) | COVERED — drafted 2026-08-03; SEC-AI-1 now pins the Bedrock model identifier and version behind SEC-CICD-2 review |
| SEC-SECRET-1, -2, -3, -4 | REQ-AUDIT-040 (partial, for logs); REQ-BUILD-010 (SEC-SECRET-1 — no secret material in the committed tree); REQ-INFRA-030 (SEC-SECRET-2, -3) | COVERED — Secrets Manager entries and protected Terraform state drafted 2026-08-03; SEC-SECRET-4 rides the auth drafts |
| SEC-LOG-1, -2 | REQ-AUDIT-010, REQ-AUDIT-020 | COVERED |
| SEC-LOG-3 | REQ-AUDIT-040 | COVERED |
| SEC-LOG-4 | REQ-AUTH-050, REQ-AUTHZ-040 | COVERED |
| SEC-LOG-5, -7 | REQ-INFRA-040 | COVERED — retention, append-only privileges, and the pseudonymized hash-chained archive produced by the nightly scheduled execution (drafted 2026-08-03) |
| SEC-LOG-6 | REQ-AUDIT-030, REQ-PLAN-070, REQ-PLAN-080 | COVERED |
| SEC-ERR-1 | REQ-API-040 | COVERED |
| SEC-EXT-1, -2 | REQ-PRIVACY-050, REQ-PLAN-040, REQ-FOOD-010 | COVERED — SEC-EXT-1 now references SEC-EXT-3 for the mail channel (2026-08-03) |
| SEC-EXT-3 | REQ-INFRA-060 | COVERED — rule added 2026-08-03: in-account SES behind the internal mail interface, named egress, no health data, non-enumerating send behavior |
| SEC-CICD-1, -2 | REQ-INFRA-010 | COVERED — per-environment OIDC roles and Terraform-only reviewed change (drafted 2026-08-03; formerly BLOCKED — PQ-19) |
| SEC-CICD-3 | REQ-INFRA-020 | COVERED — VPC tiering with named egress including SES (drafted 2026-08-03) |
| SEC-CICD-4 | REQ-INFRA-070 | COVERED — merge-blocking gate set drafted 2026-08-03 |
| SEC-CICD-5 | REQ-PIPE-020, REQ-INFRA-010 | COVERED — account-separation enforcement drafted 2026-08-03 |
| DEP-1 … DEP-8 | REQ-PIPE-010; REQ-BUILD-010 (DEP-7 — committed lockfile and frozen `npm ci` install); REQ-INFRA-070 (automated osv-scanner gate) | COVERED — the automated gate formerly blocked by PQ-19 is drafted 2026-08-03 |

## 4. Blocked scope — no issue drafted

| Scope | Requirements | Blocked by |
|---|---|---|
*None — every previously blocked area was resolved by 2026-08-01 and drafted by 2026-08-03; the coverage matrices show no `BLOCKED` or `AWAITING DRAFT` row. Still none as of 2026-08-03.*

## 5. Hierarchy

```
REQ-EPIC-001  Implement the specified subscription fitness web application
├─ Foundation
│  └─ REQ-BUILD-010  Workspace scaffolding and toolchain baseline
├─ Platform and delivery
│  ├─ REQ-PLATFORM-010  Design tokens
│  ├─ REQ-PLATFORM-020  Responsive layout and reflow
│  ├─ REQ-PLATFORM-030  Keyboard, focus, reduced motion
│  └─ REQ-PLATFORM-040  TLS and security response headers
├─ API boundary
│  ├─ REQ-API-010  Schema validation
│  ├─ REQ-API-020  Server-controlled field binding
│  ├─ REQ-API-030  Parameterized database access
│  ├─ REQ-API-040  Error hygiene and diagnostic exclusion
│  └─ REQ-API-050  Rate limits, body sizes, time-bounded handling  (2026-08-03)
├─ Authorization
│  ├─ REQ-AUTHZ-010  Deny-by-default authentication gate
│  ├─ REQ-AUTHZ-020  Object-level ownership scoping
│  ├─ REQ-AUTHZ-030  Admin-only plan lifecycle
│  ├─ REQ-AUTHZ-040  Denial response and logging
│  ├─ REQ-AUTHZ-050  Central typed policy module  (2026-08-03)
│  └─ REQ-AUTHZ-060  Admin health-data prohibition  (2026-08-03)
├─ Session
│  ├─ REQ-SESSION-010  JWT verification
│  ├─ REQ-SESSION-020  Token claim allow-list
│  ├─ REQ-SESSION-030  Server-side session records and resolution
│  ├─ REQ-SESSION-040  Logout and revocation
│  └─ REQ-SESSION-050  Cookie transport and CSRF
├─ Identity
│  ├─ REQ-AUTH-010  One role per account
│  ├─ REQ-AUTH-020  Passkey authentication for privileged roles
│  ├─ REQ-AUTH-030  Passkey registration and replacement
│  ├─ REQ-AUTH-040  Non-disclosing authentication failures
│  ├─ REQ-AUTH-050  Security event logging
│  ├─ REQ-AUTH-060  Anti-automation throttling
│  ├─ REQ-AUTH-070  Password credential storage
│  ├─ REQ-AUTH-080  Subscriber registration
│  ├─ REQ-AUTH-090  Email verification and health-data gate
│  ├─ REQ-AUTH-100  Subscriber password authentication
│  ├─ REQ-AUTH-110  MFA enrolment, recovery codes, disablement
│  ├─ REQ-AUTH-120  MFA challenge and recovery-code redemption
│  ├─ REQ-AUTH-130  Password reset
│  ├─ REQ-AUTH-140  Privileged provisioning and first passkey
│  ├─ REQ-AUTH-150  Privileged minimums and passkey recovery
│  ├─ REQ-AUTH-160  Vetting record on privileged invitations  (2026-08-03)
│  ├─ REQ-AUTH-170  Privileged deprovisioning  (2026-08-03)
│  └─ REQ-AUTH-180  Email-address change  (2026-08-03)
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
│  ├─ REQ-PRIVACY-060  Response field minimization
│  ├─ REQ-PRIVACY-070  Health-data definition  (2026-08-03)
│  ├─ REQ-PRIVACY-080  Synchronous JSON export  (2026-08-03)
│  ├─ REQ-PRIVACY-090  Synchronous account deletion  (2026-08-03)
│  ├─ REQ-PRIVACY-100  Audit tombstoning  (2026-08-03)
│  └─ REQ-PRIVACY-110  Out-of-band deletion channel  (2026-08-03)
├─ Entitlement  (2026-08-03)
│  ├─ REQ-ENTITLE-010  Subscription entitlement gate
│  ├─ REQ-ENTITLE-020  View own subscription status
│  ├─ REQ-ENTITLE-030  Admin period grant, extension, revocation
│  └─ REQ-ENTITLE-040  Retention across lapse
├─ Plan library (admin)
│  ├─ REQ-PLAN-010  Exercise plan model
│  ├─ REQ-PLAN-020  Diet plan model
│  ├─ REQ-PLAN-030  Create and edit
│  ├─ REQ-PLAN-040  Citation management
│  ├─ REQ-PLAN-050  Publication gate
│  ├─ REQ-PLAN-060  Unpublication
│  ├─ REQ-PLAN-070  One-time verification operation  (2026-08-03)
│  └─ REQ-PLAN-080  Exercise catalog management  (2026-08-03)
├─ Catalog (subscriber)
│  ├─ REQ-CATALOG-010  Browse and view exercise plans
│  ├─ REQ-CATALOG-020  Browse and view diet plans
│  └─ REQ-CATALOG-030  Safe rendering
├─ Selection  (2026-08-03)
│  ├─ REQ-SELECT-010  Active exercise plan selection
│  ├─ REQ-SELECT-020  Active diet plan selection
│  └─ REQ-SELECT-030  Unpublication ends active selections
├─ Customization
│  ├─ REQ-CUSTOM-010  Copy-on-customize
│  ├─ REQ-CUSTOM-020  Persist, list, retrieve
│  └─ REQ-CUSTOM-030  Copy stability
├─ Progress
│  ├─ REQ-PROGRESS-010  Body weight logging
│  ├─ REQ-PROGRESS-020  Workout logging
│  ├─ REQ-PROGRESS-030  Validation and error reporting
│  ├─ REQ-PROGRESS-040  Body measurement logging  (2026-08-03)
│  ├─ REQ-PROGRESS-050  Unit system and display-only conversion  (2026-08-03)
│  └─ REQ-PROGRESS-060  Trend charts and paired tables  (2026-08-03)
├─ Food and nutrition  (2026-08-03)
│  ├─ REQ-FOOD-010  Nutrition dataset import, versioning, search
│  ├─ REQ-FOOD-020  Food log entry with attribution
│  ├─ REQ-FOOD-030  Daily intake versus targets
│  ├─ REQ-FOOD-040  AI-assisted estimation flow
│  └─ REQ-FOOD-050  In-boundary inference configuration
├─ Consultants
│  ├─ REQ-CONSULT-010  Engagement-scoped access
│  ├─ REQ-CONSULT-020  Engagement termination revokes access
│  ├─ REQ-CONSULT-030  Engagement lifecycle by admin action  (2026-08-03)
│  └─ REQ-CONSULT-040  Capabilities within an active engagement  (2026-08-03)
├─ Build and environments
│  ├─ REQ-PIPE-010  Dependency policy
│  └─ REQ-PIPE-020  No real health data in non-production
├─ Infrastructure  (2026-08-03)
│  ├─ REQ-INFRA-010  Environment accounts and pipeline identities
│  ├─ REQ-INFRA-020  Network tiering and encryption
│  ├─ REQ-INFRA-030  Secrets and signing-key management
│  ├─ REQ-INFRA-040  Audit retention and hash-chained archive
│  ├─ REQ-INFRA-050  Break-glass operational access
│  ├─ REQ-INFRA-060  Transactional email via in-account SES
│  └─ REQ-INFRA-070  Merge-blocking CI security gates
└─ Operations  (2026-08-03)
   └─ REQ-OPS-010  Incident-response runbook and readiness
```

Workstream issues were deliberately omitted; the prompt makes them optional and epic-to-leaf keeps the manifest reviewable. The groupings above are organizational only; groups marked (2026-08-03) were added with the new drafts.

## 6. Manifest

Effort is engineer-days; LOC is human-authored changed lines, excluding generated files and lockfiles. Model recommendations use verified identifiers from this session's environment (`claude-opus-5`, `claude-fable-5`). **No OpenAI model is recommended: no official OpenAI model list was retrieved in this session, so any specific GPT identifier would be `NOT VERIFIED`.** Rows 0–61 are the previously filed leaves and keep their numbers; rows 62–99 are the 2026-08-03 additions.

| # | ID | File | Effort | LOC | Depends on | Model |
|---|---|---|---|---|---|---|
| 0 | REQ-BUILD-010 | `005-REQ-BUILD-010.md` | 1–2 | 300–600 | — | Opus |
| 1 | REQ-PLATFORM-010 | `010-REQ-PLATFORM-010.md` | 0.5–1 | 150–350 | 0 | Opus |
| 2 | REQ-PLATFORM-020 | `020-REQ-PLATFORM-020.md` | 1–2 | 300–700 | 1 | Opus |
| 3 | REQ-PLATFORM-030 | `030-REQ-PLATFORM-030.md` | 1–2 | 300–650 | 1, 2 | Opus |
| 4 | REQ-PLATFORM-040 | `040-REQ-PLATFORM-040.md` | 0.5–1.5 | 150–400 | 0 | Opus |
| 5 | REQ-API-010 | `050-REQ-API-010.md` | 1–2 | 400–900 | 0 | Opus |
| 6 | REQ-API-020 | `060-REQ-API-020.md` | 1–1.5 | 250–600 | 5 | Opus |
| 7 | REQ-API-030 | `070-REQ-API-030.md` | 1–2 | 250–600 | 5 | Opus |
| 8 | REQ-API-040 | `080-REQ-API-040.md` | 0.5–1.5 | 200–450 | 5 | Opus |
| 9 | REQ-SESSION-010 | `130-REQ-SESSION-010.md` | 0.5–1.5 | 150–400 | 0 | Opus |
| 10 | REQ-SESSION-020 | `140-REQ-SESSION-020.md` | 0.5–1 | 100–250 | 9 | Opus |
| 11 | REQ-AUTHZ-010 | `090-REQ-AUTHZ-010.md` | 0.5–1.5 | 200–450 | 9 | Opus |
| 12 | REQ-AUTHZ-020 | `100-REQ-AUTHZ-020.md` | 1–2 | 300–700 | 11, 7 | Opus |
| 13 | REQ-AUTH-010 | `150-REQ-AUTH-010.md` | 0.5–1 | 100–300 | 6, 7 | Opus |
| 14 | REQ-AUTHZ-030 | `110-REQ-AUTHZ-030.md` | 0.5–1 | 150–350 | 11, 13, 6 | Opus |
| 15 | REQ-AUTHZ-040 | `120-REQ-AUTHZ-040.md` | 0.5–1 | 150–350 | 11, 8 | Opus |
| 16 | REQ-AUDIT-040 | `230-REQ-AUDIT-040.md` | 1–1.5 | 200–450 | 0 | Opus |
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
| 47 | REQ-PIPE-010 | `470-REQ-PIPE-010.md` | 0.5–1 | 50–200 | 0 | Opus |
| 48 | REQ-PIPE-020 | `480-REQ-PIPE-020.md` | 0.5–1.5 | 150–400 | 29, 30, 43, 44 | Opus |
| 49 | REQ-SESSION-030 | `141-REQ-SESSION-030.md` | 1–1.5 | 250–500 | 0, 9, 7 | Opus |
| 50 | REQ-SESSION-040 | `142-REQ-SESSION-040.md` | 1–1.5 | 250–500 | 49, 17, 8 | Opus |
| 51 | REQ-SESSION-050 | `143-REQ-SESSION-050.md` | 1.5–2 | 350–700 | 49, 4, 8 | Opus |
| 52 | REQ-AUTH-060 | `144-REQ-AUTH-060.md` | 1–2 | 300–650 | 0, 8, 22 | Opus |
| 53 | REQ-AUTH-070 | `191-REQ-AUTH-070.md` | 1–1.5 | 200–450 | 0, 7, 16 | Opus |
| 54 | REQ-AUTH-080 | `192-REQ-AUTH-080.md` | 1.5–2 | 400–850 | 53, 52, 5, 6, 13, 17 | Opus |
| 55 | REQ-AUTH-090 | `193-REQ-AUTH-090.md` | 1.5–2 | 350–700 | 54, 52, 5, 11, 17 | Opus |
| 56 | REQ-AUTH-100 | `194-REQ-AUTH-100.md` | 1.5–2 | 350–750 | 53, 13, 22, 52, 49, 51, 17 | Opus |
| 57 | REQ-AUTH-110 | `195-REQ-AUTH-110.md` | 2 | 500–900 | 56, 53, 52, 50, 12, 17, 3 | Opus |
| 58 | REQ-AUTH-120 | `196-REQ-AUTH-120.md` | 1.5–2 | 350–700 | 56, 57, 53, 52, 49, 17 | Opus |
| 59 | REQ-AUTH-130 | `197-REQ-AUTH-130.md` | 1.5–2 | 350–700 | 53, 55, 52, 50, 58, 17, 8 | Opus |
| 60 | REQ-AUTH-140 | `198-REQ-AUTH-140.md` | 2 | 500–950 | 13, 20, 21, 14, 52, 17, 19, 6 | Opus |
| 61 | REQ-AUTH-150 | `199-REQ-AUTH-150.md` | 1.5–2 | 300–650 | 60, 21, 20, 13, 14, 17 | Opus |
| 62 | REQ-AUTHZ-050 | `750-REQ-AUTHZ-050.md` | 1.5–2 | 600–1100 | 11, 49 | Fable |
| 63 | REQ-PRIVACY-070 | `670-REQ-PRIVACY-070.md` | 0.5–1 | 100–250 | 62 | Opus |
| 64 | REQ-AUTHZ-060 | `580-REQ-AUTHZ-060.md` | 1–1.5 | 250–500 | 62, 13, 63 | Opus |
| 65 | REQ-ENTITLE-010 | `490-REQ-ENTITLE-010.md` | 1.5–2 | 400–800 | 11, 49, 62 | Opus |
| 66 | REQ-ENTITLE-020 | `500-REQ-ENTITLE-020.md` | 0.5–1 | 100–250 | 65 | Opus |
| 67 | REQ-ENTITLE-030 | `510-REQ-ENTITLE-030.md` | 1–1.5 | 300–600 | 65, 17, 62 | Opus |
| 68 | REQ-ENTITLE-040 | `520-REQ-ENTITLE-040.md` | 0.5–1 | 150–350 | 65 | Opus |
| 69 | REQ-SELECT-010 | `530-REQ-SELECT-010.md` | 1–1.5 | 300–600 | 35, 26, 55, 65 | Opus |
| 70 | REQ-SELECT-020 | `540-REQ-SELECT-020.md` | 0.5–1 | 200–400 | 36, 26, 55, 65 | Opus |
| 71 | REQ-SELECT-030 | `550-REQ-SELECT-030.md` | 1–1.5 | 250–500 | 40, 69, 70 | Opus |
| 72 | REQ-PLAN-070 | `560-REQ-PLAN-070.md` | 0.5–1 | 150–350 | 31, 33, 19 | Opus |
| 73 | REQ-PLAN-080 | `570-REQ-PLAN-080.md` | 1–1.5 | 300–600 | 14, 19 | Opus |
| 74 | REQ-PROGRESS-050 | `600-REQ-PROGRESS-050.md` | 0.5–1 | 200–450 | 54, 5 | Opus |
| 75 | REQ-PROGRESS-040 | `590-REQ-PROGRESS-040.md` | 1–1.5 | 350–700 | 43, 42, 74 | Opus |
| 76 | REQ-PROGRESS-060 | `610-REQ-PROGRESS-060.md` | 1.5–2 | 600–1200 | 43, 44, 75, 2, 3 | Fable |
| 77 | REQ-FOOD-010 | `620-REQ-FOOD-010.md` | 1–1.5 | 300–600 | 0, 5 | Opus |
| 78 | REQ-FOOD-020 | `630-REQ-FOOD-020.md` | 1.5–2 | 400–800 | 77, 42, 26 | Opus |
| 79 | REQ-FOOD-030 | `640-REQ-FOOD-030.md` | 1–1.5 | 250–500 | 78, 70 | Opus |
| 80 | REQ-INFRA-010 | `790-REQ-INFRA-010.md` | 1.5–2 | 500–1000 | 0 | Opus |
| 81 | REQ-INFRA-020 | `800-REQ-INFRA-020.md` | 1.5–2 | 500–1000 | 80 | Opus |
| 82 | REQ-INFRA-030 | `810-REQ-INFRA-030.md` | 1–1.5 | 250–550 | 80, 81 | Opus |
| 83 | REQ-INFRA-040 | `820-REQ-INFRA-040.md` | 1.5–2 | 450–900 | 17, 80, 82 | Opus |
| 84 | REQ-INFRA-050 | `830-REQ-INFRA-050.md` | 1–2 | 300–700 | 80, 81 | Opus |
| 85 | REQ-INFRA-060 | `840-REQ-INFRA-060.md` | 1–2 | 400–800 | 80, 81 | Opus |
| 86 | REQ-INFRA-070 | `850-REQ-INFRA-070.md` | 1–1.5 | 300–600 | 0, 80 | Opus |
| 87 | REQ-FOOD-050 | `660-REQ-FOOD-050.md` | 1–1.5 | 200–450 | 80 | Opus |
| 88 | REQ-FOOD-040 | `650-REQ-FOOD-040.md` | 1.5–2 | 500–900 | 78, 87, 5, 26 | Fable |
| 89 | REQ-API-050 | `780-REQ-API-050.md` | 1–1.5 | 300–600 | 5, 8, 11, 52 | Opus |
| 90 | REQ-AUTH-160 | `720-REQ-AUTH-160.md` | 0.5–1 | 150–350 | 60 | Opus |
| 91 | REQ-AUTH-170 | `730-REQ-AUTH-170.md` | 1–1.5 | 350–700 | 60, 61, 46 | Opus |
| 92 | REQ-PRIVACY-100 | `700-REQ-PRIVACY-100.md` | 1–1.5 | 200–450 | 17, 82 | Opus |
| 93 | REQ-PRIVACY-080 | `680-REQ-PRIVACY-080.md` | 1–1.5 | 300–600 | 12, 49, 18, 89, 63 | Opus |
| 94 | REQ-PRIVACY-090 | `690-REQ-PRIVACY-090.md` | 1.5–2 | 500–900 | 92, 91, 83, 46, 50, 63 | Opus |
| 95 | REQ-PRIVACY-110 | `710-REQ-PRIVACY-110.md` | 1.5–2 | 500–900 | 94, 85, 55 | Opus |
| 96 | REQ-AUTH-180 | `740-REQ-AUTH-180.md` | 1–1.5 | 400–750 | 55, 59, 95 | Opus |
| 97 | REQ-CONSULT-030 | `760-REQ-CONSULT-030.md` | 1–1.5 | 350–650 | 62, 17, 5, 6 | Opus |
| 98 | REQ-CONSULT-040 | `770-REQ-CONSULT-040.md` | 1–2 | 400–800 | 45, 97, 38, 62, 18, 5, 6 | Opus |
| 99 | REQ-OPS-010 | `860-REQ-OPS-010.md` | 1–2 | 400–900 | 84 | Opus |

**Totals**: 100 leaves; 99–161.5 engineer-days; roughly 28,350–59,700 human-authored lines (rows 0–61: 59.5–101 days, 15,900–34,650 lines; rows 62–99: 39.5–60.5 days, 12,450–25,050 lines). Every leaf is within the 0.5–2 day and 1,500-line bounds.

### Model assignment rationale

- **`claude-opus-5` (93 of 100 issues; counts recomputed from the manifest 2026-08-03)** — the default here because the specification is security-dense: most issues are bounded implementations of an authorization, validation, audit, or privacy control where a plausible-looking implementation that passes the functional test still fails the security requirement (fail-closed behavior, response uniformity, structural audit dependency, copy-versus-reference semantics). These reward careful bounded reasoning rather than autonomy. All 35 Opus-assigned 2026-08-03 additions fit the same profile.
- **`claude-fable-5` (7 issues: REQ-PLAN-010, REQ-PLAN-020, REQ-CATALOG-010, REQ-CATALOG-020, and from 2026-08-03 REQ-AUTHZ-050, REQ-PROGRESS-060, REQ-FOOD-040)** — foundational data models and their paired read surfaces, which span persistence, API, and client, and whose main risk is divergence between the exercise and diet halves and between what is modelled and what several downstream issues assume. Cross-boundary consistency over a longer horizon is the distinguishing need. REQ-AUTHZ-050 is Fable-assigned because the central typed policy module touches every protected route and encodes the capability map nearly every downstream issue assumes, so long-horizon cross-cutting consistency dominates over bounded per-endpoint reasoning. REQ-PROGRESS-060 is Fable-assigned because it spans persistence queries, API response shaping, and the DESIGN.md chart-and-paired-table contract across four data families in both themes, where chart-versus-table divergence is the principal failure mode. REQ-FOOD-040 is Fable-assigned because a single flow crosses the client preview, the API's consent and verification gates, the SEC-INPUT-7 photo pipeline, and the inference boundary, and the transient-photo and estimate-labelling guarantees must hold identically at every layer.
- **OpenAI GPT — `NOT VERIFIED`.** No OpenAI model identifier is recommended, because no official OpenAI model list was consulted in this session and `REQUIREMENT_TEMPLATE.md`'s prohibition on unverified identifiers applies equally here. If OpenAI tooling is used, the exact model identifier must be verified against OpenAI's documentation at execution time and recorded in the issue.

## 7. Topological creation order

The manifest's `#` column is the creation order; the dependency graph is acyclic, and for rows 62–99 every `Depends on` entry precedes its dependent (re-verified 2026-08-03). Three legacy edges among the already-filed rows run forward of the `#` order — row 37 (REQ-CUSTOM-030) depends on rows 38 and 39, and row 38 (REQ-CUSTOM-010) depends on row 41 — an ordering artifact of the original pass that is moot for creation, because rows 0–61 are already live and keep the numbers their issues were filed under. Create the epic first, then leaves in `#` order, each with `--parent` set to the epic's number. Rows 0–61 are already created; rows 62–99 are created after this branch merges (section 8). File-name prefixes reflect grouping rather than dependency order; the manifest's `#` column is the authority.

REQ-BUILD-010 is numbered 0 rather than renumbering the original 48, whose numbers are already referenced across the manifest's `Depends on` column. It is the root: every leaf depends on it transitively, and the six leaves that previously had no in-plan predecessor — 1, 4, 5, 9, 16, and 47 — now depend on it directly.

Rows 62–99 (added 2026-08-03) extend the same order rather than replacing it: original rows keep their numbers, the extension begins at 62, and every `Depends on` entry of each new row — whether an original row 0–61 or another new row — carries a lower number than the row that names it. The extended graph was re-verified row by row before appending and remains acyclic; no cycle exists among the new rows or between new and original rows.

## 8. Creation commands

Run from the repository root. All commands in the first block below have been executed: the epic is live as issue #8, and all 62 original leaves (manifest rows 0–61) are live as issues #9–#56 (rows 1–48), #60 (row 0, REQ-BUILD-010), and #66–#78 (rows 49–61), each linked as a sub-issue of #8.

```sh
# Epic
gh issue create --title "[REQ-EPIC-001] Implement the specified subscription fitness web application" --body-file ".planning/github-issues/000-REQ-EPIC-001.md"

# Leaves, in topological order (add --parent <epic-number> to each)
gh issue create --title "[REQ-BUILD-010] Workspace scaffolding and toolchain baseline" --body-file ".planning/github-issues/005-REQ-BUILD-010.md"
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
gh issue create --title "[REQ-SESSION-030] Server-side session records and per-request resolution" --body-file ".planning/github-issues/141-REQ-SESSION-030.md"
gh issue create --title "[REQ-SESSION-040] Logout and session revocation on credential or authorization change" --body-file ".planning/github-issues/142-REQ-SESSION-040.md"
gh issue create --title "[REQ-SESSION-050] Cookie session transport and cross-site request forgery protection" --body-file ".planning/github-issues/143-REQ-SESSION-050.md"
gh issue create --title "[REQ-AUTH-060] Anti-automation throttling on authentication and recovery paths" --body-file ".planning/github-issues/144-REQ-AUTH-060.md"
gh issue create --title "[REQ-AUTH-070] Password credential storage with Argon2id" --body-file ".planning/github-issues/191-REQ-AUTH-070.md"
gh issue create --title "[REQ-AUTH-080] Subscriber registration with email and password" --body-file ".planning/github-issues/192-REQ-AUTH-080.md"
gh issue create --title "[REQ-AUTH-090] Email verification and the health-data write gate" --body-file ".planning/github-issues/193-REQ-AUTH-090.md"
gh issue create --title "[REQ-AUTH-100] Subscriber password authentication" --body-file ".planning/github-issues/194-REQ-AUTH-100.md"
gh issue create --title "[REQ-AUTH-110] MFA enrolment, recovery codes, and disablement" --body-file ".planning/github-issues/195-REQ-AUTH-110.md"
gh issue create --title "[REQ-AUTH-120] MFA challenge and recovery-code redemption" --body-file ".planning/github-issues/196-REQ-AUTH-120.md"
gh issue create --title "[REQ-AUTH-130] Password reset" --body-file ".planning/github-issues/197-REQ-AUTH-130.md"
gh issue create --title "[REQ-AUTH-140] Privileged provisioning by invitation and first passkey enrolment" --body-file ".planning/github-issues/198-REQ-AUTH-140.md"
gh issue create --title "[REQ-AUTH-150] Privileged account minimums and passkey recovery" --body-file ".planning/github-issues/199-REQ-AUTH-150.md"
```

After creation, every `{{ISSUE_URL:<ID>}}` placeholder in `000-REQ-EPIC-001.md` was replaced with the created issue URL and the epic body was updated; the file now contains no placeholders.

### 2026-08-03 additions — TO BE EXECUTED AFTER THIS BRANCH MERGES

The 38 commands below have **not** been run. They are to be executed from the repository root after this branch merges, in manifest order (rows 62–99), each with `--parent 8` so the new leaves join the epic's existing sub-issue set. The draft files are the source of truth; live bodies sync from them.

```sh
# Leaves 62–99, in topological order (add --parent 8 to each)
gh issue create --title "[REQ-AUTHZ-050] Central typed authorization policy module" --body-file ".planning/github-issues/750-REQ-AUTHZ-050.md"
gh issue create --title "[REQ-PRIVACY-070] Health-data definition binding the consent, verification, audit, and admin gates" --body-file ".planning/github-issues/670-REQ-PRIVACY-070.md"
gh issue create --title "[REQ-AUTHZ-060] Admin health-data prohibition and administrative account views" --body-file ".planning/github-issues/580-REQ-AUTHZ-060.md"
gh issue create --title "[REQ-ENTITLE-010] Subscription entitlement gate on plan, customization, and progress access" --body-file ".planning/github-issues/490-REQ-ENTITLE-010.md"
gh issue create --title "[REQ-ENTITLE-020] View own subscription status" --body-file ".planning/github-issues/500-REQ-ENTITLE-020.md"
gh issue create --title "[REQ-ENTITLE-030] Admin subscription-period grant, extension, and revocation" --body-file ".planning/github-issues/510-REQ-ENTITLE-030.md"
gh issue create --title "[REQ-ENTITLE-040] Record retention across subscription lapse" --body-file ".planning/github-issues/520-REQ-ENTITLE-040.md"
gh issue create --title "[REQ-SELECT-010] Active exercise plan selection" --body-file ".planning/github-issues/530-REQ-SELECT-010.md"
gh issue create --title "[REQ-SELECT-020] Active diet plan selection" --body-file ".planning/github-issues/540-REQ-SELECT-020.md"
gh issue create --title "[REQ-SELECT-030] Unpublication ends active selections" --body-file ".planning/github-issues/550-REQ-SELECT-030.md"
gh issue create --title "[REQ-PLAN-070] One-time plan verification operation" --body-file ".planning/github-issues/560-REQ-PLAN-070.md"
gh issue create --title "[REQ-PLAN-080] Admin exercise catalog management" --body-file ".planning/github-issues/570-REQ-PLAN-080.md"
gh issue create --title "[REQ-PROGRESS-050] Per-account unit system and display-only conversion" --body-file ".planning/github-issues/600-REQ-PROGRESS-050.md"
gh issue create --title "[REQ-PROGRESS-040] Body measurement entry logging" --body-file ".planning/github-issues/590-REQ-PROGRESS-040.md"
gh issue create --title "[REQ-PROGRESS-060] Progress history display with trend charts and paired tables" --body-file ".planning/github-issues/610-REQ-PROGRESS-060.md"
gh issue create --title "[REQ-FOOD-010] Bundled nutrition dataset import, versioning, and search" --body-file ".planning/github-issues/620-REQ-FOOD-010.md"
gh issue create --title "[REQ-FOOD-020] Food log entry with calorie and macronutrient attribution" --body-file ".planning/github-issues/630-REQ-FOOD-020.md"
gh issue create --title "[REQ-FOOD-030] Daily intake versus selected diet plan targets" --body-file ".planning/github-issues/640-REQ-FOOD-030.md"
gh issue create --title "[REQ-INFRA-010] Environment accounts, pipeline identities, and deployment flow" --body-file ".planning/github-issues/790-REQ-INFRA-010.md"
gh issue create --title "[REQ-INFRA-020] Network tiering and encryption at rest and in transit" --body-file ".planning/github-issues/800-REQ-INFRA-020.md"
gh issue create --title "[REQ-INFRA-030] Secrets and signing-key management" --body-file ".planning/github-issues/810-REQ-INFRA-030.md"
gh issue create --title "[REQ-INFRA-040] Audit retention, append-only privileges, and the hash-chained archive" --body-file ".planning/github-issues/820-REQ-INFRA-040.md"
gh issue create --title "[REQ-INFRA-050] Break-glass operational access" --body-file ".planning/github-issues/830-REQ-INFRA-050.md"
gh issue create --title "[REQ-INFRA-060] Transactional email delivery via in-account SES" --body-file ".planning/github-issues/840-REQ-INFRA-060.md"
gh issue create --title "[REQ-INFRA-070] Merge-blocking CI security gates" --body-file ".planning/github-issues/850-REQ-INFRA-070.md"
gh issue create --title "[REQ-FOOD-050] In-boundary inference service configuration" --body-file ".planning/github-issues/660-REQ-FOOD-050.md"
gh issue create --title "[REQ-FOOD-040] AI-assisted nutrition estimation flow" --body-file ".planning/github-issues/650-REQ-FOOD-040.md"
gh issue create --title "[REQ-API-050] Rate limits, body-size limits, and time-bounded request handling" --body-file ".planning/github-issues/780-REQ-API-050.md"
gh issue create --title "[REQ-AUTH-160] Vetting record required on privileged invitations" --body-file ".planning/github-issues/720-REQ-AUTH-160.md"
gh issue create --title "[REQ-AUTH-170] Privileged deprovisioning" --body-file ".planning/github-issues/730-REQ-AUTH-170.md"
gh issue create --title "[REQ-PRIVACY-100] Audit tombstoning on account deletion" --body-file ".planning/github-issues/700-REQ-PRIVACY-100.md"
gh issue create --title "[REQ-PRIVACY-080] Synchronous JSON data export" --body-file ".planning/github-issues/680-REQ-PRIVACY-080.md"
gh issue create --title "[REQ-PRIVACY-090] Synchronous account deletion with deletion-ledger write" --body-file ".planning/github-issues/690-REQ-PRIVACY-090.md"
gh issue create --title "[REQ-PRIVACY-110] Out-of-band deletion channel" --body-file ".planning/github-issues/710-REQ-PRIVACY-110.md"
gh issue create --title "[REQ-AUTH-180] Email-address change" --body-file ".planning/github-issues/740-REQ-AUTH-180.md"
gh issue create --title "[REQ-CONSULT-030] Consultant engagement lifecycle by admin action" --body-file ".planning/github-issues/760-REQ-CONSULT-030.md"
gh issue create --title "[REQ-CONSULT-040] Consultant capabilities within an active engagement" --body-file ".planning/github-issues/770-REQ-CONSULT-040.md"
gh issue create --title "[REQ-OPS-010] Incident-response runbook and readiness" --body-file ".planning/github-issues/860-REQ-OPS-010.md"
```

When the 2026-08-03 batch is filed, the epic body is updated the same way with the new issue URLs.

## 9. Notes on drafting decisions

- **The consolidated refresh pass promised throughout this plan was executed 2026-08-03.** All 62 previously filed drafts were refreshed against the amended specifications in the same pass that produced the 38 new drafts: `Alerting` fields now route threshold alerts to the security lead as SEC-OPS-2 detection inputs per SQ-11 (or record `N/A` with a reason where a requirement genuinely has no alert condition); `Regulatory` fields name the SQ-1 regime set; and ASVS/AISVS/NIST mappings remain deferred to the pre-launch assessment per SQ-10. Live GitHub bodies sync from the drafts after this branch merges.
- **`OWASP ASVS 5.0.0` and `NIST SP 800-53 Rev. 5` mappings are `TO BE DECIDED` in every issue.** `REQUIREMENT_TEMPLATE.md` states that only mappings verified against the cited version may be included and that control identifiers must not be guessed. Neither catalog was retrieved in this session; `SECURITY.md` SQ-10 has since fixed the target (Level 3) and the verifier (independent pre-launch assessment), and mappings stay `TO BE DECIDED` until verified there — the 2026-08-03 refresh confirmed this position across all 100 drafts and extends it to `OWASP AISVS 1.0` for the AI-component issues. CWE identifiers and the `SECURITY.md` threat-model identifiers are cited where confident.
- **`Regulatory` fields were `TO BE DECIDED` wherever health data is involved**, because PQ-21 originally recorded the jurisdiction question as unresolved by the specification itself. Superseded at the 2026-08-03 refresh: with SQ-1 resolved, every issue touching health or personal data now names the regime set — GDPR/UK GDPR for EU/UK data subjects; CCPA/CPRA, Washington My Health My Data, and the FTC Health Breach Notification Rule for US users; HIPAA not applicable — while statute-section precision stays per-issue `TO BE DECIDED` except where a specification document states a section (GDPR Art. 12(6) in FR-9.11; the GDPR 72-hour and FTC HBNR 60-day clocks in SEC-OPS-2).
- **`Alerting` fields were `TO BE DECIDED` throughout the original pass.** Written before the alerting model existed; the thresholds are fixed (PQ-17 RESOLVED) and destinations/process too (SQ-11 RESOLVED: alerts route to the security lead as SEC-OPS-2 detection inputs). The per-issue `Alerting` fields were updated accordingly at the 2026-08-03 consolidated refresh.
- **No issue resolves an open question.** Where an issue had to proceed alongside one — REQ-PROGRESS-020 logging against an accessible plan while PQ-12 was open, REQ-PLAN-030 leaving verification untouched on edit while PQ-13 was open — the position was labelled provisional in that issue's Open Decisions and named here. Both provisional positions were confirmed by the later resolutions (PQ-12, PQ-13; the FR-5.4 catalog and FR-4.8) and folded into the affected drafts at the 2026-08-03 refresh. One deliberate surfacing remains: REQ-PROGRESS-050 records that no specification document states the initial unit preference for a new account, and asks rather than assumes.
- **No specification file was modified by the drafting passes.** The 2026-08-03 amendments were made in the specification documents themselves, upstream of this plan; the plan and the drafts remain downstream consumers of `REQUIREMENTS.md`, `ARCHITECTURE.md`, `SECURITY.md`, and `DESIGN.md`.
