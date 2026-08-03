# [REQ-INFRA-020] Network tiering and encryption at rest and in transit

## Metadata

- **ID**: REQ-INFRA-020
- **Title**: Network tiering and encryption at rest and in transit
- **Version**: 1.0.1
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `SECURITY.md` SEC-CICD-3, SEC-TB-2, SEC-DATA-1, SQ-6, SQ-7 (RESOLVED); `REQUIREMENTS.md` FR-9.8; `ARCHITECTURE.md` DR-5

## Requirement

- **Statement**: Each environment MUST be provisioned as a VPC in which CloudFront is the single public origin — serving the static client from S3 and forwarding the API path prefix to the ALB — with the API in private subnets, RDS PostgreSQL in isolated subnets reachable only through the `ARCHITECTURE.md` DR-5 sanctioned paths, NAT egress restricted to named endpoints including SES, KMS encryption at rest on database storage, backups, snapshots, and replicas, and TLS enforced on all external traffic and on every database connection.
- **Rationale**: SEC-TB-2 requires that Relational Persistence accept connections only from the REST API Application and Identity and Session Handling and never be reachable from the public network; SEC-DATA-1 requires health data encrypted in transit and at rest including backups, snapshots, and replicas; SEC-CICD-3 requires private data-tier placement, no wildcard IAM, blocked public storage access, and NAT egress restricted to named endpoints — the email service among them (SEC-EXT-3, 2026-08-03). `SECURITY.md` SQ-7's 2026-08-03 addendum fixes CloudFront as the single public origin forwarding the API path prefix to the ALB, realizing SQ-6's same-origin surface. FR-9.8 makes restricted egress a functional obligation: health data must be unable to leave the boundary.
- **Assumptions**: The environment accounts, Terraform pipeline, and reviewed-apply path exist (REQ-INFRA-010); KMS keys are managed within each environment account.
- **Out of Scope**: Provisioning accounts, pipeline identities, and the deployment flow (REQ-INFRA-010); secret and signing-key storage (REQ-INFRA-030); the log-archive account's audit and ledger storage (REQ-INFRA-040); break-glass access through the network to production data (SEC-OPS-1 — REQ-INFRA-050); SES sending behavior and the mail interface (SEC-EXT-3 — REQ-INFRA-060); the Bedrock endpoint's inference-specific configuration (SEC-AI-1 — REQ-FOOD-050, which adds Bedrock to the named-egress set this issue implements); application-layer TLS behavior such as HSTS headers (SEC-HTTP-1 — REQ-API-010 family).
- **Design Traceability**: N/A — `DESIGN.md` governs the browser client, not network infrastructure.
- **Architecture Traceability**: `ARCHITECTURE.md` — Deployment model ("CloudFront is the single public origin: it serves the static client from S3 and forwards the API path prefix to the ALB"); trust boundary 3 (REST API Application → Relational Persistence); DR-5 (sanctioned persistence paths: the REST API Application including its scheduled executions, Identity and Session Handling, the CI/CD migration-and-import path via boundary 4, and the SEC-OPS-1 break-glass process via boundary 5).
- **Security Traceability**: SEC-CICD-3, SEC-TB-2, SEC-DATA-1; supports SEC-TB-3, SEC-HTTP-1, SEC-HTTP-3, SEC-EXT-3, SEC-AI-1, FR-9.8.

## Scope

- **Applies To**: Multiple — Server-Side Application deployment, API surface exposure, persistence placement
- **Components**: Deployment (Terraform, `infra` workspace); Relational Persistence (placement and encryption); REST API Application and Identity and Session Handling (network placement); Browser Client (served via CloudFront/S3)
- **Interfaces / Operations**: VPC, subnet, security-group, and routing definitions; CloudFront distribution and origin configuration; ALB listeners; RDS instance, backup, snapshot, and replica encryption settings; NAT gateway and egress rules; database TLS enforcement
- **Actors**: Anonymous internet attacker (probing the public surface); operators and CI/CD identities changing network configuration (boundary 4); the application's runtime identities
- **Preconditions**: REQ-INFRA-010 accounts and pipeline provisioned
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data — the network and encryption posture is what stands between the public internet and the health-data store
- **Jurisdiction / Regulatory Scope**: Per `SECURITY.md` SQ-1 (RESOLVED): GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable. Single US primary region.

## Security Context

- **Security Objectives**: Confidentiality, Integrity, Availability, Privacy
- **Control Layers**: Architecture, Data Protection
- **Threat References**: `SECURITY.md` TM-I-6 (backups, replicas, snapshots hold unencrypted health data), TM-I-5 (health data leaks through outbound paths), TM-T-2 (the data tier as an injection target — network placement is its depth defense); CWE-311 (missing encryption of sensitive data); further CWE/CAPEC mappings TO BE DECIDED (not verified this session)
- **Abuse / Misuse Case**: An attacker connects to the database or ALB directly from the internet, bypassing CloudFront; a compromised in-VPC process exfiltrates health data to an arbitrary internet host through unrestricted egress; an unencrypted snapshot or replica is copied or shared; a plaintext database connection is sniffed inside the VPC; an S3 bucket is flipped public.
- **Trust Boundary**: Boundary 3 (REST API Application → Relational Persistence) for reachability; boundary 1's network edge (the public surface is CloudFront alone); boundary 4 for every change to this configuration.
- **Untrusted Inputs or Assertions**: All traffic arriving at the public surface; any connection attempt to the data tier regardless of origin; any claim that an egress destination is "needed" without appearing in the named-endpoint set.
- **Authoritative Enforcement Point**: AWS network controls defined in Terraform — security groups, subnet routing, CloudFront origin configuration, NAT egress rules — plus RDS/KMS encryption settings and the RDS TLS-enforcement parameter; changed only through the SEC-CICD-2 reviewed path.
- **Independent Verification**: Reachability probes run from outside the VPC and from non-sanctioned segments inside it, independently of what the Terraform author asserts; checkov reads the plan; encryption settings are read from the live account.
- **Zero Trust Relevance**: TO BE DECIDED — not verified against NIST SP 800-207 in this session.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — mapping deferred to the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A — no AI-enabled component in this requirement.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: Encryption at rest and in transit and boundary containment of health data support the SQ-1 regime set (GDPR/UK GDPR; CCPA/CPRA; Washington My Health My Data; FTC HBNR). Specific article/section mappings: TO BE DECIDED — the spec documents state no section for this behavior.
- **Other**: `REF-PROMPT-TF-AWS`, `REF-ASVS-5` (cited by SEC-TB-2 and SEC-DATA-1); `REF-IAC` (SEC-CICD-3).
- **Mapping Basis**: The listed references are those SEC-TB-2, SEC-DATA-1, and SEC-CICD-3 themselves cite.

## Acceptance Criteria

1. **AC-01 — Expected behavior (single public origin)**: Given a deployed environment, when its public surface is enumerated from the internet, then CloudFront is the only reachable origin — it serves the client from S3 and forwards the API path prefix to the ALB — and the ALB, API tasks, and RDS instance are not directly reachable from the public network.
2. **AC-02 — Boundary or failure behavior (data-tier reachability)**: Given the deployed network, when connection to the RDS endpoint is attempted from the public internet and from an in-VPC segment outside the DR-5 sanctioned paths, then every attempt fails at the network layer, while the REST API Application's tasks — including its scheduled executions — connect successfully.
3. **AC-03 — Prohibited behavior (encryption)**: Given the RDS instance and its automated backups, snapshots, and any replica, when their configuration is read from the account, then all are KMS-encrypted, S3 buckets block public access, and a database connection attempt without TLS is refused; an unencrypted snapshot or replica MUST NOT be creatable from the Terraform-managed resources.
4. **AC-04 — Prohibited behavior (restricted egress)**: Given the API's private subnets, when egress reachability is probed, then only the named endpoints — including SES (SEC-EXT-3) and the endpoints later added under REQ-FOOD-050 — are reachable, and connections to arbitrary internet hosts are denied, so no path exists for health data to leave the boundary (FR-9.8).

## Failure Behavior

- **On Invalid Input**: N/A — this requirement governs infrastructure configuration, not request content.
- **On Authentication Failure**: N/A — network-layer denial precedes authentication; application authentication is governed elsewhere.
- **On Authorization Failure**: A connection attempt from a non-sanctioned origin is dropped or refused at the security-group/routing layer; no application response discloses the data tier's existence.
- **On Security-Decision Failure**: Deny by default — security groups and NACLs default-deny; an absent or failed rule blocks traffic rather than admitting it; a failed TLS negotiation to the database refuses the connection rather than downgrading.
- **On External Dependency Failure**: CloudFront or ALB failure makes the service unavailable; it MUST NOT expose a bypass origin. KMS unavailability blocks storage operations rather than proceeding unencrypted.
- **On System Error**: A failed Terraform apply leaves the previous reviewed topology in effect; partial network changes are surfaced by the pipeline and drift detection.
- **Logging / Audit**: VPC Flow Logs on the data-tier and API subnets; CloudTrail for configuration changes; rejected-connection records available for review. No health data appears in network logs (SEC-LOG-3 governs application logs; flow logs carry metadata only).
- **Alerting**: Configuration drift on network or encryption resources and anomalous denied-connection volume against the data tier route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: N/A for application code — the static layer is checkov/IaC policy checks against the Terraform plan: encryption flags on RDS and backups, public-access blocks on S3, no `0.0.0.0/0` ingress to private tiers, no wildcard IAM, TLS-enforcement parameter set.
- **Integration Tests**: End-to-end request through CloudFront reaching the API and the client assets; database connection from an API task succeeding over TLS; a scheduled-execution task connecting through the same sanctioned identity.
- **Security Tests**: External scan asserting only CloudFront answers publicly (AC-01); reachability probes to the RDS endpoint from the internet and from a non-sanctioned in-VPC host asserting failure (SEC-TB-2 verification); plaintext database connection attempt asserting refusal; egress probes from the API subnet asserting only named endpoints resolve and connect (FR-9.8 egress review); attempt to create a public S3 object asserting the block.
- **Compliance Tests / Evidence**: The encryption-configuration evidence (storage, backups, replicas) and the egress review record, retained for the SQ-1 counsel review and the SQ-10 assessment.
- **Acceptance-Criteria Traceability**: AC-01 — external enumeration test; AC-02 — reachability probe suite; AC-03 — encryption configuration checks and plaintext-connection test; AC-04 — egress probe suite.
- **Coverage Target**: Project coverage threshold 90% line and branch (`CLAUDE.md`, 2026-08-03); every tier boundary MUST have at least one allowed-path and one denied-path probe.
- **Required Test Environment**: A deployed dev-account environment with an in-VPC probe host (or equivalent SSM-run probe) in a non-sanctioned segment, an external scan runner, and checkov in CI.

## Dependencies

- **Upstream Requirements**: REQ-INFRA-010 (accounts, pipeline identities, and the reviewed Terraform path this topology is applied through)
- **Downstream Requirements**: REQ-INFRA-030 (secrets resolved inside this network), REQ-INFRA-040 (log-archive delivery crosses this egress posture), REQ-INFRA-050 (break-glass is the sole sanctioned human path into the data tier), REQ-INFRA-060 (SES as a named egress endpoint), REQ-FOOD-050 (Bedrock as a named egress endpoint), and every runtime feature issue that assumes the data tier is unreachable except through the API
- **External Dependencies**: AWS VPC, CloudFront, S3, ALB, RDS, KMS, NAT; the Terraform AWS provider
- **Dependency Assumptions**: AWS enforces security-group, routing, KMS, and Object-ownership semantics as documented — verified by the probe tests rather than assumed; CloudFront origin access control keeps S3 private.
- **Failure Impact**: A tiering or egress misconfiguration is a direct health-data exposure path (TM-I-5, TM-I-6): a public database, an unencrypted snapshot, or open egress each defeat multiple application-layer controls at once.

## Implementation Notes

- **Constraints**: Terraform in the `infra` workspace applied via REQ-INFRA-010's reviewed pipeline; single US primary region (SQ-1); RDS PostgreSQL Multi-AZ with 35-day automated backups (SQ-7, SQ-5); the topology is fixed by SQ-7 and its 2026-08-03 addendum — deviations are spec changes, not implementation choices.
- **Prohibited Approaches**: Exposing the ALB, API tasks, or RDS directly to the internet; a second public origin bypassing CloudFront (it would break the SQ-6 same-origin surface); unrestricted (`0.0.0.0/0`) egress from private subnets; unencrypted storage, backups, snapshots, or replicas; disabling database TLS enforcement; wildcard IAM actions or resources (SEC-CICD-3); security-group rules referencing broad CIDR ranges where security-group references suffice.
- **Implementation Guidance**: Prefer VPC interface endpoints for AWS services (SES, Secrets Manager, and later Bedrock) over NAT-path internet egress — it makes the named-endpoint set enforceable as endpoint policies and shrinks the NAT rule surface. Reference security groups by ID (API-SG → DB-SG) rather than CIDR so the sanctioned-path set in DR-5 is structural. Enforce database TLS with the RDS parameter (`rds.force_ssl`) plus certificate verification in the application's connection configuration.
- **AI Development Guidance**: `REF-PROMPT-TF-AWS`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the security-group graph, egress endpoint set, CloudFront origin configuration, and KMS key policies; architecture review that the deployed topology matches SQ-7 and DR-5.
- **Open Decisions**: None. The named-egress set grows only through reviewed changes that themselves cite a rule (SES here; Bedrock via REQ-FOOD-050).

**Estimated effort**: 1.5–2 engineer-days. **Estimated changed lines**: 500–1000 (Terraform network, CloudFront, RDS, and KMS modules plus probe fixtures).
**Recommended model**: Claude Opus (`claude-opus-5`) — the security-group graph, origin configuration, and egress set must be exactly right; every error here is a silent boundary hole.
