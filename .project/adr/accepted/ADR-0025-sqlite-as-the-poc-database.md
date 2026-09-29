# ADR-0025: SQLite as the Database for the Local POC

**Date:** 2026-09-29
**Status:** Accepted
**Data Impact:** Yes
**Deciders:** Amra Hadzibajric (project lead)

## Source

**Type:** Internal Discussion
**Reference:** Project lead scope change in working session, 2026-09-29 (local POC, no infrastructure); follows ADR-0024. Related: ADR-0016 (EF Core, Azure SQL), ADR-0020 (concurrency), ADR-0006 and NFR-023 (case-insensitive email)
**Participants / Authors:** Amra Hadzibajric (project lead); recorded by Solution Architect

---

## Context

ADR-0024 makes the current phase a local POC with no infrastructure. The application still needs a relational database. ADR-0016 selected Azure SQL Database, which is not available locally without installing SQL Server or running a container, and the project lead does not want to think about infrastructure in this phase. EF Core is provider-based, so the database engine can differ between the POC and the hosted pilot.

## Decision Drivers

- Zero installation and no container dependency for a developer or for a demonstration machine.
- Cross-platform, file-based, trivial to reset and re-seed before each demonstration.
- Keep the model and code provider-neutral so moving to Azure SQL is a provider and migration change, not a redesign.
- Preserve the no-double-booking guarantee (NFR-005) and case-insensitive email uniqueness (NFR-023) in the POC.

## Options Considered

### Option A: SQLite through EF Core (file database)
- **Pros:** Nothing to install; one file; fast reset; works on any developer machine; supported by EF Core with migrations.
- **Cons:** Different engine from Azure SQL: no row locks, no `rowversion`, no `datetimeoffset` comparisons, case-sensitive text comparison by default, limited `ALTER TABLE` support in migrations.

### Option B: SQL Server LocalDB or Developer edition
- **Pros:** Same engine as Azure SQL.
- **Cons:** Windows-only (LocalDB) or a heavier install; contradicts "no infrastructure".

### Option C: SQL Server in a container
- **Pros:** Same engine as Azure SQL; already assumed by the v1.1 test strategy.
- **Cons:** Requires Docker on every machine used for the POC.

### Option D: EF Core in-memory provider
- **Pros:** Fastest.
- **Cons:** Not relational; ignores constraints and transactions; unsuitable for demonstrating persistence or proving NFR-005.

## Decision

**We will use SQLite through EF Core as the database for the POC (Option A), with a model that stays provider-neutral so the hosted pilot can switch to Azure SQL by changing the provider and regenerating migrations.**

Rules for keeping the switch cheap:

1. **Provider-neutral model.** No raw SQL or provider-specific types in domain code. Provider differences live in one persistence configuration point.
2. **Instants as UTC `DateTime`.** All instants are stored as UTC `DateTime` values (EF Core stores them as ISO-8601 text in SQLite) with a value converter that enforces `Kind = Utc`. `DateTimeOffset` is not used, because SQLite cannot reliably compare or order it. This is already the data model's convention (`*_utc`, ADR-0019).
3. **Case-insensitive keys.** Person email and room name unique indexes use the SQLite `NOCASE` collation in the POC. Azure SQL uses a case-insensitive collation. `NOCASE` folds only ASCII letters, which is acceptable for the synthetic POC data and must be re-tested against accented names and emails in the pilot.
4. **Concurrency token.** The `row_version` column cannot be database-generated in SQLite. In the POC it is an application-managed token (for example a GUID or counter replaced on each update and marked as a concurrency token). On Azure SQL it becomes a `rowversion`. The logical attribute is unchanged.
5. **No double booking in the POC.** SQLite allows one writer at a time. The single booking-write service (ADR-0020) runs the overlap check and the insert inside one write transaction that takes the write lock at the start (an immediate transaction, not a deferred one, so that two concurrent requests cannot both read before either writes). The database is used in write-ahead logging mode with a busy timeout so a waiting writer waits instead of failing. A parallel-request integration test proves that exactly one of N identical simultaneous requests succeeds. The exact SQLite API for an immediate transaction is to be verified against the current Microsoft.Data.Sqlite documentation at build time.
6. **ADR-0020 on Azure SQL.** The per-room row lock (UPDLOCK on the room row) applies when the application moves to Azure SQL. Serialising all writes in SQLite is coarser than the per-room lock but is more than enough at this scale and gives the same guarantee.
7. **Migrations.** Migrations are generated per provider. The POC keeps its own SQLite migrations; at the switch to Azure SQL a new baseline is generated rather than carrying SQLite migrations across.
8. **Seed data.** A startup seed creates exactly the four rooms from the specification (Focus 2, Team 6, Board 12 active; Old Room 4 inactive; no other rooms), sample people including at least one admin and several employees, and sample bookings covering the demonstration scenarios. Seed data is synthetic. A sample CSV in the ADR-0011 format, using the seeded rooms and people, supports testing and demonstrating the full import (ADR-0024). Resetting the demonstration means deleting the database file and restarting.
9. **ASP.NET Core data protection keys.** For the POC the key ring is stored on the local file system (or the same SQLite database), not in Azure SQL. It is not a domain entity.

## Consequences

### Positive
- Anyone can run the POC with only the .NET SDK.
- Resetting to a clean demonstration state is instant.
- The domain model, rules engine and pages are provider-independent, so the pilot's move to Azure SQL is contained.

### Negative / Trade-offs
- SQLite behaviour is not evidence for SQL Server behaviour. Concurrency, collation and `rowversion` semantics must be re-verified on Azure SQL before hosting; the v1.1 test strategy's rejection of SQLite (it cannot prove row locks or SQL Server collation) still holds for the pilot and is restored then.
- Serialising all writes means the POC cannot say anything about per-room parallelism or throughput on Azure SQL.
- Some later schema changes (dropping or altering columns) are awkward in SQLite migrations; acceptable because the POC schema is small and disposable.
- ADR-0016 remains Accepted with Azure SQL as the target database, so the POC diverges from it knowingly.

### Risks / Follow-up Actions
- [ ] Verify the immediate-transaction API and the WAL and busy-timeout settings against current Microsoft.Data.Sqlite and EF Core documentation at build start. Owner: delivery team.
- [ ] Add a guard test that fails if a `DateTimeOffset` or provider-specific type appears in the model. Owner: delivery team.
- [ ] At the post-POC switch: generate an Azure SQL baseline migration, apply `rowversion` and the case-insensitive collation, implement the UPDLOCK per-room lock, and re-run the concurrency and collation tests on Azure SQL via Testcontainers. Owner: delivery team.

## Change Log
- 2026-09-29: Created and Accepted on the project lead's instruction (ADR-0024).
