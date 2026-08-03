# [REQ-INFRA-010] Environment accounts, pipeline identities, and deployment flow

## Metadata

- **ID**: REQ-INFRA-010
- **Title**: Environment accounts, pipeline identities, and deployment flow
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Operational
- **Source / Parent**: REQ-EPIC-001; `SECURITY.md` SEC-CICD-1, SEC-CICD-2, SEC-CICD-5, SQ-7 (RESOLVED); `CLAUDE.md` (Repository state — CI platform and AWS topology)

## Requirement

- **Statement**: The system's infrastructure MUST be provisioned as three separate AWS accounts — dev, staging, and production, under AWS Organizations — defined entirely as Terraform code applied only after review, deployed by GitHub Actions authenticating through OIDC federation to a distinct least-privilege IAM role per environment account with no long-lived static credential, where staging deploys automatically from `main`, production deploys only after manual approval, and real subscriber health data exists only in the production account.
- **Rationale**: SEC-CICD-1 requires least-privilege, short-lived, per-environment pipeline identities; SEC-CICD-2 requires Terraform-only infrastructure change with review, because the CI/CD-and-IaC path can rewrite the system itself (trust boundary 4, threat TM-T-6); SEC-CICD-5 requires that non-production accounts never hold real health data. `SECURITY.md` SQ-7 resolved the concrete topology: separate accounts, GitHub Actions OIDC roles, staging auto-deploy from `main`, production behind manual approval.
- **Assumptions**: The npm-workspace repository with the `infra` Terraform workspace exists (REQ-BUILD-010); the operator controls an AWS Organizations management account and the GitHub repository settings needed for OIDC trust and environment protection rules.
- **Out of Scope**: In-environment network tiering, encryption, and database reachability (SEC-CICD-3, SEC-TB-2, SEC-DATA-1 — REQ-INFRA-020); secret and signing-key management including Terraform state protection (SEC-SECRET-2, SEC-SECRET-3, SEC-SESSION-7 — REQ-INFRA-030); the merge-blocking CI security gate set (SEC-CICD-4 — REQ-INFRA-070); the synthetic-data and provenance discipline for non-production datasets (SEC-CICD-5's data half — REQ-PIPE-020); break-glass operational access (SEC-OPS-1 — REQ-INFRA-050).
- **Design Traceability**: N/A — `DESIGN.md` governs the browser client, not deployment infrastructure.
- **Architecture Traceability**: `ARCHITECTURE.md` — Deployment model ("Terraform-managed infrastructure on AWS"; "three environments in separate AWS accounts"; "GitHub Actions deploys via per-environment OIDC roles"); trust boundary 4 (CI/CD-and-IaC path → production environment).
- **Security Traceability**: SEC-CICD-1, SEC-CICD-2, SEC-CICD-5; supports SEC-SECRET-3, SEC-CICD-3, SEC-CICD-4, SEC-OPS-1.

## Scope

- **Applies To**: Multiple — the deployment path itself (infrastructure and pipeline); no application runtime behavior
- **Components**: Deployment (Terraform, `infra` workspace); the GitHub Actions pipeline; all five runtime components as deployed artifacts
- **Interfaces / Operations**: Terraform plan and apply per environment; the GitHub Actions deploy workflows; the OIDC trust relationships and per-environment IAM deploy roles; GitHub environment protection rules for the production approval gate
- **Actors**: CI/CD pipeline identities (GitHub Actions OIDC principals); operators who review Terraform changes and approve production deploys; compromised CI/CD identity or dependency (adversary, `SECURITY.md` Threat Model)
- **Preconditions**: REQ-BUILD-010 scaffolding merged; AWS Organizations accounts created; GitHub OIDC provider registered in each account
- **Data Classification**: Restricted — the pipeline can rewrite the system that holds health data, and the account boundary is what keeps health data out of non-production
- **Personal or Regulated Data**: Health Data — indirectly: the production account holds it, and this requirement fixes which account may
- **Jurisdiction / Regulatory Scope**: Per `SECURITY.md` SQ-1 (RESOLVED): GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable. Single US primary region.

## Security Context

- **Security Objectives**: Integrity, Authorization, Accountability, Availability
- **Control Layers**: Supply Chain, Architecture, Authorization
- **Threat References**: `SECURITY.md` TM-T-6 (CI/CD or Terraform-state compromise deploys attacker-controlled code or infrastructure), TM-T-7 (malicious or vulnerable dependency enters the build); CWE-798 (use of hard-coded credentials) for the static-key prohibition; further CWE/CAPEC mappings TO BE DECIDED (not verified this session)
- **Abuse / Misuse Case**: An attacker who obtains a pipeline credential attempts to deploy attacker-controlled code or infrastructure to production; a leaked long-lived AWS key is replayed from outside CI; a dev-scoped identity reaches production resources; an operator bypasses review with a console change; production data is copied into a weaker-controlled non-production account.
- **Trust Boundary**: Boundary 4 — the CI/CD-and-IaC path into each environment, most critically production.
- **Untrusted Inputs or Assertions**: Any deploy request not carrying a valid, repository- and environment-scoped OIDC assertion; workflow definitions and third-party actions in the pipeline; any claim that a manual change was "reviewed" outside the Terraform path.
- **Authoritative Enforcement Point**: AWS IAM — the OIDC trust policy and per-environment role permissions; GitHub environment protection rules for the production approval; the Terraform review requirement enforced by branch protection on the `infra` workspace.
- **Independent Verification**: IAM policy review and automated IaC scanning read the deployed trust and permission configuration from AWS, independently of what workflow files assert; drift detection compares live infrastructure against Terraform state.
- **Zero Trust Relevance**: TO BE DECIDED — not verified against NIST SP 800-207 in this session.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — mapping deferred to the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A — no AI-enabled component in this requirement.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: The account separation is what confines real health data to production (SEC-CICD-5), supporting the SQ-1 regime set (GDPR/UK GDPR; CCPA/CPRA; Washington My Health My Data; FTC HBNR). Specific article/section mappings: TO BE DECIDED — the spec documents state no section for this behavior.
- **Other**: `REF-CICD`, `REF-SSDF`, `REF-PROMPT-TF-AWS` (cited by SEC-CICD-1); `REF-IAC` (SEC-CICD-2).
- **Mapping Basis**: The listed references are those SEC-CICD-1 and SEC-CICD-2 themselves cite.

## Acceptance Criteria

1. **AC-01 — Expected behavior (per-environment OIDC deploys)**: Given the three provisioned accounts, when a deploy workflow runs for a given environment, then it obtains short-lived credentials solely by OIDC federation to that environment's dedicated IAM role, and the role's permissions cover only the resources that environment's deploy requires — no wildcard actions or resources.
2. **AC-02 — Expected behavior (deployment flow)**: Given a merge to `main` that passes the CI gates, when the pipeline runs, then staging deploys automatically, and the production deploy proceeds only after a named human approves it through the environment protection rule; no workflow path deploys production without that approval.
3. **AC-03 — Prohibited behavior (credentials and cross-environment reach)**: Given the pipeline configuration and IAM state, when they are inspected, then no long-lived AWS access key exists for any pipeline identity, and an attempt to use the dev or staging deploy role against production-account resources is denied by IAM.
4. **AC-04 — Boundary or failure behavior (Terraform-only change)**: Given a manual change made to production infrastructure outside Terraform, when drift detection runs, then the drift is detected and surfaced for review, and the reviewed Terraform definition remains the authority for the next apply; an apply MUST NOT run from an unreviewed change to the `infra` workspace.
5. **AC-05 — Prohibited behavior (health-data confinement)**: Given the dev and staging accounts, when their data stores are reviewed under the REQ-PIPE-020 provenance discipline, then no real subscriber health data is present in any non-production account, and no pipeline step copies production data into a non-production account.

## Failure Behavior

- **On Invalid Input**: N/A — this requirement governs infrastructure and pipeline configuration, not request content.
- **On Authentication Failure**: A deploy attempt without a valid, correctly scoped OIDC assertion is denied by the IAM trust policy; the denial is visible in CloudTrail. No fallback credential path exists.
- **On Authorization Failure**: An identity acting outside its environment scope is denied by IAM; the denial is a CloudTrail-visible event and a review finding, not a silent no-op.
- **On Security-Decision Failure**: Deny by default — if the approval gate, OIDC trust, or review status cannot be established, the deploy does not proceed.
- **On External Dependency Failure**: GitHub Actions or AWS control-plane unavailability stops deployment; the running system is unaffected. Interrupted Terraform applies are surfaced by the pipeline and resolved by re-running the reviewed plan; no partial apply is silently accepted.
- **On System Error**: A failed apply leaves the previous reviewed configuration in effect; pipeline errors never disable the approval gate or widen a role's permissions.
- **Logging / Audit**: CloudTrail records every role assumption and API action in each account; the pipeline records who approved each production deploy and which commit deployed; Terraform changes carry their review record. No credentials appear in workflow logs (SEC-SECRET-1).
- **Alerting**: Drift detection findings, denied cross-environment role use, and failed production-approval bypass attempts route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: N/A for application code — this requirement produces Terraform and workflow configuration; its "unit" layer is static policy checks (checkov rules for wildcard IAM, missing OIDC conditions, public-access defaults) run against the Terraform plan.
- **Integration Tests**: A deploy exercised end-to-end in the dev account asserting OIDC role assumption, scoped permissions, and successful apply; a staging deploy triggered by a `main` merge; a production deploy blocked until approval is granted.
- **Security Tests**: Attempted role assumption with a mismatched repository or environment claim, asserting denial; attempted use of the dev role against production resources, asserting denial; inventory scan asserting no IAM user access keys exist for pipeline identities; a deliberate out-of-band console change in a non-production environment asserting drift detection reports it.
- **Compliance Tests / Evidence**: The IAM policy and trust-configuration review record; the production approval log; retained for the SQ-10 pre-launch assessment.
- **Acceptance-Criteria Traceability**: AC-01 — OIDC deploy integration test plus IAM policy scan; AC-02 — deployment-flow integration test; AC-03 — credential inventory and cross-environment denial tests; AC-04 — drift-detection exercise and branch-protection check; AC-05 — REQ-PIPE-020 provenance review evidence plus pipeline-step review.
- **Coverage Target**: Project coverage threshold TO BE DECIDED (`CLAUDE.md`); every IAM trust and permission policy MUST be exercised by at least one allow and one deny case.
- **Required Test Environment**: The dev AWS account with CloudTrail enabled; a GitHub repository with environments and protection rules configured; checkov in CI.

## Dependencies

- **Upstream Requirements**: REQ-BUILD-010 (the repository, workspace layout, and `infra` workspace this pipeline builds and applies)
- **Downstream Requirements**: REQ-INFRA-020, REQ-INFRA-030, REQ-INFRA-040, REQ-INFRA-050, REQ-INFRA-060, REQ-INFRA-070, REQ-FOOD-050, REQ-PIPE-020 (all assume the accounts, roles, and reviewed-apply path exist)
- **External Dependencies**: GitHub Actions (CI platform); AWS Organizations, IAM, and CloudTrail; the Terraform AWS provider
- **Dependency Assumptions**: GitHub's OIDC issuer signs assertions correctly and environment protection rules enforce the approval; AWS IAM evaluates the trust conditions as configured. Both are verified by the deny-case tests rather than assumed.
- **Failure Impact**: A compromised or over-scoped pipeline identity is a full-system compromise path (TM-T-6): it can rewrite the infrastructure that holds health data. Pipeline unavailability blocks deployment but not the running service.

## Implementation Notes

- **Constraints**: Terraform in the `infra` workspace, GitHub Actions as the platform, per-environment OIDC roles, staging auto / production manual (`CLAUDE.md`, `SECURITY.md` SQ-7 — all fixed); single US primary region (SQ-1); IAM policies avoid wildcard actions and resources (SEC-CICD-3).
- **Prohibited Approaches**: Long-lived AWS access keys or repository-secret cloud credentials for deployment (SEC-CICD-1); one shared deploy role across environments; console or CLI changes to production infrastructure outside the reviewed Terraform path (SEC-CICD-2); OIDC trust policies without repository and environment subject conditions; granting the deploy role standing read access to production health data (SEC-OPS-1 governs human emergency access; the deploy role needs none).
- **Implementation Guidance**: Structure the Terraform as per-environment root modules sharing composable child modules, so an environment's blast radius is its own state. Scope each OIDC trust policy to the exact repository and GitHub environment. Wire drift detection as a scheduled plan whose non-empty diff is a surfaced finding. Keep the production approval list short and named.
- **AI Development Guidance**: `REF-PROMPT-TF-AWS`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of every IAM trust and permission policy and of the environment protection configuration; architecture review that the account topology matches SQ-7.
- **Open Decisions**: None for this issue. Branch naming, PR template, and merge workflow remain TO BE DECIDED (`CLAUDE.md`) but do not block the pipeline's security posture.

**Estimated effort**: 1.5–2 engineer-days. **Estimated changed lines**: 500–1000 (Terraform account/role/pipeline modules and GitHub Actions workflows).
**Recommended model**: Claude Opus (`claude-opus-5`) — the trust policies and approval gating are the system's boundary-4 defense; scoping errors here are silent and total.
