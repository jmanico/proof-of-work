# [REQ-PRIVACY-050] No external transmission of health data

## Metadata

- **ID**: REQ-PRIVACY-050
- **Title**: No external transmission of health data
- **Version**: 1.1.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Privacy
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-9.8; `SECURITY.md` SEC-TB-3, SEC-EXT-1, SEC-EXT-2, SEC-EXT-3; threat TM-I-5

## Requirement

- **Statement**: The system MUST NOT transmit user health data to any external service, including analytics, error-reporting, logging, and monitoring destinations outside the system boundary, and MUST NOT issue server-side requests to URLs derived from user or admin input — including plan citation URLs — without an explicit allow-list. Outbound traffic MUST be limited to the named egress endpoints (SEC-CICD-3), and the sanctioned in-boundary channels — transactional email via in-account SES (SEC-EXT-3) and AI inference via in-account Bedrock (SEC-AI-1) — MUST NOT carry health data.
- **Rationale**: FR-9.8 forbids external transmission of health data; SEC-TB-3 restates it across all outbound paths; SEC-EXT-2 closes the server-side request-forgery path that citation URLs would otherwise open. `SECURITY.md` now records exactly two sanctioned in-boundary AWS service channels — Bedrock for inference and SES for transactional email — and says future integrations remain anticipated but unspecified, so the prohibition must be enforced by construction rather than by the current absence of integrations, and each sanctioned channel must be tested for health-data leakage rather than trusted.
- **Assumptions**: The system is self-contained (`ARCHITECTURE.md`, API style and integration model) and has no external-system trust boundary; SES and Bedrock are in-account services reached through the named-egress set, not third-party application services.
- **Out of Scope**: Log content redaction, which REQ-AUDIT-040 owns and this issue depends on; encryption in transit for the system's own traffic (SEC-DATA-1, delivered by REQ-INFRA-020); the SES mail interface and its delivery mechanics (SEC-EXT-3, delivered by REQ-INFRA-060); the Terraform network tiering that realizes the named-egress restriction (SEC-CICD-3, delivered by REQ-INFRA-020); the process for approving a future integration (SEC-EXT-1), which is a review gate rather than code.
- **Design Traceability**: `DESIGN.md` — Product Patterns → Plans, citations, and review state ("The UI never fetches citation URLs server-side (SEC-EXT-2)"); citation links are rendered for the reader to follow, a client-side navigation, not a server-side fetch.
- **Architecture Traceability**: `ARCHITECTURE.md` — "There is no external-system boundary: health data does not leave the system (FR-9.8)"; DR-7; FR-9.8 traceability row ("System boundary — no external integration exists by construction"); the mail interface is owned by the sending component (SEC-EXT-3).
- **Security Traceability**: SEC-TB-3, SEC-EXT-1, SEC-EXT-2, SEC-EXT-3; supports SEC-LOG-3, SEC-CICD-3, SEC-AI-1.

## Scope

- **Applies To**: Server-Side Application, API, Web Client, External Integration
- **Components**: REST API Application; Identity and Session Handling; Browser Client; deployment network configuration
- **Interfaces / Operations**: Every outbound network path from the server, including the SES mail interface and the Bedrock inference path; every client-side outbound request; citation URL handling
- **Actors**: All actors indirectly; an attacker supplying a hostile citation URL
- **Preconditions**: None
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED)

## Security Context

- **Security Objectives**: Confidentiality, Privacy, Integrity
- **Control Layers**: Architecture, Data Protection, Logging and Monitoring, Supply Chain
- **Threat References**: `SECURITY.md` TM-I-5 (health data leaking to external analytics or monitoring), TM-T-4 (hostile admin-supplied content); the 2026-08-03 threat-model addendum's SEC-EXT-3 threats (enumeration through send behavior, token leakage, mail misconfiguration); CWE-918 (server-side request forgery), CWE-200 (exposure of sensitive information), CWE-359 (exposure of private personal information)
- **Abuse / Misuse Case**: A developer adds an error-reporting or analytics SDK that ships request payloads containing health data; an admin supplies a citation URL pointing at an internal address and the server fetches it, turning the citation field into an SSRF primitive; or a transactional email template interpolates a health value into a message that leaves the boundary through the sanctioned mail channel.
- **Trust Boundary**: The system egress boundary, and boundary 4 (CI/CD and IaC), since an integration can be introduced by dependency or by infrastructure change.
- **Untrusted Inputs or Assertions**: Citation URLs and any other user- or admin-supplied URL; any dependency's autonomous network behavior; content interpolated into outbound email.
- **Authoritative Enforcement Point**: Network egress configuration (NAT egress restricted to named endpoints, SEC-CICD-3) plus an application-level policy that no outbound client exists for user-supplied URLs and that all mail flows through the internal mail interface (SEC-EXT-3).
- **Independent Verification**: Egress review of application and infrastructure configuration, plus a test asserting no health field appears in any outbound payload (SEC-TB-3 verification), including email payloads.
- **Zero Trust Relevance**: N/A — this is a data-flow prohibition, not a resource-access decision.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — per-issue mappings are verified during the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A — the AI inference channel's own controls are REQ-FOOD-050's scope.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — per-issue mappings are verified during the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **NIST SP 800-207**: N/A
- **Regulatory**: Every regime in the SQ-1 set treats onward transfer of health data as a regulated disclosure — GDPR/UK GDPR for EU/UK data subjects; CCPA/CPRA, Washington My Health My Data, and the FTC Health Breach Notification Rule for US users. Statute-section precision: TO BE DECIDED (per-issue verification work, `SECURITY.md` SQ-1).
- **Other**: `REF-PC-2024` as cited by SEC-EXT-2; `REF-LOG` as cited by SEC-TB-3.
- **Mapping Basis**: FR-9.8, SEC-TB-3, SEC-EXT-2, and SEC-EXT-3 are the normative sources and name these references; CWE-918 names the SSRF class SEC-EXT-2 prevents.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given the deployed system, when its outbound network behavior is reviewed, then no request carrying health data leaves the system boundary, outbound destinations are limited to the named egress endpoints (SEC-CICD-3) — the in-account mail and inference services among them — and no analytics, error-reporting, session-replay, or monitoring destination outside the boundary is configured.
2. **AC-02 — Boundary or failure behavior**: Given a plan citation URL pointing at an internal address, a loopback address, a cloud metadata endpoint, or any external host, when the plan is created, edited, viewed, or published, then the server never issues a request to that URL, and the URL is stored and rendered as data only.
3. **AC-03 — Prohibited behavior**: Given the application, when it is built and run, then it MUST NOT contain an outbound HTTP client that accepts a user- or admin-supplied URL without an explicit allow-list, and MUST NOT transmit any health field in an outbound payload for any purpose including telemetry, crash reporting, or feature flags.
4. **AC-04 — Additional criterion**: Given the Browser Client, when a page loads, then it makes no request to a third-party origin, consistent with the Content Security Policy fixed by SEC-HTTP-2 (`connect-src 'self'`; REQ-PLATFORM-040).
5. **AC-05 — Additional criterion**: Given a change that introduces an outbound dependency or destination, when it is reviewed, then it is rejected unless an explicit documented change to `REQUIREMENTS.md` permits it (SEC-EXT-1).
6. **AC-06 — Additional criterion**: Given any transactional email produced by the system, when its content is inspected, then it contains no health data, no credentials, and no secret beyond the single-use token the message exists to deliver, and it is sent only through the internal mail interface to the in-account mail service (SEC-EXT-3); no other component constructs or sends email.

## Failure Behavior

- **On Invalid Input**: A citation URL failing scheme or format validation is rejected per REQ-PLAN-040; it is never fetched to determine validity.
- **On Authentication Failure**: N/A
- **On Authorization Failure**: N/A
- **On Security-Decision Failure**: If it cannot be determined whether a destination is inside the boundary, treat it as external and refuse the transmission (fail closed).
- **On External Dependency Failure**: If the in-account mail or inference service is unavailable, the affected operation fails or degrades without rerouting to any other destination; no fallback external service exists by construction.
- **On System Error**: Diagnostics remain server-side (SEC-ERR-1); the fault is never reported to an external service.
- **Logging / Audit**: Log any attempted outbound request that the policy blocks, with destination and correlation identifier, and no health values (SEC-LOG-3).
- **Alerting**: Blocked-egress events route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-11 RESOLVED).

## Test Strategy

- **Unit Tests**: URL handling stores and renders citation URLs without dereferencing them; the outbound policy rejects any destination not on the allow-list; the mail interface refuses payloads containing health fields.
- **Integration Tests**: Exercise plan create, edit, view, and publish with hostile citation URLs and assert, via a network sink or interception layer, that no server-side request is made; capture each transactional email template rendered with fully populated accounts and assert no health field appears.
- **Security Tests**: SSRF suite covering internal, loopback, and metadata addresses, redirects, and DNS-rebinding-shaped hosts; an outbound-payload scan asserting no health field appears in any egress, including email; dependency review for libraries that phone home; client-side network assertion for third-party origins.
- **Compliance Tests / Evidence**: Egress review record and the outbound-payload scan, retained as evidence for FR-9.8.
- **Acceptance-Criteria Traceability**: AC-01 — egress review against the named-endpoint set; AC-02 — SSRF suite; AC-03 — outbound client inventory and payload scan; AC-04 — client network assertion; AC-05 — documented review gate; AC-06 — email content and mail-interface tests.
- **Coverage Target**: Every code path capable of outbound network access is inventoried; every citation-handling operation covered by the SSRF suite; every email template covered by the content scan.
- **Required Test Environment**: A network interception layer or egress-restricted test environment; deployment configuration access for the review. Topology is fixed (`SECURITY.md` SQ-7 RESOLVED): VPC per environment with NAT egress restricted to named endpoints, SES and Bedrock among them.

## Dependencies

- **Upstream Requirements**: REQ-AUDIT-040, REQ-PLATFORM-040
- **Downstream Requirements**: REQ-PLAN-040, REQ-CATALOG-030, REQ-INFRA-020, REQ-INFRA-060
- **External Dependencies**: None by construction. Any dependency with autonomous network behavior is disqualified under DEP-6 and DEP-8.
- **Dependency Assumptions**: No third-party library initiates network requests at runtime; this must be verified during dependency review (REQ-PIPE-010).
- **Failure Impact**: A single telemetry SDK, one server-side citation fetch, or one health value in an email template converts a self-contained health application into one with an undocumented external data flow — the failure mode FR-9.8 exists to prevent.

## Implementation Notes

- **Constraints**: Deployment topology is fixed (`SECURITY.md` SQ-7 RESOLVED): Terraform-managed AWS with a VPC per environment and NAT egress restricted to named endpoints (SEC-CICD-3). The network-level restriction is delivered by REQ-INFRA-020 and the mail channel by REQ-INFRA-060; this issue owns the application-level prohibitions and the health-data-content tests on the sanctioned channels.
- **Prohibited Approaches**: Fetching a citation URL to validate it, to generate a preview, or to check for link rot; adding an error-reporting or analytics SDK "with PII disabled"; allow-listing by URL substring rather than by parsed host; constructing or sending email outside the internal mail interface; relying on the current absence of integrations as the control.
- **Implementation Guidance**: Citation URLs are rendered as links by the client and followed by the user's own browser (`DESIGN.md`; SEC-RENDER-3 governs the safe rendering). Keep every outbound capability behind a single interface owned by the sending component — the SEC-EXT-3 mail interface is the model — so the prohibition has one place to be enforced per channel (DR-7). Email sending behavior must not reveal whether an address is registered (SEC-EXT-3, SEC-AUTHN-3); the non-enumeration tests live with the sending flows and the mail-channel issue (REQ-INFRA-060), not here.
- **AI Development Guidance**: `REF-PC-2024`, `REF-PROMPT-API`, `REF-PROMPT-NODE`, `REF-PROMPT-TF-AWS`; `CLAUDE.md`.
- **Required Human Review**: Security review of egress configuration and the outbound-client inventory; privacy review of any proposed telemetry and of the email templates.
- **Open Decisions**: `SECURITY.md` records that future integrations remain anticipated but unspecified (TO BE DECIDED there); SEC-EXT-1 requires an explicit `REQUIREMENTS.md` change before any health data crosses such a boundary. Nothing else in this issue is blocked: the topology and named-egress set are fixed (SQ-7), and the sanctioned channels are specified (SEC-EXT-3, SEC-AI-1).

**Estimated effort**: 0.5–1.5 engineer-days. **Estimated changed lines**: 150–400.
**Recommended model**: Claude Opus (`claude-opus-5`) — a prohibition that must be enforced by construction and verified by inventory rather than by a feature test.
