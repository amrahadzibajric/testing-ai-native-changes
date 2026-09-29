# E-13: Azure SQL Migration and Hosted Data Verification

**EPIC ID:** E-13
**Status:** Deferred — Post-POC (ADR-0024, ADR-0025)
**Delivery Phase:** Post-POC (hosted pilot)
**Dependencies:** E-12 (Azure SQL exists); POC persistence (E-02) and write service (E-06)
**ADR References:** ADR-0016, ADR-0020 (Proposed), ADR-0025, ADR-0006 (NFR-023 collation), ADR-0019
**Architecture References:** arch v1.2 (Concurrency, Target Deployment Topology — Switch from SQLite); data-model README (POC storage note); test-strategy v1.2 (Framework Evaluation, Target state)

## Description

This deferred EPIC moves the application from SQLite to Azure SQL and re-proves everything that SQLite could not prove: the per-room UPDLOCK concurrency guarantee, SQL Server `rowversion`, case-insensitive collation including accented characters, `datetime2` handling around daylight-saving transitions, and the audit, report-purge and sweeper behaviour. Because the model was kept provider-neutral (ADR-0025), the switch is a provider and migration change; this EPIC also holds the verification work that ADR-0025 and the test strategy explicitly deferred.

## Scope Boundaries

- **Includes:** Azure SQL baseline migration and provider configuration; per-room lock in the write service; Testcontainers-based SQL Server test runs and re-verification.
- **Excludes:** hosting resources (E-12); pipeline integration of these tests (E-15).

## Key Technical Components

Persistence configuration point, booking-write service, SQL Server Testcontainers fixture.

## Key User Roles Affected

Delivery team.

## Done Criteria

- The application runs on Azure SQL with a baseline migration, `rowversion` and a case-insensitive collation.
- N simultaneous identical booking requests still produce exactly one booking with the per-room lock.
- Collation, DST, audit, purge and sweeper tests pass on SQL Server.

## Features

| Feature | Description | Stories | Surfaces |
|---------|-------------|---------|----------|
| [F-13.01: Azure SQL Provider Switch](../features/_deferred/F-13.01-azure-sql-provider-switch.md) | Baseline migration, provider configuration, hosted seeding | 3 | Engineering |
| [F-13.02: Per-room Lock Concurrency](../features/_deferred/F-13.02-per-room-lock-concurrency.md) | UPDLOCK per-room lock and re-run of concurrency proof | 3 | Engineering |
| [F-13.03: SQL Server Behaviour Re-verification](../features/_deferred/F-13.03-sql-server-behaviour-re-verification.md) | Collation, time and lifecycle behaviour on SQL Server | 3 | Engineering |
