# [REQ-INFRA-050] Break-glass operational access

## Metadata

- **ID**: REQ-INFRA-050
- **Title**: Break-glass operational access
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `SECURITY.md` SEC-OPS-1 (SQ-13 RESOLVED); threats TM-I-8, TM-R-2

## Requirement

- **Statement**: Human access to production health data below the application layer — direct database access, backup restoration, and support tooling — MUST be denied by default and possible only through a dedicated break-glass IAM role that activates only with a second admin's approval, is time-boxed to 4 hours, records a per-use justification, logs the full session via CloudTrail and SSM Session Manager, and receives a post-use review within 7 days; no standing human credential may read production health data.
- **Rationale**: SEC-OPS-1 defines the mechanism and SQ-13 fixed its parameters. Operators and holders of persistence or backup credentials reach health data without traversing the REST API's enforcement point (`ARCHITECTURE.md` trust boundary 5), so the application-layer controls (SEC-AUTHZ-*, SEC-LOG-1) cannot see or stop them; the threat model rates direct operator reads of production health data (TM-I-8) as high severity. Two-person activation, a hard time box, session recording, and mandatory review convert an invisible standing capability into a rare, justified, fully observable event.
- **Assumptions**: The environment topology is fixed (`SECURITY.md` SQ-7 RESOLVED): production is a separate AWS account with RDS PostgreSQL in isolated subnets, and only the production account holds real health data (SEC-CICD-5). Tamper evidence for audit storage below the application is delivered separately by SEC-LOG-7 (REQ-INFRA-040); this issue governs who can reach the data at all.
- **Out of Scope**: Application-layer authorization (REQ-AUTHZ-010 through REQ-AUTHZ-060); audit append-only privileges and the hash-chained archive (REQ-INFRA-040 — SEC-LOG-7); the incident-response process that consumes break-glass review items (REQ-OPS-010 — SEC-OPS-2); the CI/CD migration path and its identities (REQ-INFRA-010 — the drizzle-kit and dataset-import path over trust boundary 4 is a sanctioned machine path under DR-5, not human access).
- **Design Traceability**: N/A — `DESIGN.md` governs the browser client; break-glass access has no user-facing surface.
- **Architecture Traceability**: `ARCHITECTURE.md` — trust boundary 5 (application → human operational access below it); DR-5, which enumerates the SEC-OPS-1 break-glass process as the only sanctioned human path to Relational Persistence outside the application and CI/CD paths.
- **Security Traceability**: SEC-OPS-1 (primary); SEC-CICD-3 (IAM least privilege, no wildcards); SEC-AUTHZ-9 (operational emergencies use break-glass, never an admin application capability); SEC-OPS-2 (every activation is auto-reviewed as a potential incident signal); SEC-LOG-7 (tamper evidence complements this rule).

## Scope

- **Applies To**: Multiple — AWS infrastructure (Terraform) and the operational process bound to it
- **Components**: Relational Persistence and its backups; deployment infrastructure (IAM, CloudTrail, SSM Session Manager); operational-access boundary (trust boundary 5)
- **Interfaces / Operations**: Direct database connections, backup restoration, and any support tooling that reads production data; break-glass role activation, approval, session logging, and post-use review
- **Actors**: Operators, support staff, and any human holding AWS credentials in the production account; the second admin who approves activation; the security lead who reviews use
- **Preconditions**: Production environment provisioned (REQ-INFRA-010) with network tiering and encryption in place (REQ-INFRA-020)
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED)

## Security Context

- **Security Objectives**: Confidentiality, Accountability, Authorization, Privacy
- **Control Layers**: Architecture, Authorization, Logging and Monitoring, Data Protection
- **Threat References**: `SECURITY.md` TM-I-8 (operator reads production health data directly), TM-R-2 (audit entries alterable below the application); STRIDE Information Disclosure / Elevation of Privilege at trust boundary 5; CWE-284 (improper access control)
- **Abuse / Misuse Case**: A malicious or careless operator uses a standing database, backup, or support-tooling credential to read or exfiltrate subscriber health data without any application-layer audit entry; or an attacker who compromises one operator identity attempts to reach production data alone.
- **Trust Boundary**: Boundary 5 — between the application and human operational access below it.
- **Untrusted Inputs or Assertions**: Any human claim of operational necessity — justification is recorded and reviewed, never self-certified into standing access; a single approver's identity assertion (a second admin must approve).
- **Authoritative Enforcement Point**: AWS IAM in the production account: deny-by-default policies for human principals over the data tier, with the dedicated break-glass role as the only grant path, defined in Terraform (SEC-CICD-2).
- **Independent Verification**: Activation requires a second admin's approval, so no single actor can grant themselves access; CloudTrail and SSM session logs are produced by the platform, independently of the actor being logged.
- **Zero Trust Relevance**: NIST SP 800-207 — access is granted per-session, on a per-use decision, with no standing entitlement. Exact tenet: TO BE DECIDED (not verified against the publication in this session).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session (SQ-10 pre-launch assessment).
- **OWASP AISVS 1.0**: N/A — no AI-enabled component is involved.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: The SQ-1 regime set applies to the health data this control protects — GDPR/UK GDPR, CCPA/CPRA, Washington My Health My Data, FTC HBNR; HIPAA not applicable. Statute-section mappings: TO BE DECIDED (SQ-1 counsel review).
- **Other**: `REF-PROMPT-TF-AWS`, `REF-LOG`, `REF-ASVS-5` as cited by SEC-OPS-1; `REF-IAC` via SEC-CICD-2/SEC-CICD-3.
- **Mapping Basis**: SEC-OPS-1 names its references directly; the IAM implementation is governed by the Terraform and IaC rules; CWE-284 describes the improper-access-control class this rule closes.

## Acceptance Criteria

1. **AC-01 — Deny by default**: Given the production account with no active break-glass grant, when any human principal attempts direct database access, backup restoration, or support-tooling reads of health data, then the attempt is denied, and a review of production IAM policies finds no standing human credential with read access to production health data.
2. **AC-02 — Expected activation behavior**: Given a requesting operator, a recorded per-use justification, and a second admin's approval, when the break-glass role is activated, then the issued credentials are time-boxed to at most 4 hours, the full session is logged via CloudTrail and SSM Session Manager, and a post-use review item is opened with a 7-day deadline.
3. **AC-03 — Boundary or failure behavior**: Given an activation request without a second admin's approval, or without a recorded justification, when activation is attempted, then it is refused; and given an active grant, when the 4-hour time box elapses, then the credentials no longer permit access.
4. **AC-04 — Prohibited behavior**: Given any single actor, including the requesting operator acting as their own approver, when they attempt to activate the break-glass role alone, then activation MUST NOT succeed; and no activation may occur without producing the justification record, the session logs, and the review item.
5. **AC-05 — Drill evidence**: Given a break-glass drill in a pre-production rehearsal of the production configuration, when the full activate-access-expire cycle runs, then the expected audit trail — approval record, justification, CloudTrail events, SSM session log, review item — is produced and retrievable.

## Failure Behavior

- **On Invalid Input**: A malformed or incomplete activation request (missing justification, unidentifiable approver) is refused with no grant issued.
- **On Authentication Failure**: N/A at the application layer — AWS IAM authentication governs; an unauthenticated principal reaches nothing in the data tier.
- **On Authorization Failure**: Deny. Denied attempts to reach the data tier are visible in CloudTrail; no resource existence needs concealment from authenticated operators.
- **On Security-Decision Failure**: Deny by default. If the approval mechanism, logging pipeline, or session-manager path is unavailable or errors, activation MUST NOT proceed — an unlogged grant is worse than a delayed one.
- **On External Dependency Failure**: If CloudTrail or SSM Session Manager cannot record the session, access MUST NOT be granted; there is no degraded unlogged mode.
- **On System Error**: An error during activation leaves no partial grant; credentials are either fully issued with logging active or not issued.
- **Logging / Audit**: Per activation: requesting identity, approving admin, justification text, activation and expiry times, and the complete CloudTrail and SSM session records. No health values are copied into review artifacts (SEC-LOG-3).
- **Alerting**: Every break-glass activation alerts the security lead and automatically opens a review item as a potential incident signal (SEC-OPS-1, SEC-OPS-2); threshold alerts route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: N/A for application code — this control is IAM policy and process; Terraform module logic is exercised by IaC scanning and plan assertions.
- **Integration Tests**: Terraform plan/apply assertions in a pre-production account: break-glass role exists with a 4-hour maximum session duration; activation path requires the approval step; SSM Session Manager is the only interactive path to the data tier.
- **Security Tests**: checkov IaC scan (SEC-CICD-4) plus explicit policy assertions that no human principal outside the break-glass role holds read access to RDS, its snapshots, or backups; attempted direct access without an active grant is denied; attempted self-approval fails; expired credentials are refused.
- **Compliance Tests / Evidence**: The documented break-glass procedure; the drill record with its full audit trail (AC-05); the post-use review log showing 7-day completion.
- **Acceptance-Criteria Traceability**: AC-01 — IAM policy review plus denied-access test; AC-02 — activation integration test and audit-trail assertion; AC-03 — refusal and expiry tests; AC-04 — self-approval negative test; AC-05 — documented drill evidence.
- **Coverage Target**: Every human access path to production health data enumerated and asserted denied or break-glass-gated; both negative activation cases (no approval, no justification) tested.
- **Required Test Environment**: A pre-production AWS account mirroring the production IAM and network configuration (real health data never leaves production, SEC-CICD-5); Terraform with checkov in the pipeline; two test operator identities and one approver identity.

## Dependencies

- **Upstream Requirements**: REQ-INFRA-010 (environment accounts and pipeline identities), REQ-INFRA-020 (network tiering and encryption — the data tier this rule guards)
- **Downstream Requirements**: REQ-OPS-010 (break-glass activations feed incident review); REQ-INFRA-040 (tamper-evident audit storage complements this control against the same actors)
- **External Dependencies**: AWS IAM, CloudTrail, and SSM Session Manager in the production account
- **Dependency Assumptions**: CloudTrail and SSM session logs are written to storage the accessing operator cannot alter (log-archive separation per SEC-LOG-7); IAM enforces the 4-hour maximum session duration platform-side.
- **Failure Impact**: If this control is absent or misconfigured, any credential holder can read production health data invisibly — the exact TM-I-8 scenario — and no application-layer control compensates.

## Implementation Notes

- **Constraints**: Terraform-managed AWS infrastructure only (SEC-CICD-2); IAM policies without wildcard actions or resources (SEC-CICD-3); the break-glass role and its approval mechanism are code-reviewed infrastructure, not console-created.
- **Prohibited Approaches**: Standing human read credentials to the production database, snapshots, or backups; shared operator accounts; self-approved activation; granting the application's database identity to humans; an "emergency" path that bypasses session logging; building an admin application capability that reads health data instead (SEC-AUTHZ-9 forbids it).
- **Implementation Guidance**: Model activation as an IAM role assumption gated on a second party — e.g. a permission-set or role whose assumption requires an approval workflow — with `MaxSessionDuration` at 4 hours and interactive access forced through SSM Session Manager so sessions are recorded. Keep the justification record and review item in the same tracking system SEC-OPS-2 uses, so the auto-review wiring is one integration, not two.
- **AI Development Guidance**: `REF-PROMPT-TF-AWS`; `CLAUDE.md` (checkov is a merge-blocking gate; infra lives in `infra/`).
- **Required Human Review**: Security review of the IAM policy set and activation workflow; the security lead confirms the review-item wiring into SEC-OPS-2 before production.
- **Open Decisions**: None. SQ-13 fixed the mechanism and parameters; SQ-7 fixed the account topology.

**Estimated effort**: 1–2 engineer-days. **Estimated changed lines**: 300–700.
**Recommended model**: Claude Opus (`claude-opus-5`) — IAM deny-by-default policy design where a single overlooked grant silently defeats the control.
