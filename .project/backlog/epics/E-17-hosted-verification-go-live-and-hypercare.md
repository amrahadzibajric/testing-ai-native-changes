# E-17: Hosted Verification, Go-Live and Hypercare

**EPIC ID:** E-17
**Status:** Deferred — Post-POC (ADR-0024)
**Delivery Phase:** Post-POC (hosted pilot go-live)
**Dependencies:** E-12, E-13, E-14, E-15; Office Operations preparation of the real CSV (ADR-0012); go-live date (ADR-0001, not fixed)
**ADR References:** ADR-0001, ADR-0010, ADR-0012 (Proposed), ADR-0014, ADR-0015, ADR-0021, ADR-0024
**Architecture References:** arch v1.2 Target State; test-strategy v1.2 Target state (pre-go-live stage); security-assessment v1.2 (ADR Triggers: re-run against the hosted design); nfrs.md NFR-003, NFR-004, NFR-007, NFR-008, NFR-011

## Description

This deferred EPIC takes the hosted pilot live: the checks that could not be done in the POC (hosted security re-assessment, hosted performance, backup restore rehearsal, gate reachability and HTTPS-only), the one-off import with the real spreadsheet export before opening time on the go-live day, retirement of the spreadsheet, and a short hypercare window. Hypercare is included because the pilot's post-go-live triage is expected from the delivery team; it is sized at one week for a small, single-application pilot with no formal SLA. It cannot be estimated until the hosted phase is triggered and a go-live date exists.

## Scope Boundaries

- **Includes:** pre-go-live verification; real-data import at go-live, people onboarding, spreadsheet retirement and disabling the import page; one week of hypercare.
- **Excludes:** company-wide rollout beyond 50 users and a fixed rollout date (future phase, ADR-0001).

## Key Technical Components

Hosted test and production environments, import module in production, monitoring.

## Key User Roles Affected

Office Operations (export, clean-up), Admins (import, people onboarding), all pilot users.

## Done Criteria

- The security assessment is re-run against the hosted design and there are no unresolved High or Critical findings.
- The restore rehearsal and hosted performance and reachability checks pass.
- The real import has run before opening time, every rejected row is accounted for, and the import page is disabled afterwards.
- Hypercare closes with a handover and a list of follow-up items.

## Features

| Feature | Description | Stories | Surfaces |
|---------|-------------|---------|----------|
| [F-17.01: Pre-go-live Verification](../features/_deferred/F-17.01-pre-go-live-verification.md) | Security re-assessment, restore rehearsal, hosted performance, E2E and gate checks | 4 | Azure |
| [F-17.02: Pilot Go-live Import and Cutover](../features/_deferred/F-17.02-pilot-go-live-import-and-cutover.md) | Go-live decision, real import, spreadsheet retirement | 3 | Web, Admin |
| [F-17.03: Hypercare](../features/_deferred/F-17.03-hypercare.md) | One-week post-go-live triage and handover | 2 | Azure |
