<p align="center">
  <img src="./logo.svg" width="96" height="96" alt="">
</p>

<h1 align="center">Proof of Work</h1>

<p align="center">
  Evidence-led fitness plans. A clear record of effort. Progress you can inspect.
</p>

<p align="center">
  <strong>Status:</strong> design and specification · no application code exists yet
</p>

---

Proof of Work is a planned subscription fitness web application for following science-based exercise and diet plans, adapting them into private copies, and recording progress over time. Every published plan has peer-reviewed citations and an explicit admin review record.

The product is not a social network, a leaderboard, a medical service, or a cryptocurrency product. It is a calm, private training record built around evidence and user control.

## Product shape

Proof of Work serves three roles:

- **Subscribers** select exercise and diet plans, customize private copies, log workouts, food, weight, and measurements, and inspect full-granularity progress.
- **Fitness consultants** work only with subscribers who have an active paid engagement, viewing progress and editing subscriber-owned plan copies within a tightly defined scope.
- **Admins** author, cite, review, publish, and unpublish plans and administer access with audited actions.

The planned experience includes:

- a cited exercise and diet plan library;
- one active exercise plan and one active diet plan per subscriber;
- private plan customization that never mutates the published source;
- workout, nutrition, weight, and body-measurement logging;
- charts paired with accessible data tables over 4 weeks, 3 months, 1 year, and all time;
- food lookup from a bundled USDA dataset plus optional, editable AI estimates processed inside the system boundary;
- explicit health-data consent, export, correction, withdrawal, and deletion controls;
- subscriber-controlled consultant access with immediate revocation.

## Design direction

> Training log meets evidence notebook.

The interface combines warm paper-like surfaces, dark ink, grounded forest green, and a small signal-lime accent. It is disciplined and quietly encouraging—credible without looking clinical, motivating without hype, shame, streak pressure, or competitive gamification.

The Proof mark above pairs a custom `P` with a small datum: the product name and one observation in an accumulating record. It deliberately avoids completion badges as well as coin, chain, mining, hexagon, and other cryptocurrency cues.

The design system specifies:

- responsive subscriber, consultant, and admin workspaces;
- light, dark, and system color modes;
- WCAG 2.2 AA as the accessibility target;
- keyboard-complete workflows, visible focus, reduced motion, and 320px reflow;
- inline source references connected to a full Evidence section;
- separate acknowledgement flows for the medical disclaimer and health-data consent;
- neutral target language and clear labelling of AI-generated estimates;
- no lifestyle photography or idealized transformation imagery.

See [DESIGN.md](./DESIGN.md) for the normative brand, logo, token, layout, component, content, accessibility, and verification rules.

## Current repository state

This repository currently contains specifications only. Features described here are requirements and design commitments, not claims about a deployed application.

| Document | Purpose |
|---|---|
| [CLAUDE.md](./CLAUDE.md) | Canonical repository instructions and fixed toolchain decisions |
| [REQUIREMENTS.md](./REQUIREMENTS.md) | Product behavior, roles, privacy rights, and open product decisions |
| [DESIGN.md](./DESIGN.md) | Brand system, responsive experience, component behavior, and accessibility |
| [ARCHITECTURE.md](./ARCHITECTURE.md) | Components, boundaries, data flows, and dependency rules |
| [SECURITY.md](./SECURITY.md) | Threat model, controls, privacy posture, and assurance target |
| [REQUIREMENT_TEMPLATE.md](./REQUIREMENT_TEMPLATE.md) | Required structure for new implementation issues |
| [.planning/github-issues/ISSUE_PLAN.md](./.planning/github-issues/ISSUE_PLAN.md) | Planned issue sequence and implementation coverage |

The normative specification files take precedence over this README.

## Planned implementation

The fixed application stack is:

- TypeScript with npm workspaces;
- Vue 3, Vite, and `vue-router` for the browser client;
- plain CSS custom properties and scoped component styles—no component library;
- Node.js and Fastify for the same-origin REST API;
- PostgreSQL with Drizzle ORM and drizzle-kit;
- Vitest plus Playwright and axe-core;
- Terraform-managed AWS infrastructure using ECS Fargate, S3 and CloudFront, RDS PostgreSQL, Amazon Bedrock, and AWS Secrets Manager;
- GitHub Actions with short-lived OIDC deployment roles.

The intended workspace layout is `apps/web`, `apps/api`, `packages/shared`, `db/migrations`, and `infra`. It will be created when implementation scaffolding begins.

## Working in this repository

Read [AGENTS.md](./AGENTS.md) and then [CLAUDE.md](./CLAUDE.md) before proposing or changing anything.

Key rules:

1. Specifications precede code. Do not implement behavior that is absent from the normative documents.
2. Preserve unresolved `TO BE DECIDED` and `UNKNOWN` markers outside design until the user explicitly resolves them.
3. Trace non-trivial implementation work to requirement, architecture, security, and design identifiers.
4. Use [REQUIREMENT_TEMPLATE.md](./REQUIREMENT_TEMPLATE.md) for every new GitHub issue.
5. Do not treat client-side validation or presentation as the authority for authorization, entitlement, ownership, or data validity.

## Product name

Use **Proof of Work** in full. Do not shorten the product to “PoW” in user-facing copy.
