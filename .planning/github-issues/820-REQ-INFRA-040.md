# [REQ-INFRA-040] Audit retention, append-only privileges, and the hash-chained archive

## Metadata

- **ID**: REQ-INFRA-040
- **Title**: Audit retention, append-only privileges, and the hash-chained archive
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `SECURITY.md` SEC-LOG-5, SEC-LOG-7, SQ-8, SQ-13 (RESOLVED); SEC-DATA-4 (deletion-ledger placement and tombstone derivation, 2026-08-03); `REQUIREMENTS.md` FR-9.7, FR-9.10

## Requirement

- **Statement**: Audit storage MUST enforce the resolved SQ-8 regime: the application's database role holds INSERT and SELECT only on audit tables (UPDATE and DELETE revoked in raw-SQL migrations); a nightly scheduled execution of the REST API Application appends audit batches — with every account identifier replaced by its SEC-DATA-4 tombstone derivation and a SHA-256 hash chain computed over the pseudonymized content — to S3 Object Lock (governance mode) in the separate log-archive account, where the deletion-ledger prefix with 35-day object expiry also lives; security logs are retained 12 months and audit entries 3 years, then deleted, legal holds excepted; and audit entries are readable only by `admin` accounts through an investigative interface whose reads are themselves audited.
- **Rationale**: SEC-LOG-7 makes audit storage tamper-evident against actors below the application layer — SEC-LOG-2 binds only the application, so database-privilege revocation plus a hash-chained Object Lock archive in a separate account is what detects out-of-band edits (threat TM-R-2). Batch-time pseudonymization via the SEC-DATA-4 keyed tombstone derivation means account deletion never requires an archive rewrite, holding the FR-9.10 posture across the archive (2026-08-03). SEC-LOG-5 bounds retention on purpose — access-audit records are themselves sensitive under the SQ-1 GDPR ceiling (threat TM-P-4) — and restricts reads to audited admin-only investigative access. SEC-DATA-4 places the deletion ledger in the log-archive account, outside the restored dataset, expiring at the 35-day backup horizon.
- **Assumptions**: The audit entry model and application-layer append-only behavior exist (REQ-AUDIT-010); the environment accounts and reviewed Terraform path exist (REQ-INFRA-010); the tombstone-derivation key is a managed secret (REQ-INFRA-030, SEC-DATA-4). `ARCHITECTURE.md` (2026-08-03) fixes the nightly archival as a scheduled execution of the REST API Application — same codebase, image, and database identity — not a new component.
- **Out of Scope**: Which events produce audit entries (REQ-AUDIT-010, REQ-AUDIT-020, REQ-AUDIT-030; SEC-LOG-1, SEC-LOG-4, SEC-LOG-6); log-content redaction (SEC-LOG-3 — REQ-AUDIT-040); writing deletion-ledger entries at deletion execution and restore re-deletion (SEC-DATA-4 behavior — REQ-PRIVACY-090); tombstoning audit rows in the primary store on account deletion (FR-9.10 — REQ-PRIVACY-100); break-glass access below the application (SEC-OPS-1 — REQ-INFRA-050); the incident-response restore runbook step (SEC-OPS-2 — REQ-OPS-010).
- **Design Traceability**: `DESIGN.md` — Information Architecture: the admin workspace's Audit destination is the investigative interface's surface; this issue provisions its storage and access constraints, not its pages.
- **Architecture Traceability**: `ARCHITECTURE.md` — Relational Persistence (audit retention resolved: "12-month logs, 3-year audit entries, hash-chained archive"); Initial Architecture ("The REST API Application additionally runs scheduled executions — same codebase, image, and database identity — for the nightly audit archival (SEC-LOG-7), deletion-ledger maintenance (SEC-DATA-4)..."); DR-5 (scheduled executions are a sanctioned persistence path); Data model expectations ("The deletion ledger lives in the log-archive account, outside the restored dataset"); trust boundary 5.
- **Security Traceability**: SEC-LOG-5, SEC-LOG-7; implements the storage half of SEC-DATA-4's ledger; supports SEC-LOG-2, FR-9.10, SEC-AUTHZ-9 (audit reads reference data without containing it).

## Scope

- **Applies To**: Multiple — Server-Side Application (scheduled execution, migrations, investigative access control) and deployment (log-archive account resources)
- **Components**: Relational Persistence (audit tables, database-role privileges); REST API Application (nightly scheduled execution, investigative interface authorization); Deployment (Terraform: log-archive account, S3 Object Lock buckets, lifecycle rules, cross-account write role)
- **Interfaces / Operations**: Raw-SQL privilege migrations on audit tables; the nightly batch-archival execution; the archive and deletion-ledger S3 prefixes with Object Lock and lifecycle configuration; retention deletion jobs; the admin investigative read operations; the chain-verification procedure
- **Actors**: The application database role; the scheduled execution's identity; `admin` accounts (investigative reads); malicious or careless operator / insider holding database or backup credentials (adversary, threat TM-R-2); compromised admin (bounded by audited reads)
- **Preconditions**: REQ-AUDIT-010 audit tables exist; REQ-INFRA-010 accounts provisioned; REQ-INFRA-030 tombstone key available
- **Data Classification**: Restricted — audit entries are sensitive personal data (threat TM-P-4), though value-free (SEC-LOG-3)
- **Personal or Regulated Data**: Personal Data — audit entries reference accounts and health-data actions but never contain health values (SEC-LOG-3); archived batches carry pseudonymized identifiers only
- **Jurisdiction / Regulatory Scope**: Per `SECURITY.md` SQ-1 (RESOLVED): GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable. Retention is deliberately bounded because audit records are sensitive under the GDPR ceiling (SQ-8).

## Security Context

- **Security Objectives**: Integrity, Accountability, Privacy, Confidentiality
- **Control Layers**: Data Protection, Logging and Monitoring, Authorization, Architecture
- **Threat References**: `SECURITY.md` TM-R-2 (audit entries alterable below the application by a DB-credential holder or operator), TM-P-4 (the audit trail as sensitive personal data with retention and access risk), TM-R-1 (repudiable admin actions — the archive preserves the accountability record); CWE-778-adjacent (insufficient logging) and log-tampering classes — exact CWE mappings TO BE DECIDED (not verified this session)
- **Abuse / Misuse Case**: A hostile insider with database credentials edits or deletes audit rows to erase evidence of health-data access; a backup-credential holder alters restored audit history; an admin browses audit records without leaving a trace; audit data accumulates indefinitely, becoming a surveillance liability; a deleted account's identifiers persist in archived batches, defeating FR-9.10.
- **Trust Boundary**: Boundary 5 — the application versus human operational access below it; the cross-account line between production and the log-archive account; boundary 4 for changes to the archive infrastructure.
- **Untrusted Inputs or Assertions**: Any audit content read back from the primary store during verification (it may have been tampered with below the application — the chain is the authority); any claim that an out-of-band change did not occur; investigative-read requests, which must pass the SQ-4 policy module like every protected operation.
- **Authoritative Enforcement Point**: PostgreSQL privilege grants applied in raw-SQL migrations (below the application); S3 Object Lock governance mode plus the log-archive account boundary (below the operator's production credentials); the SQ-4 policy module for investigative reads.
- **Independent Verification**: Chain verification recomputes SHA-256 hashes over the pseudonymized archived content and compares against the stored chain, detecting edits independently of database state; the archive lives in a separate account so no production credential can rewrite it.
- **Zero Trust Relevance**: TO BE DECIDED — not verified against NIST SP 800-207 in this session.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — mapping deferred to the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A — no AI-enabled component in this requirement.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: Bounded retention (12-month logs, 3-year audit entries) and pseudonymized archives implement data-minimization posture under the SQ-1 regime set (GDPR/UK GDPR; CCPA/CPRA; Washington My Health My Data; FTC HBNR); FR-9.7's audit obligation and FR-9.10's tombstoning are the requirement-level anchors. Specific article/section mappings: TO BE DECIDED — the spec documents state no section for this behavior.
- **Other**: `REF-LOG`, `REF-PROMPT-TF-AWS` (cited by SEC-LOG-7); `REF-ASVS-5` (SEC-LOG-5).
- **Mapping Basis**: The listed references are those SEC-LOG-5 and SEC-LOG-7 themselves cite.

## Acceptance Criteria

1. **AC-01 — Expected behavior (nightly pseudonymized, chained archival)**: Given audit entries written during a day, when the nightly scheduled execution of the REST API Application runs, then it writes one batch to the log-archive account's Object Lock (governance mode) bucket in which every account identifier has been replaced by its SEC-DATA-4 tombstone derivation, the batch carries a SHA-256 hash chained to the previous batch computed over the pseudonymized content, and no raw account identifier appears anywhere in the archived object.
2. **AC-02 — Boundary or failure behavior (append-only below the application)**: Given the migrated database privileges, when the application's database role attempts UPDATE or DELETE on any audit table, then PostgreSQL refuses the statement at the privilege layer, while INSERT and SELECT succeed; the revocation is expressed in the raw-SQL migrations, not application code.
3. **AC-03 — Prohibited behavior (tamper evidence)**: Given an out-of-band modification to an archived batch or to primary audit rows already archived, when chain verification runs, then the modification is detected and reported; and an attempt by production-account credentials to overwrite or delete an archived object within its Object Lock retention is denied.
4. **AC-04 — Expected behavior (retention and ledger lifecycle)**: Given the configured lifecycle rules, when retention horizons pass, then security logs are deleted after 12 months, audit entries after 3 years — except entries under a recorded legal hold — and deletion-ledger objects in their dedicated prefix expire at 35 days; nothing deletes earlier, and no rule retains longer absent a hold.
5. **AC-05 — Prohibited behavior (investigative access)**: Given the investigative interface, when an audit-entry read is attempted by a `subscriber` or `consultant` account, then it is denied by the SQ-4 policy module; and every permitted `admin` read itself produces an audit entry recording the acting admin, the query scope, and the time.

## Failure Behavior

- **On Invalid Input**: N/A for the archival path — it consumes persisted audit rows, not client input; investigative-read query parameters are schema-validated per SEC-INPUT-1.
- **On Authentication Failure**: Investigative reads require an authenticated `admin` session (passkey-backed, SEC-AUTHN-2); unauthenticated access is denied by the REQ-AUTHZ-010 gate.
- **On Authorization Failure**: Non-admin investigative reads are denied by the policy module with no disclosure of audit content or existence; cross-account archive writes are denied by IAM to every identity except the scheduled execution's role.
- **On Security-Decision Failure**: Deny by default — a failed policy evaluation denies the read (SEC-AUTHZ-7); a failed tombstone derivation fails the batch rather than archiving raw identifiers.
- **On External Dependency Failure**: If the log-archive account or S3 is unreachable, the nightly execution fails visibly and retries on its next run; unarchived entries remain in the append-only primary store, so no audit data is lost — the failure narrows tamper-evidence coverage until resolved, which is why it alerts.
- **On System Error**: A partially written batch is not chained; the next successful run re-archives from the last verified chain point. Errors never delete or mutate primary audit rows.
- **Logging / Audit**: The scheduled execution logs run outcome, batch identifier, entry count, and chain head — never entry contents; investigative reads produce audit entries per SEC-LOG-5; chain-verification results are recorded. No health values, credentials, or raw identifiers in any of it (SEC-LOG-3).
- **Alerting**: A failed or missed nightly archival run, a chain-verification mismatch, and denied archive-write or Object Lock-violation attempts route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-11 RESOLVED); a chain mismatch is a potential incident signal.

## Test Strategy

- **Unit Tests**: Batch construction replaces every identifier field with the keyed tombstone derivation and rejects a batch containing an unpseudonymized identifier; chain computation over pseudonymized content is deterministic and links to the prior head; derivation-failure and empty-day paths fail closed; the investigative-read policy function denies non-admin subjects.
- **Integration Tests**: Migration applies the INSERT/SELECT-only grants and UPDATE/DELETE attempts fail against real PostgreSQL; a full nightly run against a seeded audit table lands a chained, pseudonymized object in a test archive bucket; lifecycle rules on archive, log, and ledger prefixes match the 3-year, 12-month, and 35-day horizons; an admin investigative read succeeds and emits its own audit entry.
- **Security Tests**: Tamper exercise — modify an archived object copy and an already-archived primary row out of band, assert chain verification reports both; attempt archive overwrite/delete with production-account credentials, assert Object Lock denial; attempt audit mutation through every application role and interface, assert denial (SEC-LOG-2 verification); scan an archived batch for raw identifiers, assert none.
- **Compliance Tests / Evidence**: The documented retention configuration and legal-hold mechanism (SEC-LOG-5 verification: "Documented retention policy exists and is enforced by configuration"); chain-verification run records, retained for the SQ-10 assessment and SQ-1 counsel review.
- **Acceptance-Criteria Traceability**: AC-01 — nightly-run integration test plus pseudonymization unit tests; AC-02 — privilege-migration integration test; AC-03 — tamper exercise and Object Lock denial test; AC-04 — lifecycle-configuration test; AC-05 — investigative-access policy tests and the read-audit assertion.
- **Coverage Target**: Project coverage threshold TO BE DECIDED (`CLAUDE.md`); every pseudonymization, chaining, privilege, and access-control branch MUST have positive and negative tests.
- **Required Test Environment**: PostgreSQL with the migrated grants; a test log-archive bucket with Object Lock enabled (governance mode); the tombstone key fixture (REQ-INFRA-030); seeded audit fixtures across all three roles; Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-AUDIT-010 (audit entry model and application-layer append-only writes); REQ-INFRA-010 (the log-archive account, cross-account roles, and reviewed Terraform path); REQ-INFRA-030 (the SEC-DATA-4 tombstone-derivation key as a managed secret)
- **Downstream Requirements**: REQ-PRIVACY-090 (writes deletion-ledger entries to the prefix this issue provisions), REQ-PRIVACY-100 (tombstoning consistency with the archive derivation), REQ-INFRA-050 (break-glass sessions are reviewed against this tamper-evident record), REQ-OPS-010 (chain verification and restore re-deletion as runbook steps)
- **External Dependencies**: AWS S3 Object Lock, S3 lifecycle management, IAM cross-account roles, EventBridge/ECS scheduled task execution (or equivalent scheduler for the Fargate task); PostgreSQL privilege system
- **Dependency Assumptions**: Object Lock governance mode denies overwrite and delete within retention as documented — verified by the denial test; PostgreSQL enforces revoked privileges regardless of application behavior; the scheduler fires nightly and surfaces missed runs.
- **Failure Impact**: Without this issue, an insider with database credentials can silently erase the health-data access record (TM-R-2) and FR-9.7's accountability guarantee is hollow below the application. An archival outage leaves audit data intact but un-notarized; a lost tombstone key would orphan ledger matching — key custody is REQ-INFRA-030's concern.

## Implementation Notes

- **Constraints**: The archival job is a scheduled execution of the REST API Application — same codebase, image, and database identity (`ARCHITECTURE.md` 2026-08-03; DR-5) — not a new component or Lambda; privilege revocation lives in raw-SQL drizzle-kit migrations (`CLAUDE.md`); the archive and ledger live in the separate log-archive account (SQ-8, SEC-DATA-4); governance-mode Object Lock is the fixed mechanism (SEC-LOG-7); retention periods are named constants matching SQ-8.
- **Prohibited Approaches**: Archiving raw account identifiers (SEC-LOG-7 forbids it — pseudonymize at batch time); computing chain hashes over pre-pseudonymization content (deletion would then break verification); granting the application role UPDATE or DELETE on audit tables for "maintenance"; retention deletion implemented as application-role DELETE (use lifecycle rules and a separately privileged retention path); placing the ledger or archive in the production account; exposing investigative reads outside the SQ-4 enforcement point; copying audit values (rather than references) into entries (SEC-LOG-3).
- **Implementation Guidance**: Store the chain head alongside each batch and in the batch object's metadata so verification needs only the archive. Make the batch format append-friendly newline-delimited JSON with a manifest record carrying batch ID, entry count, previous head, and own hash. Drive the 3-year and 12-month deletions from partitioned tables or indexed timestamps so retention deletion is cheap and hold-aware — the legal-hold marker exempts a partition or entry set from the retention job. Reuse the REQ-PRIVACY-100 derivation function for batch pseudonymization so archive and tombstone outputs match by construction.
- **AI Development Guidance**: `REF-PROMPT-TF-AWS`, `REF-PROMPT-NODE`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the privilege migrations, cross-account IAM, Object Lock configuration, and chain design; privacy review that archived batches and ledger entries carry no raw identifiers and retention matches SQ-8.
- **Open Decisions**: None. Legal-hold invocation is an operator process; the mechanism this issue builds only has to honor a recorded hold.

**Estimated effort**: 1.5–2 engineer-days. **Estimated changed lines**: 450–900 (raw-SQL migrations, the scheduled-execution batch/chain module, investigative-access policy wiring, and Terraform for the archive account resources).
**Recommended model**: Claude Opus (`claude-opus-5`) — the chain, pseudonymization ordering, and privilege boundaries interact subtly; an ordering mistake silently voids either tamper evidence or FR-9.10.
