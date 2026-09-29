# ADR-0007: Employees Can Edit Their Bookings; Edits Are Not Flagged as Late

**Date:** 2026-09-29
**Status:** Accepted
**Deciders:** Office Operations (client stakeholder)

## Source

**Type:** Stakeholder Interview
**Reference:** client-qa-2026-09-29.md, Answer 3 and Follow-up Answer 10
**Participants / Authors:** Client stakeholder (Office Operations); recorded by Amra Hadzibajric

---

## Context

The request says employees "manage their own bookings" but lists only booking and cancelling explicitly *(Source: room-booking-spec.md, Who will use it; Cancelling)*. The client confirmed employees can edit bookings (time or room) *(Source: client-qa-2026-09-29.md, Answer 3)*, and that editing close to the start is acceptable and is never flagged. Only cancellations less than 2 hours before start are flagged *(Source: client-qa-2026-09-29.md, Answer 10)*.

## Decision Drivers

- Flexibility for employees.
- Rule integrity (overlaps, hours, limits).
- Late-cancellation reporting must stay meaningful.

## Options Considered

### Option A: Editing allowed, re-validated, never flagged
- **Pros:** Matches client answer; simple for users.
- **Cons:** Editing can be used to avoid a late-cancel flag (accepted by client).

### Option B: Cancel and rebook only
- **Pros:** Simpler.
- **Cons:** Contradicts the client answer.

## Decision

**We will allow employees to edit the time and room of their bookings; every edit re-runs all booking rules, and edits are never flagged as late. Only cancellations less than 2 hours before start are flagged.**

## Consequences

### Positive
- Consistent rule enforcement on every change.

### Negative / Trade-offs
- Last-minute edits are a known way around late-cancellation flags; the client accepted this.

### Risks / Follow-up Actions
- [ ] Define in the product definition how editing interacts with the 3-booking limit and with bookings that have already started (assumed not editable, consistent with the cancel rule; confirm at product definition).
