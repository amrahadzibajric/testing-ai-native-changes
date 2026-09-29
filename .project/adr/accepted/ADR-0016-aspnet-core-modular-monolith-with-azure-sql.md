# ADR-0016: ASP.NET Core Modular Monolith with Azure SQL Database

**Date:** 2026-09-29
**Status:** Accepted
**Data Impact:** Yes
**Deciders:** Amra Hadzibajric (project lead), on the Solution Architect's recommendation

## Source

**Type:** Internal Discussion
**Reference:** Phase 4 architecture design, 2026-09-29; confirmed by project lead in working session, 2026-09-29 (the delivery team works in .NET). Constraints from ADR-0001 (pilot, up to 50 users), ADR-0002 (Azure), ADR-0006 (Microsoft-ready identity), nfrs.md NFR-001, NFR-024
**Participants / Authors:** Solution Architect; Amra Hadzibajric (project lead)

---

## Context

The pilot serves up to 50 people with a few thousand bookings in total *(Source: nfrs.md, NFR-001, NFR-002)*. It is hosted on Azure *(Source: ADR-0002)* and is expected to adopt Microsoft sign-in later *(Source: ADR-0006)*. Cost should be minimal *(Source: nfrs.md, NFR-024)*. When first proposed, no source stated the delivery team's language or framework; on 2026-09-29 the project lead confirmed that the delivery team works in .NET, which removes the main open assumption.

## Decision Drivers

- Simplicity: one deployable unit, one language, minimal moving parts.
- Low cost and low operational effort at pilot scale.
- Strong, first-class support for Microsoft Entra ID sign-in later.
- Strong relational integrity for the no-overlap rule (NFR-005).
- Phone-friendly web UI without a separate front-end build chain.
- Delivery team skills (.NET, confirmed 2026-09-29).

## Options Considered

### Option A: ASP.NET Core (Razor Pages, server-rendered) modular monolith, EF Core, Azure SQL Database
- **Pros:** One project and one deployment; server-rendered pages suit a small workflow app and phones; built-in anti-forgery, authorisation policies and time abstractions (TimeProvider); Microsoft.Identity.Web makes the later Entra ID step small; Azure SQL is the cheapest managed relational option and supports managed identity and point-in-time restore; matches the team's .NET skills.
- **Cons:** Less interactive than an SPA unless small amounts of JavaScript are added.

### Option B: ASP.NET Core Web API plus React single-page application, Azure SQL
- **Pros:** Rich interactivity; API reusable by a future mobile client.
- **Cons:** Two build chains and two deployables for a 50-user tool; more surface (CORS, token handling) for no pilot benefit.

### Option C: Node.js (for example Next.js) with PostgreSQL
- **Pros:** Single language front to back if the team is JavaScript-first; PostgreSQL exclusion constraints can enforce no-overlap declaratively.
- **Cons:** Not the team's stack (the team works in .NET); adds a non-Microsoft stack to a Microsoft-aligned client; PostgreSQL flexible server costs more than the smallest Azure SQL tier.

## Decision

**We will build an ASP.NET Core (current LTS) modular monolith with server-rendered Razor Pages, Entity Framework Core and Azure SQL Database (Option A).**

The application is one deployable unit with internal modules mirroring M-01 to M-06 (room booking, booking lifecycle, rooms, people, oversight, import). It fits the pilot scale, the Azure constraint, the Microsoft sign-in path and the delivery team's .NET skills. Azure SQL tier and sizing remain governed by ADR-0017 (Proposed) and the performance test; nothing in this acceptance fixes a tier.

## Consequences

### Positive
- Lowest complexity and cost; one pipeline, one runtime, one database.
- Later Entra ID sign-in is an additive change (ADR-0022).
- No re-skilling; risk 6 in the architecture document (team not .NET-skilled) is closed.

### Negative / Trade-offs
- Ties the pilot to the .NET ecosystem and SQL Server dialect.
- Growth beyond the pilot may justify splitting modules or adding an API; the module boundaries keep that option open.
- The project lead confirmed the .NET basis. Razor Pages as the UI approach is accepted as part of the recommendation; the team may choose a different .NET UI style (for example MVC or Blazor) with an amendment to this ADR if it prefers.

### Risks / Follow-up Actions
- [x] Confirm the delivery team's language and framework preference before build. Confirmed: .NET (2026-09-29).
- [ ] Confirm the .NET version is the current LTS at build start.

## Change Log
- 2026-09-29: Moved from Proposed to Accepted on the basis that the delivery team works in .NET (project lead). Azure SQL and tier detail unchanged; tier remains subject to ADR-0017 and the performance test. Added Data Impact tag (SQL Server dialect, EF Core, case-insensitive collation for email uniqueness).
- 2026-09-29: For the local POC the database is SQLite through EF Core (ADR-0025, with ADR-0024). Azure SQL Database remains the target database for the hosted pilot; the ASP.NET Core, Razor Pages, EF Core and modular monolith decisions are unchanged. Status stays Accepted. The model is kept provider-neutral so the switch to Azure SQL is a provider and migration change.
