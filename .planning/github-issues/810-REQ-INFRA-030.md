# [REQ-INFRA-030] Secrets and signing-key management

## Metadata

- **ID**: REQ-INFRA-030
- **Title**: Secrets and signing-key management
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `SECURITY.md` SEC-SECRET-2, SEC-SECRET-3, SEC-SESSION-7, SQ-7 (RESOLVED); SEC-AUTHN-11 and SEC-DATA-4 (the dedicated HMAC keys they require, 2026-08-03)

## Requirement

- **Statement**: All runtime secrets — database credentials, JWT signing keys, the SEC-AUTHN-11 token-HMAC key, and the SEC-DATA-4 tombstone-derivation key — MUST be stored in AWS Secrets Manager as distinct secrets resolved at runtime by the task's IAM role and rotatable without code change, with JWT signing keys rotated via `kid` on a 90-day period, no secret appearing in source, client bundles, container images, logs, or Terraform state in plaintext, and Terraform state itself encrypted, access-restricted, and never committed to source control.
- **Rationale**: SEC-SECRET-2 requires runtime resolution from a managed store with rotation and fixes the service (AWS Secrets Manager via the task's IAM role); SEC-SESSION-7 fixes `kid`-based 90-day rotation for token signing keys with the algorithm as a named constant from the SEC-SESSION-1 allow-list; SEC-SECRET-3 makes Terraform state sensitive because it can carry infrastructure secrets; SEC-AUTHN-11 (2026-08-03) requires a dedicated HMAC-SHA-256 key from the secret store for machine-held token hashing, and SEC-DATA-4 (2026-08-03) requires a dedicated key for the FR-9.10 tombstone derivation — both must exist as managed, access-scoped secrets before the features that consume them can be built.
- **Assumptions**: The environment accounts, task IAM roles, and reviewed Terraform path exist (REQ-INFRA-010); the network placement of Secrets Manager access is provided by REQ-INFRA-020.
- **Out of Scope**: The consuming behavior — JWT issuance and verification against the `kid` (REQ-SESSION-010 family), token hashing under the HMAC key (SEC-AUTHN-11 — REQ-AUTH drafts), tombstone derivation and the deletion ledger (SEC-DATA-4 — REQ-PRIVACY-090, REQ-PRIVACY-100); secret scanning as a merge-blocking CI gate (gitleaks, SEC-CICD-4 — REQ-INFRA-070, with REQ-BUILD-010 holding the no-secrets-in-tree baseline); Argon2id password hashing, which uses per-credential salts, not a managed key (SEC-AUTHN-5).
- **Design Traceability**: N/A — `DESIGN.md` governs the browser client, not secret storage.
- **Architecture Traceability**: `ARCHITECTURE.md` — Deployment model ("AWS Secrets Manager for secrets and signing keys"); Identity and Session Handling (key storage resolved: "AWS Secrets Manager with `kid`-based 90-day rotation, SEC-SESSION-7"); DR-8 (deployment choices sit behind component interfaces).
- **Security Traceability**: SEC-SECRET-2, SEC-SECRET-3, SEC-SESSION-7; enables SEC-AUTHN-11, SEC-DATA-4; supports SEC-SECRET-1, SEC-SESSION-1.

## Scope

- **Applies To**: Multiple — Server-Side Application runtime configuration and deployment
- **Components**: Deployment (Terraform, `infra` workspace); Identity and Session Handling (signing and token-HMAC keys); REST API Application (database credentials, tombstone key)
- **Interfaces / Operations**: Secrets Manager secret definitions and resource policies; the task IAM role's secret-read permissions; the runtime secret-resolution seam; the Terraform state backend configuration; the signing-key rotation procedure
- **Actors**: The REST API Application's task IAM role (sole runtime reader); operators and CI/CD identities managing secrets and state (boundary 4); malicious or careless operator / insider (adversary, `SECURITY.md` Threat Model)
- **Preconditions**: REQ-INFRA-010 accounts and roles provisioned
- **Data Classification**: Restricted — these secrets gate every session, recovery token, and the health-data store itself
- **Personal or Regulated Data**: None directly stored; the keys protect Health Data and credentials — compromise of any of them is a health-data incident
- **Jurisdiction / Regulatory Scope**: Per `SECURITY.md` SQ-1 (RESOLVED): GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable. Single US primary region.

## Security Context

- **Security Objectives**: Confidentiality, Integrity, Authenticity
- **Control Layers**: Data Protection, Architecture, Supply Chain
- **Threat References**: `SECURITY.md` TM-T-3 (JWT signature compromise — key theft is its worst case), TM-T-6 (Terraform-state compromise), TM-S-5 (token theft and replay — forging beats stealing if the signing key leaks); CWE-798 (hard-coded credentials), CWE-522 (insufficiently protected credentials); further mappings TO BE DECIDED (not verified this session)
- **Abuse / Misuse Case**: An attacker reads a signing key from a committed file, a client bundle, a container layer, a log line, or plaintext Terraform state and mints valid sessions; an insider with state-backend access harvests database credentials; a stale signing key never rotates, extending the value of an undetected leak; one shared secret blurs the blast radius between token hashing and tombstone derivation.
- **Trust Boundary**: Boundary 4 (CI/CD-and-IaC path — Terraform state and secret provisioning) and boundary 5 (operators below the application); at runtime, the IAM-enforced line between the task role and every other identity.
- **Untrusted Inputs or Assertions**: Any secret value found outside Secrets Manager; any identity other than the task role requesting secret reads; environment variables or files claiming to carry key material.
- **Authoritative Enforcement Point**: AWS Secrets Manager resource policies plus the task IAM role's scoped read permissions, both Terraform-managed through the SEC-CICD-2 reviewed path; the state backend's encryption and access policy.
- **Independent Verification**: CI secret scanning (gitleaks) over history and artifacts, client-bundle inspection, and container-image scanning verify absence of secret material independently of author assertions; IAM policy review reads the deployed grants from AWS.
- **Zero Trust Relevance**: TO BE DECIDED — not verified against NIST SP 800-207 in this session.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — mapping deferred to the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A — no AI-enabled component in this requirement.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: Key management underpins the encryption and access controls the SQ-1 regime set expects for health data (GDPR/UK GDPR; CCPA/CPRA; Washington My Health My Data; FTC HBNR). Specific article/section mappings: TO BE DECIDED — the spec documents state no section for this behavior.
- **Other**: `REF-SECRETS`, `REF-PROMPT-TF-AWS` (cited by SEC-SECRET-2/-3); `REF-PROMPT-JWT` (SEC-SESSION-7); `REF-IAC` (SEC-SECRET-3).
- **Mapping Basis**: The listed references are those SEC-SECRET-2, SEC-SECRET-3, and SEC-SESSION-7 themselves cite.

## Acceptance Criteria

1. **AC-01 — Expected behavior (runtime resolution and rotation)**: Given a deployed environment, when the application starts, then it resolves database credentials, the JWT signing key set, the token-HMAC key, and the tombstone-derivation key from AWS Secrets Manager using only the task IAM role, and rotating any secret in Secrets Manager takes effect without a code change.
2. **AC-02 — Expected behavior (kid rotation)**: Given the signing-key secret, when the 90-day rotation produces a new key under a new `kid`, then newly issued tokens carry the new `kid` while verification continues to accept the configured key set, and the rotation is executable as a documented procedure without service interruption.
3. **AC-03 — Prohibited behavior (no plaintext exposure)**: Given the repository history, client bundles, container images, application logs, and Terraform state, when each is scanned or inspected, then no secret value appears in any of them in plaintext, and the four secrets exist as distinct Secrets Manager entries — no key is shared across the signing, token-HMAC, and tombstone purposes.
4. **AC-04 — Boundary or failure behavior (access scope and state protection)**: Given the deployed IAM and state-backend configuration, when a secret read is attempted by an identity other than the environment's task role, then it is denied and CloudTrail records the attempt; and the Terraform state backend is encrypted, access-restricted to the pipeline and named operators, and no state file is present in source control.

## Failure Behavior

- **On Invalid Input**: N/A — this requirement governs configuration and key material, not request content.
- **On Authentication Failure**: N/A at this layer — IAM denies unauthorized secret reads; there is no user-facing surface.
- **On Authorization Failure**: A secret read by a non-authorized identity is denied by the resource policy and visible in CloudTrail; the denial discloses nothing about secret contents.
- **On Security-Decision Failure**: Deny by default — if a required secret cannot be resolved at startup, the application refuses to serve rather than falling back to a baked-in or default value; a missing `kid` match refuses the token (SEC-SESSION-1, SEC-SESSION-2).
- **On External Dependency Failure**: Secrets Manager unavailability at startup fails the task closed; already-running tasks continue with resolved material until restart. No cached secret is written to disk.
- **On System Error**: Errors in secret resolution are logged with correlation identifiers but never with secret values or ARN-embedded material beyond the secret name (SEC-LOG-3, SEC-SECRET-1); failed rotation leaves the previous key set active and surfaces the failure.
- **Logging / Audit**: CloudTrail records every secret read, write, and rotation with the acting identity; application logs record resolution success/failure by secret name only; pino redaction paths cover any field that could carry secret material (SEC-LOG-3).
- **Alerting**: Denied secret-read attempts, rotation failures, and a signing key exceeding its 90-day period route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: The secret-resolution seam fails closed on missing or malformed secrets; no code path reads key material from environment variables, files, or literals; the signing-key set exposes keys by `kid` and refuses unknown `kid` values; the token-HMAC and tombstone keys are resolved as separate named secrets.
- **Integration Tests**: A deployed task resolves all four secrets via the task role and serves; a secret rotation in the dev account takes effect without code change; a `kid` rotation exercise issues under the new key while old-`kid` verification behavior matches the configured key set.
- **Security Tests**: gitleaks over full history and build artifacts (SEC-CICD-4 gate); client-bundle and container-image inspection for secret material; secret-read attempt under a non-task-role identity asserting denial; Terraform state inspected to confirm secret values are absent or encrypted and the backend rejects unauthenticated access; checkov rules on the state backend and secret resources.
- **Compliance Tests / Evidence**: The rotation-procedure record and the state-backend access policy review, retained for the SQ-10 assessment.
- **Acceptance-Criteria Traceability**: AC-01 — startup-resolution integration test and rotation exercise; AC-02 — `kid` rotation exercise; AC-03 — gitleaks/bundle/image/state scans plus the distinct-secrets check; AC-04 — denied-read test and state-backend review.
- **Coverage Target**: Project coverage threshold TO BE DECIDED (`CLAUDE.md`); every resolution and fail-closed branch MUST have positive and negative tests.
- **Required Test Environment**: The dev AWS account with Secrets Manager, CloudTrail, and the state backend provisioned; gitleaks and checkov in CI; a non-task-role test identity.

## Dependencies

- **Upstream Requirements**: REQ-INFRA-010 (accounts, task IAM roles, and the reviewed Terraform path); REQ-INFRA-020 (network reachability of Secrets Manager from the API's private subnets)
- **Downstream Requirements**: REQ-SESSION-010 family (JWT signing and verification), the REQ-AUTH drafts consuming the token-HMAC key (SEC-AUTHN-11), REQ-PRIVACY-090 and REQ-PRIVACY-100 (tombstone-derivation key, SEC-DATA-4), REQ-INFRA-040 (archive pseudonymization uses the same tombstone derivation)
- **External Dependencies**: AWS Secrets Manager, IAM, CloudTrail, KMS (secret encryption); the Terraform AWS provider
- **Dependency Assumptions**: Secrets Manager enforces its resource policies and encrypts at rest as documented — the denial and scan tests verify the deployed posture rather than assuming it; the state backend honors its access policy.
- **Failure Impact**: A leaked signing key lets an attacker mint sessions for any account (TM-T-3); a leaked token-HMAC key enables offline forgery of recovery tokens; a leaked tombstone key breaks the irreversibility of FR-9.10 pseudonymization; leaked database credentials bypass the application entirely. Secrets Manager unavailability blocks task startup but never degrades to plaintext fallbacks.

## Implementation Notes

- **Constraints**: AWS Secrets Manager resolved by the task IAM role (SEC-SECRET-2, SQ-7 — fixed); `kid`-based 90-day rotation for signing keys with the algorithm a named constant from the SEC-SESSION-1 allow-list, ES256 recommended non-normatively (SEC-SESSION-7); Terraform in the `infra` workspace via REQ-INFRA-010's reviewed pipeline; distinct secrets per purpose so IAM grants and rotation schedules stay independently scoped.
- **Prohibited Approaches**: Committing any secret or state file to source control (SEC-SECRET-1, SEC-SECRET-3); baking secrets into container images or client bundles; reusing one key across signing, token-HMAC, and tombstone purposes; logging secret values or resolved material (SEC-LOG-3); granting secret reads to identities beyond the environment's task role; storing key material in plain environment variables supplied outside the Secrets Manager path; hand-rolled cryptography around the managed keys (DEP-1).
- **Implementation Guidance**: Keep the resolution seam a single first-party module so redaction, fail-closed behavior, and caching policy live in one reviewable place (`REF-PROMPT-QUALITY`). Model the signing-key secret as a small keyed set (`kid` → key) so rotation is an additive write followed by retirement. Use a Secrets Manager VPC endpoint (REQ-INFRA-020) so resolution never leaves the boundary. Terraform should create the secret containers and policies but never render secret values into state — seed values out of band or with lifecycle-ignored placeholders.
- **AI Development Guidance**: `REF-PROMPT-TF-AWS`, `REF-PROMPT-JWT`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the secret inventory, IAM grants, rotation procedure, and state-backend policy; review that no Terraform resource renders a secret value into state.
- **Open Decisions**: The concrete signing-algorithm constant is chosen from the SEC-SESSION-1 allow-list at implementation (SEC-SESSION-7 records ES256 as a non-normative recommendation). Does not block this issue.

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 250–550 (Terraform secret/state resources, the resolution seam, and rotation procedure).
**Recommended model**: Claude Opus (`claude-opus-5`) — key separation, IAM scope, and fail-closed resolution are small in code and absolute in consequence.
