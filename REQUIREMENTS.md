# Required Requirement Inputs

- **Project purpose:** A fitness web application that delivers science-based, admin-curated exercise and diet plans, lets users customize those plans, and tracks their progress over time. Access requires a paid subscription.
- **Primary users / actors:** Subscriber (registered end user who follows plans and logs progress); Fitness consultant / helper (available to subscribers as a paid option); Admin (authors, cites, verifies, and publishes plan content)
- **Core workflows:** Register and authenticate; subscribe; browse and select an exercise or diet plan; customize a plan into a personal copy; log workouts, food intake, weight, and measurements; view progress over time; admin authors and verifies plans
- **Business objects / data entities:** User account; subscription; exercise plan (with cited sources); diet plan (meals, calorie/macro targets, cited sources); user plan copy (customized); workout log entry; food log entry; body weight entry; body measurement entry
- **External integrations:** None — the system is self-contained
- **Authentication / roles:** Account required. Subscribers authenticate with standard consumer authentication: email + password minimum, with optional user-enabled MFA. Admins and fitness consultants MUST authenticate with a passkey. Roles: subscriber, consultant, admin
- **Regulatory or privacy constraints:** US consumer-health laws apply to user health data — the FTC Health Breach Notification Rule, state consumer-health laws (e.g. Washington My Health My Data), and CCPA/CPRA; HIPAA is not applicable because no covered-entity or business-associate relationship exists (OQ-3 RESOLVED; SECURITY.md SQ-1); GDPR and UK GDPR govern EU/UK data subjects, and GDPR-grade data-subject rights are granted to all users; a medical disclaimer stating the content is not medical advice must be shown

# Functional Requirements

### Delivery Channel

- **FR-1.1** The system MUST be delivered as a web application accessible through current desktop and mobile web browsers.
- **FR-1.2** The system MUST present a usable, responsive layout at both desktop and mobile viewport sizes, with no loss of function or content on mobile.

### Accounts and Authentication

- **FR-2.1** The system MUST require a user account for all plan, customization, and progress functionality, and MUST deny unauthenticated access to those functions.
- **FR-2.2** The system MUST allow a user to register an account with an email address and a password.
- **FR-2.3** The system MUST allow a registered user to authenticate with email and password, and MUST reject invalid credentials without revealing which factor was wrong.
- **FR-2.4** The system MUST allow a user to end their session (log out).
- **FR-2.5** The system MUST allow a user to enable and disable multi-factor authentication on their own account. MFA MUST be optional, not required.
- **FR-2.6** When MFA is enabled for an account, the system MUST require a successful second factor before granting an authenticated session.
- **FR-2.7** The system MUST assign every account exactly one of the roles `subscriber`, `consultant`, or `admin`.
- **FR-2.8** The system MUST require passkey authentication for accounts with the `admin` or `consultant` role, and MUST NOT allow those accounts to authenticate with a password alone.
- **FR-2.9** The system MUST allow an `admin` or `consultant` account to register a passkey and to register a replacement passkey.
- **FR-2.10** The system MUST create `admin` and `consultant` accounts only through an invitation issued by an existing `admin`, MUST allow the invited person to register their first passkey through that invitation, and MUST NOT allow a role to be selected or changed by the account holder. The first `admin` account is created by a one-time out-of-band provisioning step that is unavailable once an `admin` exists. *(SECURITY.md SEC-AUTHN-9; threats TM-S-4, TM-E-1.)*
- **FR-2.11** The system MUST require a user to verify control of the email address they registered, and MUST refuse to record health data for an account whose address is not yet verified. An unverified account MAY otherwise exist and authenticate. *(SECURITY.md SEC-AUTHN-8; threat TM-S-3.)*
- **FR-2.12** The system MUST allow a subscriber to reset a forgotten password using a single-use, time-limited token sent to their verified email address. Completing a reset MUST NOT satisfy, disable, or bypass an enabled second factor, and MUST terminate all of that account's existing sessions. *(SECURITY.md SEC-AUTHN-10, SEC-AUTHN-12; threat TM-S-2.)*
- **FR-2.13** The system MUST issue single-use recovery codes when a subscriber enables MFA, MUST present them exactly once, and MUST accept one in place of the second factor. The system MUST NOT provide any other route to bypass an enabled second factor, and no `admin` may clear or reset a subscriber's MFA. *(SECURITY.md SEC-AUTHN-10, SEC-AUTHN-11; threat TM-S-2.)*
- **FR-2.14** The system MUST state, before a subscriber enables MFA, that losing every factor and every recovery code makes the account permanently unrecoverable. *(This is a deliberate trade of recoverability for health-data protection; its tension with FR-9.3 and FR-9.5 is recorded in OQ-17.)*
- **FR-2.15** The system MUST require every `admin` and `consultant` account to have at least two registered passkeys, MUST maintain at least two `admin` accounts, and MUST recover a privileged account that has lost passkey access only by a fresh invitation from another `admin` under FR-2.10. No password or email-possession path may restore privileged access. *(SECURITY.md SEC-AUTHN-2, SEC-AUTHN-9, SEC-AUTHN-10; threats TM-S-2, TM-S-4.)*

### Subscription and Access Control

- **FR-3.1** The system MUST require an active subscription for a subscriber to access exercise plans, diet plans, customization, and progress tracking.
- **FR-3.2** The system MUST deny plan and progress-tracking access to authenticated users without an active subscription, and MUST tell them a subscription is required.
- **FR-3.3** The system MUST allow a user to view their current subscription status.
- **FR-3.4** The system MUST retain a user's existing progress records and plan customizations when their subscription lapses, and MUST restore access to them when the subscription becomes active again.
- **FR-3.5** The system MUST allow an `admin` to grant, extend, and revoke a subscription period on a subscriber account, and MUST record an audit entry for each such action capturing the acting admin, the affected account, the action, the period bounds, and the time. *(Resolves OQ-1; SECURITY.md SEC-AUTHZ-8, threat TM-E-4.)*
- **FR-3.6** The system MUST treat a subscription as active exactly when the current time falls within a granted period on the account, and MUST NOT provide any other mechanism that activates a subscription. Payment collection occurs out of band in v1 (OQ-1, OQ-18).

### Plan Library and Content Verification

- **FR-4.1** The system MUST provide a library of exercise plans and diet plans authored by admins.
- **FR-4.2** The system MUST NOT allow subscribers to author plans, submit plans for publication, or share plans with other users.
- **FR-4.3** The system MUST allow an admin to create, edit, publish, and unpublish plans.
- **FR-4.4** The system MUST require every plan to carry at least one citation to a peer-reviewed source before it can be published, and MUST block publication of a plan with no citation.
- **FR-4.5** The system MUST require explicit admin verification of a plan before publication, and MUST record which admin verified it and when.
- **FR-4.6** The system MUST display a plan's citations to the user when the plan is viewed.
- **FR-4.7** The system MUST show only published plans to subscribers.

### Exercise Plans

- **FR-5.1** The system MUST allow a subscriber to browse published exercise plans and view a plan's full contents, including its exercises and prescribed sets and repetitions.
- **FR-5.2** The system MUST allow a subscriber to select an exercise plan to follow.
- **FR-5.3** The system MUST maintain at most one active exercise plan selection per subscriber, where the selection names either a published exercise plan or one of the subscriber's own customized exercise plan copies; selecting another MUST replace the current selection, and neither selection nor replacement may alter any logged history. *(Resolves OQ-6.)*

### Diet Plans

- **FR-6.1** The system MUST allow a subscriber to browse published diet plans and view a plan's full contents.
- **FR-6.2** Each diet plan MUST specify its meals and its daily calorie and macronutrient targets.
- **FR-6.3** The system MUST allow a subscriber to select a diet plan to follow.
- **FR-6.4** The system MUST maintain at most one active diet plan selection per subscriber, where the selection names either a published diet plan or one of the subscriber's own customized diet plan copies; selecting another MUST replace the current selection, and neither selection nor replacement may alter any logged history. The FR-8.5 target comparison uses the currently selected diet plan. *(Resolves OQ-6.)*

### Plan Customization

- **FR-7.1** The system MUST allow a subscriber to customize a published exercise or diet plan by editing its contents.
- **FR-7.2** The system MUST save a customization as a private copy owned by that subscriber, and MUST NOT modify the published plan.
- **FR-7.3** The system MUST persist a subscriber's customized plans across sessions and MUST make them retrievable by that subscriber.
- **FR-7.4** The system MUST NOT allow a subscriber to view or modify another subscriber's customized plans.
- **FR-7.5** The system MUST preserve an existing customized copy unchanged when the published plan it was derived from is later edited or unpublished.

### Progress Tracking

- **FR-8.1** The system MUST allow a subscriber to log a body weight entry with a date.
- **FR-8.2** The system MUST allow a subscriber to log body measurement entries with a date. The measurement fields are waist, chest, hips, upper arm, and thigh — each a length in the account's unit system (FR-8.10) — and body-fat percentage, which is unitless and MUST be within 0–100. *(OQ-4 RESOLVED.)*
- **FR-8.3** The system MUST allow a subscriber to record completion of a workout from their plan, including sets, repetitions, and weight used per exercise.
- **FR-8.4** The system MUST allow a subscriber to log food intake and MUST attribute calories and macronutrients to each logged entry.
- **FR-8.5** The system MUST display logged calories and macronutrients for a given day against the targets of the subscriber's selected diet plan.
- **FR-8.6** The system MUST display a subscriber's logged history over time for body weight, body measurements, and workout performance.
- **FR-8.7** The system MUST allow a subscriber to edit and delete their own log entries.
- **FR-8.8** The system MUST allow a subscriber to log entries for a past date, not only the current date.
- **FR-8.9** The system MUST reject log entries with a non-numeric, negative, or absent required value and MUST report the specific invalid field to the user.
- **FR-8.10** The system MUST maintain a per-account unit-system preference — metric (centimetres, kilograms) or imperial (inches, pounds) — applied to body measurements, body weight, and workout load; MUST store every logged value together with the unit it was entered in; and MUST convert between unit systems only at display time, never by mutating stored values. *(Resolves OQ-4 together with DESIGN.md OQ-8's unit half.)*

### Data Rights, Privacy, and Safety

- **FR-9.1** The system MUST scope every plan copy and log entry to its owning subscriber, and MUST NOT expose one subscriber's data to another subscriber.
- **FR-9.2** The system MUST obtain the user's explicit consent to collect and process their health data before any health data is recorded.
- **FR-9.3** The system MUST allow a user to export all of their own account, plan, and progress data in a machine-readable form.
- **FR-9.4** The system MUST allow a user to request deletion of their account and all associated personal and health data, and MUST complete the deletion. The completion deadline is TO BE DECIDED.
- **FR-9.5** The system MUST allow a user to view and correct the personal data held about them.
- **FR-9.6** The system MUST display a disclaimer stating that the plans and content are not medical advice, and MUST require the user to acknowledge it before they first use a plan.
- **FR-9.7** The system MUST record an audit entry for each access to or modification of a user's health data, capturing the acting account, the action, and the time.
- **FR-9.8** The system MUST NOT transmit user health data to any external service.
- **FR-9.9** The system MUST allow a user to withdraw their previously given consent to health-data collection, and MUST NOT record new health data for that user while consent is withdrawn. Existing records remain subject to FR-9.3–FR-9.5. *(Threat-model-derived: SECURITY.md TM-P-2.)*

### Administration

- **FR-10.1** The system MUST restrict plan authoring, verification, publication, and unpublication to accounts with the `admin` role, and MUST deny those actions to subscribers.
- **FR-10.2** The system MUST record an audit entry for every admin plan lifecycle action — create, edit, verify, publish, unpublish — capturing the acting admin, the action, the affected plan, and the time. *(Threat-model-derived: SECURITY.md TM-R-1, TM-T-5.)*

### Fitness Consultants

- **FR-11.1** The system MUST offer subscribers access to a fitness consultant / helper as a paid option in addition to the base subscription.
- **FR-11.2** The system MUST NOT grant a consultant access to a subscriber's plans or health data unless that subscriber has an active paid consultant engagement with them.
- **FR-11.3** The system MUST allow a subscriber to end a consultant engagement, after which the system MUST revoke that consultant's access to the subscriber's data.
- **FR-11.4** The system MUST record an audit entry for each consultant access to a subscriber's health data, per FR-9.7.
- **FR-11.5** The system MUST create a consultant engagement only through an `admin` action that names the consultant and the subscriber, MUST treat the engagement as active from creation until it is ended under FR-11.3 or revoked by an `admin`, and MUST record an audit entry for each creation and administrative revocation. Payment for the option occurs out of band in v1 (OQ-1, OQ-18). *(Resolves the purchase half of OQ-13.)*

# Open Questions

- **OQ-1** RESOLVED (2026-08-01). Subscriptions are admin-granted periods: an `admin` grants, extends, or revokes a period (FR-3.5), a subscription is active exactly while the current time falls within a granted period (FR-3.6), and every change is audited. Payment collection is out of band in v1 — the system stays self-contained with no payments integration (SECURITY.md SEC-EXT-1). Self-serve purchase via a payment processor is deferred to OQ-18; the paid consultant option is recorded the same way (FR-11.5). FR-3.1–FR-3.4 are now fully testable.
- **OQ-2** RESOLVED (2026-08-01). No trial period, promotional tier, or free tier exists. Access requires a paid subscription; an `admin` MAY grant a time-boxed courtesy period through the ordinary FR-3.5 mechanism, which needs no additional machinery, states, or per-person trial tracking.
- **OQ-3** RESOLVED (SECURITY.md SQ-1, 2026-08-01). HIPAA is not applicable: the system has no covered-entity or business-associate relationship — it is direct-to-consumer, bills no insurance, and integrates with no provider — so no BAA is required and FR-9.7's audit obligation stands on its own. In scope for US users: the FTC Health Breach Notification Rule, state consumer-health laws (e.g. Washington My Health My Data), and CCPA/CPRA. GDPR/UK GDPR govern EU/UK data subjects, with GDPR-grade rights granted to all users. The breach-notification workflow is specified under SECURITY.md SQ-11; counsel review before launch is required.
- **OQ-4** RESOLVED (2026-08-01). Measurement fields are waist, chest, hips, upper arm, thigh, and body-fat percentage (FR-8.2). Units are configurable per account — metric (cm, kg) or imperial (in, lb) — covering measurements, body weight, and workout load; every record stores its value with its unit, and switching preference converts display only (FR-8.10). Body-fat percentage is unitless and bounded 0–100.
- **OQ-5** Food logging requires nutrition data, but no external nutrition database is in scope. Do subscribers enter calories and macros manually, or does the system ship its own food catalog, and who maintains it?
- **OQ-6** RESOLVED (2026-08-01). One active plan of each type: at most one active exercise plan selection and one active diet plan selection per subscriber (FR-5.3, FR-6.4). A selection names a published plan or the subscriber's own customized copy; selecting another replaces it, logged history is never altered, and FR-8.5's daily comparison reads the currently selected diet plan exactly as written. The FR-9.6 disclaimer gates the first plan use, including first selection.
- **OQ-7** What time period and granularity must progress history cover (per day, per week, all-time), and are charts required or is a list sufficient?
- **OQ-8** RESOLVED. Password policy follows NIST SP 800-63B — 8-character minimum with 15 or more encouraged, no composition rules, no forced rotation, known-breached passwords refused — and anti-automation is exponential backoff throttling rather than account lockout, so no third party can permanently lock an account (SECURITY.md SEC-AUTHN-6, threat TM-D-2). Recovery: password reset by single-use token to the verified address, never bypassing an enabled second factor and always terminating existing sessions (FR-2.12); single-use recovery codes issued at MFA enrolment, with no admin-assisted MFA reset (FR-2.13); permanent unrecoverability on total factor loss, disclosed up front (FR-2.14); and privileged passkey recovery only by fresh invitation, with a two-passkey and two-admin minimum (FR-2.15).
- **OQ-9** RESOLVED. Subscribers may enable TOTP (RFC 6238) or a passkey as an optional second factor. SMS and email codes are excluded — NIST SP 800-63B treats SMS as a restricted authenticator, and an email code reduces the second factor to the security of the email account.
- **OQ-10** Is admin verification of a plan a one-time gate, or must a plan be re-verified after each edit?
- **OQ-11** RESOLVED. The accessibility conformance target is WCAG 2.2 AA (DESIGN.md, Accessibility; ARCHITECTURE.md, Note on DESIGN.md, which records that DESIGN.md resolves this question).
- **OQ-12** What can a fitness consultant actually do — view a subscriber's plans and logs, edit their plans, message them, or something else? FR-11.x currently only bounds their access, not their capabilities.
- **OQ-13** PARTIALLY RESOLVED (2026-08-01): the paid consultant option is recorded by admin action (FR-11.5) with payment out of band, closing the purchase half alongside OQ-1. Still open: how consultants are onboarded and vetted, and whether they are platform staff or third parties.
- **OQ-14** Is offline use required for logging entries without a connection? Not selected, so assumed out of scope.
- **OQ-15** RESOLVED *(threat-model-derived: SECURITY.md TM-S-3)*. An account may be created and may authenticate before verification, but every health-data write is refused until control of the address is proven, and the address cannot be used in recovery until then (FR-2.11, SEC-AUTHN-8). The token is single-use, short-lived, and invalidated on use or replacement; resend is rate-limited and neither request nor response reveals whether an address is registered. Concrete token lifetime and resend interval remain with SECURITY.md SQ-3.
- **OQ-16** *(Threat-model-derived: SECURITY.md TM-T-5.)* Should plan verification (FR-4.5) require an admin other than the plan's author (dual control)? A single compromised admin account can currently author, verify, and publish harmful exercise or diet content alone. Note that FR-2.15 now guarantees at least two `admin` accounts exist, so dual control is operationally possible if chosen.
- **OQ-17** FR-2.14 accepts that a subscriber who loses every authentication factor and every recovery code is permanently locked out of their own health data. That protects the data against account takeover, but it also puts the export right (FR-9.3) and the correction right (FR-9.5) permanently out of that user's reach, and FR-9.4 deletion cannot be requested either. Whether an identity-proofing escalation is legally required — and what proofing standard would apply in a self-contained system with no proofing capability — depends on the governing regimes (SECURITY.md SQ-1, since RESOLVED — GDPR is the ceiling) and the incident process (SQ-11, still open). Recorded rather than resolved.
- **OQ-18** Deferred payments decision: when self-serve purchase is wanted, which payment processor is used, and what checkout, renewal, cancellation, refund, and dunning behavior applies? Introducing one is the system's first external integration and requires an explicit SECURITY.md SEC-EXT-1 scope change and a threat-model delta. The FR-3.5/FR-3.6 subscription-period record is the seam: processor events would create and revoke periods, and the entitlement model does not change.
