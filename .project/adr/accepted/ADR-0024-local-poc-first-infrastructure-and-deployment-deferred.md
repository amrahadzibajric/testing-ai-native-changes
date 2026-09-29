# ADR-0024: Local POC First; Infrastructure and Deployment Deferred

**Date:** 2026-09-29
**Status:** Accepted
**Deciders:** Amra Hadzibajric (project lead)

## Source

**Type:** Internal Discussion
**Reference:** Project lead scope change in working session, 2026-09-29: "We are complicating it for this MVP phase. I don't want to think about infrastructure. I want the app built locally so the client can see a POC. Deployment and infra should be later."
**Participants / Authors:** Amra Hadzibajric (project lead); recorded by Solution Architect

---

## Context

The Phase 4 architecture (arch-meeting-room-booking-v1.1.md) designed a hosted pilot: Azure App Service and Azure SQL in an EU region (ADR-0017), a tenant-only Easy Auth sign-in gate (ADR-0021), Azure DevOps pipelines with deployment to the client subscription (ADR-0023), Application Insights, Bicep, managed identity and a set of pipeline checks. Most of that work depends on client IT (subscription, region, Entra app registration, service connection), so it carries lead time and effort that has nothing to do with showing the client what the product does.

The project lead has decided that the current phase is a local proof of concept (POC). The purpose is to let the client see the working product. Infrastructure and deployment come later. Product scope is unchanged.

## Decision Drivers

- Show the client a working product as early as possible.
- Remove client IT dependencies (subscription, Entra app registration, service connection) from the critical path of the first demonstration.
- Avoid infrastructure effort during the POC (project lead).
- Keep every product rule and the module design intact so that nothing built for the POC is thrown away.
- Keep a clean path to the hosted pilot.

## Options Considered

### Option A: Local POC now; hosting, gate and pipelines deferred to a post-POC phase
- **Pros:** No infrastructure work or client IT dependency; fastest route to a demonstration; the application code is the same code that is hosted later.
- **Cons:** The client can only see the product when the delivery team demonstrates it; the hosted-environment risks (gate, Azure SQL behaviour, deployment) are discovered later.

### Option B: Continue with the hosted pilot design and demonstrate on Azure
- **Pros:** Client could click through the product remotely; hosting risks surface early.
- **Cons:** Requires everything the project lead wants to avoid now, and depends on client IT.

### Option C: Local build now, plus a temporary public tunnel for remote client access
- **Pros:** Remote access without Azure.
- **Cons:** Exposes an unauthenticated pick-a-user app to the internet; contradicts the security posture of ADR-0021. **Rejected.**

## Decision

**We will build and demonstrate the application locally as a proof of concept. Infrastructure and deployment are deferred to a post-POC phase.**

Definition of the POC phase:

1. The application runs on a developer machine (`dotnet run`, localhost only) and is demonstrated to the client by the delivery team, in person or by screen share. There is no remote client access and no live trial with real employees (project lead, 2026-09-29).
2. The database is SQLite (ADR-0025).
3. Seed data is synthetic: exactly the four rooms from the specification (Focus fits 2, Team fits 6, Board fits 12 and Old Room fits 4, inactive because it is not in use; no other rooms), sample people including at least one admin, and sample bookings for demonstration. No real client personal data is loaded. The delivery team owns and prepares the sample import CSV and the synthetic seed people and bookings before development ends.
4. Tests run on developer machines; no pipeline is required.
5. **Production-ready quality, no demo-only features.** The application is built to production-ready quality with no demo-only features or shortcuts. In particular there is no demonstration clock control. Time-based rules (check-in window, no-show release, done marking, late cancellation) run on the real clock. An injectable clock abstraction (ADR-0019) is allowed and expected for automated tests only; it must not be exposed in the running application (no endpoint, setting or UI).
6. **First-admin bootstrap.** The application has no login and admins are created by admins, so an admin must exist at startup. No earlier artifact (architecture v1.2, data model, module definitions, ADRs) defined a bootstrap; the module rule PI-07 only protects the last active admin. Decision: a configured bootstrap admin is seeded on first run if no active admin exists. Production-safe behaviour: the name and email come from configuration (a `BootstrapAdmin` settings section) with no hard-coded default account; the step does nothing when an active admin already exists, so it is idempotent and never overwrites or re-promotes anyone; if no admin exists and the configuration is missing or invalid, startup fails with a clear message; the creation is recorded as a system action in the audit trail; the bootstrap admin is an ordinary person record that can be deactivated or demoted once another active admin exists (PI-07). The demonstration sample people are separate from this step.
7. **Localhost binding only.** The POC is bound to localhost and not deployed. The run instructions must forbid LAN binding and public tunnels (SEC-13).

**Full functional scope (project lead clarification, 2026-09-29).** The POC contains all functional scope; infrastructure is the only exclusion. Every module M-01 to M-06 is fully in the POC, including M-06 Booking Import. The import is complete behaviour, not a demonstration stub: the ADR-0011 CSV format, per-row rule validation, the rejected-rows report with downloadable CSV, duplicate detection and the five-working-day window rules (ADR-0014, ADR-0015 decision 5). It is tested and demonstrated with a synthetic or anonymised CSV sample that uses the seeded fictional people's emails and the seeded room names. Nothing functional is deferred; only the infrastructure, deployment, CI/CD and identity-provider items in the table below are.

**Data-model decisions confirmed as accepted (project lead, 2026-09-29).** (1) "Cancelled by admin" is a display label derived from `cancellation_origin` on a Cancelled booking, not a separate status. (2) An admin cancelling their own booking is recorded as `OwnerCancelled` and can be late-flagged. (3) Import report rows are purged after the import is settled (this refines the "retain everything" reading of NFR-021 for that one table).

**Kept unchanged**

| Area | Decision |
|------|----------|
| Product scope and rules, modules M-01 to M-06 | All ADRs on product behaviour (ADR-0001, ADR-0003 to ADR-0010, ADR-0014, ADR-0015; ADR-0011 to ADR-0013 remain Proposed) |
| Stack | ASP.NET Core, Razor Pages, modular monolith, EF Core (ADR-0016). Only the database provider changes for the POC (ADR-0025) |
| Time | UTC storage, evaluation in office local time, Europe/Berlin assumed (ADR-0019) |
| Background processing | In-process hosted service for no-show and done (ADR-0018) |
| Concurrency | No double booking through one booking-write service and a per-room approach (ADR-0020), adapted to the local database (ADR-0025) |
| Identity | Pick-a-user (ADR-0005), email as the key (ADR-0006), all rules reading the person through the `ICurrentUser` seam (ADR-0022, design only) |
| Audit | Audit events recorded on every state change; the gate identity fields stay in the model and are simply empty in the POC |

**Deferred to a post-POC phase**

| Area | Decision affected | What is deferred |
|------|-------------------|------------------|
| Azure hosting | ADR-0017 (Proposed), ADR-0002 | App Service, Azure SQL, EU region choice, subscription, SKUs, Bicep, Application Insights, managed identity |
| Tenant access gate | ADR-0021 (Accepted) | Easy Auth configuration, Entra app registration, gate reachability checks, gate identity capture (fields stay in the model, empty locally) |
| CI/CD | ADR-0023 (Accepted) | Azure DevOps organisation, YAML pipelines, service connection, deployment stages, pipeline gate tokens |
| Entra ID sign-in path | ADR-0022 (Proposed) | No Entra work. The `ICurrentUser` seam stays in the code design |
| Operational items | ADR-0015 decision 2 (operational part) | Hosting in an EU region and the published privacy note. The synthetic-data rule replaces them during the POC |
| Non-functional testing against deployed environments | Test strategy | k6 and OWASP ZAP against a hosted test environment, backup restore rehearsal |

Deferred does not mean cancelled or rejected. The deferred ADRs keep their status and are expected to apply to the hosted pilot. They are annotated in their change logs.

**Security stance during the POC.** With the application reachable on localhost only, the findings tied to network exposure (SEC-01 and SEC-12 in the security assessment) do not apply. They re-activate the moment hosting begins, and no hosted deployment may go ahead before ADR-0021 (or a successor) is in place.

## Consequences

### Positive
- Removes client IT from the critical path of the first demonstration.
- No infrastructure spend or effort in this phase.
- The application code, module boundaries and data model are unchanged, so the POC is the foundation of the pilot, not a throwaway.

### Negative / Trade-offs
- The client cannot click through the POC themselves remotely; demonstrations need the delivery team present or sharing a screen.
- Hosted-environment risks (gate configuration, Conditional Access, Azure SQL locking behaviour, deployment permissions, cost) are not retired by the POC and stay open.
- The POC runs on a different database engine from the target (ADR-0025); behaviour that depends on SQL Server must be re-verified before hosting.
- Deferred ADRs, the architecture document's target-state sections and the deferred NFRs risk drifting while parked; they are labelled "Target state (post-POC)" so they are not mistaken for current work.

### Risks / Follow-up Actions
- [x] Confirmed 2026-09-29: the client does not use the POC remotely; the delivery team demonstrates it.
- [x] Confirmed 2026-09-29: the POC is a demonstration to the client, not a live trial with real employees.
- [ ] Post-POC phase (hosting and infrastructure): to be decided after the POC demo. It has no date or owner yet. When it is decided, revisit ADR-0017, ADR-0021, ADR-0022 and ADR-0023. Owner: to be assigned by the project lead.
- [ ] Write the run instructions so that they forbid LAN binding and public tunnels (SEC-13). Owner: delivery team.
- [ ] Prepare the sample import CSV and the synthetic seed people and bookings before development ends. Owner: delivery team.
- [ ] Agree the bootstrap admin settings (name and email) for the demonstration. Owner: delivery team with project lead.
- [ ] The target-state note in ADR-0021 (follow-up about a test-only clock control on the test environment) no longer applies, since no clock control exists. Correct it when ADR-0021 is next revised. Owner: solution architect.
- [ ] Before any hosted deployment, re-run the security assessment against the hosted design and rebuild the Azure SQL concurrency and collation tests (ADR-0025).
- [ ] The backlog (Phase 7) must separate POC work from post-POC infrastructure work. Not started here.

## Change Log
- 2026-09-29: Created and Accepted on the project lead's instruction.
- 2026-09-29: Same-day clarification from the project lead recorded: the POC contains all functional scope (M-01 to M-06 including the full M-06 import), rooms seeded exactly as in the specification, and the three data-model decisions are accepted. Infrastructure remains the only exclusion.
- 2026-09-29: Amended after a further project lead working session (Amra Hadzibajric). Status stays Accepted. (1) The delivery team demonstrates the POC; no remote client access and no live trial with real employees (Definition items 1; two follow-ups closed). (2) There is no demonstration clock control; time-based rules run on the real clock; an injectable clock is for automated tests only and is not exposed in the running application (new Definition item 5; architecture, security assessment and test strategy v1.2 amended with revision notes). (3) The delivery team prepares the sample import CSV and synthetic seed people and bookings before development ends; rooms remain exactly as in the specification (Definition item 3). The absence of a first-admin bootstrap was found and the approach recorded: a configured bootstrap admin seeded on first run if no admin exists, with production-safe behaviour (new Definition item 6). (4) Localhost only and not deployed; run instructions must forbid LAN binding and public tunnels, SEC-13 (new Definition item 7). (5) The post-POC phase (hosting and infrastructure) will be decided after the POC demo, with no date or owner yet (follow-up updated). The Context and Options sections are unchanged.
- 2026-09-29: The project lead confirmed the first-admin bootstrap approach (Definition item 6) in a working session. It is no longer a proposal. Status stays Accepted.
