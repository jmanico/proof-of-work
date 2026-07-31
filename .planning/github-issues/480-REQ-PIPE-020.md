# [REQ-PIPE-020] Non-production environments contain no real health data

## Metadata

- **ID**: REQ-PIPE-020
- **Title**: Non-production environments contain no real health data
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-07-31
- **Priority**: High
- **Requirement Type**: Privacy
- **Source / Parent**: REQ-EPIC-001; `SECURITY.md` SEC-CICD-5; `REQUIREMENTS.md` FR-9.1, FR-9.8

## Requirement

- **Statement**: Non-production environments MUST NOT contain real subscriber health data, and any dataset used in development, testing, or staging MUST be synthetic or verifiably de-identified with a documented provenance.
- **Rationale**: SEC-CICD-5 states the rule. Non-production environments have weaker access controls, broader operator access, and no break-glass discipline, so a copy of production data there defeats SEC-OPS-1, SEC-DATA-1, and the audit obligations in FR-9.7 simultaneously.
- **Assumptions**: More than one environment will exist. `SECURITY.md` records environment topology as UNKNOWN (SQ-7), so this issue establishes the rule and the provenance discipline rather than a specific environment set.
- **Out of Scope**: The environment topology and AWS service selection, blocked by `SECURITY.md` SQ-7; encryption at rest for non-production stores (SEC-DATA-1, partly blocked by SQ-7); operator access controls (SEC-OPS-1, blocked by SQ-13); production backup handling and deletion mechanics (SEC-DATA-4, blocked by SQ-5).
- **Design Traceability**: N/A
- **Architecture Traceability**: `ARCHITECTURE.md` — trust boundary 4 (CI/CD and IaC to production) and trust boundary 5 (operational and human access below the application); Relational Persistence ("Stores no data that originates outside the system").
- **Security Traceability**: SEC-CICD-5; supports SEC-DATA-1, SEC-DATA-4, SEC-OPS-1, SEC-TB-3, SEC-LOG-3.

## Scope

- **Applies To**: Background Processing, Server-Side Application, External Integration
- **Components**: Relational Persistence in non-production environments; the CI pipeline; any data-seeding or refresh process
- **Interfaces / Operations**: Environment provisioning; test data seeding; any production-to-non-production data movement
- **Actors**: Developers; operators; the CI/CD identity
- **Preconditions**: A non-production environment exists
- **Data Classification**: Restricted — the rule exists to keep restricted data out of a lower-controlled context
- **Personal or Regulated Data**: Health Data
- **Jurisdiction / Regulatory Scope**: TO BE DECIDED (`SECURITY.md` SQ-1, `REQUIREMENTS.md` OQ-3)

## Security Context

- **Security Objectives**: Confidentiality, Privacy
- **Control Layers**: Data Protection, Architecture, Supply Chain
- **Threat References**: `SECURITY.md` TM-I-8 (operator or support staff reads production health data), TM-I-6 (backups, replicas, and snapshots hold over-retained health data), TM-P-1 (identifiability); CWE-359 (exposure of private personal information), CWE-1272 (sensitive information uncleared before debug or transfer)
- **Abuse / Misuse Case**: A production database snapshot is restored into staging to reproduce a bug; the snapshot then sits in an environment with shared credentials, no audit trail, and no deletion process, where every developer can read subscribers' weight, measurement, and workout history.
- **Trust Boundary**: Boundary 5 — human and operational access below the application, which non-production environments expose far more broadly than production.
- **Untrusted Inputs or Assertions**: Any dataset whose provenance is not documented; a claim that a dataset has been anonymized without verification.
- **Authoritative Enforcement Point**: The environment provisioning and data-seeding process, plus a documented provenance review.
- **Independent Verification**: A data-provenance review of every non-production dataset, which is the verification SEC-CICD-5 itself specifies.
- **Zero Trust Relevance**: N/A — data placement, not a resource-access decision.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session.
- **OWASP AISVS 1.0**: N/A
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: N/A
- **Regulatory**: TO BE DECIDED — blocked by `SECURITY.md` SQ-1; copying health data into a lower-controlled environment is a processing activity under every regime `REQUIREMENTS.md` names.
- **Other**: `REF-SSDF` as cited by SEC-CICD-5.
- **Mapping Basis**: SEC-CICD-5 is the normative rule and cites `REF-SSDF`; the CWE identifiers name the exposure and uncleared-data classes.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a non-production environment, when its datasets are reviewed, then every record has documented synthetic or verifiably de-identified provenance, and no record traces to a real subscriber.
2. **AC-02 — Boundary or failure behavior**: Given an attempt to copy, restore, or replicate production data into a non-production environment, when it is initiated, then it is refused by process and, where the topology permits, by configuration, and the attempt is recorded.
3. **AC-03 — Prohibited behavior**: Given any non-production environment, when it is used, then it MUST NOT contain a production database snapshot, a production backup restore, production log exports containing personal data, or an export artifact generated for a real subscriber (SEC-DATA-6).
4. **AC-04 — Additional criterion**: Given the test data generator, when it produces health records, then the values are synthetic and no generated identifier, email address, or name collides with a real account.
5. **AC-05 — Additional criterion**: Given a bug that appears to require production data to reproduce, when it is investigated, then the investigation uses the break-glass process against production (SEC-OPS-1) rather than moving data to a lower environment — noting that SEC-OPS-1's mechanism is blocked by `SECURITY.md` SQ-13, so this route is currently undefined.

## Failure Behavior

- **On Invalid Input**: A dataset without documented provenance is not loaded.
- **On Authentication Failure**: N/A
- **On Authorization Failure**: N/A
- **On Security-Decision Failure**: If a dataset's provenance cannot be established, treat it as production-derived and refuse it (fail closed).
- **On External Dependency Failure**: N/A
- **On System Error**: A failed seeding run leaves the environment empty rather than partially populated from an unknown source.
- **Logging / Audit**: Record each non-production dataset's provenance and load time. Any attempt to move production data downward is recorded as a security-relevant event (SEC-LOG-4).
- **Alerting**: TO BE DECIDED — no alerting model exists in the source documents; a production-snapshot restore into a non-production account is a strong alert candidate once the AWS topology is defined (SQ-7).

## Test Strategy

- **Unit Tests**: The test data generator produces values from synthetic ranges and never reads from a production source.
- **Integration Tests**: Seed a non-production environment and assert every record originates from the generator.
- **Security Tests**: Data-provenance review of every non-production dataset — the verification SEC-CICD-5 specifies; a scan of non-production stores for patterns matching real account identifiers or email domains; review that no pipeline step copies data from production.
- **Compliance Tests / Evidence**: The provenance review record for each environment, retained as privacy evidence.
- **Acceptance-Criteria Traceability**: AC-01 — provenance review; AC-02 — pipeline and process review; AC-03 — non-production store scan; AC-04 — generator collision test; AC-05 — documented investigation procedure.
- **Coverage Target**: Every non-production environment and every seeded dataset reviewed.
- **Required Test Environment**: At least one non-production environment and the seeding process — environment topology TO BE DECIDED (`SECURITY.md` SQ-7).

## Dependencies

- **Upstream Requirements**: REQ-PLAN-010, REQ-PLAN-020, REQ-PROGRESS-010, REQ-PROGRESS-020 — the entity models the generator must produce
- **Downstream Requirements**: Every issue whose test environment needs seeded health data
- **External Dependencies**: None
- **Dependency Assumptions**: The chosen deployment topology allows non-production environments to be isolated from production data stores; this cannot be confirmed while SQ-7 is open.
- **Failure Impact**: A production snapshot in staging exposes all subscriber health data to everyone with development access, outside every control the application enforces, and outside the audit trail FR-9.7 requires.

## Implementation Notes

- **Constraints**: The CI platform is GitHub Actions (`CLAUDE.md`), but AWS services and environment topology remain UNKNOWN (`SECURITY.md` SQ-7), so the configuration half of this control — account separation, network isolation, IAM boundaries preventing cross-environment data movement — cannot be specified. This issue delivers the synthetic data generator and the provenance discipline; the infrastructure enforcement is blocked.
- **Prohibited Approaches**: Restoring a production snapshot "temporarily"; ad-hoc anonymization scripts applied to production exports without verification; seeding from a support ticket attachment; sharing one database across environments.
- **Implementation Guidance**: Build the generator alongside the entity models so that every test environment has realistic data from the start and no one is tempted to reach for production. Generated values should be recognizably synthetic so that a scan can distinguish them.
- **AI Development Guidance**: `REF-SSDF`, `REF-PROMPT-TF-AWS`; `CLAUDE.md`.
- **Required Human Review**: Privacy review of the generator and of each environment's provenance record.
- **Open Decisions**: `SECURITY.md` SQ-7 (environment topology and AWS services), SQ-13 (the break-glass process AC-05 depends on), SQ-5 (deletion mechanics, which determine what happens to any non-production data derived from a deleted account). This issue's coverage of SEC-CICD-5 is therefore process-level, with infrastructure enforcement blocked.

**Estimated effort**: 0.5–1.5 engineer-days. **Estimated changed lines**: 150–400 (generator and documentation).
**Recommended model**: Claude Opus (`claude-opus-5`) — modest code, but the judgement about what counts as de-identified is where this control usually fails.
