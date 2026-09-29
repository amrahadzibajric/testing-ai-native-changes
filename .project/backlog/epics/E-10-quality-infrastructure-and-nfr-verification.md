# E-10: Quality Infrastructure and NFR Verification Tooling

**EPIC ID:** E-10
**Delivery Phase:** Phase 1 — POC (build, cross-cutting)
**Dependencies:** E-02 (scaffold, persistence, clock). The harness is used by the acceptance-criteria tests of every functional EPIC.
**ADR References:** ADR-0016, ADR-0019, ADR-0024, ADR-0025
**Architecture References:** test-strategy-meeting-room-booking-v1.2.md (Test Levels, Test Infrastructure, Test Data Strategy, Coverage Targets, NFR to test mapping); security-assessment v1.2

## Description

This EPIC builds the local test infrastructure that the test strategy v1.2 prescribes and the tooling needed to verify the non-functional requirements that apply in the POC. It does not contain story-level test cases: per-story tests are derived by QA from each story's acceptance criteria. Before this EPIC there is no way to run integration, API or E2E tests; afterwards a developer machine can run `dotnet test` with a temporary SQLite file per test class, a fake clock, an in-memory app host and Playwright against the running app in desktop and phone viewports, plus bulk data generators, an optional k6 smoke run and a dependency vulnerability check. No pipeline is involved (deferred to E-15).

## Scope Boundaries

- **Includes:** integration, API and E2E test harnesses and fixtures; reference seed and builders; setup helpers for time-relative E2E data; accessibility hook; bulk data and import file generators; optional k6 and ZAP guidance; dependency check script.
- **Excludes:** individual test cases and test code per story (QA derives from acceptance criteria); unit tests (developer-owned); CI/CD, gate tokens and hosted-environment performance or security testing (E-15); SQL Server Testcontainers (E-13).

## Key Technical Components

xUnit integration project with SQLite temp-file fixture and `FakeTimeProvider`; `WebApplicationFactory` API/contract fixture; Playwright (.NET) E2E fixture; data generators; scripts.

## Key User Roles Affected

Delivery team (developers, QA).

## Done Criteria

- `dotnet test` runs integration and API tests against a temporary SQLite file with a fake clock, with no Docker or cloud account.
- The E2E fixture starts the app on a random localhost port with a fresh SQLite file and seed data and runs in desktop and phone viewports with axe checks.
- Generators can produce 50 people, the four rooms and about 5,000 bookings, and 300- and 500-row import files.
- A single command checks dependencies for known vulnerabilities before a demonstration build.

## Features

| Feature | Description | Stories | Surfaces |
|---------|-------------|---------|----------|
| [F-10.01: Test Harness and Fixtures](../features/F-10.01-test-harness-and-fixtures.md) | Integration, API and E2E harnesses, builders, time-relative helpers, accessibility hook | 4 | Engineering |
| [F-10.02: Local NFR Verification Tooling](../features/F-10.02-local-nfr-verification-tooling.md) | Bulk data, optional k6 and ZAP, dependency check | 3 | Engineering |
