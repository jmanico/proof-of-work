# CLAUDE.md

Canonical agent instructions for this repository.

## Specifications

These are normative. Read them before proposing or writing anything, and follow them directly rather than re-deriving their content.

@REQUIREMENTS.md
@ARCHITECTURE.md
@SECURITY.md
@DESIGN.md

`REQUIREMENT_TEMPLATE.md` is not imported — it is the authoring format for new requirements (see Issues below).

## Repository state

No application code exists. The repository contains specifications only. The toolchain below is decided; nothing has been built with it yet.

The toolchain was selected on 2026-07-31, resolving ISSUE_PLAN.md PQ-1; the client half was completed on 2026-08-01. ARCHITECTURE.md and SECURITY.md already fixed Node.js, Vue.js, REST over HTTPS, relational persistence, Terraform on AWS, and the JWT session format; these choices sit within those constraints and do not revisit them.

- **Language, package manager, build tooling:** TypeScript, npm with workspaces, and Vite as the client build tool. Production and CI installs use `npm ci` against the committed `package-lock.json` (DEP-7).
- **Client rendering, routing, and styling:** A single-page application with `vue-router`. There is no server-rendering tier — ARCHITECTURE.md defines four components and adding one would be an architecture change, not a build-tool choice. Vite emits static assets and the server stays a pure REST API, which keeps DR-1 intact and makes SEC-HTTP-2's no-inline-script CSP reachable. Styling is plain CSS custom properties plus scoped single-file-component styles: DESIGN.md already fixes the tokens, and they map one-to-one onto custom properties, so no CSS framework is introduced (DEP-2). A component library MUST NOT be adopted without an explicit decision — it would import a competing visual system and accessibility posture over DESIGN.md's WCAG 2.2 AA rules.
- **Node.js server framework:** Fastify. Route-level JSON Schema validation (SEC-INPUT-1), response serialization against a declared schema (SEC-DATA-5), lifecycle hooks as the single authorization enforcement point (SEC-AUTHZ-5), and pino redaction paths (SEC-LOG-3) are the reasons it was chosen; use those built-ins rather than reimplementing them per endpoint.
- **RDBMS engine and migration tooling:** PostgreSQL, with Drizzle ORM and drizzle-kit. Migrations may contain raw SQL, which is how row-level security policies and append-only audit enforcement (SEC-LOG-2, SEC-LOG-7) are expressed below the application layer.
- **Test framework and runner commands:** Vitest for unit and component tests across both workspaces; Playwright with axe-core for end-to-end and WCAG 2.2 AA accessibility testing. DESIGN.md's commitments that need a real browser — 200% zoom, 320px reflow without horizontal scroll, keyboard operability with no traps, visible focus, reduced motion — are Playwright's responsibility, not Vitest's. Runner commands: TO BE DECIDED — the scripts do not exist until the scaffolding lands. Code-coverage threshold: TO BE DECIDED.
- **Lint and format tooling:** ESLint (flat config) with typescript-eslint, eslint-plugin-vue, and eslint-plugin-security, plus Prettier. SEC-RENDER-1 and SEC-INPUT-5 name lint rules as their verification method; `vue/no-v-html` and a dynamic-query-construction rule are required, not optional.
- **CI platform and pipeline:** GitHub Actions is the platform, using OIDC federation to AWS IAM roles rather than static keys (SEC-CICD-1). Pipeline design, the merge-blocking gate set (SEC-CICD-4), secret-management service, and AWS service topology remain TO BE DECIDED (SECURITY.md SQ-7; ISSUE_PLAN.md PQ-19).
- **Local development and run instructions:** TO BE DECIDED — derived from the toolchain above, but unwritten because no runnable code exists yet.
- **Directory layout:** npm workspaces — `apps/api` (Fastify REST API, with Identity and Session Handling as an internal module so ARCHITECTURE.md's module-versus-service decision stays open per DR-8), `apps/web` (Vue single-page application built by Vite), `packages/shared` (request and response contracts and validation schemas), `db/migrations`, `infra` (Terraform).

`packages/shared` carries contract shape only. Every authorization, entitlement, ownership, and validation rule is still enforced in `apps/api`; a rule that exists only in shared or client code violates DR-2.

Do not invent, assume, or silently pick anything still marked above. When work requires one, state that it is undecided and ask. When one is decided, replace the placeholder here in the same change that introduces it.

## Working rules

- **Specs precede code.** A behavior that is not in REQUIREMENTS.md, ARCHITECTURE.md, SECURITY.md, or DESIGN.md is not agreed. Propose a spec change first; do not encode an unspecified behavior in code and call it settled.
- **Do not resolve open questions unilaterally.** REQUIREMENTS.md OQ-*, DESIGN.md OQ-*, SECURITY.md SQ-*, and every `TO BE DECIDED` / `UNKNOWN` marker are open. Surface them; let the user decide. Never quietly close one by writing code that depends on one answer.
- **Preserve the markers.** `TO BE DECIDED` and `UNKNOWN` are deliberate. Do not replace either with a plausible-sounding value.
- **Trace every change.** Any non-trivial implementation change must name the requirement, architecture, security, and design identifiers it satisfies. If nothing traces, the change is out of scope.
- **Report gaps rather than closing them.** If a requested change cannot be made without deciding something open, do the parts that are unblocked, and say explicitly what was left and which open item blocks it.

## GitHub issues

Every new GitHub issue MUST follow `REQUIREMENT_TEMPLATE.md`, so that each issue is a structured, testable requirement — metadata, atomic normative statement, scope, security context, standards alignment, acceptance criteria, failure behavior, test strategy, dependencies, and implementation notes.

- Use `N/A` only for a field evaluated and found inapplicable; use `TO BE DECIDED` for one that is unresolved. Never leave a bracketed placeholder in a filed issue.
- Split compound behavior across separate issues. One issue is one atomic requirement.
- Do not invent standards mappings or control identifiers. An unverified mapping is `TO BE DECIDED`.

## Branch and change workflow

- Default branch is `main`. History to date is squash-merged pull requests.
- Do not commit or push unless asked. If asked while on `main`, branch first.
- Everything else about the workflow — branch naming, PR template, required reviewers, merge strategy, release process: TO BE DECIDED.
