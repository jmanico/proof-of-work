# [REQ-INFRA-070] Merge-blocking CI security gates

## Metadata

- **ID**: REQ-INFRA-070
- **Title**: Merge-blocking CI security gates
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `SECURITY.md` SEC-CICD-4 (gate set fixed by SQ-7); `CLAUDE.md` Repository state (CI platform and pipeline); threats TM-T-6, TM-T-7

## Requirement

- **Statement**: The CI pipeline MUST run the fixed gate set on every pull request — lint and typecheck, the Vitest unit and component suites, the Playwright with axe end-to-end suite, the osv-scanner dependency vulnerability audit, the gitleaks secret scan, the checkov IaC scan, and the authorization test suite — and a failure of any gate MUST block merge to `main`.
- **Rationale**: SEC-CICD-4 requires security-relevant automated checks to run in CI and block merge on failure, and SQ-7 fixed the gate set; `CLAUDE.md` names GitHub Actions as the platform and this exact set as the merge-blocking gates. The build and deploy path can rewrite the system itself (`ARCHITECTURE.md` trust boundary 4), so an advisory-only check is no control: TM-T-6 (pipeline compromise) and TM-T-7 (malicious or vulnerable dependency) are both rated high severity, and several security rules name a gate as their verification method — SEC-RENDER-1 and SEC-INPUT-5 (lint rules), SEC-SECRET-1 (secret scanning), SEC-CICD-3 (IaC scanning), DEP-5 (vulnerability audit), SEC-AUTHZ-1 (authorization tests).
- **Assumptions**: The workspace scaffolding, runner scripts, and lint configuration the gates invoke exist (REQ-BUILD-010); runner commands and the code-coverage threshold remain TO BE DECIDED in `CLAUDE.md` and are not fixed here — the gates run whatever documented commands REQ-BUILD-010 establishes. Staging auto-deploys from `main` and production requires manual approval (SEC-CICD-1), so `main` is the deployment source the gates protect.
- **Out of Scope**: The content of the suites the gates execute — the authorization test suite (REQ-AUTHZ-010 through REQ-AUTHZ-060 and their feature tests), accessibility coverage (DESIGN.md Design Verification), dependency policy itself (REQ-PIPE-010 — DEP-1 … DEP-8); deployment identities and environment promotion (REQ-INFRA-010 — SEC-CICD-1, SEC-CICD-2); branch naming, PR template, and reviewer rules (`CLAUDE.md` — TO BE DECIDED, deliberately not resolved here).
- **Design Traceability**: `DESIGN.md`, Design Verification — the browser-level commitments "belong in the Playwright and axe suite named by CLAUDE.md"; that suite is one of the gates.
- **Architecture Traceability**: `ARCHITECTURE.md` — trust boundary 4 (CI/CD-and-IaC path → production environment); DR-5 (the CI/CD migration-and-import path is a sanctioned path whose integrity these gates protect).
- **Security Traceability**: SEC-CICD-4 (primary); SEC-CICD-1 (pipeline identities the gates run under); SEC-SECRET-1 (gitleaks), SEC-CICD-3 (checkov), DEP-5/DEP-7 (osv-scanner over the committed lockfile), SEC-RENDER-1 and SEC-INPUT-5 (lint-rule verification), SEC-AUTHZ-1–SEC-AUTHZ-9 (authorization suite as a gate).

## Scope

- **Applies To**: Multiple — CI pipeline over all workspaces (`apps/api`, `apps/web`, `packages/shared`, `db/migrations`, `infra`)
- **Components**: CI pipeline (GitHub Actions); it exercises the Browser Client, REST API Application, Identity and Session Handling, Relational Persistence migrations, and Terraform code
- **Interfaces / Operations**: Pull-request checks and the merge protection on `main`; the seven named gates
- **Actors**: Developers and reviewers submitting changes; the CI service identity; an attacker attempting to land malicious or vulnerable code through the pipeline
- **Preconditions**: Repository scaffolding with documented build, test, and lint commands (REQ-BUILD-010); GitHub Actions configured with per-environment OIDC identities (REQ-INFRA-010)
- **Data Classification**: Internal — pipeline configuration and results; no production data is present in CI (SEC-CICD-5)
- **Personal or Regulated Data**: None — CI operates on code and synthetic fixtures only
- **Jurisdiction / Regulatory Scope**: N/A — no personal data is processed by the gates themselves; the health-data regimes attach to the system the gates protect, not to this pipeline control

## Security Context

- **Security Objectives**: Integrity, Availability, Accountability
- **Control Layers**: Supply Chain, Architecture, Logging and Monitoring
- **Threat References**: `SECURITY.md` TM-T-6 (CI/CD or Terraform-state compromise deploys attacker-controlled code), TM-T-7 (malicious or vulnerable dependency enters the build); STRIDE Tampering / Elevation of Privilege at trust boundary 4; CWE-1357 (reliance on insufficiently trustworthy component) for the dependency-audit gate
- **Abuse / Misuse Case**: A developer or compromised account merges code that fails the authorization suite, carries a known-vulnerable dependency, embeds a secret, or weakens IaC posture, because a gate was advisory, skipped, or removable in the same pull request it should have blocked.
- **Trust Boundary**: Boundary 4 — the CI/CD-and-IaC path into the production environment; merge to `main` is the entry point, since staging deploys from `main` automatically.
- **Untrusted Inputs or Assertions**: The changed code under test, including workflow-file changes in the same pull request; dependency manifests and lockfile changes; a contributor's claim that a failure is a false positive (requires review, not a bypass).
- **Authoritative Enforcement Point**: The source-control platform's required-status-check protection on `main` — merge is refused by the platform while any gate fails, not by convention.
- **Independent Verification**: Gates run in the CI service, not on developer machines; branch protection is platform-enforced and cannot be satisfied by a local claim of passing tests.
- **Zero Trust Relevance**: N/A — this is a supply-chain integrity control, not a resource-access decision on user requests.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session (SQ-10 pre-launch assessment).
- **OWASP AISVS 1.0**: N/A — the gates do not evaluate an AI component; AISVS applicability for the AI Inference component is tracked with SQ-10.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: N/A.
- **Regulatory**: N/A — no statutory obligation attaches to the pipeline control itself; the SQ-1 regimes govern the data the protected system handles.
- **Other**: `REF-SSDF`, `REF-CICD`, `REF-DEPS` as cited by SEC-CICD-4; `REF-SUPPLY` via the dependency rules the osv-scanner gate enforces.
- **Mapping Basis**: SEC-CICD-4 names its references directly; NIST SSDF and the OWASP CI/CD and dependency cheat sheets are the selected authorities for merge-gating and dependency auditing in `SECURITY.md`.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a pull request against `main`, when CI runs, then all seven gates execute — lint and typecheck, Vitest unit and component, Playwright with axe, osv-scanner, gitleaks, checkov, and the authorization test suite — and each reports a distinct status to the pull request.
2. **AC-02 — Blocking on failure**: Given a pull request with any single gate failing, when a merge is attempted, then the platform refuses the merge; this is verified per gate with a deliberate failing case — a lint violation, a failing unit test, a failing end-to-end or axe check, a dependency with a known vulnerability, a planted test secret, an insecure Terraform resource, and a failing authorization test — each individually blocking.
3. **AC-03 — Prohibited behavior**: Given the branch protection configuration, when a pull request fails a gate, then no merge path — administrator merge, bypass label, or re-run-until-green without a code change — is available as a routine action; and removing or disabling a gate in the workflow files MUST NOT itself merge without the still-active branch protection flagging the missing required status.
4. **AC-04 — Full-set coverage**: Given the branch protection settings and workflow definitions, when they are reviewed, then every gate in the SEC-CICD-4 set is a required status check on `main`, and no gate runs as advisory-only.

## Failure Behavior

- **On Invalid Input**: A pull request that cannot build or whose workflows fail to start reports failure and blocks merge — an unrunnable gate is a failed gate, not a skipped one.
- **On Authentication Failure**: N/A — platform authentication to GitHub governs; pipeline AWS identities are covered by REQ-INFRA-010.
- **On Authorization Failure**: N/A — repository permissions are platform-managed; no gate result may be overridden by repository role as a routine action.
- **On Security-Decision Failure**: Fail closed: a gate that errors, times out, or cannot fetch its tooling reports failure and blocks merge; it MUST NOT report success or neutral.
- **On External Dependency Failure**: If a scanner's vulnerability database or an action dependency is unavailable, the gate fails and blocks; retry is manual and visible. No gate silently degrades to a pass.
- **On System Error**: CI infrastructure errors surface as failed checks; merge remains blocked until a genuine green run completes.
- **Logging / Audit**: Gate results, run logs, and the merge history are retained by the platform per run; scanner findings are visible in the pull request. Secrets detected by gitleaks are reported by location, never echoed in logs.
- **Alerting**: A gitleaks finding on any branch and repeated gate-bypass attempts are surfaced to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-11 RESOLVED); routine test failures alert only the pull-request author.

## Test Strategy

- **Unit Tests**: N/A — the gates are pipeline configuration; the suites they run carry their own unit tests.
- **Integration Tests**: A pipeline verification run on a scratch branch exercising AC-01: all seven gates trigger and report on a representative pull request touching each workspace.
- **Security Tests**: The deliberate failing-case matrix of AC-02 — one seeded failure per gate, each demonstrated to block merge — recorded as evidence and re-run when workflows materially change; a workflow-tamper case asserting that deleting a required check does not unblock merge (AC-03).
- **Compliance Tests / Evidence**: Branch protection configuration export listing the seven required status checks; the failing-case evidence set (SEC-CICD-4 verification: "pipeline configuration review; deliberate failing-case test").
- **Acceptance-Criteria Traceability**: AC-01 — pipeline verification run; AC-02 — per-gate failing-case matrix; AC-03 — bypass and workflow-tamper checks against branch protection; AC-04 — configuration review evidence.
- **Coverage Target**: All seven gates covered by both a passing and a deliberately failing case; no gate advisory-only.
- **Required Test Environment**: The GitHub repository with Actions enabled and branch protection on `main`; seeded failing fixtures (a known-vulnerable test dependency pin, a planted dummy secret, an intentionally insecure Terraform sample) kept on scratch branches, never merged.

## Dependencies

- **Upstream Requirements**: REQ-BUILD-010 (workspace scaffolding, runner scripts, lint configuration the gates invoke), REQ-INFRA-010 (pipeline identities and environments; staging deploys from the `main` these gates protect)
- **Downstream Requirements**: Every feature and infrastructure issue — their tests become merge-blocking through this gate set; REQ-PIPE-010 (osv-scanner is the enforcement point for DEP-5 audits)
- **External Dependencies**: GitHub Actions and branch protection; osv-scanner, gitleaks, and checkov as CI tooling with their vulnerability and rule databases
- **Dependency Assumptions**: CI tooling is pinned to exact versions under DEP-7 discipline; scanner databases update out-of-band and may introduce new findings on unchanged code — such findings block until triaged, which is the intended fail-closed behavior.
- **Failure Impact**: If gates are advisory or bypassable, trust boundary 4 is open: vulnerable dependencies, committed secrets, insecure infrastructure, and authorization regressions reach staging automatically on merge and production behind a single manual approval.

## Implementation Notes

- **Constraints**: GitHub Actions is the platform (`CLAUDE.md`); the gate set is fixed by SEC-CICD-4/SQ-7 and MUST NOT be reduced without a `SECURITY.md` change; runner commands come from REQ-BUILD-010 — this issue MUST NOT invent commands or fix the coverage threshold (both TO BE DECIDED in `CLAUDE.md`).
- **Prohibited Approaches**: Advisory-only checks; `continue-on-error` on any gate; routine administrator bypass or merge-override labels; gates that pass when their tooling fails to run; resolving floating tool or action versions at run time (DEP-7); echoing detected secrets into logs.
- **Implementation Guidance**: One workflow per gate (or clearly separated jobs) so each reports a distinct required status and a failure is attributable at a glance; make the authorization suite a named, separate status rather than folding it into the general Vitest job, since SEC-CICD-4 lists it as its own gate. Keep the failing-case fixtures on dedicated scratch branches so the evidence run is repeatable.
- **AI Development Guidance**: `REF-CICD`, `REF-SSDF`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of workflow definitions and branch protection settings; re-review on any change to the gate set or its bypass surface.
- **Open Decisions**: Runner commands and the code-coverage threshold are TO BE DECIDED in `CLAUDE.md` (delivered by REQ-BUILD-010 and a coverage decision respectively); branch naming, PR template, and reviewer requirements are TO BE DECIDED and deliberately untouched here.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 300–600.
**Recommended model**: Claude Opus (`claude-opus-5`) — pipeline configuration where fail-closed semantics and an unbypassable required-check surface are the acceptance bar.
