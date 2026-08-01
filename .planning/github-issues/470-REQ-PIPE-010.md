# [REQ-PIPE-010] Dependency policy and reproducible resolution

## Metadata

- **ID**: REQ-PIPE-010
- **Title**: Dependency policy and reproducible resolution
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: High
- **Requirement Type**: Operational
- **Source / Parent**: REQ-EPIC-001; `SECURITY.md` DEP-1 … DEP-8; threat TM-T-7

## Requirement

- **Statement**: Production and CI builds MUST resolve dependencies to exact versions through a committed lockfile with frozen or reproducible resolution, MUST NOT resolve floating versions at build or deployment time, and every dependency addition MUST be justified against the DEP-1 … DEP-8 policy before it is merged.
- **Rationale**: DEP-7 states the lockfile and frozen-resolution rule; DEP-2 requires per-dependency justification in the pull request; DEP-3 through DEP-6 and DEP-8 supply the maintenance, version, vulnerability, transitive-graph, and selection criteria. The threat model rates a malicious or vulnerable dependency entering the build (TM-T-7) as high severity against trust boundary 4.
- **Assumptions**: No implementation and no dependencies exist yet (`SECURITY.md`, Dependency Security Rules — "No dependency has been assessed"), so this policy applies from the first dependency onward.
- **Out of Scope**: The pipeline configuration and blocking gate set — including the automated dependency-scanning gate SEC-CICD-4 requires — blocked by `SECURITY.md` SQ-7; the CI platform itself is decided (GitHub Actions with OIDC federation, `CLAUDE.md`); secret scanning and IaC scanning (also SEC-CICD-4, blocked); build and deployment identities (SEC-CICD-1, pipeline design blocked by SQ-7).
- **Design Traceability**: N/A
- **Architecture Traceability**: `ARCHITECTURE.md` — trust boundary 4 (CI/CD-and-IaC to production); DR-8 (unresolved decisions must not be hard-coded). The language and package manager are TypeScript and npm (`CLAUDE.md`).
- **Security Traceability**: DEP-1 … DEP-8; supports SEC-CICD-4, SEC-SECRET-1, SEC-TB-3 (a dependency that phones home would breach it).

## Scope

- **Applies To**: Server-Side Application, Web Client, Background Processing
- **Components**: All components; the build process
- **Interfaces / Operations**: Dependency installation and resolution in local, CI, and production builds; the dependency review step in a pull request
- **Actors**: Developers; the CI/CD identity; a compromised upstream maintainer as adversary
- **Preconditions**: None — TypeScript and npm are recorded in `CLAUDE.md`, so this issue is unblocked
- **Data Classification**: Internal
- **Personal or Regulated Data**: None directly — but a compromised dependency reaches all health data
- **Jurisdiction / Regulatory Scope**: N/A

## Security Context

- **Security Objectives**: Integrity, Availability, Confidentiality
- **Control Layers**: Supply Chain, Architecture
- **Threat References**: `SECURITY.md` TM-T-7 (malicious or vulnerable dependency enters the build), TM-T-6 (CI/CD compromise); CWE-1104 (use of unmaintained third-party components), CWE-1357 (reliance on insufficiently trustworthy component), CWE-829 (inclusion of functionality from an untrusted control sphere)
- **Abuse / Misuse Case**: A floating version resolves at deploy time to a newly published malicious release; or a small utility drags in a large unvetted transitive tree; or an abandoned package with a known unpatched vulnerability is introduced because no review step existed.
- **Trust Boundary**: Boundary 4 — the build and deploy path can rewrite the system itself.
- **Untrusted Inputs or Assertions**: Every third-party package and its full transitive graph; registry metadata.
- **Authoritative Enforcement Point**: The committed lockfile plus the frozen-install flag in CI and production builds; the pull request review gate for additions.
- **Independent Verification**: A build from a clean checkout produces the same resolved graph as the lockfile records.
- **Zero Trust Relevance**: N/A — supply-chain integrity rather than a resource-access decision.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: N/A
- **Regulatory**: N/A
- **Other**: `REF-SUPPLY`, `REF-DEPS`, `REF-SSDF`, `REF-PROMPT-NODE`, `REF-PROMPT-VUE` — the applicable sources `SECURITY.md` names for the dependency rules.
- **Mapping Basis**: DEP-1 … DEP-8 are the normative rules and `SECURITY.md` names these sources as applicable to them; the CWE identifiers name the unmaintained-component and untrusted-inclusion classes.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a clean checkout, when a CI or production build runs, then dependencies resolve exclusively from the committed lockfile with frozen resolution, and the resolved graph is identical to the previous build for an unchanged lockfile.
2. **AC-02 — Boundary or failure behavior**: Given a build environment where the lockfile is absent, out of date relative to the manifest, or would require resolving a floating version, when the build runs, then it fails rather than resolving at build time.
3. **AC-03 — Prohibited behavior**: Given a proposed dependency, when it is reviewed, then it MUST NOT be merged if it is deprecated, abandoned, end-of-life, or pre-release; if it shows no stable release, security response, or substantive maintainer activity within the previous 12 months without a documented exception; or if it carries a known unpatched vulnerability applicable to its intended use without explicit, time-bounded risk acceptance and a remediation plan.
4. **AC-04 — Additional criterion**: Given a proposed dependency, when the pull request is opened, then it states the dependency's purpose and why the standard library or straightforward first-party code is insufficient (DEP-2), and records a review of the complete transitive graph (DEP-6).
5. **AC-05 — Additional criterion**: Given a security-critical function — cryptography, authentication, authorization, protocol parsing, output encoding, HTML sanitization — when it is implemented, then a vetted library is used rather than custom code written to avoid a dependency (DEP-1).

## Failure Behavior

- **On Invalid Input**: A malformed or inconsistent lockfile fails the build rather than triggering a re-resolution.
- **On Authentication Failure**: N/A
- **On Authorization Failure**: N/A
- **On Security-Decision Failure**: If a dependency's maintenance status or vulnerability state cannot be determined, do not merge it (fail closed).
- **On External Dependency Failure**: If the package registry is unavailable, the build fails; it MUST NOT fall back to an alternative registry or an unpinned source.
- **On System Error**: The build fails visibly; a partially resolved dependency graph is never deployed.
- **Logging / Audit**: The lockfile diff and the dependency justification in the pull request are the audit record. Build logs MUST NOT contain registry credentials (SEC-SECRET-1).
- **Alerting**: TO BE DECIDED — automated vulnerability alerting depends on the CI platform, blocked by `SECURITY.md` SQ-7.

## Test Strategy

- **Unit Tests**: N/A — this is a build and process control rather than runtime code.
- **Integration Tests**: Clean-checkout build reproducing the locked graph; a deliberately stale lockfile causing a build failure; a floating-version manifest entry causing a build failure.
- **Security Tests**: Vulnerability check across the direct and transitive graph before merge (DEP-5); a review check that no dependency initiates network requests at runtime, which REQ-PRIVACY-050 depends on; verification that no security-critical function is implemented as custom code (DEP-1).
- **Compliance Tests / Evidence**: The committed lockfile, the per-dependency justification records, and the vulnerability check output, retained as supply-chain evidence for `REF-SSDF`.
- **Acceptance-Criteria Traceability**: AC-01 — reproducible build test; AC-02 — stale and floating lockfile failure tests; AC-03 and AC-04 — pull request review checklist with recorded evidence; AC-05 — code review of security-critical paths.
- **Coverage Target**: Every dependency in the graph covered by the vulnerability check; every addition covered by a justification record.
- **Required Test Environment**: A clean build environment with npm, and a GitHub Actions runner; the pipeline gate set remains TO BE DECIDED (`SECURITY.md` SQ-7).

## Dependencies

- **Upstream Requirements**: None
- **Downstream Requirements**: Every issue that introduces a library — REQ-API-010 (validation), REQ-API-030 (database driver), REQ-SESSION-010 (JWT), REQ-AUTH-020 (WebAuthn), REQ-CATALOG-030 (sanitizer, if rich text is adopted)
- **External Dependencies**: The package registry for the chosen ecosystem.
- **Dependency Assumptions**: npm supports a committed `package-lock.json` and `npm ci` frozen installs, which is what DEP-7 requires.
- **Failure Impact**: A compromised dependency executes inside the trust boundary that holds all health data, with none of the application's authorization controls applying to it.

## Implementation Notes

- **Constraints**: TypeScript with npm workspaces, and GitHub Actions as the CI platform (`CLAUDE.md`). `npm ci` against the committed `package-lock.json` supplies DEP-7's frozen resolution. The policy is implementable now as a documented review gate and lockfile discipline; the SEC-CICD-4 automated gate set remains TO BE DECIDED (`SECURITY.md` SQ-7) and is tracked separately. This issue's coverage of the dependency rules is therefore process-and-lockfile, not enforcement-in-pipeline.
- **Prohibited Approaches**: Version ranges resolved at install time in CI or production; a lockfile excluded from source control; adding a dependency without the DEP-2 justification; writing custom cryptography, authentication, authorization, parsing, encoding, or sanitization to avoid a dependency (DEP-1); accepting a large opaque transitive tree for a small convenience (DEP-6).
- **Implementation Guidance**: `SECURITY.md` DEP-2 prefers zero new dependencies, while DEP-1 forbids replacing vetted security functionality with custom code — the two together mean the dependencies this project should accept are precisely the security-critical ones, and few others.
- **AI Development Guidance**: `REF-SUPPLY`, `REF-DEPS`, `REF-SSDF`, `REF-PROMPT-NODE`, `REF-PROMPT-VUE`; `CLAUDE.md`.
- **Required Human Review**: Security review of every dependency addition.
- **Open Decisions**: The pipeline gate set (`SECURITY.md` SQ-7). GitHub Actions is selected, but until the blocking-check set is designed, the automated scanning gate (SEC-CICD-4) is not built and this control depends on human review.

**Estimated effort**: 0.5–1 engineer-day. **Estimated changed lines**: 50–200 (configuration and documentation; excludes the lockfile itself, which is a generated file).
**Recommended model**: Claude Opus (`claude-opus-5`) — a small but consequential supply-chain discipline where the judgement calls (DEP-1 versus DEP-2) matter more than the code.
