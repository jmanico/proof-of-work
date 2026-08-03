# [REQ-INFRA-060] Transactional email delivery via in-account SES

## Metadata

- **ID**: REQ-INFRA-060
- **Title**: Transactional email delivery via in-account SES
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `SECURITY.md` SEC-EXT-3 (added 2026-08-03), SQ-7 addendum; `ARCHITECTURE.md` Initial Architecture (mail interface, 2026-08-03)

## Requirement

- **Statement**: All transactional email — address verification, password reset, privileged invitations, pending-deletion notices, and incident notifications — MUST be sent only through in-account Amazon SES, reached exclusively through an internally defined mail interface owned by the sending component and listed as a named egress endpoint; message content MUST NOT contain health data, credentials, or any secret beyond the single-use token the message exists to deliver, and sending behavior MUST NOT reveal whether an address is registered.
- **Rationale**: SEC-EXT-3 states the rule. Email is the one channel through which system-generated content leaves the boundary, so it concentrates three risks the 2026-08-03 threat-model addendum names: account enumeration through send and bounce behavior, single-use-token leakage through forwarded or compromised mailboxes, and mail-service misconfiguration. Confining all mail to one in-account service behind one owned interface makes the content rules (no health data — SEC-TB-3, FR-9.8; no secrets beyond the delivered token — SEC-AUTHN-11) and the non-enumeration rule (SEC-AUTHN-3) enforceable and testable in one place.
- **Assumptions**: Amazon SES is fixed as the service (`SECURITY.md` SQ-7 addendum, 2026-08-03) and is included in the SEC-CICD-3 named-egress set. Email is a delivery channel for notices and single-use tokens only — it is not an authentication factor (REQUIREMENTS.md OQ-9 RESOLVED). The tokens the messages deliver are generated and stored under SEC-AUTHN-11 by their owning flows, not by this issue.
- **Out of Scope**: The flows that compose messages and their token semantics — email verification (REQ-AUTH-090), password reset (REQ-AUTH-130), privileged invitations (REQ-AUTH-140), email change (REQ-AUTH-180), out-of-band deletion notices (REQ-PRIVACY-110), incident notifications (REQ-OPS-010); rate limiting of resend requests (REQ-AUTH-060 — SEC-AUTHN-6/SEC-AUTHN-8); the network-level egress restriction itself (REQ-INFRA-020 — SEC-CICD-3), which this issue consumes by naming SES as an endpoint.
- **Design Traceability**: `DESIGN.md`, "Credentials, account security, and administration" — request confirmations read "If that address is registered, we sent a message"; no send, resend, or verification response may reveal whether an address exists.
- **Architecture Traceability**: `ARCHITECTURE.md` — Initial Architecture: outbound transactional email leaves the system through an in-account managed email service behind an internally defined mail interface; it carries no health data and is a channel, not a component. Identity and Session Handling outputs name the mail interface; data flow 9 (out-of-band deletion notice); DR-7's interface discipline applied to the mail path.
- **Security Traceability**: SEC-EXT-3 (primary); SEC-AUTHN-3 (non-enumeration), SEC-AUTHN-11 (single-use tokens, never logged), SEC-TB-3 and FR-9.8 (no health data outbound), SEC-CICD-3 (named egress), SEC-EXT-1 (integration posture).

## Scope

- **Applies To**: Multiple — Server-Side Application and deployment infrastructure
- **Components**: Identity and Session Handling and REST API Application (senders, through the mail interface); deployment infrastructure (SES, IAM, VPC egress)
- **Interfaces / Operations**: The internal mail interface; every message-producing operation: address verification and resend, password reset, privileged invitation, email change, pending-deletion notice, incident notification
- **Actors**: The system as sender; subscribers, invited privileged users, and notified individuals as recipients; anonymous attackers probing send behavior for enumeration
- **Preconditions**: Production-like environment with SES provisioned in-account (REQ-INFRA-010) and NAT egress restricted to named endpoints (REQ-INFRA-020)
- **Data Classification**: Confidential — message content includes addresses and single-use tokens; never health data
- **Personal or Regulated Data**: Personal Data — email addresses and account-lifecycle notices. Health data is prohibited in this channel.
- **Jurisdiction / Regulatory Scope**: GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users (SEC-OPS-2 uses this channel for individual breach notification); HIPAA not applicable (`SECURITY.md` SQ-1 RESOLVED)

## Security Context

- **Security Objectives**: Confidentiality, Privacy, Integrity, Accountability
- **Control Layers**: Architecture, Data Protection, Output Encoding, Logging and Monitoring
- **Threat References**: `SECURITY.md` Threat Model, Addendum 2026-08-03 (readiness review): account enumeration through send and bounce behavior, single-use-token leakage through forwarded or compromised mailboxes, mail-service misconfiguration; TM-I-5 (health data leaks into outbound paths); TM-I-4 (account-existence enumeration); CWE-204 (observable response discrepancy), CWE-200 (exposure of sensitive information)
- **Abuse / Misuse Case**: An attacker submits registration, reset, or resend requests for arbitrary addresses and infers account existence from response differences, send behavior, or bounce handling; a developer adds a direct SMTP or third-party mail dependency that bypasses the interface; a template change leaks a health value, a credential, or an extra token into a message.
- **Trust Boundary**: The system boundary at the mail egress path — content leaving the system toward external mailboxes; recipient mailboxes are outside the boundary and untrusted.
- **Untrusted Inputs or Assertions**: Recipient addresses supplied by users (validated by the owning flows); bounce and delivery feedback from the mail service, which MUST NOT alter response behavior in ways that reveal registration state.
- **Authoritative Enforcement Point**: The internally defined mail interface in the server-side application — the only code path that can reach SES — combined with IAM (only the task role may call SES) and VPC egress restriction (SES is the only reachable mail endpoint).
- **Independent Verification**: Template content rules are asserted by tests independent of the flows that send; egress review confirms no other mail path exists at the network and IAM layers, independent of application code discipline.
- **Zero Trust Relevance**: N/A — this rule governs outbound content and channel confinement, not resource-access decisions.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — not verified against `REF-ASVS-5` in this session (SQ-10 pre-launch assessment).
- **OWASP AISVS 1.0**: N/A — no AI-enabled component is involved.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: N/A — see Zero Trust Relevance.
- **Regulatory**: The SQ-1 regime set applies — GDPR/UK GDPR, CCPA/CPRA, Washington My Health My Data, FTC HBNR (this channel delivers the SEC-OPS-2 individual breach notifications on the 60-day HBNR clock). Statute-section mappings: TO BE DECIDED (SQ-1 counsel review).
- **Other**: `REF-ASVS-5`, `REF-AUTH`, `REF-PROMPT-TF-AWS` as cited by SEC-EXT-3; `REF-SECRETS` via SEC-AUTHN-11.
- **Mapping Basis**: SEC-EXT-3 names its references directly; CWE-204 describes the enumeration-by-discrepancy class SEC-AUTHN-3 closes and CWE-200 the content-leakage class the template rules close.

## Acceptance Criteria

1. **AC-01 — Expected behavior**: Given a message-producing flow (verification, reset, invitation, email change, pending-deletion notice, incident notification), when it sends, then the message goes through the internal mail interface to in-account SES, and the message contains only the notice content and, where applicable, the single one-time token that flow exists to deliver.
2. **AC-02 — Non-enumeration**: Given a send or resend request for a registered address and an identical request for an unregistered address, when each is submitted, then the observable responses do not differ in content, status, or timing beyond configured thresholds, and neither discloses whether the address is registered (SEC-AUTHN-3).
3. **AC-03 — Prohibited content**: Given any message template and any rendered message in the test capture, when inspected, then no health value, no credential, no session token, and no secret other than the single delivered one-time token appears; a template that would include such content MUST fail its content assertion.
4. **AC-04 — Channel confinement**: Given the deployed environment, when egress and IAM configuration are reviewed and a non-SES mail connection is attempted from the application network, then SES is the only reachable and only authorized mail path, and the attempt fails.
5. **AC-05 — Test fixture**: Given the automated test suites, when any suite exercises a message-producing flow, then a mail-capture fixture receives the message without real delivery, and suites can assert on recipient, template identity, and token presence exactly once per message.

## Failure Behavior

- **On Invalid Input**: An invalid or malformed recipient address is rejected by the owning flow's validation (SEC-INPUT-1) before the mail interface is invoked; the rejection response is as non-disclosing as a success (AC-02).
- **On Authentication Failure**: N/A — sending is a server-side action; recipient-facing flows govern their own authentication.
- **On Authorization Failure**: N/A — no user-invocable operation sends arbitrary mail; only the enumerated flows can invoke the interface.
- **On Security-Decision Failure**: Fail closed on content: a message failing template content validation is not sent. Failure to send MUST NOT alter the user-visible response in a way that reveals registration state.
- **On External Dependency Failure**: If SES is unavailable or throttling, the interface returns a failure to the owning flow for its own retry policy; user-visible responses remain neutral ("If that address is registered, we sent a message"); no fallback to any other mail service or direct SMTP is permitted.
- **On System Error**: Errors in the mail path are logged server-side with a correlation identifier (SEC-ERR-1); the recipient address may be logged, the token MUST NOT be (SEC-AUTHN-11, SEC-LOG-3).
- **Logging / Audit**: Each send attempt logs the sending flow, template identity, recipient account reference, timestamp, and outcome; message bodies and tokens are never logged. Bounce and complaint feedback is logged for operations without being surfaced to requesters.
- **Alerting**: Sustained send failures and bounce-rate anomalies alert the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-11 RESOLVED) — mail-service misconfiguration is a named threat.

## Test Strategy

- **Unit Tests**: Mail-interface contract: template rendering produces expected content; content assertions reject templates containing health-data fields, credentials, or more than the single expected token; interface refuses send on content-validation failure.
- **Integration Tests**: Each message-producing flow sends exactly one message through the capture fixture with the correct template and recipient; SES client configuration targets the in-account service; send failure leaves the flow's user-visible response unchanged.
- **Security Tests**: Differential response and timing tests on send and resend paths for registered versus unregistered addresses (SEC-AUTHN-3); egress test asserting no non-SES mail endpoint is reachable from the application network; IaC scan asserting SES-scoped IAM permissions with no wildcard; log assertion that no token or message body appears in logs.
- **Compliance Tests / Evidence**: Egress review record naming SES as the sole mail path (SEC-EXT-3 verification); template inventory review confirming no health data in any message.
- **Acceptance-Criteria Traceability**: AC-01 — per-flow integration tests; AC-02 — differential enumeration suite; AC-03 — template content assertions; AC-04 — egress and IAM tests; AC-05 — mail-capture fixture used by all mail-touching suites.
- **Coverage Target**: Every message template and every message-producing flow covered by content and capture assertions; both enumeration cases (registered, unregistered) tested on every send-triggering endpoint.
- **Required Test Environment**: Mail-capture fixture standing in for SES in Vitest and Playwright suites; a pre-production AWS account for egress and IAM verification; test accounts in registered and unregistered states.

## Dependencies

- **Upstream Requirements**: REQ-INFRA-010 (SES provisioned in-account with pipeline identities), REQ-INFRA-020 (named-egress restriction that confines the mail path)
- **Downstream Requirements**: REQ-AUTH-090 (email verification), REQ-AUTH-130 (password reset), REQ-AUTH-140 (privileged invitations), REQ-AUTH-180 (email change), REQ-PRIVACY-110 (pending-deletion notices), REQ-OPS-010 (incident notifications)
- **External Dependencies**: Amazon SES in the system's own AWS account (in-boundary managed service per SEC-EXT-3, not a third-party application service)
- **Dependency Assumptions**: SES delivery is best-effort; the owning flows tolerate delayed or failed delivery without weakening their token semantics (single-use, time-limited under SEC-AUTHN-11); SES configuration (verified sending domain, feedback handling) is Terraform-managed (SEC-CICD-2).
- **Failure Impact**: An unavailable mail path delays verification, reset, invitation, and notice delivery — flows degrade to neutral confirmations without disclosure; a misconfigured path risks enumeration, token leakage, or a silent second mail channel, which is why egress confinement is part of the requirement.

## Implementation Notes

- **Constraints**: The mail interface is first-party code owned by the sending component (`ARCHITECTURE.md`); SES is reached via the task IAM role (SEC-SECRET-2 posture — no static mail credentials); templates are structured plain text consistent with the `DESIGN.md` content voice and its neutral confirmation copy.
- **Prohibited Approaches**: Direct SMTP or any third-party email service; scattering SES client calls across flows instead of the single owned interface; including more than one token, any credential, or any health value in a message; varying send/resend responses by registration state; logging tokens or message bodies; treating email as an authentication factor (OQ-9).
- **Implementation Guidance**: Keep the interface narrow — send(templateId, recipientRef, params) — so content assertions can enumerate templates statically. Have the fixture implement the same interface as the SES adapter so suites swap the transport, not the flows. Bounce handling should feed operations logging only; never surface bounce state through any user-facing response.
- **AI Development Guidance**: `REF-PROMPT-NODE`, `REF-PROMPT-TF-AWS`, `REF-PROMPT-QUALITY`; `CLAUDE.md`.
- **Required Human Review**: Security review of the mail interface, every template, and the egress and IAM configuration; privacy review that no message class carries health data.
- **Open Decisions**: None. SEC-EXT-3 and the SQ-7 addendum fix the service, the interface ownership, and the content rules.

**Estimated effort**: 1–2 engineer-days. **Estimated changed lines**: 400–800.
**Recommended model**: Claude Opus (`claude-opus-5`) — a cross-cutting channel where enumeration resistance and content confinement must hold across every flow that sends.
