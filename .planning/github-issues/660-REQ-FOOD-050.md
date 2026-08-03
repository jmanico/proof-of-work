# [REQ-FOOD-050] In-boundary inference service configuration

## Metadata

- **ID**: REQ-FOOD-050
- **Title**: In-boundary inference service configuration
- **Version**: 1.0.1
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-8.13, FR-9.8; `SECURITY.md` SEC-AI-1, SQ-7 (RESOLVED)

## Requirement

- **Statement**: AI inference for nutrition estimation MUST execute only on Amazon Bedrock within the system's own AWS account, configured for zero prompt retention and no training use with that configuration verified rather than assumed, with the model identifier and version pinned in configuration and changeable only through the reviewed SEC-CICD-2 infrastructure path, and with no third-party inference endpoint reachable from the system.
- **Rationale**: FR-8.13 requires inference inside the system's cloud boundary with no user data transmitted to any third-party application service (FR-9.8); SEC-AI-1 makes the zero-retention configuration a verified property and pins the Bedrock model identifier and version behind SEC-CICD-2 review (2026-08-03) so a model swap is a reviewed infrastructure change, not a runtime drift. `SECURITY.md` SQ-7 resolved the service choice: Bedrock in-account with zero-retention, no-training configuration.
- **Assumptions**: The per-environment AWS accounts, Terraform pipeline, and OIDC deploy roles exist (REQ-INFRA-010); the estimation request/response behavior at trust boundary 6 is specified separately (REQ-FOOD-040).
- **Out of Scope**: The estimation flow itself — consent gates, photo validation, response validation, rate limits (REQ-FOOD-040; SEC-AI-2, SEC-AI-3, SEC-INPUT-7, SEC-HTTP-5); general network tiering and NAT egress restriction (REQ-INFRA-020; SEC-CICD-3) — this issue adds the inference-specific endpoint to that regime and verifies no third-party alternative is reachable; model quality, sizing, and cost, which `ARCHITECTURE.md` records as riding with deployment.
- **Design Traceability**: N/A — `DESIGN.md` governs the estimation presentation (REQ-FOOD-040), not the inference infrastructure.
- **Architecture Traceability**: `ARCHITECTURE.md` — AI Inference component ("a managed in-account model service configured for zero prompt retention and no training use"; open decisions: none); trust boundary 6 (REST API Application → AI Inference, "inference runs in-account under zero-retention configuration"); Deployment model (Amazon Bedrock in-account); DR-7 (an internally defined interface owned by the REST API Application, no vendor-specific behavior propagating past it).
- **Security Traceability**: SEC-AI-1; supports SEC-TB-3, SEC-CICD-2, SEC-CICD-3, SEC-EXT-1.

## Scope

- **Applies To**: Server-Side Application
- **Components**: AI Inference; REST API Application (the sole caller); deployment (Terraform, `infra` workspace)
- **Interfaces / Operations**: The Bedrock invocation interface owned by the REST API Application; the Terraform resources and configuration that pin the model identifier and version, IAM invocation policy, and egress rules
- **Actors**: The REST API Application's task IAM role (only permitted invoker); operators and CI/CD identities that change infrastructure (boundary 4)
- **Preconditions**: Production-shaped environment provisioned per REQ-INFRA-010
- **Data Classification**: Restricted — subscriber food descriptions and transient photos transit this service
- **Personal or Regulated Data**: Health Data — an estimation request is collection and processing of health data (FR-9.12)
- **Jurisdiction / Regulatory Scope**: Per `SECURITY.md` SQ-1 (RESOLVED): GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable. Single US primary region.

## Security Context

- **Security Objectives**: Confidentiality, Privacy, Integrity, Safety
- **Control Layers**: Architecture, Data Protection, Supply Chain, Authorization
- **Threat References**: `SECURITY.md` Threat Model addendum 2026-08-01 — model/dataset supply chain; TM-T-6 (IaC compromise deploys attacker-controlled infrastructure, the path a hostile model swap would take); TM-I-5 (health data leaking outward); CWE-1104-adjacent supply-chain concern for the pinned model version — exact CWE mapping TO BE DECIDED (not verified this session)
- **Abuse / Misuse Case**: An operator or compromised pipeline repoints inference at a third-party endpoint or an unreviewed model version, silently changing where subscriber food data flows or what output quality subscribers receive; a retention or training toggle is left at a default that stores prompts; a component other than the REST API Application invokes the model directly.
- **Trust Boundary**: Boundary 6 (REST API Application → AI Inference) for placement; boundary 4 (CI/CD-and-IaC → production) for every change to this configuration.
- **Untrusted Inputs or Assertions**: Vendor console defaults and any undocumented retention behavior — the zero-retention posture MUST be verified against the account's actual configuration, not assumed from documentation; any proposed model-identifier change arriving outside the reviewed Terraform path.
- **Authoritative Enforcement Point**: Terraform-managed configuration in the `infra` workspace (model pin, IAM invocation policy, egress rules), applied only through the SEC-CICD-2 reviewed pipeline; the REST API Application's internal inference interface (DR-7) as the sole invocation site.
- **Independent Verification**: Infrastructure review and automated checks read the deployed configuration from AWS, independently of what the application or the Terraform author asserts; egress verification probes from inside the VPC rather than trusting the rule set.
- **Zero Trust Relevance**: TO BE DECIDED — not verified against NIST SP 800-207 in this session.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — mapping deferred to the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: TO BE DECIDED — SEC-AI-3 records the AISVS mapping for the AI component as pending with SQ-10.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: Keeping health-data processing inside the operator's own boundary supports the SQ-1 regime set (GDPR/UK GDPR; CCPA/CPRA; Washington My Health My Data; FTC HBNR) by avoiding any third-party processor for inference. Specific article/section mappings: TO BE DECIDED — the spec documents state no section for this behavior.
- **Other**: `REF-ASVS-5`, `REF-PROMPT-TF-AWS` (cited by SEC-AI-1); `REF-IAC` (SEC-CICD-2).
- **Mapping Basis**: The listed references are those SEC-AI-1 and SEC-CICD-2 themselves cite.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given the provisioned environment, when the REST API Application requests an estimation, then the invocation goes to Amazon Bedrock in the system's own AWS account and region using the task's IAM role, against exactly the model identifier and version pinned in the Terraform-managed configuration.
2. **AC-02 — Boundary or failure behavior (verified zero retention)**: Given the deployed environment, when the infrastructure verification runs, then it produces evidence read from the account's actual Bedrock configuration that prompt retention is disabled and no training use is enabled, and the verification fails — blocking the pipeline — if either setting deviates; documentation alone MUST NOT satisfy it.
3. **AC-03 — Prohibited behavior (no third-party inference)**: Given the environment's egress rules, when reachability of third-party inference endpoints is tested from the API's network segment, then no such endpoint is reachable, Bedrock is reachable only via the named in-account endpoint, and no IAM identity other than the REST API Application's task role holds Bedrock invocation permission.
4. **AC-04 — Prohibited behavior (unreviewed model change)**: Given the pinned model identifier and version, when a change is attempted outside the SEC-CICD-2 reviewed Terraform path — a console edit, an environment-variable override, or a runtime setting — then the running system continues to invoke the pinned model or the drift is detected and surfaced for review; the model pin MUST NOT be resolvable from any source that bypasses review.

## Failure Behavior

- **On Invalid Input**: N/A — this requirement governs configuration, not request content; hostile estimation inputs are handled by REQ-FOOD-040 (SEC-AI-2, SEC-INPUT-7).
- **On Authentication Failure**: N/A at this layer — IAM denies invocation to any identity other than the task role; the denial is a configuration-verification finding, not a user-facing response.
- **On Authorization Failure**: A Bedrock invocation attempt by any non-authorized identity is denied by IAM policy and visible in CloudTrail.
- **On Security-Decision Failure**: Deny by default — if the pinned model configuration cannot be resolved at startup or the verification check fails, the estimation feature is unavailable rather than falling back to an unpinned or third-party model; the two non-AI food entry methods remain (FR-8.11).
- **On External Dependency Failure**: Bedrock unavailability makes estimation fail closed with a generic error to the caller (handled per REQ-FOOD-040); it MUST NOT trigger failover to any endpoint outside the system boundary.
- **On System Error**: Terraform apply failures leave the previous reviewed configuration in effect; partial applies are surfaced by the pipeline; no error path substitutes an unreviewed model or endpoint.
- **Logging / Audit**: Bedrock invocations and IAM denials are recorded in CloudTrail; configuration changes carry the SEC-CICD-2 review record; application logs record inference errors with correlation identifiers but never the prompt content, photo bytes, or estimation values (SEC-LOG-3, SEC-AI-3).
- **Alerting**: Configuration drift on the inference resources (SEC-CICD-2 drift detection) and denied invocation attempts by non-authorized identities route to the security lead as SEC-OPS-2 detection inputs (SQ-11).

## Test Strategy

- **Unit Tests**: The inference interface resolves the model identifier and version solely from the pinned configuration; startup fails closed (estimation disabled, non-AI paths unaffected) when the pin is absent or malformed; no code path accepts a model identifier from request data or ambient environment overrides.
- **Integration Tests**: An estimation invocation in a deployed environment reaches Bedrock in-account with the pinned identifier (asserted via CloudTrail or SDK instrumentation); invocation under a non-task-role identity is denied.
- **Security Tests**: Egress probe from the API's network segment asserting third-party inference endpoints are unreachable (SEC-AI-1 egress review); checkov/IaC scan rules asserting the retention, training, IAM, and egress settings (SEC-CICD-4 gate); a deliberate out-of-band model change in a non-production environment asserting drift detection surfaces it.
- **Compliance Tests / Evidence**: The zero-retention/no-training verification evidence and the egress review record, retained for the SQ-1 pre-launch counsel review and the SQ-10 assessment.
- **Acceptance-Criteria Traceability**: AC-01 — invocation integration test; AC-02 — retention-verification pipeline check; AC-03 — egress probe plus IAM policy test; AC-04 — drift-detection exercise and configuration-resolution unit tests.
- **Coverage Target**: Project coverage threshold 90% line and branch (`CLAUDE.md`, 2026-08-03); every configuration-resolution and fail-closed branch MUST have positive and negative tests.
- **Required Test Environment**: A deployed dev-account environment with Terraform state, CloudTrail access, and an in-VPC probe host or equivalent; checkov in CI.

## Dependencies

- **Upstream Requirements**: REQ-INFRA-010 (environment accounts, pipeline identities, and deployment flow — the SEC-CICD-2 review path this requirement's change control rides on)
- **Downstream Requirements**: REQ-FOOD-040 (the estimation flow assumes this service and its guarantees)
- **External Dependencies**: Amazon Bedrock (in-account managed service); AWS IAM, CloudTrail, and the Terraform AWS provider
- **Dependency Assumptions**: Bedrock's zero-retention and no-training configuration behaves as configured — which is exactly why SEC-AI-1 requires verification of the account's actual settings rather than reliance on vendor documentation; the model version remains available until a reviewed change replaces it.
- **Failure Impact**: Bedrock outage disables estimation only — dataset search and manual entry continue (FR-8.11, FR-8.12). A misconfiguration here is a health-data exposure: prompts retained by the vendor, or subscriber food data flowing to an unreviewed destination.

## Implementation Notes

- **Constraints**: Terraform in the `infra` workspace (`CLAUDE.md`); single US primary region (SQ-1); the invocation interface lives in the REST API Application behind an internally defined seam so no Bedrock-specific type or error propagates into business logic (DR-7); egress is restricted to named endpoints (SEC-CICD-3), with Bedrock added to that named set.
- **Prohibited Approaches**: Assuming zero retention from documentation instead of verifying deployed configuration (SEC-AI-1); resolving the model identifier from a runtime-mutable source outside the reviewed path; granting Bedrock invocation to any identity beyond the API task role; wildcard IAM actions or resources on the inference policy (SEC-CICD-3); any fallback to a third-party inference endpoint; console changes to production inference resources (SEC-CICD-2).
- **Implementation Guidance**: Prefer a VPC endpoint for Bedrock so invocation never traverses the public internet and the egress posture stays auditable. Express the model pin as a named constant in Terraform-supplied configuration consumed at startup, so a change is a diff in review. Encode the retention/training assertions as pipeline checks against the live account rather than a one-time manual review, so AC-02 holds on every deploy.
- **AI Development Guidance**: `REF-PROMPT-TF-AWS`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the IAM policy, egress rules, and retention verification; architecture review that the invocation seam satisfies DR-7. Every subsequent model-version change requires SEC-CICD-2 review by construction.
- **Open Decisions**: OWASP AISVS mapping (`SECURITY.md` SQ-10). Does not block implementation.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 200–450 (Terraform, configuration seam, and verification checks).
**Recommended model**: Claude Opus (`claude-opus-5`) — small surface, but the value is in getting the IAM scope, retention verification, and fail-closed pinning exactly right.
