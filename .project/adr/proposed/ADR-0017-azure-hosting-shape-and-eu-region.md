# ADR-0017: Azure Hosting Shape: App Service, Azure SQL, EU Region

**Date:** 2026-09-29
**Status:** Proposed
**Deciders:** Solution Architect (recommendation); client IT and project lead to confirm

## Source

**Type:** Internal Discussion
**Reference:** Phase 4 architecture design, 2026-09-29; ADR-0002 (Azure), ADR-0015 (EU region), nfrs.md NFR-007, NFR-008, NFR-011, NFR-019, NFR-024
**Participants / Authors:** Solution Architect

---

## Context

Azure is confirmed as the host and the deployment must be in an EU region *(Source: ADR-0002, ADR-0015)*. Targets are best-effort availability during office hours, RPO 24 hours and RTO one business day, HTTPS only, basic logging, and the smallest suitable tiers *(Source: nfrs.md, NFR-007, NFR-008, NFR-011, NFR-019, NFR-024)*. The specific Azure services, subscription and network arrangements were left to architecture *(Source: integration-points.md, Azure hosting)*.

## Decision Drivers

- Lowest cost at pilot scale, with minimal operations effort.
- Built-in backup and TLS.
- Ability to restrict network access to the trusted group (ADR-0021).
- EU data residency.

## Options Considered

### Option A: Azure App Service (Linux, Basic B1, single instance) plus Azure SQL Database (Basic or smallest DTU tier), Application Insights
- **Pros:** Fully managed; approximately tens of dollars per month (indicative, to be verified with the Azure pricing calculator); built-in point-in-time restore; managed identity to the database (no database password); access restrictions and Always On available; simple to deploy from a pipeline.
- **Cons:** Single instance, so no automatic failover; a platform restart causes a short outage (acceptable under NFR-007).

### Option B: Azure Container Apps plus Azure SQL
- **Pros:** Container portability; scale to zero.
- **Cons:** More concepts for a single web app; scale-to-zero cold starts hurt the once-a-minute background processing; no pilot benefit.

### Option C: Azure Static Web Apps plus Azure Functions plus serverless SQL
- **Pros:** Cheapest at idle.
- **Cons:** Splits the app; serverless SQL auto-pause adds cold-start delay; timer-based work and session handling become more complicated.

### Option D: Virtual machine
- **Pros:** Full control.
- **Cons:** Patching and operating burden with no benefit at this scale.

## Decision

**We propose Option A, deployed in a single EU region.**

Recommended region: an EU region chosen with client IT (candidates: West Europe or Germany West Central; final choice follows any client subscription policy). Infrastructure is defined as code (Bicep) so environments can be recreated. Two environments: a test environment (smallest tiers, may be stopped outside working hours) and production. The exact region and SKUs are to be confirmed against the client's subscription and current Azure pricing.

## Consequences

### Positive
- Meets every hosting-related NFR with the smallest managed footprint.
- Backups and TLS are platform features, not custom work.

### Negative / Trade-offs
- No high availability across zones or regions; a regional outage takes the pilot down (accepted, NFR-007 is best effort).
- Basic-tier SQL has limited performance headroom (adequate for NFR-001 to NFR-004; to be verified in the performance test).

### Risks / Follow-up Actions
- [ ] Client IT to confirm subscription, region and any naming, tagging or policy requirements.
- [ ] Verify indicative costs in the Azure pricing calculator for the chosen region.
- [x] Confirm the CI/CD platform. Confirmed as Azure DevOps Pipelines, recorded in ADR-0023 (2026-09-29).

## Change Log
- 2026-09-29: CI/CD platform follow-up closed by ADR-0023 (Accepted). This ADR stays Proposed: region, subscription, SKUs and naming/policy requirements still await client IT. Access restriction for the trusted group is now provided by the Easy Auth tenant gate (ADR-0021, Accepted), not by network restrictions; the driver "restrict network access to the trusted group" is satisfied that way.
- 2026-09-29: Deferred to the post-POC phase by ADR-0024 (Accepted). The current phase is a local POC on SQLite (ADR-0025) with no Azure hosting. Status stays Proposed and the content is unchanged; it is the target-state hosting design for the pilot after the POC.
