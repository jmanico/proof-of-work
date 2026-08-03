# [REQ-PRIVACY-100] Audit tombstoning on account deletion

## Metadata

- **ID**: REQ-PRIVACY-100
- **Title**: Audit tombstoning on account deletion
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Privacy
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-9.10 (derivation fixed 2026-08-03); `SECURITY.md` SEC-DATA-4 (tombstone derivation), SEC-LOG-7 (archive pseudonymization consumer)

## Requirement

- **Statement**: On account deletion, the system MUST replace the deleted account's identifiers in retained audit entries with an irreversible tombstone identifier computed as a keyed one-way derivation — HMAC-SHA-256 of the account identifier under a dedicated key held in the managed secret store — and that same derivation MUST be the single source of tombstones for deletion-ledger keying (SEC-DATA-4) and archive-batch pseudonymization (SEC-LOG-7).
- **Rationale**: FR-9.10 resolves the tension between the erasure right (FR-9.4) and accountability (FR-9.7): audit entries survive deletion, but nothing identifying survives in them — and because audit entries never contain health values (SEC-LOG-3), nothing health-bearing survives either. The keyed derivation (fixed 2026-08-03) makes the tombstone deterministic — so the restore procedure can re-match ledger entries and the archive needs no rewrite on deletion — while remaining irreversible without the key, unlike a plain hash of a small identifier space.
- **Assumptions**: The audit entry model stores subject and actor identifiers as structurally replaceable reference fields, not embedded in free text (REQ-AUDIT-010 implementation guidance); the managed secret store is available at runtime via the task IAM role (REQ-INFRA-030; SEC-SECRET-2).
- **Out of Scope**: The deletion execution itself, the ledger write, and the restore re-deletion procedure (REQ-PRIVACY-090); the nightly archive batching that consumes the derivation at batch time (SEC-LOG-7; REQ-INFRA-040); audit-entry creation and append-only enforcement (REQ-AUDIT-010); log redaction (REQ-AUDIT-040).
- **Design Traceability**: `DESIGN.md` — Account, privacy, and destructive actions: the deletion page summarizes audit tombstoning to the user. No further client-side behavior; the derivation is server-side.
- **Architecture Traceability**: `ARCHITECTURE.md` — Relational Persistence (audit entries); the deletion ledger lives in the log-archive account, outside the restored dataset (Data model expectations); data flow 8 (deletion writes its audit entry); scheduled executions consume the same derivation for archival (SEC-LOG-7).
- **Security Traceability**: SEC-DATA-4 (derivation definition), SEC-LOG-7 (archive pseudonymization uses the same derivation), SEC-LOG-3 (entries are value-free, so tombstoning completes de-identification), SEC-LOG-2 (append-only interplay), SEC-SECRET-2 and SEC-SECRET-4 (key custody), FR-9.10.

## Scope

- **Applies To**: Server-Side Application, API
- **Components**: REST API Application (derivation function and tombstoning step); Relational Persistence (audit tables); the log-archive account as a consumer (via REQ-PRIVACY-090 and REQ-INFRA-040)
- **Interfaces / Operations**: The account-deletion execution path (REQ-PRIVACY-090 invokes tombstoning); the shared derivation function consumed by ledger keying and archive batching
- **Actors**: The deleting user (indirectly); the system itself — no actor invokes tombstoning except through deletion
- **Preconditions**: An account deletion is executing (FR-9.4, via either the in-session or FR-9.11 path); audit entries referencing the account exist
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Personal Data — audit entries reference identity but never contain health values (SEC-LOG-3); the tombstone removes the identifying reference
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED)

## Security Context

- **Security Objectives**: Privacy, Accountability, Integrity, Confidentiality
- **Control Layers**: Data Protection, Logging and Monitoring, Architecture
- **Threat References**: `SECURITY.md` TM-P-1 (identifiability — deletion must de-identify derived copies), TM-P-4 (the audit trail is itself sensitive personal data), TM-R-2 (audit alteration below the application — the tombstoning path must not weaken append-only posture); CWE-328 (use of weak hash — an unkeyed hash over a small identifier space is reversible by enumeration)
- **Abuse / Misuse Case**: An insider or attacker with audit-table or archive access re-identifies a deleted user by brute-forcing an unkeyed hash of sequential account identifiers; the tombstoning path is abused as a write channel to alter audit history for non-deleted accounts; the derivation key leaks into logs or source, making every tombstone reversible.
- **Trust Boundary**: Boundary 3 (REST API Application → Relational Persistence) for the replacement; boundary 5 for the archive and ledger copies the derivation protects against below-application readers.
- **Untrusted Inputs or Assertions**: None from clients — tombstoning is triggered only by the server-side deletion path. The derivation input is the persisted account identifier.
- **Authoritative Enforcement Point**: The single shared derivation function in the REST API Application, and the deletion execution path that applies it to audit rows.
- **Independent Verification**: The irreversibility property rests on the key never leaving the secret store custody path (SEC-SECRET-1, SEC-SECRET-2); determinism is verified by matching tombstones across the three consumers in tests.
- **Zero Trust Relevance**: N/A — no resource-access decision is made here; this is a data-protection transformation. (Evaluated; NIST SP 800-207 concerns access, not de-identification.)

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session (SQ-10 pre-launch assessment).
- **OWASP AISVS 1.0**: N/A — no AI component is involved.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: N/A — see Zero Trust Relevance.
- **Regulatory**: The SQ-1 regime set — GDPR/UK GDPR (erasure and storage-limitation posture: bounded, de-identified audit retention per SQ-8), CCPA/CPRA, Washington My Health My Data, FTC HBNR; HIPAA not applicable. Statute-section mappings: TO BE DECIDED (SQ-1 counsel review).
- **Other**: HMAC-SHA-256 per the construction named in `SECURITY.md` SEC-DATA-4; exact RFC citation for HMAC: TO BE DECIDED (not verified against a standards text in this session).
- **Mapping Basis**: SEC-DATA-4 names the HMAC-SHA-256 construction and key custody; CWE-328 describes the unkeyed-hash weakness the keyed derivation exists to avoid.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given an executing account deletion, when tombstoning runs, then every retained audit entry referencing the account — as acting account or as affected subject — has those identifier fields replaced by the tombstone, while the entry's action, time, and non-identifying fields remain intact and the entry remains value-free (SEC-LOG-3).
2. **AC-02 — Boundary or failure behavior**: Given the same account identifier and the same key, when the derivation is computed by the tombstoning step, the deletion-ledger keying, and the archive-batch pseudonymization, then all three produce the identical tombstone — so a restore can match ledger entries and the archive posture holds without rewrite; and given the secret store is unavailable, when tombstoning is attempted, then the deletion does not complete with plaintext identifiers left behind — the operation fails closed and is retriable (REQ-PRIVACY-090 failure ordering).
3. **AC-03 — Prohibited behavior**: Given a completed deletion, when audit rows, archive batches, and ledger entries are inspected, then no plaintext or reversibly encoded account identifier of the deleted account remains in any of them; the derivation MUST NOT be an unkeyed hash or truncation of the identifier; and the derivation key MUST NOT appear in source control, logs, error responses, client bundles, or query text (SEC-SECRET-1).
4. **AC-04 — Additional criterion**: Given the tombstoning path, when it executes, then it replaces identifier fields only on entries referencing the deleted account and alters no other field and no other account's entries — the audit trail remains append-only from the application's perspective for every purpose other than this sanctioned identifier replacement (SEC-LOG-2; FR-9.10's "tombstoned in place", SEC-LOG-5).

## Failure Behavior

- **On Invalid Input**: N/A — no client input reaches this path; the derivation input is the persisted account identifier of the account being deleted.
- **On Authentication Failure**: N/A — authentication and re-authentication are enforced by the deletion path (REQ-PRIVACY-090) before tombstoning is reachable.
- **On Authorization Failure**: N/A — tombstoning has no direct interface to authorize; it executes only inside deletion.
- **On Security-Decision Failure**: Fail closed — if the key cannot be resolved or the replacement cannot be completed for every referencing entry, the deletion does not complete and reports failure to the deletion path; partial tombstoning MUST NOT be reported as success.
- **On External Dependency Failure**: Secret-store unavailability aborts the operation with retry; the derivation key is resolved at runtime via the task IAM role (SEC-SECRET-2), never cached to disk.
- **On System Error**: The replacement participates in the deletion transaction's consistency guarantees (REQ-PRIVACY-090) — a mid-operation failure leaves either the pre-deletion state or the fully tombstoned state, never a mixture presented as complete.
- **Logging / Audit**: The deletion's own audit entry records that tombstoning occurred (action and time); operational logs record counts, never the mapping between identifier and tombstone, never the key, and never the pre-replacement identifiers alongside their tombstones (SEC-LOG-3, SEC-SECRET-1).
- **Alerting**: A tombstoning failure is a deletion-completion failure — alerted to the security lead as a SEC-OPS-2 detection input, since an incomplete erasure is a potential notifiable non-compliance (`SECURITY.md` SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: Derivation produces HMAC-SHA-256 output under the configured key; determinism — identical input and key yield identical tombstones; distinct identifiers yield distinct tombstones; key change yields different tombstones (rotation surface); replacement targets exactly the actor and subject identifier fields and only for the deleted account.
- **Integration Tests**: End-to-end deletion fixture asserting every audit entry referencing the account carries the tombstone afterwards with all other fields byte-identical; cross-consumer test asserting the tombstoning step, ledger keying (REQ-PRIVACY-090), and archive pseudonymization (REQ-INFRA-040) produce the same tombstone for the same account; secret-store-outage fixture asserting fail-closed behavior with no partial plaintext state.
- **Security Tests**: Irreversibility review — assert no unkeyed hash path exists and attempt identifier recovery by enumerating the identifier space against captured tombstones without the key (must fail); secret-scanning assertion that the key appears in no source, log output, or error response (SEC-SECRET-1, gitleaks gate); append-only regression — attempt to reach the replacement path outside deletion and against a non-deleted account, asserting denial.
- **Compliance Tests / Evidence**: Post-deletion inspection evidence (audit rows, archive batch, ledger entry all identifier-free) for the SQ-1 counsel review; retention configuration evidence rides with REQ-INFRA-040 (SQ-8).
- **Acceptance-Criteria Traceability**: AC-01 — end-to-end tombstoning fixture; AC-02 — cross-consumer determinism test and secret-store-outage fixture; AC-03 — irreversibility and secret-scanning tests; AC-04 — append-only regression and field-scope unit tests.
- **Coverage Target**: Project-defined; the derivation function and every replacement and failure path covered positive and negative.
- **Required Test Environment**: Vitest; audit-table fixtures with entries referencing the target account as both actor and subject plus entries for other accounts; a secret-store stub with an outage mode; archive and ledger stand-ins for the cross-consumer test.

## Dependencies

- **Upstream Requirements**: REQ-AUDIT-010, REQ-INFRA-030
- **Downstream Requirements**: REQ-PRIVACY-090 (ledger keying and tombstoning at deletion), REQ-INFRA-040 (archive-batch pseudonymization)
- **External Dependencies**: None at runtime beyond the system's own managed secret store (AWS Secrets Manager via the task IAM role; SEC-SECRET-2, SQ-7 RESOLVED).
- **Dependency Assumptions**: The audit schema keeps identifiers in structurally replaceable reference fields (REQ-AUDIT-010); the dedicated key exists in Secrets Manager, is distinct from signing keys, and is resolvable at runtime without appearing in configuration files.
- **Failure Impact**: Without the keyed derivation, deleted users remain identifiable in the audit trail, archive, and ledger — an erasure-right breach (TM-P-1) — or, with an unkeyed hash, re-identifiable by enumeration (CWE-328); a leaked key reverses every tombstone at once.

## Implementation Notes

- **Constraints**: Node.js runtime; HMAC-SHA-256 via the platform crypto library — no new dependency (DEP-1). The key lives in AWS Secrets Manager and is resolved via the task IAM role (SEC-SECRET-2); PostgreSQL with Drizzle ORM and raw-SQL migrations where below-application privileges are involved (`CLAUDE.md`).
- **Prohibited Approaches**: Unkeyed or truncated hashes of the identifier; per-consumer derivation variants (the derivation MUST be one shared function); storing an identifier-to-tombstone mapping table (it would reverse the tombstone by lookup); hardcoding or committing the key; logging pre-replacement identifiers with their tombstones; deleting audit entries instead of tombstoning them.
- **Implementation Guidance**: Expose the derivation as one function consumed by all three call sites so drift is impossible. The identifier replacement must coexist with SEC-LOG-7's revocation of UPDATE/DELETE for the application's database role; how the sanctioned in-place replacement is expressed below the application (for example, a narrowly scoped routine defined in the raw-SQL migrations that permits identifier-column replacement only) is schema-level design decided with the code per `ARCHITECTURE.md` — it MUST NOT be solved by granting the application role broad UPDATE on audit tables.
- **AI Development Guidance**: `REF-PROMPT-NODE` (crypto usage), `REF-PROMPT-TF-AWS` (key custody), `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the derivation, key custody, and the migration-level replacement mechanism; privacy review that post-deletion state satisfies the erasure position recorded for the SQ-1 counsel review.
- **Open Decisions**: None — the derivation, key store, and consumers are fixed (FR-9.10, SEC-DATA-4, SEC-LOG-7, 2026-08-03); the exact below-application replacement mechanism is implementation-level schema design decided with the code (`ARCHITECTURE.md`, Relational Persistence open decisions).

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 200–450.
**Recommended model**: Claude Opus (`claude-opus-5`) — a small cryptographic control where irreversibility, single-source determinism, and append-only interplay carry the entire privacy guarantee.
