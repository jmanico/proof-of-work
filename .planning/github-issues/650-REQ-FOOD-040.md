# [REQ-FOOD-040] AI-assisted nutrition estimation flow

## Metadata

- **ID**: REQ-FOOD-040
- **Title**: AI-assisted nutrition estimation flow
- **Version**: 1.0.1
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: Critical
- **Requirement Type**: Security
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-8.12, FR-9.12, FR-9.2, FR-9.9, FR-2.11; `SECURITY.md` SEC-AI-2, SEC-AI-3, SEC-INPUT-7, SEC-HTTP-5

## Requirement

- **Statement**: The REST API Application MUST offer a subscriber AI-assisted estimation of calories and the FR-6.2 macronutrients (protein, carbohydrate, fat) from a food description or photo, returned as an editable pre-fill carrying a persistent estimate label that the subscriber reviews and confirms before any food log entry is saved; the estimation request MUST be refused for an account without recorded consent, with consent withdrawn, or with an unverified email address, the photo MUST pass SEC-INPUT-7 content validation and be processed transiently with metadata stripped, and the model's response MUST be schema-validated with FR-8.9 range checks before presentation.
- **Rationale**: FR-8.12 defines the estimation flow and its consent gates: an estimation request is itself collection and processing of health data (FR-9.12), so it is gated identically to a health-data write even though nothing is stored until the subscriber confirms. SEC-AI-2 treats descriptions, photos, and model output as untrusted (prompt-injection vectors and unvalidated numerics); SEC-AI-3 keeps photos transient and the estimate label end-to-end; SEC-INPUT-7 validates the system's only file-upload surface; SEC-HTTP-5 bounds inference cost (threat: denial-of-wallet, `SECURITY.md` Threat Model addendum 2026-08-01).
- **Assumptions**: The in-boundary inference service exists and is configured per REQ-FOOD-050 (SEC-AI-1); the food log entry write path exists (REQ-FOOD-020) and enforces FR-8.9 on save; consent state is recorded and queryable (REQ-PRIVACY-010).
- **Out of Scope**: The inference service's account placement, zero-retention verification, and model pinning (REQ-FOOD-050); dataset-search and manual food entry, which are the other two equal entry methods (REQ-FOOD-010, REQ-FOOD-020); the FR-8.5 daily target comparison (REQ-FOOD-030); consent capture and withdrawal mechanics themselves (REQ-PRIVACY-010, REQ-PRIVACY-020); email-verification mechanics (REQ-AUTH-090); the shared rate-limiting machinery (REQ-API-050) — this issue fixes only the estimation-specific thresholds.
- **Design Traceability**: `DESIGN.md` — Product Patterns → "Logging and AI-assisted food estimates" (three equal entry methods, AI never the default, the pre-estimation notice "Your description or photo is used only to create this estimate. Photos are not saved.", editable fields with a persistent `Estimate` label and "Review and adjust before saving", confirmation saves subscriber-edited values); Imagery (photo shown only as a transient local preview labelled "Not saved", removed on confirm, cancel, or failure); Status chips (`Estimate` vocabulary); Design Verification (no photos retained in browser storage).
- **Architecture Traceability**: `ARCHITECTURE.md` — AI Inference component (inputs, outputs, "owns no durable data"); trust boundary 6 (REST API Application → AI Inference: model output is untrusted input revalidated before use; descriptions and photos are prompt-injection vectors); trust boundary 1 (client-supplied content); DR-2, DR-9.
- **Security Traceability**: SEC-AI-2, SEC-AI-3, SEC-INPUT-7, SEC-HTTP-5; supports SEC-DATA-2 (estimation refused without consent), SEC-INPUT-1, SEC-LOG-3, SEC-TB-3/FR-9.8.

## Scope

- **Applies To**: Multiple — API, Server-Side Application, Web Client
- **Components**: REST API Application; AI Inference; Browser Client
- **Interfaces / Operations**: The AI estimation request operation (description variant and photo-upload variant); the food log entry save operation insofar as it carries the estimate label; the client estimation view
- **Actors**: Subscriber (only role that logs food); anonymous attacker and other roles are denied upstream
- **Preconditions**: Authenticated subscriber session; active subscription entitlement; recorded, unwithdrawn consent (FR-9.2, FR-9.9); verified email address (FR-2.11)
- **Data Classification**: Restricted
- **Personal or Regulated Data**: Health Data — an estimation request is collection and processing of health data (FR-9.12)
- **Jurisdiction / Regulatory Scope**: Per `SECURITY.md` SQ-1 (RESOLVED): GDPR and UK GDPR for EU/UK data subjects; CCPA/CPRA, US state consumer-health laws (e.g. Washington My Health My Data), and the FTC Health Breach Notification Rule for US users; HIPAA not applicable

## Security Context

- **Security Objectives**: Privacy, Confidentiality, Integrity, Availability, Safety
- **Control Layers**: Input Validation, Business-Rule Validation, Sanitization, Data Protection, Availability, Authorization
- **Threat References**: `SECURITY.md` Threat Model addendum 2026-08-01 — prompt injection through food descriptions and images, unsafe or materially inaccurate estimation output, inference cost abuse (denial-of-wallet); TM-I-5 (health data leaking into logs); CWE-434 (unrestricted upload of file with dangerous type), CWE-409 (resource consumption amplification / decompression), CWE-200 (exposure of sensitive information — EXIF/GPS), CWE-20 (improper input validation of model output)
- **Abuse / Misuse Case**: A subscriber submits a crafted description or image that attempts to escape the estimation task or elicit tool/retrieval behavior; an attacker uploads a decompression bomb, a polyglot file with mismatched magic bytes, or a GPS-bearing photo hoping location metadata reaches or persists in the system; a scripted client burns inference spend with rapid requests; a client submits an estimation request for an account whose consent is absent or withdrawn or whose email is unverified.
- **Trust Boundary**: Boundary 1 (Browser Client → REST API Application: description, photo, and confirmed values enter) and boundary 6 (REST API Application → AI Inference: model output re-enters as untrusted data).
- **Untrusted Inputs or Assertions**: The food description; the uploaded photo bytes, their declared `Content-Type`, and filename; the model's estimation response; any client assertion of consent, verification, or entitlement state.
- **Authoritative Enforcement Point**: The REST API Application — consent/verification gates before dispatch, SEC-INPUT-7 validation before decode, schema and FR-8.9 range validation of the model response before presentation, and FR-8.9 validation again on save.
- **Independent Verification**: Consent, withdrawal, and email-verification state are read from persisted state, never from client claims (DR-3); the estimate label on a saved entry is set server-side from the request path taken, not from a client flag.
- **Zero Trust Relevance**: TO BE DECIDED — not verified against NIST SP 800-207 in this session; the operative principle is that model output crossing boundary 6 is revalidated before use.

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — mapping deferred to the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: TO BE DECIDED — SEC-AI-3 records the AISVS mapping as pending with SQ-10.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — not verified against the catalog in this session.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: Consent-gated processing of health data under the SQ-1 regime set (GDPR/UK GDPR; CCPA/CPRA; Washington My Health My Data; FTC HBNR). Specific article/section mappings: TO BE DECIDED — the spec documents state no section for this behavior.
- **Other**: `REF-INPUT`, `REF-PROMPT-API`, `REF-ASVS-5` (SEC-INPUT-7, SEC-AI-2); `REF-LOG` (SEC-AI-3).
- **Mapping Basis**: The listed references are those the governing SECURITY.md rules themselves cite; CWE identifiers describe the upload, decompression, metadata-exposure, and output-validation weakness classes this requirement closes.

## Acceptance Criteria

1. **AC-01 — Expected behavior (description path)**: Given a verified, consented subscriber with an active subscription, when they submit a food description for estimation, then the REST API Application dispatches it to AI Inference, schema-validates the response, checks calories and protein/carbohydrate/fat against the FR-8.9 rules, and returns the values as an editable pre-fill marked as an estimate; when the subscriber edits and confirms, the saved food log entry contains the subscriber-confirmed values, passes FR-8.9 validation, and carries the estimate label.
2. **AC-02 — Expected behavior (photo path)**: Given the same subscriber, when they upload a conforming JPEG, PNG, WebP, or HEIC photo within 10 MB and 12 decoded megapixels, then magic-byte inspection accepts it, EXIF and GPS metadata are stripped before the image reaches AI Inference, HEIC is transcoded in memory, and after the estimation response is returned no copy of the photo exists in persistence, logs, or any storage inspectable after the request completes.
3. **AC-03 — Boundary or failure behavior (consent and verification gates)**: Given an account with no consent record, with consent withdrawn, or with an unverified email address, when an estimation request is submitted (description or photo), then it is refused before anything is sent to AI Inference, no data reaches the model, and the refusal states the applicable reason class so the client can present the DESIGN.md permission/withdrawn-consent state.
4. **AC-04 — Boundary or failure behavior (photo validation)**: Given an upload whose content bytes do not match an accepted format (regardless of extension or declared `Content-Type`), whose decoded dimensions exceed 12 megapixels, that exhibits a decompression anomaly, or whose body exceeds 10 MB, when it is submitted, then it is rejected with a non-descriptive error before full decoding or inference, and no partial image data is retained.
5. **AC-05 — Boundary or failure behavior (rate limits)**: Given a subscriber who has made 5 estimation requests within the current minute or 50 within the current day, when they submit another, then it is refused under the SEC-HTTP-5 thresholds without invoking AI Inference, and the counters are per account.
6. **AC-06 — Prohibited behavior**: Given a description or image crafted as a prompt-injection attempt, when it is processed, then the inference context grants the model no tool, retrieval, or data access beyond the estimation task and no content outside the estimation schema is ever presented to the subscriber; a model response that fails schema or range validation MUST NOT be presented or saved; a saved entry created via the estimate path MUST NOT lack the estimate label; and no photo bytes or health values MUST ever appear in application logs.

## Failure Behavior

- **On Invalid Input**: Reject per SEC-INPUT-1/SEC-INPUT-7 before inference — malformed descriptions fail schema validation with the failing field named; non-conforming photos are rejected with a non-descriptive error (no decoder internals, no format detail); nothing is dispatched to the model and nothing is stored.
- **On Authentication Failure**: Denied upstream by the deny-by-default guard (REQ-AUTHZ-010); uniform response per SEC-AUTHN-3.
- **On Authorization Failure**: Deny — non-subscriber roles, lapsed entitlement (SEC-AUTHZ-8), missing/withdrawn consent, and unverified email all refuse the request; the consent and verification refusals name their reason class (they concern the actor's own account state, so disclosure is appropriate), and no resource existence question arises.
- **On Security-Decision Failure**: Deny by default — an error resolving consent, verification, or entitlement state refuses the estimation request (SEC-AUTHZ-7 discipline).
- **On External Dependency Failure**: If AI Inference (Bedrock) times out or errors, the request fails with a generic error and correlation identifier; the transient photo is discarded; the subscriber can retry within rate limits or fall back to dataset search or manual entry (DESIGN.md: three equal methods); no automatic retry loop amplifies inference spend.
- **On System Error**: Generic error with correlation identifier (SEC-ERR-1); the photo and description are not persisted by any error path; no partial food log entry is created.
- **Logging / Audit**: Log estimation-request acceptance/refusal with account, reason class, and correlation identifier (SEC-LOG-4 for denials); MUST NOT log the description content, photo bytes, estimation values, or any health value (SEC-LOG-3, SEC-AI-3). The audit entry for the eventual health-data write occurs at entry save (REQ-FOOD-020, FR-9.7); this requirement adds no separate audit obligation because nothing is stored until confirmation (FR-9.12).
- **Alerting**: Rate-limit threshold breaches on the estimation endpoints route to the security lead as SEC-OPS-2 detection inputs (`SECURITY.md` SQ-3, SQ-11) — inference endpoints are a named denial-of-wallet target.

## Test Strategy

- **Unit Tests**: Consent/withdrawal/verification gate logic over all account-state combinations; magic-byte detection for JPEG, PNG, WebP, HEIC, and rejection of mismatches and unknown formats; megapixel-cap and decompression-anomaly rejection; EXIF/GPS strip function leaves no metadata; estimation-response schema and FR-8.9 range validation including out-of-range and non-numeric model outputs; estimate-label assignment from the request path.
- **Integration Tests**: End-to-end description and photo estimation against a stubbed inference service asserting pre-fill shape, label, and confirmed-save values; gate refusals asserting no inference dispatch occurred; save path revalidating per FR-8.9; storage inspection after photo estimation asserting no photo residue in the database, filesystem, or object storage.
- **Security Tests**: Prompt-injection corpus over descriptions and images asserting no context escape and schema containment (SEC-AI-2 verification); upload abuse suite — polyglot files, extension/content mismatch, decompression bombs, oversized bodies, metadata-bearing photos (SEC-INPUT-7 verification); burst tests at 5/minute and 50/day boundaries; log-content assertions that no description, photo, or health value is emitted; client storage inspection asserting no photo persists after confirm/cancel/failure (DESIGN.md Design Verification, SEC-RENDER-4).
- **Compliance Tests / Evidence**: Evidence that estimation requests are refused without consent, retained for the SQ-1 pre-launch counsel review.
- **Acceptance-Criteria Traceability**: AC-01 — description-path integration suite; AC-02 — photo-path integration suite plus residue inspection; AC-03 — gate-refusal suite; AC-04 — upload abuse suite; AC-05 — burst tests; AC-06 — injection corpus, invalid-model-output tests, label assertions, and log-content assertions.
- **Coverage Target**: Project coverage threshold 90% line and branch (`CLAUDE.md`, 2026-08-03); all gates, validation branches, and error paths MUST have positive and negative tests.
- **Required Test Environment**: Vitest with an inference-service stub returning conforming, non-conforming, and hostile responses; fixture images per format including malformed, oversized, bomb, and EXIF/GPS-bearing samples; accounts in each consent/verification state; Playwright for the client estimate flow and "Not saved" preview behavior.

## Dependencies

- **Upstream Requirements**: REQ-FOOD-020 (food log entry save path and attribution), REQ-FOOD-050 (in-boundary inference service), REQ-API-010 (allow-list schema validation), REQ-PRIVACY-010 (consent record and refusal)
- **Downstream Requirements**: None — the estimate path terminates in the REQ-FOOD-020 entry.
- **External Dependencies**: Amazon Bedrock, in-account (SEC-AI-1, via REQ-FOOD-050); an image decoding/transcoding dependency for HEIC and metadata stripping, subject to DEP-1–DEP-8.
- **Dependency Assumptions**: Bedrock is configured for zero prompt retention and no training use, verified per REQ-FOOD-050; the decoder dependency is vetted, current, and pinned (DEP-4, DEP-7) — image decoders are a classic memory-safety attack surface.
- **Failure Impact**: Inference unavailability degrades to the two non-AI entry methods with no data loss; a compromised or misconfigured decoder or a validation bypass would expose the server to hostile file content, and a gate failure would process health data without consent.

## Implementation Notes

- **Constraints**: Node.js/Fastify with route-level JSON Schema validation (`CLAUDE.md`); photo upload body limit 10 MB and JSON 1 MB (SEC-HTTP-5); estimation thresholds 5/minute burst and 50/day per account as named constants (SQ-3); photos held only in memory or ephemeral storage for the duration of the request (SEC-AI-3).
- **Prohibited Approaches**: Trusting file extension or declared `Content-Type` (SEC-INPUT-7); forwarding an unstripped image to inference; persisting or logging photo bytes on any path, including error paths; presenting or saving model output that has not passed schema and range validation; deriving the estimate label from a client-supplied flag; making AI the default or only entry method (DESIGN.md); coercing out-of-range model values instead of rejecting the response.
- **Implementation Guidance**: Run SEC-INPUT-7 checks in order of increasing cost — size, magic bytes, dimension header, bounded decode — so bombs are rejected before decoding. Treat the model call as a narrow function: fixed instruction context, one description or one image, structured-output schema; the subscriber's identity and other data never enter the prompt. The manual-entry path is the estimate-free case of the same save flow (FR-8.12), so implement one save endpoint with the label set server-side.
- **AI Development Guidance**: `REF-PROMPT-API`, `REF-PROMPT-NODE`, `REF-PROMPT-QUALITY`; `CLAUDE.md`. Human review of the inference context and the SEC-INPUT-7 pipeline is mandatory before merge.
- **Required Human Review**: Security review of the upload validation pipeline, the inference context, and the gate ordering; privacy review that the estimation request is gated as health-data collection (FR-9.12).
- **Open Decisions**: OWASP AISVS mapping (`SECURITY.md` SQ-10); project coverage threshold (`CLAUDE.md`). Neither blocks implementation.

**Estimated effort**: 1.5–2 engineer-days. **Estimated changed lines**: 500–900.
**Recommended model**: Claude Fable (`claude-fable-5`) — a multi-surface flow (upload pipeline, inference boundary, consent gates, client pre-fill) whose breadth benefits from Fable; every gate and validation branch still carries exhaustive negative tests.
