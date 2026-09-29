# ADR-0018: Background Processing for Automatic No-show, Room Release and Done Marking

**Date:** 2026-09-29
**Status:** Proposed
**Deciders:** Solution Architect (recommendation); project lead to confirm

## Source

**Type:** Internal Discussion
**Reference:** Phase 4 architecture design, 2026-09-29; booking-lifecycle.md LC-08 to LC-10; nfrs.md NFR-006
**Participants / Authors:** Solution Architect

---

## Context

No-show marking, room release and done marking are automatic and must occur within 1 minute of the trigger moment *(Source: nfrs.md, NFR-006; booking-lifecycle.md, LC-08, LC-10)*. The freed remainder of a no-show must be bookable straight away *(Source: ADR-0008)*. The application is a single deployable on one App Service instance (ADR-0016, ADR-0017).

## Decision Drivers

- Correctness must not depend on a scheduler having run.
- Simplicity and low cost.
- Safe if the job runs twice or the platform restarts.

## Options Considered

### Option A: In-process hosted service (timer every 30 to 60 seconds) with idempotent set-based updates, plus effective status computed on read
- **Pros:** No extra Azure resource; no extra deployment; idempotent updates are safe if two instances ever run; because reads treat "Confirmed and now is at least start plus 15 minutes" as a no-show, booking rules are correct even if the job is late.
- **Cons:** Requires Always On so the process is not idled; runs inside the web process.

### Option B: Azure Functions timer trigger
- **Pros:** Separates the job from the web app; independent scaling.
- **Cons:** A second deployable and resource for one small job; duplicated data access code.

### Option C: Azure Logic Apps or Azure Automation schedule calling an endpoint
- **Pros:** No code for scheduling.
- **Cons:** Minimum recurrence and cold behaviour add latency risk against the 1-minute target; extra cost and an endpoint that needs protecting.

### Option D: Compute status on read only, no persisted transition
- **Pros:** No scheduler at all.
- **Cons:** Status history and admin no-show reports need persisted state; the late-flag and reporting queries become complex.

## Decision

**We propose Option A.**

A hosted service runs about every 30 seconds and applies idempotent transitions in office-time-aware terms: Confirmed bookings whose start plus 15 minutes has passed become No-show; Checked-in bookings whose end has passed become Done. The overlap and availability logic additionally uses effective status computed from the clock, so the rules stay correct between job runs and after a restart.

## Consequences

### Positive
- Meets NFR-006 with no extra resources.
- Robust to restart and to accidental double execution.

### Negative / Trade-offs
- The scheduler shares the web process; a heavy import could delay it briefly (mitigated by the read-time safety net).
- Scaling out later would run the job on each instance; the idempotent updates keep this safe, but a distributed lock or a Functions timer would be preferable then.

### Risks / Follow-up Actions
- [ ] Enable Always On on the App Service.
- [ ] Test transitions with a controllable clock, including around the DST change (see ADR-0019).
