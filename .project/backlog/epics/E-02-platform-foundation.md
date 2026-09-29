# E-02: Platform Foundation

**EPIC ID:** E-02
**Delivery Phase:** Phase 1 — POC (build, infrastructure-first)
**Dependencies:** E-01 (Discovery exit criteria met)
**ADR References:** ADR-0006, ADR-0015, ADR-0016, ADR-0018, ADR-0019, ADR-0022, ADR-0024, ADR-0025
**Architecture References:** arch v1.2 (Component view: Office clock, Audit recorder, EF Core data access, Startup seeder; Identity Design — first-admin bootstrap; Time Handling; Local Run View); data-model (all entities); security-assessment v1.2 (SEC-02, 03, 08, 09, 11, 13)

## Description

This EPIC builds the shared infrastructure that every functional EPIC stands on: the solution scaffold and module structure, the EF Core and SQLite persistence layer with the full data model and migrations, the office clock and time handling, the audit recorder, the first-admin bootstrap, the synthetic seed data and the localhost-only hosting baseline. Following the feature pipeline's infrastructure-first rule, this EPIC claims all migrations, all aggregate roots and the shared cross-cutting seams so that later stories consume them and never redefine them. After this EPIC the application starts with `dotnet run` on localhost, creates and migrates its SQLite file, guarantees an admin exists, seeds the specification's rooms and synthetic sample data, and exposes no business behaviour yet.

## Scope Boundaries

- **Includes:** solution and module scaffold; configuration and logging; all entities and the initial migration; SQLite provider configuration; office clock; audit recorder; bootstrap admin; seed data; loopback binding; anti-forgery, request limits and data protection key ring.
- **Excludes:** the `ICurrentUser` picker implementation and authorisation policies (E-04); booking rules and the booking-write service (E-06); the sweeper (E-07); UI design system and shell (E-03); test harness projects (E-10); anything Azure, gate, pipeline or Entra (E-12 to E-16).

## Key Technical Components

Solution and modules, Persistence (EF Core, SQLite), Office clock, Audit recorder, Startup seeder and bootstrap step, Host configuration (Kestrel, cookies, data protection, logging).

## Key User Roles Affected

Delivery team and the demonstration presenter (indirectly: the first admin and the seeded people). No end-user behaviour.

## Done Criteria

- `dotnet run` starts one process bound to loopback only and refuses non-loopback binding.
- A fresh start creates the SQLite file, applies the initial migration, seeds exactly Focus 2, Team 6, Board 12 (active) and Old Room 4 (inactive), guarantees an admin exists, and seeds synthetic people and bookings.
- All six entities exist with the data-model constraints; no `DateTimeOffset` or provider-specific type is in the model.
- The office clock is the only source of "now"; no clock control exists in the running application.
- Audit events can be written in the same transaction as any change.

## Features

| Feature | Description | Stories | Surfaces |
|---------|-------------|---------|----------|
| [F-02.01: Solution Scaffold and Module Structure](../features/F-02.01-solution-scaffold-and-module-structure.md) | Solution, module boundaries, configuration and logging | 4 | Engineering |
| [F-02.02: Persistence Foundation (EF Core and SQLite)](../features/F-02.02-persistence-foundation-ef-core-sqlite.md) | All entities, SQLite configuration, indexes, migrations, provider-neutrality guard | 5 | Engineering |
| [F-02.03: Office Clock and Time Handling](../features/F-02.03-office-clock-and-time-handling.md) | Injectable clock, office-zone conversions, DST safety, no clock control | 4 | Engineering, Web |
| [F-02.04: Audit Trail Recorder](../features/F-02.04-audit-trail-recorder.md) | Append-only audit events written in the change transaction | 3 | Engineering |
| [F-02.05: First-Admin Bootstrap](../features/F-02.05-first-admin-bootstrap.md) | Configured, idempotent, fail-fast first admin | 3 | Engineering |
| [F-02.06: Seed Data (Rooms, People, Demonstration Bookings)](../features/F-02.06-seed-data-rooms-people-and-demonstration-bookings.md) | Rooms exactly as specified, synthetic people and bookings | 4 | Engineering |
| [F-02.07: Localhost Hosting and Local Security Baseline](../features/F-02.07-localhost-hosting-and-local-security-baseline.md) | Loopback-only binding, anti-forgery, request limits, key ring | 4 | Engineering, Web |
