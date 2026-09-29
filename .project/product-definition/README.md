# Product Definition

This folder is the **live, structured source of truth** for the project's product scope. It is written and maintained by the Product Strategist agent during and after the intake session.

All downstream agents read from here — not from the proposal. The proposal is a composed client deliverable assembled from this folder plus estimates and architecture artifacts.

## Contents

| File / Folder | Purpose | Owner |
|---------------|---------|-------|
| `modules/README.md` | Module registry — index of all modules | Product Strategist |
| `modules/<slug>.md` | One file per module | Product Strategist |
| `scope.md` | Explicit in scope / out of scope / deferred | Product Strategist |
| `nfrs.md` | Non-functional requirements table | Product Strategist |
| `e2e-scenarios.md` | Cross-module end-to-end workflows | Product Strategist |
| `integration-points.md` | External systems and dependencies | Product Strategist |

**Last updated:** 2026-09-29 (Phase 3 complete for the Meeting Room Booking pilot: six modules, scope, NFRs, 14 E2E scenarios, integration points). Decisions relied upon: ADR-0001 to ADR-0010 and ADR-0014 (Accepted); ADR-0011 to ADR-0013 (Proposed, pending client confirmation).

## Project summary

| Item | Detail |
|------|--------|
| Project | Meeting Room Booking pilot for Office Operations |
| Modules | M-01 Room Booking and Availability; M-02 Booking Lifecycle; M-03 Room Management; M-04 People and Identity; M-05 Admin Booking Oversight and Reporting; M-06 Booking Import |
| Module index | [modules/README.md](modules/README.md) |

## Who reads what

| Agent | Reads |
|-------|-------|
| Solution Architect | `modules/`, `integration-points.md`, `nfrs.md` |
| Delivery Lead | `modules/`, `scope.md` |
| Requirements Writer | `modules/<slug>.md` (relevant module only) |
| Solution Spec Writer | `modules/<slug>.md` (relevant module only) |
| QA | `e2e-scenarios.md`, `nfrs.md` |
| Product Strategist | All — maintains and composes from here |

## Update rules

- Changes to scope, modules, or NFRs go here first, then the proposal is recomposed
- Never edit the proposal to change scope — change it here and recompose
- Append to files; do not delete history — mark outdated items as `[Superseded]`
