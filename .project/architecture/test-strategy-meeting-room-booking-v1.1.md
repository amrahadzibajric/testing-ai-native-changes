# Test Automation Strategy: Meeting Room Booking Pilot

**Version:** 1.1
**Date:** 2026-09-29
**Status:** Superseded by test-strategy-meeting-room-booking-v1.2.md (2026-09-29)

## Change Log

| Version | Date | Change | Impact on QA artifacts |
|---------|------|--------|------------------------|
| 1.0 | 2026-09-29 | Initial strategy | — |
| 1.1 | 2026-09-29 | CI/CD platform confirmed as Azure DevOps Pipelines (ADR-0023) and stack confirmed as .NET (ADR-0016): assumptions removed, pipeline stages unchanged. Access gate (ADR-0021, Accepted) added: the deployed test environment is gated like production, so E2E, ZAP and k6 must pass the gate; NFR-009 now has an automated reachability check; SEC-12 checks added | No framework changes. Any test scaffold that targets the deployed test environment must obtain a gate token (new section). Nothing existing breaks; no QA test plans or automation code were produced against v1.0 yet |

## Context

This strategy selects the tools and pipeline stages the QA agent and delivery team use to verify the Meeting Room Booking pilot. It is derived from `arch-meeting-room-booking-v1.1.md` (ASP.NET Core modular monolith with Razor Pages, Azure SQL, hosted status sweeper; stack Accepted in ADR-0016, CI/CD Accepted in ADR-0023, access gate Accepted in ADR-0021, other technology choices Proposed in ADR-0017 to ADR-0020 and ADR-0022), from `.project/product-definition/nfrs.md`, from the two integration assessments, and from `security-assessment-meeting-room-booking-v1.1.md`.

Test-writing standards (level, volume, naming, comments, skips, traceability, automation candidates) come from the `test-standards` skill and are not restated here.

**Unit testing:** Unit tests are developer-owned and outside the scope of this strategy. Framework selection for unit tests is an implementation team decision.

Three characteristics of this system shape the strategy. First, many behaviours are time-driven (check-in window, no-show after 15 minutes, done, 2-hour late rule, DST), so every level needs a controllable clock (ADR-0019). Second, the correctness-critical property is that a room is never double booked under concurrency (NFR-005, ADR-0020), which needs real database tests, not mocks. Third, the deployed environments sit behind a Microsoft sign-in gate (ADR-0021), so anything that drives the deployed test environment must pass the gate first, while tests below the deployed level do not see the gate at all.

---

## Test Levels

| Level | Scope | Framework / Tool | Rationale |
|-------|-------|------------------|-----------|
| Integration | Outcomes observable across the application-database boundary: persisted booking state read back through the app, concurrency refusal, status sweeper transitions with a fake clock, import outcomes, DST behaviour, audit content including gate identity fields | xUnit with Testcontainers for SQL Server and a fake clock (`FakeTimeProvider`) | Real SQL Server semantics are required to prove the per-room lock and transactional overlap check; containers give repeatable local and CI runs |
| API / Contract | Behaviour of the app's HTTP endpoints and pages per acceptance criterion, including named refusal messages and role boundaries (employee versus admin, owner-only check-in) | xUnit with `WebApplicationFactory` plus HTTP client assertions against the in-memory app and containerised database | No separate public API exists; the HTTP surface of the app is the contract. The gate is not present in the in-memory host; gate headers are simulated where audit tests need them |
| E2E / UI | Full user journeys through the running application in desktop and phone viewports (E2E-001 to E2E-014) | Playwright (.NET) | Same language as the app, first-class mobile viewport emulation for check-in, built-in tracing and accessibility hooks |
| Performance | NFR-003 page response, NFR-004 import time, NFR-005 concurrent booking correctness under load | k6 | Small scripts, threshold assertions map directly to NFR targets, runs from CI |
| Security | Web vulnerability baseline, dependency vulnerabilities, auth and access boundary checks, gate reachability (NFR-009, SEC-01, SEC-12) | OWASP ZAP baseline scan; `dotnet list package --vulnerable` plus Azure DevOps dependency scanning; access-boundary checks in the API / Contract suite; an unauthenticated reachability probe in the pipeline | Matches the design-level findings (SEC-01 to SEC-12) with free tools sized for a pilot |

---

## Framework Evaluation

### xUnit with Testcontainers for Integration and API / Contract

- **Alternatives considered:** NUnit with Testcontainers; SQLite or in-memory EF provider; a shared test SQL database.
- **Key criteria:** fidelity to Azure SQL locking behaviour, speed, language alignment with the confirmed .NET stack, CI friendliness.
- **Decision:** xUnit plus Testcontainers (SQL Server image). SQLite and the in-memory provider do not reproduce row locks or SQL Server collation (case-insensitive email uniqueness), so they cannot prove NFR-005 or NFR-023. A shared database creates cross-test interference. NUnit is equivalent; xUnit is the default in `WebApplicationFactory` samples. Microsoft-hosted Azure DevOps agents run Docker, which Testcontainers needs.

### Playwright (.NET) for E2E

- **Alternatives considered:** Playwright (TypeScript), Cypress, Selenium.
- **Key criteria:** language alignment, mobile viewport emulation, reliability, tracing, low maintenance.
- **Decision:** Playwright for .NET, so one language and one build across app and tests.

### k6 for Performance

- **Alternatives considered:** Azure Load Testing (managed JMeter), JMeter, Artillery.
- **Key criteria:** threshold assertions, scripting simplicity, cost, CI use.
- **Decision:** k6 run from the CI agent against the test environment. Load is small (up to about 20 concurrent virtual users); a managed load service is unnecessary cost.

### OWASP ZAP baseline for Security

- **Alternatives considered:** Burp Suite (licensed), commercial scanners, manual only.
- **Key criteria:** cost (free), CI automation, coverage of OWASP Top 10 basics.
- **Decision:** ZAP baseline scan against the test environment (authenticated through the gate, see below), plus dependency scanning. Not a substitute for a pre-go-live infrastructure review (see Open Questions).

### Controllable clock for all levels

- **Decision:** The application reads time only through an injectable clock (ADR-0019). Integration and API tests inject `FakeTimeProvider`. For E2E on a deployed test environment, a test-only endpoint sets an offset to the clock; it is compiled out or disabled outside the test environment (SEC-11) and the pipeline asserts it is absent in production.

### Passing the access gate on the deployed test environment

- **Problem:** The deployed test environment is gated by Easy Auth (ADR-0021) because it exposes a test-only clock control and must not be reachable by outsiders. Interactive Microsoft sign-in, especially with MFA, is unsuitable for automated runs.
- **Alternatives considered:** leave the test environment ungated; allow-list the Azure DevOps agent addresses; automate interactive sign-in with a test user; present a bearer token obtained by a pipeline service principal.
- **Decision (recommended, to confirm with client IT):** the pipeline obtains an access token for the gate's app registration using a service principal (client credentials, workload identity federation where available) and sends it as a bearer token on every request from Playwright's API context, ZAP and k6; the gate app registration must accept that audience. Leaving the test environment ungated is rejected (SEC-11, SEC-12). Automated interactive sign-in is rejected as brittle. The pick-a-user step then runs as in production, using the seeded people.
- **Fallback if bearer tokens are refused by client IT:** a test user account excluded from MFA by Conditional Access, with Playwright storing its sign-in state; k6 and ZAP then reuse the session cookie. This is the weaker option.

---

## Test Infrastructure

### Local Development

```bash
# Integration and API / Contract tests (requires Docker for the SQL Server container)
dotnet test tests/RoomBooking.IntegrationTests

# E2E tests (requires running app against a local container database; no gate locally; browsers installed once)
pwsh tests/RoomBooking.E2E/bin/Debug/net*/playwright.ps1 install
dotnet test tests/RoomBooking.E2E

# Performance smoke (requires k6 and a running app)
k6 run tests/performance/booking-smoke.js
```

Project and folder names are indicative; the implementation team fixes them at scaffold time. The gate does not exist locally, so local runs never need a token.

### CI/CD Integration

The CI/CD platform is Azure DevOps Pipelines (ADR-0023), with YAML multi-stage pipelines.

| Pipeline Stage | Test Levels Run | Gate (pass required?) | Timeout |
|---------------|----------------|----------------------|---------|
| PR validation | Integration, API / Contract, dependency vulnerability scan | Yes | 10 min |
| Merge to main (deploy to test environment) | Integration, API / Contract, gate reachability probe, E2E / UI smoke set (one journey per module) | Yes | 25 min |
| Nightly | Full E2E / UI, Performance, Security (ZAP baseline) | No (alert) | 60 min |
| Pre-go-live | Full E2E, Performance, Security, gate reachability on production, backup restore rehearsal | Yes (release gate) | Manual |

Parallelism: integration classes share a single container per test run using database-per-test-class via Respawn or fresh databases; E2E runs sequentially against one test environment.

### Test Environments

| Level | Environment | Provisioning | Isolation |
|-------|------------|--------------|-----------|
| Integration | Local and CI containers | Testcontainers on demand | Fresh database or reset (Respawn) per test class |
| API / Contract | In-memory app host with container database | `WebApplicationFactory` | Seeded per test; no shared state; no gate |
| E2E / UI | Azure test environment (App Service B1, Basic SQL), same EU region as production, gated like production | Deployed from main by the pipeline; Bicep | Reset to seed data at the start of each run; test-only clock endpoint enabled; tests present a gate token |
| Performance | Same test environment, outside working hours | Scheduled | No other traffic during run; seeded with production-like volume; gate token |
| Security | Same test environment | Scan on nightly run | Read-only baseline scan; no destructive actions; gate token supplied to the scanner, plus an unauthenticated probe |

---

## Test Data Strategy

| Level | Strategy |
|-------|---------|
| Integration | Builders create rooms, people (employee and admin) and bookings per test; fixed fake-clock start time; reset between classes |
| API / Contract | Seed the reference set: rooms Focus (2), Team (6), Board (12) active, Old Room (4) inactive, one admin, several employees; each test adds only what it needs; DST cases use fake-clock dates around 25 October 2026 and 28 March 2027; gate identity headers set explicitly when the audit content is under test |
| E2E / UI | Reset endpoint (test environment only) restores the seed data; scenarios that need a three-booking user or a pending no-show create them through the app setup helpers |
| Performance | Bulk seed script: 50 people, 4 rooms, about 5,000 bookings across past and future dates; import file generators for 300 and 500 rows including known conflicts |
| Security | Scan-only; uses seeded test data with no real personal data. Real names and emails are never used in test environments |

CSV import fixtures: a valid file, a file with each rejection reason (unknown room, unknown person, outside window, off grid, wrong duration, capacity exceeded, overlap, beyond the 3rd booking, duplicate), a wrong-header file, a file with formula-leading cells (SEC-06), and an oversized file.

---

## Coverage Targets

Coverage is measured against acceptance criteria, not against code structure or endpoint matrices (`test-standards` §2, §6).

| Level | Coverage Target | Measurement | Rationale |
|-------|----------------|-------------|-----------|
| Integration + API | Every numbered acceptance criterion has exactly one passing test case (plus one per failure or access condition the requirements name) | Acceptance Criteria Traceability table in each story's `verification-report.md` | Product-recognisable behaviour is what ships |
| E2E / UI | One happy-path scenario per feature; a failure path only where requirements name one; E2E-001 to E2E-014 map to features | Scenario traceability in `e2e-verification-report.md` | Business-critical flows verified end to end |
| Performance | All NFRs with measurable thresholds (table below) | k6 threshold assertions | SLA-style compliance at pilot scale |
| Security | OWASP Top 10 baseline scan clean of High findings; access boundaries named in acceptance criteria; SEC-03, SEC-04, SEC-06 mitigations have tests; gate reachability (SEC-01, SEC-12) | ZAP report, dependency report, API boundary tests, reachability probe result | Pilot risk posture |

### NFR to test mapping

| NFR | Test level | Test | Threshold |
|-----|-----------|------|-----------|
| NFR-003 | Performance | Availability search and booking page load, 20 virtual users | p95 under 2 s |
| NFR-004 | Performance / Integration | Import of 300 and 500 rows | Under 60 s end to end |
| NFR-005 | Integration; Performance | N simultaneous identical booking requests for one slot; mixed manual booking during import | Exactly one succeeds; zero overlapping active bookings |
| NFR-006 | Integration (fake clock) | Sweeper run at start+15 min and at end | Status changed within one sweeper interval (under 1 min); availability correct even before the sweeper runs |
| NFR-009 | Security (automated on test and production, plus manual pre-go-live) | Unauthenticated requests to the site and to a direct booking or admin URL are redirected to Microsoft sign-in and never return application content; a request with a valid tenant token reaches the picker; the deployment console is not reachable anonymously | Not reachable without sign-in (ADR-0021) |
| NFR-010 | API / Contract | Employee attempts admin pages, other person's bookings, non-owner check-in | Refused every time |
| NFR-011 | Security | ZAP baseline; HTTP to HTTPS redirect | No plain HTTP |
| NFR-012 | API / Contract | Availability response contains no owner data; my bookings show only own | As stated |
| NFR-015 | Integration (fake clock) | Rules around the 25 October 2026 and 28 March 2027 transitions; weekday and office-hours tests in winter and summer offsets | Correct in both offsets |
| NFR-016, NFR-018 | E2E | Desktop and phone viewports; automated accessibility checks (axe) on main screens | No serious violations |
| NFR-020 | Integration | Audit content for cancel, admin cancel, deactivation, import, including gate sign-in name and object identifier and the picked person (mismatch recorded, not blocked) | Actor, origin and gate identity recorded |
| NFR-023 | Integration | Duplicate email differing in case is refused | Refused |
| NFR-008 | Manual rehearsal | Point-in-time restore of the test database | Recover within one working day |

External systems: the spreadsheet CSV import has its own integration and E2E fixtures above. Microsoft Entra ID is used by the platform gate in the pilot; the application does not integrate with it, so no application contract test exists. The gate is verified by the reachability checks above. When the in-app sign-in phase starts, add an integration test using a stubbed OpenID Connect provider and a manual test against a developer tenant.

---

## Reporting & Observability

- **CI dashboards:** Azure DevOps test results and code coverage tabs; the acceptance-criteria traceability table remains the coverage view.
- **Failure notifications:** Team chat or email alert (Azure DevOps notifications) on nightly failure; PR status policy blocks on gate failure.
- **Trend tracking:** Retry counts per test reported to detect flakiness; time-dependent tests must use the fake clock and never real waits.
- **Artifact retention:** Test reports, Playwright traces and screenshots, ZAP report and k6 summaries kept as pipeline artifacts for 30 days.

---

## Risks & Assumptions

| # | Risk / Assumption | Impact | Mitigation |
|---|-------------------|--------|------------|
| 1 | Stack confirmed as .NET (ADR-0016); other technology ADRs (0017 to 0020, 0022) remain Proposed | Minor rework if a Proposed choice changes | Playwright and k6 are stack-independent; revisit the Integration/API rows only if the app stack changes |
| 2 | CI/CD platform confirmed as Azure DevOps Pipelines (ADR-0023). Organisation location, hosted-agent parallelism and the service connection into the client subscription are not yet arranged | Pipeline cannot run or deploy | Resolve with project lead and client IT |
| 3 | Azure test environment does not exist yet | E2E, performance and security runs blocked | Defer those tests in the deferred test registry until the environment is provisioned; integration and API tests run without it |
| 4 | Test-only clock endpoint is needed for E2E time scenarios | Risk of it reaching production (SEC-11) | Environment-gated, compiled out of release builds, pipeline assertion |
| 5 | Import format (ADR-0011) unconfirmed | Import fixtures may change | Keep fixtures generated from a single column definition |
| 6 | Real spreadsheet not seen | Real-world rejection patterns unknown | Add an anonymised sample as a fixture when Office Operations provides one |
| 7 | Access gate needs the client's Entra app registration and a route for automated tests to authenticate (bearer token from a pipeline service principal assumed) | Automated runs against the deployed test environment blocked until client IT agrees; NFR-009 falls back to a manual check | Raise with client IT together with the app registration request; keep the fallback test-user option; run integration and API tests, which need no gate, in the meantime |
| 8 | Unit tests are developer-owned | Gaps in rule-level logic unknown to QA | Integration and API tests cover every acceptance criterion; developers own unit coverage of the rules engine |
| 9 | Intake answers other than CI platform and stack (environment, compliance) were derived from the architecture document | Assumed answers may be wrong | Confirm with the delivery lead before first test scaffold |
| 10 | Conditional Access in the client tenant could require MFA or compliant devices for the test principal or test user | Blocks automated sign-in | Ask client IT for an exclusion for the pipeline principal |

---

## Key Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Integration test framework | xUnit with Testcontainers (SQL Server) and fake clock | Real locking semantics and deterministic time |
| API / Contract test framework | xUnit with `WebApplicationFactory` | The app's HTTP surface is the contract; no separate API |
| E2E / UI framework | Playwright (.NET) | Language alignment, mobile viewports, tracing |
| Performance tool | k6 | Simple thresholds mapping to NFRs, low cost |
| Security scanning | OWASP ZAP baseline plus dependency vulnerability scanning plus gate reachability probe | Free, CI-friendly, matches design findings |
| CI/CD platform | Azure DevOps Pipelines (ADR-0023) | Team platform; Docker-capable hosted agents |
| Time in tests | Injected clock; test-only offset endpoint in test environment only | Deterministic time-based rules; guarded by SEC-11 |
| Access to the gated test environment | Pipeline service principal bearer token (recommended, confirm with client IT) | Reliable automation without ungating the test environment |

---

## Open Questions

- [ ] Will client IT accept a service principal (bearer token) for automated access to the gated test environment, and exclude it from MFA where needed? Owner: project lead with client IT.
- [ ] When will the Azure test environment be provisioned in the EU region? Owner: project lead with client IT.
- [ ] Where does the Azure DevOps organisation live and who arranges the service connection and hosted parallelism? Owner: project lead and delivery lead.
- [ ] Who runs the pre-go-live infrastructure and access-gate check (SEC-01, SEC-12, ADR-0021)? Owner: project lead.
- [ ] Will Office Operations provide an anonymised spreadsheet export for import fixtures? Owner: project lead.
- [ ] Is a formal accessibility target beyond the WCAG 2.1 AA aim needed? Owner: Office Operations.
