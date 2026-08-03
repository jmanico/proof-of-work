# [REQ-FOOD-010] Bundled nutrition dataset import, versioning, and search

## Metadata

- **ID**: REQ-FOOD-010
- **Title**: Bundled nutrition dataset import, versioning, and search
- **Version**: 1.0.0
- **Status**: Draft
- **Owner**: TO BE DECIDED
- **Author**: Jim Manico
- **Last Updated**: 2026-08-03
- **Priority**: High
- **Requirement Type**: Functional
- **Source / Parent**: REQ-EPIC-001; `REQUIREMENTS.md` FR-8.11 (OQ-5 RESOLVED); `SECURITY.md` SEC-EXT-1, DEP-5, DEP-7; `ARCHITECTURE.md` DR-5

## Requirement

- **Statement**: The system MUST import the bundled nutrition dataset — initially USDA FoodData Central — into Relational Persistence at build or deploy time as a versioned, integrity-verified artifact, MUST record the dataset version in use, MUST let an authenticated subscriber search the imported dataset and obtain calories and the FR-6.2 macronutrients (protein, carbohydrate, fat) computed for a chosen quantity, and MUST NOT issue runtime requests to any external nutrition service.
- **Rationale**: FR-8.11 keeps nutrition lookup inside the system boundary so no user data reaches a third-party nutrition service (FR-9.8) while still giving subscribers a searchable food database; version recording and integrity verification make the imported artifact auditable supply chain (DEP-5, DEP-7) rather than an unpinned download.
- **Assumptions**: The CI/CD migration-and-import path (trust boundary 4) exists per `ARCHITECTURE.md` DR-5 and REQ-BUILD-010; authentication and subscription entitlement in front of the search operation are enforced by REQ-AUTHZ-010 and REQ-ENTITLE-010.
- **Out of Scope**: Creating, editing, or deleting food log entries from a search result (REQ-FOOD-020); AI-assisted estimation (REQ-FOOD-040, REQ-FOOD-050); the daily target comparison (REQ-FOOD-030); the choice of a successor dataset version (an operator decision executed through this import path).
- **Design Traceability**: `DESIGN.md` — Product Patterns, "Logging and AI-assisted food estimates": dataset search is one of three equal entry methods; numeric values use the data face with the account's unit suffix.
- **Architecture Traceability**: `ARCHITECTURE.md` — Relational Persistence ("the bundled nutrition dataset (FR-8.11) is the sole imported external content, ingested … as a versioned, integrity-verified artifact"); entity "nutrition dataset item"; DR-5 (the CI/CD migration-and-import path via trust boundary 4 is a sanctioned persistence path); no external-integration boundary exists.
- **Security Traceability**: SEC-EXT-1 (dataset is a build- or deploy-time artifact under DEP-5/DEP-7 discipline); SEC-TB-3 and FR-9.8 (no external transmission); SEC-INPUT-1 (search input validation); SEC-AUTHN-1 (authenticated surface).

## Scope

- **Applies To**: Multiple — Server-Side Application, API, and the build/deploy import path
- **Components**: REST API Application; Relational Persistence; CI/CD import path (trust boundary 4)
- **Interfaces / Operations**: Dataset import step in the build/deploy pipeline; dataset-version read operation; authenticated food search operation with quantity-scaled nutrient computation
- **Actors**: Subscriber (search); CI/CD pipeline identity (import); anonymous internet attacker (denied)
- **Preconditions**: For search — an authenticated session with active subscription entitlement (FR-3.1). For import — a dataset artifact with a recorded version and integrity hash.
- **Data Classification**: Multiple — the imported dataset is public-domain reference data; the search operation runs inside the authenticated, subscription-gated surface
- **Personal or Regulated Data**: None — dataset items are public reference data, and this requirement stores no subscriber input; entries combining dataset values with subscriber input are REQ-FOOD-020
- **Jurisdiction / Regulatory Scope**: Global service, single US primary region, GDPR as design ceiling (`SECURITY.md` SQ-1); no health data is processed by this requirement itself

## Security Context

- **Security Objectives**: Integrity, Availability, Confidentiality
- **Control Layers**: Supply Chain, Architecture, Input Validation
- **Threat References**: `SECURITY.md` Threat Model Addendum 2026-08-01 (model/dataset supply chain, covered by DEP-5/DEP-7); TM-I-5 (data leaking outward — prevented here by having no outbound nutrition call at all); CWE-345 (insufficient verification of data authenticity)
- **Abuse / Misuse Case**: A tampered or substituted dataset artifact injects false nutrition values at import; a compromised dependency or developer shortcut reintroduces a runtime call to an external nutrition API, leaking subscriber search terms; an anonymous attacker uses the search endpoint unauthenticated or as an injection vector.
- **Trust Boundary**: Boundary 4 (CI/CD-and-IaC path → production) for the import; boundary 1 (Browser Client → REST API Application) for search; boundary 3 (REST API Application → Relational Persistence) for queries.
- **Untrusted Inputs or Assertions**: The dataset artifact until its integrity hash verifies; search terms and quantity parameters from the client.
- **Authoritative Enforcement Point**: The import step verifies artifact integrity before any write; the REST API Application validates and serves all search requests — the client never reads the dataset store directly (DR-1).
- **Independent Verification**: Artifact hash verification is performed by the import step itself, independent of where the artifact was produced; search queries are parameterized (SEC-INPUT-5) and schema-validated (SEC-INPUT-1) server-side regardless of client behavior.
- **Zero Trust Relevance**: TO BE DECIDED — per-issue NIST SP 800-207 mapping is deferred to the independent pre-launch assessment (`SECURITY.md` SQ-10).

## Standards Alignment

- **OWASP ASVS 5.0.0**: TO BE DECIDED — per-issue mapping deferred to the independent pre-launch assessment (`SECURITY.md` SQ-10).
- **OWASP AISVS 1.0**: N/A — no AI-enabled component is involved in this requirement.
- **NIST SP 800-53 Rev. 5**: TO BE DECIDED — deferred with SQ-10.
- **NIST SP 800-207**: TO BE DECIDED — see Zero Trust Relevance.
- **Regulatory**: N/A — this requirement processes public-domain reference data only; health-data regimes attach at REQ-FOOD-020 where subscriber entries are created.
- **Other**: `REF-SUPPLY`, `REF-DEPS` (artifact discipline per DEP-5/DEP-7); `REF-INPUT` (search validation); USDA FoodData Central public-domain dataset as named by FR-8.11.
- **Mapping Basis**: FR-8.11 and SEC-EXT-1 place the dataset under the dependency-security rules, whose named references are `REF-SUPPLY` and `REF-DEPS`; search input handling falls under SEC-INPUT-1's `REF-INPUT`.

## Acceptance Criteria

1. **AC-01 — Expected behavior (import and versioning)**: Given a dataset artifact whose integrity hash matches its recorded value, when the build/deploy import step runs, then the dataset items are loaded into Relational Persistence, the dataset version in use is recorded, and that version is readable through the application afterward.
2. **AC-02 — Expected behavior (search and computation)**: Given an authenticated, entitled subscriber and an imported dataset, when they search for a food term and choose a matching item and a quantity, then the response returns the item with calories, protein, carbohydrate, and fat computed for that quantity, suitable for storing on a food log entry (REQ-FOOD-020).
3. **AC-03 — Boundary or failure behavior (integrity failure)**: Given a dataset artifact whose integrity hash does not match, when the import step runs, then the import fails, the deploy does not proceed, and no partial or unverified dataset content is served; the previously imported dataset, if any, remains in use unchanged.
4. **AC-04 — Prohibited behavior (no external calls)**: Given any search, item lookup, or quantity computation at runtime, when the operation executes, then no request is issued to any external nutrition service — there is no code path, configuration, or fallback that performs a runtime external nutrition lookup.
5. **AC-05 — Prohibited behavior (history immutability)**: Given existing food log entries with values stored on the entry, when a newer dataset version is imported, then no stored entry value changes and no entry re-derives its values from the updated dataset.
6. **AC-06 — Boundary or failure behavior (unauthenticated and invalid input)**: Given a search request with no valid session, or with a malformed term, non-numeric quantity, or out-of-schema parameter, when it arrives, then it is rejected by the deny-by-default guard (REQ-AUTHZ-010) or schema validation (REQ-API-010) without executing a dataset query.

## Failure Behavior

- **On Invalid Input**: Search requests failing allow-list schema validation are rejected with the failing field named, per SEC-INPUT-1 and SEC-ERR-1; no query executes and no state changes.
- **On Authentication Failure**: Deny per REQ-AUTHZ-010 with a uniform response; the dataset is not reachable unauthenticated.
- **On Authorization Failure**: Deny per the entitlement gate (FR-3.1, REQ-ENTITLE-010) with the subscription-required explanation defined there; dataset content existence MAY be considered non-sensitive but is still not served without entitlement.
- **On Security-Decision Failure**: Deny by default (SEC-AUTHZ-7); an error resolving entitlement or session denies the search.
- **On External Dependency Failure**: N/A at runtime — there is no runtime external dependency by construction. At import time, an unavailable or corrupt artifact fails the deploy (AC-03); the system continues serving the last successfully imported dataset.
- **On System Error**: Import runs transactionally: a failed import leaves the prior dataset intact and serving. Runtime errors return a generic message with a correlation identifier (SEC-ERR-1).
- **Logging / Audit**: The import step logs the dataset version, artifact hash, item count, and outcome. Search requests are ordinary authenticated requests under SEC-LOG-4; no FR-9.7 audit entry is required because dataset items are not health data (FR-9.12) and no health record is read or written. Search terms MUST NOT be written to logs as personal profiling data (SEC-LOG-3 spirit); log the request outcome, not the term.
- **Alerting**: An import integrity-verification failure is a supply-chain signal: it fails the pipeline and routes a threshold alert to the security lead as a SEC-OPS-2 detection input (SQ-11).

## Test Strategy

- **Unit Tests**: Quantity-scaling computation for calories, protein, carbohydrate, and fat, including fractional quantities and rounding; search-parameter validation; version-recording read.
- **Integration Tests**: Full import of a fixture dataset artifact into PostgreSQL, then search and quantity computation through the API; re-import at a newer version asserting existing entry rows are untouched (with REQ-FOOD-020 fixtures); version endpoint reflects the imported version.
- **Security Tests**: Import with a corrupted artifact asserting deploy failure and no partial data; egress assertion that no outbound nutrition-service request occurs during search (network-level test double that fails the test on any unexpected egress); SQL-injection suite over the search term (SEC-INPUT-5); unauthenticated search enumeration asserting denial.
- **Compliance Tests / Evidence**: Recorded dataset version and artifact hash for each deploy, retained as supply-chain evidence under DEP-5/DEP-7.
- **Acceptance-Criteria Traceability**: AC-01 — import integration suite; AC-02 — search/computation suite; AC-03 — corrupted-artifact test; AC-04 — egress assertion test; AC-05 — re-import immutability test; AC-06 — unauthenticated and malformed-input suite.
- **Coverage Target**: Project coverage threshold is TO BE DECIDED (`CLAUDE.md`); all computation and integrity-failure paths MUST have positive and negative tests.
- **Required Test Environment**: A small fixture dataset artifact with a valid and an invalid hash; PostgreSQL via the migration tooling; HTTP test client with authenticated subscriber fixtures; a network egress recorder; Vitest as the runner.

## Dependencies

- **Upstream Requirements**: REQ-BUILD-010 (workspace, migration tooling, and the build/deploy path the import rides); REQ-API-010 (allow-list schema validation for the search operation)
- **Downstream Requirements**: REQ-FOOD-020 (entries store values computed here); REQ-FOOD-030 (daily totals aggregate those entries); REQ-FOOD-040 (estimation is the alternative entry method)
- **External Dependencies**: USDA FoodData Central as a build-time artifact only — public domain, imported and versioned, never called at runtime
- **Dependency Assumptions**: The artifact's provenance and license (public domain) hold; DEP-5 vulnerability and DEP-6 transitive-tree review apply to any parsing library used by the import step.
- **Failure Impact**: If the import path fails, food logging by search is unavailable and subscribers fall back to manual entry and estimation (REQ-FOOD-020, REQ-FOOD-040); no health data is at risk because this path stores none.

## Implementation Notes

- **Constraints**: TypeScript across workspaces; PostgreSQL with Drizzle ORM and drizzle-kit, with the import executed through the sanctioned CI/CD path of DR-5 (`CLAUDE.md`, `ARCHITECTURE.md`); Fastify route-level JSON Schema validation for the search operation.
- **Prohibited Approaches**: Runtime calls to any nutrition API, including as a "fallback" when a search misses; re-deriving stored entry values from the current dataset at read time; importing the dataset through an unsanctioned direct-to-database path outside trust boundary 4; unparameterized search queries.
- **Implementation Guidance**: Store the dataset version and artifact hash in a dedicated metadata row written in the same transaction as the import so version and content cannot diverge. Full-text or trigram search stays inside PostgreSQL; no external search service is introduced (DEP-1, DEP-2).
- **AI Development Guidance**: `REF-PROMPT-NODE`, `REF-PROMPT-API`, `REF-PROMPT-QUALITY`; `CLAUDE.md`. Human review required for the import pipeline's integrity-verification step.
- **Required Human Review**: Security review of the import path and the egress posture; architecture review that the import uses only the DR-5-sanctioned path.
- **Open Decisions**: None for this requirement. The FoodData Central subset, item count, and search-index tuning are implementation-level choices decided with the code (`ARCHITECTURE.md`, Relational Persistence open decisions).

**Estimated effort**: 1–1.5 engineer-days. **Estimated changed lines**: 300–600.
**Recommended model**: Claude Opus (`claude-opus-5`) — an import pipeline with integrity gating plus a search surface whose egress and injection posture must be exactly right.
