# [REQ-BUILD-010] Workspace scaffolding and toolchain baseline

## Metadata

- **ID**: REQ-BUILD-010
- **Title**: Workspace scaffolding and toolchain baseline
- **Version**: 1.1.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Operational
- **Source / Parent**: REQ-EPIC-001; `CLAUDE.md` Repository state (resolves `ISSUE_PLAN.md` PQ-1); `ARCHITECTURE.md` DR-1, DR-5, DR-6, DR-8

## Requirement

- **Statement**: A clean checkout MUST install with `npm ci` and MUST expose one documented command each for build, test, lint, format check, and database migration across the workspace, with the directory structure recorded in `CLAUDE.md` in place and the dependency direction of `ARCHITECTURE.md` DR-1, DR-5, and DR-6 enforced by workspace boundaries rather than by convention.
- **Rationale**: Several issues have no in-plan predecessor and REQ-PIPE-010's precondition is now satisfied, yet no other drafted issue creates the manifest, the migration mechanism, the test harness, or the lint configuration they all presuppose. Multiple issues name a schema migration as their delivery vehicle and no issue establishes migrations. Two security rules — SEC-RENDER-1 and SEC-INPUT-5 — name a lint rule as their *verification method*, so until the lint configuration exists those rules have no stated means of verification. This issue supplies that substrate once, so no feature issue has to invent it and diverge.
- **Assumptions**: The toolchain in `CLAUDE.md` Repository state is settled and is not reopened here. No application code exists, so nothing must be migrated or preserved.
- **Out of Scope**: Every functional requirement — this issue implements no `FR-*` and adds no product behavior. Client application code and interface work, which belong to REQ-PLATFORM-010 through -030 and the REQ-CATALOG issues; this issue stands up the `apps/web` workspace and its Vite build but ships no view, route, or component. CI workflow definitions and the merge-blocking gate set, whose content is fixed (`SECURITY.md` SEC-CICD-4, SQ-7 RESOLVED) and whose delivery is REQ-INFRA-070, with pipeline identities and deployment flow in REQ-INFRA-010. Terraform content under `infra/`, which receives a directory and nothing else — infrastructure delivery is REQ-INFRA-010 through REQ-INFRA-060. Database *schema* — tables belong to REQ-AUTH-010, REQ-AUDIT-010, REQ-PLAN-010, REQ-PLAN-020, and REQ-CUSTOM-030, and this issue delivers only the mechanism that runs them. Secret and configuration management (SEC-SECRET-2 — AWS Secrets Manager per SQ-7, delivered by REQ-INFRA-030). Any dependency beyond the recorded toolchain, which REQ-PIPE-010 governs.
- **Design Traceability**: N/A — `DESIGN.md` governs the interface, and no interface is built here. `DESIGN.md` token and component work is REQ-PLATFORM-010.
- **Architecture Traceability**: `ARCHITECTURE.md` — the four code-bearing components receive their directory: Browser Client (`apps/web`), REST API Application (`apps/api`), Identity and Session Handling (a module inside `apps/api`, preserving DR-8 because module-versus-service is still open), Relational Persistence (`db/migrations`). The fifth component, AI Inference, is a managed in-account service (Amazon Bedrock, `SECURITY.md` SQ-7) and receives no workspace directory. DR-1 (the client depends on the API only, never on persistence), DR-5 (persistence reachable at runtime only from the API — including its scheduled executions — and Identity, plus the enumerated CI/CD and break-glass paths), DR-6 (one-directional dependencies, no cycles), DR-8 (unresolved decisions sit behind component interfaces and are not hard-coded).
- **Security Traceability**: SEC-RENDER-1 and SEC-INPUT-5 (their stated verification method is a lint rule, which this issue wires); SEC-SECRET-1 (no secret material in source control); DEP-7 (committed lockfile, frozen install); SEC-CICD-2 (infrastructure defined as reviewable code); supports SEC-INPUT-1, SEC-DATA-5, SEC-AUTHZ-5, and SEC-LOG-3 by establishing the Fastify and pino baseline those rules are expected to be delivered through.

## Scope

- **Applies To**: Multiple
- **Components**: Browser Client; REST API Application; Identity and Session Handling; Relational Persistence; the build process
- **Interfaces / Operations**: Repository bootstrap; the build, test, lint, format-check, and migration commands; workspace dependency resolution
- **Actors**: Developers; the CI/CD identity
- **Preconditions**: None — the toolchain is recorded in `CLAUDE.md`
- **Data Classification**: Internal
- **Personal or Regulated Data**: None — no data of any kind is processed or stored by this issue
- **Jurisdiction / Regulatory Scope**: N/A

## Security Context

- **Security Objectives**: Integrity, Accountability
- **Control Layers**: Architecture, Supply Chain
- **Threat References**: `SECURITY.md` TM-T-7 (malicious or vulnerable dependency enters the build), TM-T-6 (CI/CD or IaC compromise); CWE-1188 (insecure default initialization), CWE-540 (inclusion of sensitive information in source code), CWE-1104 (use of unmaintained third-party components)
- **Abuse / Misuse Case**: A skeleton that permits the client workspace to import persistence or API internals directly, quietly defeating DR-1 and DR-5 before any authorization code exists; a committed `.env` or key file that leaks credentials from the first commit; a lint configuration that omits the two rules SEC-RENDER-1 and SEC-INPUT-5 depend on, so their stated verification never runs and the gap is invisible.
- **Trust Boundary**: Boundary 4 — the build path can rewrite the system itself. The workspace layout also materializes boundaries 1 and 3 as directory and dependency separation.
- **Untrusted Inputs or Assertions**: Third-party toolchain packages and their transitive graphs; anything a contributor adds to the tree that could carry secret material.
- **Authoritative Enforcement Point**: The workspace manifests and their dependency declarations, plus the lint and type-check configuration; violations fail the build rather than producing a warning.
- **Independent Verification**: A build from a clean checkout on a machine with no prior state produces a working tree and the same resolved dependency graph the lockfile records.
- **Zero Trust Relevance**: N/A — build-time structure rather than a resource-access decision.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — ASVS 5.0.0 Level 3 is the fixed design target (`SECURITY.md` SQ-10 RESOLVED); per-issue mappings are verified only at the independent pre-launch assessment.
- **OWASP AISVS 1.0**: N/A — this issue touches no AI-enabled behavior.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: N/A
- **Regulatory**: N/A — no personal or health data is involved.
- **Other**: `REF-SSDF` (NIST SP 800-218) for a secure build foundation; `REF-SECRETS` for keeping credentials out of source; `REF-PROMPT-QUALITY` for the separation-of-concerns structure the layout encodes.
- **Mapping Basis**: Only sources `SECURITY.md` already names are cited. No ASVS or NIST control identifier is asserted, because neither catalog was consulted and `REQUIREMENT_TEMPLATE.md` forbids guessing control identifiers.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a clean checkout on a machine with no prior project state, when `npm ci` runs and then each documented command for build, test, lint, format check, and migration is invoked, then every command completes successfully against the empty skeleton and each is recorded in `CLAUDE.md` Repository state under local development and run instructions.
2. **AC-02 — Boundary or failure behavior**: Given a lockfile that is absent or inconsistent with the workspace manifests, when `npm ci` runs, then it fails rather than resolving versions at install time (DEP-7).
3. **AC-03 — Prohibited behavior**: Given the workspace layout, when any module in `apps/web` attempts to import from `apps/api` internals, from `db/`, or from any database driver, then the attempt fails the type check or lint rather than producing a warning (DR-1, DR-5, DR-6).
4. **AC-04 — Additional criterion**: Given a fixture containing a raw HTML binding in a Vue component and a fixture containing a database query assembled by string concatenation, when lint runs, then each fixture fails — establishing the verification mechanism SEC-RENDER-1 and SEC-INPUT-5 name.
5. **AC-05 — Additional criterion**: Given the committed tree, when it is scanned for secret material, then no credential, signing key, connection string with a password, or populated `.env` file is present, and the ignore configuration excludes them by default (SEC-SECRET-1).
6. **AC-06 — Additional criterion**: Given a disposable PostgreSQL instance, when the migration command runs forward and then in reverse on an empty database, then it succeeds in both directions and leaves no residual objects — proving the mechanism the schema-bearing issues depend on.

## Failure Behavior

- **On Invalid Input**: A malformed workspace manifest or lint configuration fails the command with a non-zero exit rather than being skipped.
- **On Authentication Failure**: N/A — no authentication exists at this layer.
- **On Authorization Failure**: N/A
- **On Security-Decision Failure**: A lint or type-check rule that cannot be evaluated fails the run rather than passing silently; a disabled or unresolvable security rule is a build failure, not a warning.
- **On External Dependency Failure**: If the package registry is unavailable, the bootstrap fails; it MUST NOT fall back to an alternative registry or an unpinned source (REQ-PIPE-010).
- **On System Error**: A failed migration leaves the database in its prior state; a partially applied migration is never treated as success.
- **Logging / Audit**: Build and migration output only. No audit entry — this issue touches no health data and no user record. Build logs MUST NOT contain registry credentials or database passwords (SEC-SECRET-1, SEC-LOG-3).
- **Alerting**: N/A — no runtime service is introduced; CI alerting belongs to REQ-INFRA-070.

## Test Strategy

- **Unit Tests**: N/A — configuration and structure rather than runtime logic.
- **Integration Tests**: Clean-checkout bootstrap exercising every documented command; migration applied forward and reversed against a disposable PostgreSQL instance (AC-06); a deliberately stale lockfile causing `npm ci` to fail (AC-02).
- **Security Tests**: Import-boundary violation fixtures for `apps/web` reaching `apps/api` internals, `db/`, and a database driver, each asserted to fail (AC-03); the `vue/no-v-html` and dynamic-query fixtures asserted to fail lint (AC-04); a secret scan over the tree and its history (AC-05).
- **Compliance Tests / Evidence**: The recorded bootstrap transcript and the passing boundary and lint fixtures, retained as evidence that SEC-RENDER-1 and SEC-INPUT-5 have a working verification mechanism before any code depends on them.
- **Acceptance-Criteria Traceability**: AC-01 — bootstrap suite; AC-02 — stale lockfile test; AC-03 — import-boundary fixtures; AC-04 — security lint fixtures; AC-05 — secret scan; AC-06 — migration round-trip test.
- **Coverage Target**: Every documented command exercised from a clean checkout; every dependency rule named in Architecture Traceability covered by a failing fixture. A code-coverage percentage is TO BE DECIDED — Vitest is selected but no threshold has been set (`CLAUDE.md`).
- **Required Test Environment**: A clean checkout with no prior npm cache or `node_modules`; a disposable PostgreSQL instance for the migration round trip; Vitest as the runner, with Playwright browsers installed so the client end-to-end harness is proven to launch before any view exists to test.

## Dependencies

- **Upstream Requirements**: None — this is the root of the dependency graph.
- **Downstream Requirements**: All other leaf issues — the 61 remaining drafted leaves and the 38 planned in the 2026-08-03 pass. Directly and immediately: REQ-API-010, REQ-API-030, REQ-SESSION-010, REQ-AUDIT-040, REQ-PLATFORM-040, and REQ-PIPE-010, which are the filed issues with no other in-plan predecessor. REQ-AUTH-010, REQ-AUDIT-010, REQ-PLAN-010, REQ-PLAN-020, REQ-API-030, and REQ-CUSTOM-030 depend specifically on the migration mechanism.
- **External Dependencies**: The npm registry; a PostgreSQL instance for migration verification.
- **Dependency Assumptions**: npm workspaces can express the dependency direction DR-1 and DR-5 require, so a violation is a resolution or type error rather than a review finding. If that proves false, the enforcement mechanism must be reconsidered rather than downgraded to convention.
- **Failure Impact**: Nothing else can start. Every other issue either creates files in a structure this establishes or runs a command this defines.

## Implementation Notes

- **Constraints**: TypeScript with npm workspaces, Fastify, PostgreSQL with Drizzle ORM and drizzle-kit, Vite, Vitest with Playwright and axe-core, ESLint flat config with Prettier, and the `apps/api` / `apps/web` / `packages/shared` / `db/migrations` / `infra` layout (`CLAUDE.md`). `apps/web` is a Vite-built single-page application with `vue-router`, styled with plain CSS custom properties and scoped single-file-component styles; no CSS framework or component library is introduced. `packages/shared` carries contract shape only — a rule that lives there rather than in `apps/api` violates DR-2. Migrations may contain raw SQL — the mechanism this issue delivers must support it, because the append-only audit privileges (SEC-LOG-2, SEC-LOG-7) are expressed that way.
- **Prohibited Approaches**: Scaffolding anything that resolves a question still open — no CSS framework or component library, and no local-run convention beyond the documented commands. Scaffolding behavior owned by later issues — no CI workflow (REQ-INFRA-070 owns the fixed SEC-CICD-4 gate set), no Terraform content (REQ-INFRA-010…-060), no session transport (REQ-SESSION-050), no schema tables, no authentication stub. Creating placeholder route handlers or model classes that later issues must delete. Enforcing the dependency direction by documentation or review convention rather than by a mechanism that fails the build (DR-2's reasoning applies to structure as much as to business rules). Committing a `.env`, key, or connection string with real values. Adding a dependency outside the recorded toolchain without the REQ-PIPE-010 justification. Disabling a lint rule to make the skeleton pass.
- **Implementation Guidance**: The dominant risk is scaffolding too much. A skeleton that quietly pulls in a component library, invents a directory convention, or stubs an endpoint converts an open question into a fait accompli that no reviewer will revisit — which is exactly what `CLAUDE.md` forbids. Prefer an empty workspace that fails loudly over a populated one that presumes. Wire the two security lint rules before any code exists to trip them, so they are known to work rather than assumed to.
- **AI Development Guidance**: `CLAUDE.md`; `REF-PROMPT-QUALITY` (separation of concerns, testability without hidden global state); `REF-PROMPT-NODE`; `REF-PROMPT-VUE`; `REF-SECRETS`; `REF-SSDF`.
- **Required Human Review**: Architecture review that the workspace boundaries actually enforce DR-1, DR-5, and DR-6 rather than merely describing them; security review of the ignore configuration and the two security lint rules.
- **Open Decisions**: The code-coverage threshold is unset (`CLAUDE.md`, TO BE DECIDED). The concrete runner commands and local development instructions are TO BE DECIDED in `CLAUDE.md` until this issue lands and records them (AC-01). Whether Identity and Session Handling becomes a separate deployable is left open per DR-8; it is scaffolded as a module inside `apps/api` so either outcome remains available.

**Estimated effort**: 1–2 engineer-days. **Estimated changed lines**: 300–600 (manifests, configuration, and documentation; excludes the generated lockfile).
**Recommended model**: Claude Opus (`claude-opus-5`) — the difficulty is restraint rather than breadth. The failure mode is a skeleton that silently resolves an open question, and judging what to leave absent matters more than what to write.
