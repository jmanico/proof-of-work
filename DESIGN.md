# DESIGN.md

Normative product-design specification for **Proof of Work**. This document governs the browser client described in ARCHITECTURE.md. “MUST”, “SHOULD”, and “MUST NOT” are used intentionally. Functional, security, and authorization behavior remains governed by REQUIREMENTS.md, SECURITY.md, and ARCHITECTURE.md.

## Required Design Inputs

- **Product name:** Proof of Work.
- **Product promise:** Evidence-led plans, a clear record of effort, and progress the subscriber can inspect over time.
- **Primary audience:** Subscribers following admin-curated exercise and diet plans and logging workouts, food, weight, and measurements. Secondary audiences are engaged fitness consultants and admins who author, verify, and publish plans.
- **Brand personality:** Disciplined, evidence-led, calm, candid, and quietly encouraging. The product MUST feel credible without looking clinical and motivating without using hype, shame, or competitive pressure.
- **Platform targets:** Responsive web application for current desktop and mobile browsers (FR-1.1, FR-1.2). No native application is in scope.
- **Color modes:** Light and dark, plus a `System` preference. The initial preference is `System`.
- **Launch language and direction:** English and left-to-right. Components MUST use logical layout properties and tolerate 30% text expansion so future translation and right-to-left support do not require redesign. Additional languages and active RTL support are not part of v1.
- **Brand assets:** [`logo.svg`](./logo.svg), the Proof mark defined below. No earlier brand asset is retained.

## Design Principles

- **D-PRINCIPLE-1 — Show the evidence.** Plans expose their sources and review state at the point of use. The interface never describes content as “proven,” “guaranteed,” or “doctor approved.”
- **D-PRINCIPLE-2 — Make logging fast, not gamified.** Frequent entry paths minimize steps and remember no health values in browser storage (SEC-RENDER-4). No streak loss, leaderboard, confetti, guilt language, or artificial urgency is used.
- **D-PRINCIPLE-3 — Treat targets as context.** Calories, macros, weight, and workout targets are neutral reference points, not moral grades. Being above or below a target is not styled as an error unless an actual validation rule failed.
- **D-PRINCIPLE-4 — Put trust in plain sight.** Consent, consultant access, estimates, citations, subscription state, and destructive consequences are stated in direct language near the relevant action.
- **D-PRINCIPLE-5 — Prefer structure over decoration.** Hierarchy comes from type, spacing, rules, and restrained surfaces. Decorative gradients, glass effects, lifestyle photography, and excessive card nesting are not part of the system.

## Brand Identity

### Visual direction

The visual metaphor is a **field notebook for training**: warm paper-like canvas, dark ink, functional green, tabular data, and a single signal-lime accent. The result should feel considered and durable rather than medical, futuristic, or aggressive.

The initials “PoW” MUST NOT be used as the primary product label because of their strong cryptocurrency association. The full name is preferred. Coin, chain, mining, hexagon, lightning-bolt, and blockchain imagery MUST NOT be used.

### Logo

The Proof mark in `logo.svg` is a rounded ink tile with an evergreen boundary, containing a `P` and a signal-lime check. The `P` names the product; the check is the evidence that a piece of work was completed. Its simple, incomplete-record metaphor suggests ongoing work rather than a finish line.

The standard lockup consists of the mark followed by the live-text wordmark **Proof of Work**. The wordmark uses the interface sans-serif at weight 700 with `-0.02em` letter spacing. Live text is intentional: it stays crisp, introduces no font file, and avoids inaccessible text embedded in an image.

- **Primary lockup:** horizontal, mark on the left and wordmark on the right, separated by half the mark width.
- **Compact lockup:** mark alone only where the product name is already visible or programmatically available, such as a favicon or collapsed navigation.
- **Clear space:** at least 25% of the mark width on every side.
- **Minimum size:** 24×24 CSS pixels for the mark; 32×32 is preferred in application chrome. The horizontal lockup MUST NOT be displayed below 120 CSS pixels wide.
- **Backgrounds:** the self-contained ink tile is approved on light canvas, white surfaces, dark canvas, and dark surfaces. Its fixed `#2B7C55` boundary is 4.63:1 against light canvas and 3.59:1 against dark canvas; that boundary MUST NOT be removed.
- **Accessibility:** when the adjacent wordmark is visible, the SVG is decorative. When the mark stands alone as the product identifier, its accessible name is “Proof of Work.”
- **Incorrect use:** do not stretch, rotate, outline, crop, add a shadow or gradient, recolor individual paths, place it over a photograph, use the check without the `P`, or recreate the mark with an icon-font glyph.

## Color System

All interface color is assigned by semantic token. Components MUST NOT reference raw palette values. The light and dark values below are paired theme values, not separate component APIs.

| Semantic token | Light | Dark | Use |
|---|---:|---:|---|
| `canvas` | `#F5F4EE` | `#111612` | Page background |
| `surface` | `#FFFFFF` | `#182019` | Primary panels, forms, dialogs |
| `surface-subtle` | `#ECEFE9` | `#202A22` | Grouped controls, secondary regions |
| `ink` | `#17211B` | `#F3F5EF` | Headings and body text |
| `ink-muted` | `#566159` | `#B6C0B7` | Supporting copy and metadata |
| `border` | `#C9D0C8` | `#3A463D` | Non-essential separators |
| `control-border` | `#69746C` | `#839087` | Inputs and meaningful component boundaries |
| `brand` | `#1E6A48` | `#92D7AA` | Links, selected states, primary actions |
| `brand-strong` | `#15563A` | `#B7E9C7` | Hover/pressed emphasis |
| `brand-soft` | `#DCEEE3` | `#233D2D` | Selected and informational backgrounds |
| `signal` | `#D7F45B` | `#D7F45B` | Logo check, chart highlight, small emphasis |
| `error` | `#B42318` | `#FFB4AB` | Validation and destructive states |
| `error-soft` | `#FEE4E2` | `#482522` | Error callout background |
| `success` | `#18794E` | `#85D6A3` | Confirmed completion and saved state |
| `success-soft` | `#DFF5E8` | `#203D2A` | Success callout background |
| `warning` | `#8A4B0F` | `#F0C46C` | Caution and expiring state |
| `warning-soft` | `#FEF0C7` | `#46351D` | Warning callout background |
| `info` | `#2456A6` | `#AFC6FF` | Neutral system information |
| `info-soft` | `#E1EAFF` | `#24324C` | Information callout background |

Verified core contrast pairs include: `ink` on `canvas` at 15.01:1 light and 16.66:1 dark; `ink-muted` on `canvas` at 5.86:1 light and 9.77:1 dark; white on light `brand` at 6.54:1; dark `canvas` on dark `brand` at 10.91:1; and both `control-border` values at 4.87:1 or better against their adjacent surface. New combinations MUST be tested before use.

Color rules:

- `signal` is paired with `ink`, never white, and is never the only status cue.
- The primary filled button uses light `brand` with white text and dark `brand` with dark `canvas` text.
- Links are underlined. Statuses combine text with an icon or shape.
- Macro or plan-target variance uses neutral brand tones. `error` is reserved for invalid, failed, or destructive states.
- Dark mode changes color only. Spacing, hierarchy, border meaning, and elevation remain stable.

## Typography and Iconography

No web font is required. Proof of Work uses a local system stack for speed, privacy, language coverage, and predictable rendering:

```css
--font-sans: system-ui, -apple-system, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
--font-data: ui-monospace, "SFMono-Regular", Consolas, "Liberation Mono", monospace;
```

The data face is used for numeric values, sets × reps, units, dates in dense tables, and chart labels. It MUST NOT be used for paragraphs or form labels.

| Token | Fluid size / line height | Weight | Use |
|---|---|---:|---|
| `display` | `clamp(2rem, 4vw, 3rem)` / 1.05 | 750 or 700 fallback | Rare signed-out or empty-state statement |
| `h1` | `clamp(1.75rem, 3vw, 2.25rem)` / 1.15 | 700 | Page title |
| `h2` | `1.5rem` / 1.25 | 700 | Major section |
| `h3` | `1.25rem` / 1.3 | 650 or 700 fallback | Card or subsection |
| `body` | `1rem` / 1.5 | 400 | Default text |
| `small` | `0.875rem` / 1.45 | 400 or 600 | Helper text, table text, metadata |
| `micro` | `0.75rem` / 1.35 | 600 | Short badges only; never sustained reading |

Headings use sentence case. Body copy is never justified and long-form text is limited to 68ch. Numeric data uses lining tabular figures. Values and their units do not wrap onto separate lines.

Icons use a consistent 2px rounded-stroke style at 20px or 24px. They support labels; they do not replace them unless an accessible name and a conventional meaning are both present. The interface MUST use text labels rather than icons for destructive, privacy, consultant-access, publication, and verification actions.

## Imagery

Product chrome uses no photography and no decorative illustration. Charts, the logo, icons, and content diagrams supply the visual layer.

Exercise demonstrations MAY use admin-approved, neutral vector diagrams when a written instruction alone is insufficient. Demonstrations MUST:

- prioritize correct position and movement over physique;
- avoid idealized transformation imagery, sexualized framing, before/after comparisons, and gender-coded styling;
- include an equivalent text description and never carry instructions that are absent from the text;
- use the same restrained palette and remain understandable in monochrome.

Food photography is not used in plans or chrome in v1. A subscriber-supplied food photo is shown only as a local preview during the transient AI estimate flow and is labelled “Not saved” (FR-8.12, FR-8.13). It is removed from view when the estimate is confirmed, cancelled, or fails.

## Information Architecture and Navigation

Each role has a distinct workspace built from the same tokens and components. A persistent text label names the workspace so role context is never communicated by color alone.

| Role | Primary destinations | Workspace treatment |
|---|---|---|
| Subscriber | Home, Plans, Log, Progress, Account | “Proof of Work” product lockup; personal data and active-plan context |
| Consultant | Clients, Account | “Consultant workspace” label; every client view carries a persistent selected-client context bar |
| Admin | Plans, People, Access, Audit, Account | “Admin workspace” label; publishing and access-administration states are explicit |

“Log” opens a hub for workout, food, weight, and measurement entry. “Plans” contains Exercise, Diet, and My copies views. “Access” contains subscription periods and consultant engagements; payment collection is not represented because it is out of band in v1 (FR-3.5, FR-3.6, FR-11.5).

Desktop at 64rem and above uses a 15rem left navigation rail with the account and workspace switch context at the bottom. Below 64rem, the interface uses a compact top bar and a bottom navigation bar with at most five role-specific destinations. Account and secondary actions remain reachable from the top bar. Navigation labels MUST remain visible; icon-only primary navigation is not allowed.

On consultant pages, the selected-client context bar states the subscriber name, the active-engagement status, and the consultant’s scope: “View progress and selected plans; edit plan copies.” Changing clients is an explicit action. No client data remains visible after an engagement ends (FR-11.3).

Subscribers see consultant access in two places: a Home summary and Account → Consultant access. Both name the consultant, state the exact access granted by FR-11.6, and offer **End access**. A customized plan edited by a consultant shows a neutral “Edited by your consultant” provenance line with the time; it is not an endorsement badge.

## Layout, Spacing, and Responsive Behavior

- **Spacing scale:** 4, 8, 12, 16, 24, 32, 48, 64, and 96 CSS pixels. Components MUST use only these steps.
- **Grid:** 12 columns at 64rem and above, 8 columns from 48rem, and 4 columns below 48rem. Gutters are 24px desktop and 16px below 48rem.
- **Content widths:** 80rem maximum for application content; 68ch for prose; 40rem for a single-column form; 32rem for authentication.
- **Breakpoints:** 30rem, 48rem, 64rem, and 80rem. Breakpoints respond to content pressure, not named device models.
- **Density:** comfortable by default. Dense tables are allowed only in admin and consultant workspaces at desktop sizes. There is no user-facing compact-mode setting in v1.
- **Elevation:** use a border first. Menus and dialogs may use one shadow: `0 12px 32px rgb(23 33 27 / 16%)`. Cards on a page are not all raised.
- **Shape:** 8px radius for controls, 12px for panels, and a pill only for short status chips. Nested panels MUST NOT accumulate multiple rounded containers.
- **Targets:** all interactive targets are at least 44×44 CSS pixels.

At 320 CSS pixels and at 200% zoom, content reflows into one column with no page-level horizontal scrolling and no lost function. Multi-column forms stack. Action groups wrap with the primary action first in reading order. Data tables become labelled record cards below 48rem rather than forcing horizontal scrolling. Sticky regions MUST leave enough room for content and the on-screen keyboard.

## Core Components

### Actions

- **Primary button:** filled brand color, used for the action that advances the current task. A region has one primary action; a long page may have more than one region.
- **Secondary button:** bordered surface for a meaningful alternative.
- **Quiet button:** text and optional icon for low-risk utility actions.
- **Destructive button:** `error` treatment, explicit verb, and confirmation. “Delete account” is never styled as an ordinary primary action.
- **Busy state:** preserves button width, includes a text status for assistive technology, and prevents repeat submission. Disabling the only way to act without explaining why is not allowed.

Button labels use specific verbs: “Save weight”, “Select plan”, “Publish plan”, “End access”, not “Submit”, “Okay”, or “Yes”.

### Forms and validation

Every field has a persistent visible label. A placeholder is an example, never the label. Helper text precedes error text in the accessibility description. Required fields are marked with text, and forms explain once what the marker means.

Numeric health fields use the data face and an adjacent unit suffix tied to the account preference (FR-8.10). Dates use a visible calendar control plus an editable text value; launch display format uses an unambiguous month name such as “1 Aug 2026”.

Validation appears inline with an error icon and a specific correction. After failed submission, a summary at the top links to each invalid field and focus moves to the first invalid field (FR-8.9). Server-returned field errors override optimistic client assumptions. A generic system failure includes the correlation identifier from SEC-ERR-1 without internal detail.

### Status, feedback, and loading

Status chips contain a short text label and an icon or distinct outline. Approved vocabulary includes `Active`, `Inactive`, `Draft`, `Reviewed`, `Published`, `Unpublished`, `Estimate`, and `Email unverified`. Status labels use nouns or factual states, not praise.

Inline confirmation stays near the action. A toast MAY duplicate that message but MUST NOT be the only record of success or failure. Loading skeletons use `aria-busy`; shimmer is disabled under reduced motion. Empty states say what belongs in the region and provide the action that creates it. Permission, lapsed-subscription, withdrawn-consent, and ended-engagement states explain the exact reason and the available next step without exposing hidden data.

### Cards, lists, and tables

Cards group one object or one decision, not every section. Clickable cards contain a single primary link and never nest unrelated controls inside that link. Selected plan cards use a leading check, “Selected” text, and `brand-soft`; color alone is insufficient.

Tables use real table semantics, concise headers, row headers where appropriate, and tabular figures. At narrow widths each row becomes a record card with visible field labels. Sorting controls state direction in text available to assistive technology.

### Dialogs and drawers

Dialogs are reserved for blocking acknowledgement, destructive confirmation, or short focused tasks. They trap focus while open, return focus to the invoking control, close with Escape unless work would be lost, and provide a visible close action. At narrow widths, complex tasks use a full-page route; they do not become cramped modal sheets.

## Product Patterns

### Plans, citations, and review state

Every subscriber-facing plan detail starts with the plan title and a compact **Evidence reviewed** line. This is a factual review state, not a quality score. Subscriber copy reads “Evidence reviewed by Proof of Work” plus the review date; it does not expose the admin’s identity. Admin views show the verifier identity and timestamp required by FR-4.5.

Source references appear in two connected forms:

1. bracketed inline reference links such as `[1]` beside the relevant plan statement; and
2. an **Evidence** section at the end of the plan containing the full citation, peer-reviewed-source label, and a safe external link (FR-4.4–FR-4.6; SEC-RENDER-3).

The Evidence section is in the page reading order, not hidden in a tooltip. On long plans it may also be reached from an anchored “View evidence (n)” link near the title. The UI never fetches citation URLs server-side (SEC-EXT-2). Draft admin views show a publication checklist with separate Citation present and Review recorded gates.

Plan content uses structured plain text, headings, ordered steps, lists, tables, and approved diagrams. Arbitrary admin-authored HTML is not part of the v1 design.

### Medical disclaimer and health-data consent

The medical disclaimer and health-data consent are separate decisions.

- **Medical disclaimer (FR-9.6):** immediately before the subscriber first selects or customizes a plan, a focused interstitial shows a short plain-language summary, a link to the full disclaimer, a secondary Back action, and a primary **I understand — continue** action. No pre-checked checkbox is used. After acknowledgement, plan pages retain a quiet “Not medical advice · Safety information” link near the Evidence section; the blocking interstitial does not repeat unless the disclaimer materially changes.
- **Health-data consent (FR-9.2):** immediately before the first health-data write, a separate consent screen names the categories collected, purposes, withdrawal path, export/deletion rights, and the effect of withdrawal. An unchecked **I consent to the collection and use described above** checkbox plus **Continue** records the explicit action. Declining returns without saving health data. Account → Privacy makes the consent state and withdrawal action continuously available.

Neither flow uses a persistent warning banner. Banners are reserved for current actionable states such as unverified email, withdrawn consent, lapsed subscription, or pending out-of-band deletion (FR-2.11, FR-3.2, FR-9.9, FR-9.11).

### Logging and AI-assisted food estimates

Logging opens directly to the chosen entry type, with the date near the top and Save kept in a stable location. The interface does not prefill a previous health value. A successful save shows the recorded value and an Edit link.

The food flow offers three equal-entry methods: search the bundled dataset, describe or photograph food for an estimate, or enter values manually (FR-8.11, FR-8.12). AI is never the default or the only path. Before estimation, the interface says “Your description or photo is used only to create this estimate. Photos are not saved.” Estimated calories and macros appear in ordinary editable fields with a persistent `Estimate` label and the instruction “Review and adjust before saving.” Confirmation saves the subscriber-edited values, not an opaque model response.

### Progress and target comparison

Progress pages pair every trend chart with a data table showing the same full-granularity entries (FR-8.14). Range controls are 4 weeks, 3 months, 1 year, and all time. The selected range is programmatically exposed.

Charts use direct labels where space allows, visible point shapes, and a stroke of at least 2px. Multiple series differ by both color and dash or point shape. Keyboard focus can reach each data point and expose its label and value. Tooltips duplicate information; they never contain unique data. Reduced motion removes drawing animations.

Daily calorie and macro comparisons show explicit text such as “82 g of 110 g target” beside a linear indicator (FR-8.5). Going past a target continues the bar with a labelled over-target value; it does not turn red or use failure language.

### Account, privacy, and destructive actions

Account groups Profile, Units and appearance, Security, Subscription, Consultant access, and Privacy. Export is described as a JSON download and remains separate from deletion. Consent withdrawal explains that new logging stops while existing records remain available under FR-9.3–FR-9.5.

Account deletion uses a dedicated page, not a small dialog. It summarizes immediate live-system deletion, the 35-day backup horizon, audit tombstoning, and loss of access (FR-9.4, FR-9.10). The final action requires fresh re-authentication (SECURITY.md SEC-AUTHN-7) and an explicit destructive button labelled **Permanently delete account**.

## Content Voice

Proof of Work writes like a knowledgeable training partner: direct, specific, non-judgmental, and willing to state uncertainty.

- Use “record”, “entry”, “target”, “range”, “reviewed”, and “estimate”.
- Prefer “You logged 3 workouts this week” to “Great job crushing your goals!”
- Prefer “No entries yet” to “You’re falling behind”.
- Prefer “This estimate may be inaccurate; review it before saving” to “AI-powered precision”.
- Never use “clean/dirty food”, “cheat meal”, “summer body”, “burn off”, “failure”, or body-shaming language.
- Error copy explains recovery. Privacy copy names what happens. Scientific copy distinguishes a citation from proof.

## Accessibility

- **Target:** WCAG 2.2 AA (REQUIREMENTS.md OQ-11 resolved).
- **Keyboard:** all functionality is keyboard operable in a logical reading order with no traps. A skip link precedes repeated navigation. Focus is never suppressed.
- **Focus:** use a 2px high-contrast outline with a 2px offset. In light mode use `#176B45`; in dark mode use `#92D7AA`. Components near the same color add an outer canvas-colored separation ring.
- **Structure:** one `h1` per page, sequential heading levels, landmarks, labelled regions, persistent control labels, and text alternatives for meaningful graphics.
- **Reflow:** no loss of content or function at 320px width or 200% zoom. Text uses relative units.
- **Color and graphics:** never rely on color alone; meaningful graphics and control boundaries meet 3:1; text meets 4.5:1 except qualifying large text.
- **Motion:** honor `prefers-reduced-motion`. Standard transitions are 120–180ms and limited to opacity or position under 8px. No auto-playing motion, parallax, animated chart drawing, or celebratory motion is required to understand state.
- **Announcements:** errors use assertive announcements only when submission fails; saves and background updates are polite. Focus is not stolen for a successful save.
- **Charts:** the paired table is the complete text alternative, but the chart itself also has a concise accessible name and range summary.
- **Touch and pointer:** 44×44px minimum target, no hover-only disclosure, and no drag-only operation.

## Design Verification

Implementation is not design-complete until automated and manual checks cover:

- light, dark, and forced-colors/high-contrast behavior;
- 320px reflow and 200% zoom without lost content or page-level horizontal scrolling;
- keyboard-only completion of authentication, plan selection, logging, consent, disclaimer, consultant-access termination, and admin publication;
- visible focus, correct focus return, no traps, and logical route-change focus;
- axe-core checks plus manual screen-reader checks of forms, dialogs, charts/tables, and status announcements;
- contrast of every actual token pairing, including hover, disabled, focus, and error states;
- `prefers-reduced-motion` and 400% text-size stress checks;
- no health values or photos retained in browser storage after the relevant view ends (SEC-RENDER-4, FR-8.13).

These browser-level commitments belong in the Playwright and axe suite named by CLAUDE.md.

## Resolved Design Questions

- **OQ-1 — RESOLVED (2026-08-01):** The product name is **Proof of Work**. The primary identity is the Proof mark plus a live-text wordmark; the mark also supplies the favicon/app-icon strategy.
- **OQ-2 — RESOLVED (2026-08-01):** The personality is disciplined, evidence-led, calm, candid, and quietly encouraging—not clinical and not high-energy gym marketing.
- **OQ-3 — RESOLVED (2026-08-01):** Light and dark themes are in scope, with `System` as the initial preference and explicit Light/Dark choices.
- **OQ-4 — RESOLVED (2026-08-01):** Progress uses trend charts paired with full-granularity accessible tables over 4 weeks, 3 months, 1 year, and all time (FR-8.14).
- **OQ-5 — RESOLVED (2026-08-01):** Plan statements use inline numbered references linked to an in-page Evidence section. Subscribers see “Evidence reviewed by Proof of Work” with the date; admins also see verifier identity and publication gates.
- **OQ-6 — RESOLVED (2026-08-01):** A focused first-plan interstitial records medical-disclaimer acknowledgement, followed by a persistent quiet safety link. Health-data consent is a separate explicit-checkbox flow before the first health-data write.
- **OQ-7 — RESOLVED (2026-08-01):** Subscriber, consultant, and admin roles have distinct labelled workspaces within one visual system. Subscribers see consultant access on Home, in Account, and as edit provenance on affected plan copies.
- **OQ-8 — RESOLVED (2026-08-01):** Unit display follows FR-8.10. V1 launches in English/LTR; components are text-expansion tolerant and logical-property ready, while additional translations and active RTL support are deferred beyond v1.
- **OQ-9 — RESOLVED (2026-08-01):** No decorative or lifestyle photography is used. Admin-approved neutral vector exercise diagrams are allowed under the accessibility and body-image rules above; subscriber food photos appear only as transient, unsaved estimation previews.

There are no open design questions in this specification.
