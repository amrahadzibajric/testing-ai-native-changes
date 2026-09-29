# ADR-0008: Freed No-show Remainder Can Be Rebooked on the 15-Minute Grid

**Date:** 2026-09-29
**Status:** Accepted
**Deciders:** Office Operations (client stakeholder)

## Source

**Type:** Stakeholder Interview
**Reference:** client-qa-2026-09-29.md, Answer 4 and Follow-up Answer 11
**Participants / Authors:** Client stakeholder (Office Operations); recorded by Amra Hadzibajric

---

## Context

The request says a booking with no check-in within 15 minutes of start becomes a no-show and the room becomes free *(Source: room-booking-spec.md, Checking in)*. It did not say whether the remaining part of the slot can be booked. The client confirmed it can, and that start times must stay on the 15-minute grid, with no off-grid starts *(Source: client-qa-2026-09-29.md, Answers 4 and 11)*.

## Decision Drivers

- Reduce wasted room time, the core problem of the request.
- Keep one consistent time grid.

## Options Considered

### Option A: Remainder bookable, 15-minute grid starts
- **Pros:** Frees capacity while staying consistent with existing rules.
- **Cons:** The earliest bookable start is the next grid point after the no-show is recorded.

### Option B: Remainder bookable from the exact no-show moment
- **Pros:** Maximises availability.
- **Cons:** Rejected by the client.

### Option C: Remainder not bookable
- **Pros:** Simplest.
- **Cons:** Rejected by the client.

## Decision

**We will make the freed remainder of a no-show booking bookable, with start times restricted to the 15-minute grid.**

## Consequences

### Positive
- Freed time is reusable.

### Negative / Trade-offs
- Minimal extra rule complexity around bookings that start in the past relative to the original slot.

### Risks / Follow-up Actions
- [ ] Minimum duration for the remainder: see ADR-0013 (Proposed).
- [ ] Define how the "no bookings in the past" rule applies when the remainder start is after now (product definition).
