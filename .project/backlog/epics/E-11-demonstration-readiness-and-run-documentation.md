# E-11: Demonstration Readiness and Run Documentation

**EPIC ID:** E-11
**Delivery Phase:** Phase 1 — POC (close-out)
**Dependencies:** E-06 to E-09 (functional scope complete), E-10 (test running order), F-02.06 (seed), F-09.06 (sample CSV)
**ADR References:** ADR-0011 to ADR-0015 (confirmations), ADR-0017, ADR-0021, ADR-0022, ADR-0023, ADR-0024
**Architecture References:** arch v1.2 (Local Run View, Risks 1, 9, 10); security-assessment v1.2 (SEC-13, SEC-14); test-strategy v1.2 (Running order)

## Description

This EPIC closes the POC: it writes the run instructions that keep the application on localhost only, prepares the delivery team to demonstrate it, and records what was deferred so nothing parked is forgotten. The POC is demonstrated by the delivery team, in person or by screen share, with no remote client access and no live trial. Before this EPIC the application works but nobody else can safely run or present it; afterwards there is a run guide with the localhost-only safeguards, a demonstration script mapped to the E2E scenarios and seeded data, a pre-demonstration checklist, a client-facing statement of what the POC is and is not, and a handoff register for the post-POC phase. Hypercare does not apply to the POC because there is no go-live; the post-launch commitment sits in E-17.

## Scope Boundaries

- **Includes:** run guide and safeguards (SEC-13, SEC-14); troubleshooting and configuration reference; demonstration script and checklist; POC statement and client confirmation list; post-POC handoff register and feedback capture.
- **Excludes:** deployment, hosting, pipelines, privacy note publication, gate and sign-in work (E-12 to E-16); training for real employees (no live trial); go-live and hypercare (E-17).

## Key Technical Components

Documentation and checklists; no new runtime components.

## Key User Roles Affected

Delivery team and project lead (presenters); Office Operations (audience and confirmer).

## Done Criteria

- Run instructions exist, state that LAN binding and public tunnels are forbidden, and state that the POC is not for real bookings.
- A demonstration script covers E2E-001 to E2E-014 using the seeded data without any clock control.
- The pre-demonstration checklist lists the local running order and a green run is required to cut a demonstration build.
- The client confirmation list for ADR-0011 to ADR-0015 and the "what this POC is" statement are ready.
- A handoff register lists every deferred item with its ADR and trigger.

## Features

| Feature | Description | Stories | Surfaces |
|---------|-------------|---------|----------|
| [F-11.01: Local Run Guide and Safeguards](../features/F-11.01-local-run-guide-and-safeguards.md) | Run, reset, configuration reference and localhost-only rules | 3 | Documentation |
| [F-11.02: Demonstration Pack](../features/F-11.02-demonstration-pack.md) | Script, checklist and client statement | 3 | Documentation |
| [F-11.03: Post-POC Handoff Register](../features/F-11.03-post-poc-handoff-register.md) | Deferred items register and feedback capture | 2 | Documentation |
