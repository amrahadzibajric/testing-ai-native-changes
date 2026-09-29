# Test Automation Strategy: Meeting Room Booking

**Version:** 1.2
**Date:** 2026-09-29
**Status:** Draft

## Change Log

| Version | Date | Change | Impact on QA artifacts |
|---------|------|--------|------------------------|
| 1.0 | 2026-09-29 | Initial strategy | — |
| 1.1 | 2026-09-29 | CI/CD platform confirmed (ADR-0023), stack confirmed (ADR-0016), access gate added (ADR-0021) | No framework changes |
| 1.2 | 2026-09-29 | Scope change: the current phase is a local POC (ADR-0024). Tests run on developer machines against localhost with SQLite (ADR-0025); no pipeline, no deployed test environment, no gate. Integration and API tests use SQLite instead of Testcontainers for SQL Server (this reverses the v1.1 rejection of SQLite, for the POC only). E2E runs with Playwright against the local app. k6 and ZAP become optional local runs. The v1.1 pipeline, gate-token and Azure test-environment material is kept as "Target state (post-POC)". Import (M-06) is fully in the POC and has its own fixtures including a sample CSV | Breaking for any scaffold written against v1.1: the database fixture changes from a SQL Server container to SQLite; anything that expects a gate token or a deployed environment is postponed. No QA test plans or automation code were produced against v1.0 or v1.1 yet, so nothing existing breaks |
| 1.2 rev a | 2026-09-29 | Minor revision, append-only, same day, no version bump. Project lead decisions (ADR-0024): no demonstration clock control, so the Development-only clock offset control for E2E is removed everywhere (Controllable clock section, Key Decisions, Risks, target-state test environment). The injectable clock stays for integration and API tests only and is not exposed in the running application. E2E time-driven journeys run on the real clock using setup helpers that place bookings relative to the current time; no-show release and done marking are proven at integration level with the fake clock. A test asserts that no clock-control endpoint exists (SEC-11). Added tests for the configured bootstrap admin. Run instructions must forbid LAN binding and public tunnels (SEC-13). The delivery team prepares the sample CSV and seed data before development ends | Small: E2E scenarios must not depend on a clock endpoint. No QA test plans or automation code exist yet, so nothing existing breaks |

## Context

This strategy selects the tools and the way of running tests for the Meeting Room Booking application. It derives from `arch-meeting-room-booking-v1.2.md` (ASP.NET Core Razor Pages modular monolith, EF Core, SQLite for the POC and Azure SQL later, in-process status sweeper), from `.project/product-definition/nfrs.md`, from the two integration assessments and from `security-assessment-meeting-room-booking-v1.2.md`.

Test-writing standards (level, volume, naming, comments, skips, traceability, automation candidates) come from the `test-standards` skill and are not restated here.

**Unit testing:** Unit tests are developer-owned and outside the scope of this strategy. Framework selection for unit tests is an implementation team decision.

**POC testing in one paragraph.** The POC is built and demonstrated locally. All tests run on a developer machine with `dotnet test`, and there is no pipeline dependency. The functional scope is complete (M-01 to M-06 including the full import), so the test scope is complete too: every numbered acceptance criterion has a test, and the E2E scenarios cover the whole product. What is not tested in the POC is infrastructure: deployment, the sign-in gate, Azure SQL behaviour, hosted performance and hosted security scanning.

Three characteristics of this system shape the strategy. First, many behaviours are time-driven (check-in window, no-show after 15 minutes, done, 2-hour late rule, DST), so integration and API tests need a controllable clock (ADR-0019); the running application uses the real clock only. Second, the correctness-critical property is that a room is never double booked under concurrency (NFR-005, ADR-0020); in the POC this is proven against SQLite's single-writer serialisation and must be re-proven on Azure SQL before hosting. Third, the database engine differs between the POC and the target (ADR-0025), so the tests are written against behaviour, not against SQLite specifics, so they can be re-pointed at Azure SQL.

---

## Test Levels (POC)

| Level | Scope | Framework / Tool | Rationale | Runs |
|-------|-------|------------------|-----------|------|
| Integration | Outcomes observable across the application-database boundary: persisted booking state read back through the app, concurrency refusal, status sweeper transitions with a fake clock, import outcomes and report, DST behaviour, audit content, seed data | xUnit with SQLite (a temporary database file per test class, not the in-memory EF provider) and a fake clock (`FakeTimeProvider`) | Real relational behaviour, transactions and constraints with zero installation. A file database is used because concurrency tests need several connections to one database | `dotnet test` on a developer machine |
| API / Contract | Behaviour of the app's HTTP endpoints and pages per acceptance criterion, including named refusal messages and role boundaries (employee versus admin, owner-only check-in) | xUnit with `WebApplicationFactory` plus HTTP client assertions, backed by SQLite | The HTTP surface of the app is the contract; no separate public API exists | `dotnet test` |
| E2E / UI | Full user journeys through the running application in desktop and phone-size viewports (E2E-001 to E2E-014), including the import journey with the sample CSV | Playwright (.NET) against the app running on localhost | Same language as the app; mobile viewport emulation for check-in; tracing and accessibility hooks | `dotnet test` on a developer machine, before each demonstration |
| Performance | NFR-003 page response, NFR-004 import time, NFR-005 concurrent booking correctness | Optional: k6 against the local app for smoke thresholds; the import timing and the concurrency test also exist as integration tests | Local figures are indicative only and do not predict Azure figures | On request, developer machine |
| Security | Dependency vulnerabilities, access-boundary checks (NFR-010, NFR-012), anti-forgery (SEC-03), CSV formula escaping (SEC-06), localhost-only binding (SEC-13) | `dotnet list package --vulnerable`, access-boundary tests in the API / Contract suite; optional OWASP ZAP baseline against the local app | Covers the findings that apply to the POC (security assessment v1.2) with free tools | Before each demonstration build |

## Framework Evaluation (POC)

### xUnit with SQLite for Integration and API / Contract

- **Alternatives considered:** xUnit with Testcontainers for SQL Server (the v1.1 choice); EF Core in-memory provider; SQLite in-memory; NUnit.
- **Key criteria:** zero installation, no Docker requirement, speed, language alignment with .NET, fidelity of relational behaviour, ability to run the concurrency test with several connections.
- **Decision:** xUnit plus SQLite backed by a temporary file per test class. The EF in-memory provider is rejected because it has no real transactions or constraints. Shared in-memory SQLite is avoided for the concurrency test because it complicates multiple connections. SQL Server containers need Docker, which the project lead does not want to depend on now.
- **Limits, stated plainly.** SQLite does not reproduce SQL Server row locks, `rowversion` or collation, and its `NOCASE` collation only folds ASCII. The tests prove the no-double-booking outcome under SQLite's single-writer serialisation and the case-insensitive uniqueness outcome under `NOCASE`. They do not prove SQL Server behaviour. The v1.1 rejection of SQLite (it cannot prove SQL Server locking) still holds for the pilot; Testcontainers for SQL Server returns as the integration database when hosting begins.
- **Portability rule.** The database fixture is a single class that supplies the provider and connection. Tests never reference SQLite-specific behaviour, so re-pointing the fixture at a SQL Server container is a fixture change, not a test rewrite.

### Playwright (.NET) for E2E

- **Alternatives considered:** Playwright (TypeScript), Cypress, Selenium.
- **Decision:** Playwright for .NET, one language and one build across app and tests. The E2E fixture starts the app on a random localhost port with a fresh SQLite file and the seed data, and stops it afterwards, so E2E needs no separately running environment and no gate.

### k6 and OWASP ZAP (optional in the POC)

- **Decision:** Kept as the chosen tools for the hosted phase. In the POC they are optional local runs against the local app, useful for a quick smoke check, and are not required to close any POC acceptance criterion.

### Clock: fake in automated tests only

- **Decision:** The application reads time only through an injectable clock (ADR-0019). Integration and API tests inject `FakeTimeProvider`. The running application, including the one started for E2E, uses the real clock. There is no clock offset endpoint, setting or UI in any environment (ADR-0024, 2026-09-29), and one API / Contract test asserts that no such endpoint exists (SEC-11).
- **E2E consequence.** E2E journeys that depend on time set up their data relative to the real current time through setup helpers (for example a booking starting in a few minutes for the check-in journey, or a booking already beyond its check-in window for the no-show outcome). Waiting for real minutes to pass is not used. The transition rules themselves (no-show at start+15 minutes, done at end, the 2-hour late rule, DST) are proven at integration level with the fake clock.

---

## Test Infrastructure (POC)

### Local Development

Prerequisites: the .NET SDK and, for E2E, the Playwright browsers (installed once). No Docker, no cloud account, no network resources.

```bash
# Integration and API / Contract tests
dotnet test tests/RoomBooking.IntegrationTests

# E2E tests (starts the app on localhost with a fresh SQLite file; browsers installed once)
pwsh tests/RoomBooking.E2E/bin/Debug/net*/playwright.ps1 install
dotnet test tests/RoomBooking.E2E

# Optional performance smoke (requires k6 and the app running locally)
k6 run tests/performance/booking-smoke.js

# Dependency check before a demonstration build
dotnet list package --vulnerable
```

Project and folder names are indicative; the implementation team fixes them at scaffold time. No gate exists locally, so no test ever needs a token.

### Running order (no pipeline)

There is no CI in the POC. The team runs, in this order before each demonstration: build, unit tests (developer-owned), integration and API / Contract tests, dependency check, E2E. The demonstration build is cut only from a green local run. When Azure DevOps is introduced (ADR-0023, post-POC), the same commands become pipeline stages (see Target state).

### Test Environments (POC)

| Level | Environment | Provisioning | Isolation |
|-------|------------|--------------|-----------|
| Integration | Developer machine | Temporary SQLite file created by the fixture | Fresh database per test class |
| API / Contract | In-memory app host (`WebApplicationFactory`) with a temporary SQLite file | Fixture | Seeded per test; no shared state |
| E2E / UI | The app started by the test fixture on localhost | Fixture starts and stops the app | Fresh SQLite file and seed data per run |
| Performance (optional) | The local app | Started manually | Indicative only |
| Security (optional scan) | The local app | Started manually | Read-only baseline scan |

---

## Test Data Strategy

All test data is synthetic. Real names and emails are never used in any test.

| Level | Strategy |
|-------|---------|
| Integration | Builders create rooms, people (employee and admin) and bookings per test; fixed fake-clock start time; fresh database per class |
| API / Contract | Seed the reference set: exactly the specification's rooms (Focus 2, Team 6, Board 12 active; Old Room 4 inactive; no other rooms), one admin and several employees with obviously fictional names and non-routable emails; each test adds only what it needs; DST cases use fake-clock dates around 25 October 2026 and 28 March 2027 |
| E2E / UI | The app starts with the seed data (the same reference set plus sample bookings that make each E2E scenario runnable); scenarios that need a three-booking user or a pending no-show create them through setup helpers |
| Performance (optional) | Bulk seed script: 50 people, the four rooms, about 5,000 bookings across past and future dates; import file generators for 300 and 500 rows |
| Security | Scan-only; seeded synthetic data |
| Bootstrap admin | Integration tests: with no admin and valid configuration one admin is created; with an existing admin nothing changes and the step is repeatable; with no admin and missing or invalid configuration startup fails with a clear message |

### Import (M-06) fixtures and the sample CSV

M-06 is fully in the POC, so the import has a complete fixture set generated from one column definition (`room`, `person_email`, `date`, `start`, `end`, `attendees`, per ADR-0011) and relative to a go-live date supplied by the test.

| Fixture | Purpose |
|---------|---------|
| Sample CSV (demonstration and E2E) | A synthetic file using the seeded room names and the seeded fictional people's emails: valid rows spread across the five working days; a small number of deliberately faulty rows so the rejected-rows report has content; one exact duplicate; one formula-leading cell |
| Valid file | Every row imports; counts match |
| One file per rejection reason | Unknown room, inactive Old Room, unknown person, outside window, weekend date, off grid, outside office hours, too short, too long, capacity exceeded, overlap with another row, beyond the third booking for a person, in the past |
| Duplicate file | Re-importing the same file reports duplicates, not new bookings |
| Weekend go-live file | Window starts on the following Monday (ADR-0015 decision 5) |
| Wrong-header file | Whole-file refusal with a message |
| Formula-leading cells | Escaped in the report and the downloadable CSV (SEC-06) |
| Oversized file | Refused by the size and row cap |

The report data is purged once the import is settled (accepted by the project lead, 2026-09-29), so a test asserts the purge as well as the report content. If Office Operations later supplies a real export, it is anonymised before it becomes a fixture.

---

## Coverage Targets

Coverage is measured against acceptance criteria, not against code structure or endpoint matrices (`test-standards` §2, §6). In the POC this covers all six modules.

| Level | Coverage Target | Measurement | Rationale |
|-------|----------------|-------------|-----------|
| Integration + API | Every numbered acceptance criterion has exactly one passing test case (plus one per failure or access condition the requirements name) | Acceptance Criteria Traceability table in each story's `verification-report.md` | Product-recognisable behaviour is what ships |
| E2E / UI | One happy-path scenario per feature; a failure path only where requirements name one; E2E-001 to E2E-014 map to features | Scenario traceability in `e2e-verification-report.md` | Business-critical flows verified end to end |
| Performance | The NFRs marked as applying in the POC (table below), measured locally | Threshold assertions (integration timing tests; optional k6) | Indicative at pilot scale |
| Security | Access boundaries named in acceptance criteria; SEC-02, 03, 04, 06, 08, 11, 13 mitigations have tests or checks | API boundary tests, dependency report, localhost-binding check | POC risk posture |

### NFR to test mapping

| NFR | POC applicability | Test level | Test | Threshold |
|-----|-------------------|-----------|------|-----------|
| NFR-003 | Applies (indicative) | Performance (optional) / E2E timing | Availability search and booking page load locally | p95 under 2 s |
| NFR-004 | Applies | Integration | Import of 300 and 500 rows | Under 60 s end to end |
| NFR-005 | Applies (SQLite serialisation) | Integration | N simultaneous identical booking requests for one slot; mixed manual booking during import | Exactly one succeeds; zero overlapping active bookings |
| NFR-006 | Applies | Integration (fake clock) | Sweeper run at start+15 min and at end | Status changed within one sweeper interval; availability correct even before the sweeper runs |
| NFR-009 | Localhost-only check in the POC; gate check post-POC | Security | Startup binds to localhost; no LAN or tunnel binding | Not reachable from another machine |
| NFR-010 | Applies | API / Contract | Employee attempts admin pages, other person's bookings, non-owner check-in | Refused every time |
| NFR-012 | Applies | API / Contract | Availability response contains no owner data; my bookings show only own | As stated |
| NFR-015 | Applies | Integration (fake clock) | Rules around 25 October 2026 and 28 March 2027; weekday and office-hours tests in winter and summer offsets | Correct in both offsets |
| NFR-016, NFR-018 | Applies | E2E | Desktop and phone-size viewports; automated accessibility checks (axe) on main screens | No serious violations |
| NFR-017 | Applies | API / Contract | Every refusal names the rule | As stated |
| NFR-020 | Applies (gate fields empty) | Integration | Audit content for cancel, admin cancel, deactivation, import; actor, origin recorded | Recorded |
| NFR-023 | Applies (`NOCASE`) | Integration | Duplicate email differing in case is refused | Refused. Accented characters are not asserted in the POC because `NOCASE` folds ASCII only; asserted on Azure SQL later |
| NFR-007, NFR-008, NFR-011, NFR-014, NFR-022, NFR-024 | Deferred to post-POC | — | Uptime, backup restore rehearsal, HTTPS, privacy note, Azure hosting, cost | Deferred |

External systems: the spreadsheet CSV import has its own fixtures above and an E2E scenario. Microsoft Entra ID is not used in the POC, so no gate or sign-in test exists.

---

## Reporting & Observability (POC)

- **Results:** `dotnet test` console and TRX output on the developer machine; the acceptance-criteria traceability table remains the coverage view.
- **Failure notification:** None automated; the team runs the full local suite before each demonstration.
- **Flakiness:** Time-dependent tests use the fake clock and never real waits; a test that needs a retry is treated as a defect.
- **Artifacts:** Playwright traces and screenshots on failure are written to a local folder and kept until the next run.

---

## Risks & Assumptions

| # | Risk / Assumption | Impact | Mitigation |
|---|-------------------|--------|------------|
| 1 | SQLite results are taken as evidence for Azure SQL behaviour | Defects in locking, collation, `rowversion` or date handling appear only when hosted | Portable fixture; a post-POC task to re-run the concurrency, collation and audit tests on SQL Server via Testcontainers; guard test against provider-specific types (ADR-0025) |
| 2 | No CI: a demonstration build could be cut from an unverified state | A defect is shown to the client | The running order above; cut demo builds only from a green local run |
| 3 | The POC is a demonstration by the delivery team, with no remote client access and no live trial (decided 2026-09-29, ADR-0024) | If this changes, hosting and the gate would be needed | Revisit ADR-0024 |
| 4 | Import format (ADR-0011) unconfirmed by the client | Import fixtures may change | Fixtures generated from a single column definition |
| 5 | Real spreadsheet not seen | Real-world rejection patterns unknown | The sample CSV covers every named rejection reason; add an anonymised real sample when Office Operations provides one |
| 6 | E2E cannot move time, so time-driven journeys rely on data set up relative to the real clock | Some E2E setups are more elaborate | Setup helpers place bookings relative to now; transition rules are proven with the fake clock at integration level; no clock control is built (ADR-0024) |
| 7 | Unit tests are developer-owned | Gaps in rule-level logic unknown to QA | Integration and API tests cover every acceptance criterion; developers own unit coverage of the rules engine |
| 8 | Intake answers on environment and compliance were derived from the architecture document | Assumed answers may be wrong | Confirm with the delivery lead before the first test scaffold |
| 9 | Timing figures measured locally are quoted as pilot performance | Wrong expectations about hosted speed | Label local results as indicative; hosted measurement is post-POC |

---

## Key Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Integration test framework (POC) | xUnit with SQLite (temporary file per class) and fake clock | No installation, real transactions, deterministic time |
| Integration test framework (post-POC) | xUnit with Testcontainers for SQL Server | Real SQL Server locking and collation |
| API / Contract test framework | xUnit with `WebApplicationFactory` | The app's HTTP surface is the contract |
| E2E / UI framework | Playwright (.NET) against localhost | Language alignment, mobile viewports, tracing |
| Performance tool | k6 (optional locally; required post-POC) | Simple thresholds mapping to NFRs |
| Security scanning | Dependency vulnerability check and access-boundary tests now; ZAP baseline optional locally, required post-POC | Free, matches the applicable findings |
| CI/CD | None in the POC; Azure DevOps Pipelines post-POC (ADR-0023, deferred) | ADR-0024 |
| Time in tests | Injected fake clock in integration and API tests only; real clock in the running app and in E2E; no clock control exists | Deterministic time-based rules without a demo-only feature (ADR-0024; SEC-11) |

---

# Target State (post-POC)

*Carried from v1.1. Not used while the project is a local POC (ADR-0024). Applies when hosting begins.*

**Pipeline stages (Azure DevOps Pipelines, ADR-0023)**

| Pipeline Stage | Test Levels Run | Gate (pass required?) | Timeout |
|---------------|----------------|----------------------|---------|
| PR validation | Integration (Testcontainers, SQL Server), API / Contract, dependency vulnerability scan | Yes | 10 min |
| Merge to main (deploy to test environment) | Integration, API / Contract, gate reachability probe, E2E / UI smoke set (one journey per module) | Yes | 25 min |
| Nightly | Full E2E / UI, Performance (k6), Security (ZAP baseline) | No (alert) | 60 min |
| Pre-go-live | Full E2E, Performance, Security, gate reachability on production, backup restore rehearsal | Yes (release gate) | Manual |

**Deployed test environment.** Azure test environment (App Service B1, Basic SQL) in the EU region, gated like production by Easy Auth (ADR-0021), reset to seed data at the start of each run, with no clock endpoint (none exists; ADR-0024). Performance and security runs use the same environment outside working hours.

**Passing the access gate.** The pipeline obtains an access token for the gate's app registration using a service principal (client credentials, workload identity federation where available) and sends it as a bearer token from Playwright's API context, ZAP and k6; the gate app registration must accept that audience. Leaving the test environment ungated is rejected (SEC-11, SEC-12); automated interactive sign-in is rejected as brittle. Fallback if client IT refuses bearer tokens: a test user excluded from MFA by Conditional Access, with Playwright storing its sign-in state. To confirm with client IT.

**Additional NFR tests when hosted:** NFR-007 uptime probe; NFR-008 point-in-time restore rehearsal; NFR-009 unauthenticated requests redirected to Microsoft sign-in and the deployment console not reachable anonymously; NFR-011 HTTPS-only and HTTP redirect; NFR-023 accented-character collation on Azure SQL; re-run of the NFR-005 concurrency tests against the per-room UPDLOCK implementation.

**When the in-app sign-in phase starts:** add an integration test using a stubbed OpenID Connect provider and a manual test against a developer tenant.

## Open Questions

- [x] Resolved 2026-09-29 (ADR-0024): the delivery team demonstrates; no remote client access, no live trial.
- [x] Resolved 2026-09-29 (ADR-0024): the delivery team prepares the sample import CSV and synthetic seed data before development ends.
- [ ] Is a formal accessibility target beyond the WCAG 2.1 AA aim needed? Owner: Office Operations.
- [ ] Will Office Operations provide an anonymised spreadsheet export for later import fixtures? Owner: project lead.
- [ ] Post-POC: will client IT accept a service principal (bearer token) for automated access to the gated test environment? Owner: project lead with client IT.
- [ ] Post-POC: where does the Azure DevOps organisation live and who arranges the service connection and hosted parallelism? Owner: project lead and delivery lead.
- [ ] Post-POC: who runs the pre-go-live infrastructure and access-gate check? Owner: project lead.
