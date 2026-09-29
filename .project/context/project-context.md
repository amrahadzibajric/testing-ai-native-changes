# Project Context

> This file is populated incrementally as the project is elaborated. Entries are added by the product-strategist agent when new details are confirmed. Do not delete prior entries — mark outdated entries as `[Superseded]`.

<!--
Format for entries:
**[Topic]** — [Detail] *(Source: [document or session date])*
-->

**Project** — Meeting Room Booking: a small internal tool for Office Operations to replace a shared spreadsheet and Slack messages for booking meeting rooms. Problems: double bookings, rooms held "just in case", no visibility of free rooms. *(Source: room-booking-spec.md, Background)*

**Client** — Office Operations (single office, single time zone). *(Sources: room-booking-spec.md; client-qa-2026-09-29.md, Working assumptions)*

**Delivery stage** — Pilot version with no fixed go-live date. Greenfield build; the spreadsheet is retired after a one-week import. *(Source: client-qa-2026-09-29.md, Answer 1, Working assumptions; ADR-0001)*

**Users** — Employees and office admins; no more than 50 users, added manually by admins through a people-management screen. *(Source: client-qa-2026-09-29.md, Answer 2; ADR-0004)*

**Authentication** — No real login in the pilot ("pick a user"). Future sign-in is expected to be Microsoft company accounts; user identity is keyed on email. *(Sources: room-booking-spec.md, Not needed for now; client-qa-2026-09-29.md, Answer 9, Proposed import format; ADR-0005, ADR-0006)*

**Hosting** [Deferred to post-POC, ADR-0024] — Azure resources can be used. *(Source: client-qa-2026-09-29.md, Answer 8; ADR-0002)*

**Delivery format** — Web app for desktop and phone browsers (working assumption, not contradicted by client). Check-in is done from the user's own laptop or phone; no room-side devices. Native mobile app not needed. *(Sources: client-qa-2026-09-29.md, Answer 5, Working assumptions; room-booking-spec.md, Not needed for now; ADR-0003)*

**Rooms today** — Focus (2), Team (6), Board (12), Old Room (4, currently not in use). *(Source: room-booking-spec.md, Rooms we have today)*

**Core booking rules** — 30 min to 4 h, 15-minute blocks, open 08:00–20:00 Mon–Fri, no past bookings, no overlap (back-to-back allowed), attendees not above room capacity, max 3 upcoming bookings per employee (admin bookings for others count toward it; admins booking for themselves are unlimited). *(Source: room-booking-spec.md, Finding and booking a room; Stopping people from hogging rooms)*

**Cancellation and check-in rules** — Own cancellations by employees, any by admins; cancellations under 2 hours before start are flagged late; no cancellation once started. Check-in window: 10 minutes before to 15 minutes after start; otherwise no-show and room freed; finished meetings marked done. *(Source: room-booking-spec.md, Cancelling; Checking in)*

**Booking edits** — Employees can edit time and room; edits re-run all rules and are never flagged late. *(Source: client-qa-2026-09-29.md, Answers 3 and 10; ADR-0007)*

**No-show remainder** — The freed remainder can be rebooked; starts stay on the 15-minute grid. The 30-minute minimum is assumed to still apply (pending client confirmation). *(Source: client-qa-2026-09-29.md, Answers 4 and 11, Working assumptions; ADR-0008, ADR-0013)*

**Penalties** — Late cancellations and no-shows are for admin review only; no automatic penalties. *(Source: client-qa-2026-09-29.md, Answer 6; ADR-0009)*

**Data import** — One-off CSV import of upcoming-week bookings (columns room, person_email, date, start, end, attendees). Rule-violating rows are rejected into a report. Office Operations is assumed to own spreadsheet cleanup. Format and ownership pending client confirmation. *(Source: client-qa-2026-09-29.md, Answers 7 and 12, Proposed import format; ADR-0010, ADR-0011, ADR-0012)*

**Working assumptions** — Availability view shows busy rooms without revealing who booked; "upcoming" = confirmed bookings not yet started; no-show, release and done marking are automatic; the booker checks in (attendees are a head count); admins follow the same time rules as employees; taken-out-of-use rooms can be reactivated; lowering capacity leaves existing bookings unchanged; no public-holiday calendar. *(Source: client-qa-2026-09-29.md, Working assumptions)*

**Out of scope for now** — Recurring meetings, email or calendar notifications, proper login, multiple time zones, native mobile app. *(Source: room-booking-spec.md, Not needed for now)*

**Started bookings** — A booking that has started cannot be edited (same as the cancellation rule). *(Source: project lead confirmation, working session 2026-09-29; ADR-0014)*

**Edits and the 3-booking limit** — Editing a booking does not count as a new booking toward the 3-upcoming-booking limit. On import, rows are processed in date/time order per person; rows beyond a person's 3rd upcoming booking are rejected into the import report. *(Source: project lead confirmation, working session 2026-09-29; ADR-0014)*

**Person deactivation** — When an admin deactivates a person, that person's future bookings are cancelled and marked "cancelled by admin" (same pattern as room deactivation). *(Source: project lead confirmation, working session 2026-09-29; ADR-0014)*

**Import window** — The import covers the 5 working days starting on the go-live date. Refines the earlier "upcoming week" wording. *(Source: project lead confirmation, working session 2026-09-29; ADR-0014)*

**Pick-a-user list** — Shows active people only. *(Source: project lead confirmation, working session 2026-09-29; ADR-0014)*

**Office time zone** — Central European Time with daylight saving (CET/CEST), e.g. IANA Europe/Berlin or equivalent. All booking rules evaluate in office local time; DST transitions must be handled. Exact IANA zone name is an assumption to confirm with the client. *(Source: project lead confirmation, working session 2026-09-29; ADR-0015)*

**Data protection** — Host in an EU Azure region; provide a simple privacy note for the names and emails stored. No stricter compliance regime stated. *(Source: project lead confirmation, working session 2026-09-29; ADR-0015)*

**Admin cancellations and the late flag** — An admin cancelling another person's booking less than 2 hours before start is not flagged as a late cancellation; deactivation cancellations are never late-flagged. *(Source: project lead confirmation, working session 2026-09-29; ADR-0015)*

**Check-in ownership** — Only the booking owner can check in; admins cannot check in on someone's behalf. *(Source: project lead confirmation, working session 2026-09-29; ADR-0015)*

**Weekend go-live** — If go-live falls on a weekend, the 5-working-day import window starts the following Monday. *(Source: project lead confirmation, working session 2026-09-29; ADR-0015)*

**Phase 4 status** — [Superseded by the entries below] Architecture produced 2026-09-29 in .project/architecture/ (architecture document, two integration assessments, security assessment, test strategy). Technology choices are recommendations recorded as Proposed ADRs (ADR-0016 to ADR-0020, ADR-0022); pilot access gating is an Open ADR (ADR-0021). *(Source: architecture session 2026-09-29)*

**Off-network requirement** [Deferred to post-POC, ADR-0024] — The pilot must work outside the office network (phones off-network), with no VPN dependency. IP allow-listing was therefore rejected as the access-gating option. *(Source: project lead answers, working session 2026-09-29; ADR-0021)*

**Access gating** [Deferred to post-POC, ADR-0024] — A tenant-only Microsoft sign-in gate (Azure App Service built-in authentication / Easy Auth, restricted to the company Entra tenant) sits in front of the app as access gating only; pick-a-user remains the in-app identity. Client IT must permit the app registration; recommended hardening is to require user assignment for the pilot users. The gate identity is recorded in the audit trail for traceability. Residual risk: any gated person can still pick any user, including an admin. *(Source: project lead, working session 2026-09-29; ADR-0021 Accepted)*

**Technology stack** — The delivery team works in .NET. ASP.NET Core modular monolith with Razor Pages, EF Core and Azure SQL is Accepted (Azure SQL is the post-POC target; the POC uses SQLite, ADR-0025). Hosting region, subscription and SKUs remain Proposed pending client IT (ADR-0017). Other design choices (ADR-0018 to ADR-0020, ADR-0022) remain Proposed. *(Source: project lead, working session 2026-09-29; ADR-0016 Accepted)*

**CI/CD platform** [Deferred to post-POC, ADR-0024] — Azure DevOps Pipelines. Location of the Azure DevOps organisation and source repository is not yet stated. *(Source: project lead, working session 2026-09-29; ADR-0023 Accepted)*

**Phase 4 status (v1.1)** — Architecture updated 2026-09-29 to arch-meeting-room-booking-v1.1.md, security-assessment-meeting-room-booking-v1.1.md (no High or Critical residual findings; SEC-01 residual Medium) and test-strategy-meeting-room-booking-v1.1.md; v1.0 files marked superseded. *(Source: architecture update session 2026-09-29)*

**Phase 5 status** — Conceptual data model produced 2026-09-29 in .project/data-model/ (README, domain files, data-model.json). *(Source: data model session 2026-09-29)*

**Current phase: local POC** — The project lead changed the plan on 2026-09-29: "We are complicating it for this MVP phase. I don't want to think about infrastructure. I want the app built locally so the client can see a POC. Deployment and infra should be later." The current phase is a local proof of concept that runs on a developer machine (`dotnet run`, localhost only) and is demonstrated to the client by the delivery team, in person or by screen share. Assumption to confirm: the client does not need to click through it remotely (that would require hosting). *(Source: project lead, working session 2026-09-29; ADR-0024)*

**POC scope** — The POC contains all functional scope: modules M-01 to M-06, including the full M-06 import (ADR-0011 format, rule validation, rejected-rows report, five-working-day window), tested with a synthetic or anonymised CSV. Only infrastructure is excluded. Rooms are seeded exactly as in the specification (Focus 2, Team 6, Board 12 active; Old Room 4 inactive; no others). *(Source: project lead clarification, 2026-09-29; ADR-0024)*

**POC database** — SQLite through EF Core (file database, zero install). The model stays provider-neutral so moving to Azure SQL later is a provider and migration change. SQLite's single-writer serialisation satisfies the no-double-booking guarantee locally; the SQL Server per-room lock (ADR-0020) applies when hosted. *(Source: project lead, 2026-09-29; ADR-0025)*

**Deferred to post-POC** — Azure hosting (ADR-0017), Easy Auth tenant access gate (ADR-0021), Azure DevOps CI/CD (ADR-0023), Entra ID sign-in path (ADR-0022; the `ICurrentUser` seam stays in the code design), the EU-region and privacy-note operational items (ADR-0015 decision 2), Bicep, Application Insights, managed identity, pipeline gate tokens, and ZAP and k6 against deployed environments. These ADRs keep their status and are annotated. The earlier entries above on hosting, off-network requirement, access gating and CI/CD are tagged [Deferred to post-POC]; they remain valid as target-state decisions but do not describe the current phase. *(Source: project lead, 2026-09-29; ADR-0024)*

**POC security and data** — Localhost only, so SEC-01 and SEC-12 do not apply during the POC and re-activate when hosting begins. Only synthetic seed data is used (no real client personal data); an anonymised sample is used for the import. *(Source: project lead, 2026-09-29; ADR-0024; security-assessment-meeting-room-booking-v1.2.md)*

**Data-model decisions accepted** — "Cancelled by admin" is a label derived from `cancellation_origin`, not a status; an admin cancelling their own booking counts as `OwnerCancelled` and can be late-flagged; import report rows are purged after the import is settled. *(Source: project lead clarification, 2026-09-29; ADR-0024)*

**Phase 4 status (v1.2)** — Architecture updated 2026-09-29 to arch-meeting-room-booking-v1.2.md, security-assessment-meeting-room-booking-v1.2.md and test-strategy-meeting-room-booking-v1.2.md for the local POC; v1.1 files marked superseded. The Azure design is retained in each as "Target state (post-POC)". *(Source: architecture update session 2026-09-29)*

**POC demonstration mode** — The delivery team demonstrates the POC. There is no remote client access and no live trial with real employees. This confirms the two earlier "assumption to confirm" items in the Current phase entry above. *(Source: project lead, working session 2026-09-29; ADR-0024)*

**No demonstration clock control** — The application is built to production-ready quality with no demo-only features or shortcuts. Time-based rules (check-in window, no-show release, done marking, late cancellation) run on the real clock. An injectable clock abstraction is allowed for automated tests only and is not exposed in the running application. Removes the demonstration clock control proposed in arch, security and test strategy v1.2 (revision notes added). *(Source: project lead, working session 2026-09-29; ADR-0024)*

**Sample and seed data** — The delivery team owns and prepares the sample import CSV and the synthetic seed people and bookings before development ends. Rooms are seeded exactly as in the specification: Focus fits 2, Team fits 6, Board fits 12, Old Room fits 4 and is inactive. *(Source: project lead, working session 2026-09-29; ADR-0024)*

**First-admin bootstrap** — No login exists and no earlier artifact defined how the first admin is created. Decision: a configured bootstrap admin (name and email from configuration) is seeded on first run if no active admin exists; idempotent, no hard-coded default account, startup fails clearly if no admin exists and configuration is missing, audited as a system action. *(Source: solution architect proposal recorded in ADR-0024; confirmed by the project lead in working session 2026-09-29)*

**POC runs locally only** — Bound to localhost and not deployed. The run instructions must forbid LAN binding and public tunnels (SEC-13). *(Source: project lead, working session 2026-09-29; ADR-0024; security-assessment-meeting-room-booking-v1.2.md)*

**Post-POC phase** — Hosting and infrastructure will be decided after the POC demo. It has no date and no owner yet. *(Source: project lead, working session 2026-09-29; ADR-0024)*

**Phase 3 status** — Product definition produced 2026-09-29 in .project/product-definition/ (six modules). The five rules above were confirmed by the project lead, not directly by the client stakeholder; to be shown to Office Operations. *(Source: product definition session 2026-09-29)*
