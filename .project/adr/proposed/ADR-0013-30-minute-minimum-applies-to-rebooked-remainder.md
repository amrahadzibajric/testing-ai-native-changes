# ADR-0013: 30-Minute Minimum Applies to a Rebooked No-show Remainder

**Date:** 2026-09-29
**Status:** Proposed
**Deciders:** Office Operations (client stakeholder) to confirm

## Source

**Type:** Stakeholder Interview
**Reference:** client-qa-2026-09-29.md, "Working assumptions carried forward"
**Participants / Authors:** Delivery team (assumption); recorded by Amra Hadzibajric

---

## Context

Meetings must be at least 30 minutes and at most 4 hours *(Source: room-booking-spec.md, Finding and booking a room)*. The client confirmed the freed remainder of a no-show can be rebooked on the 15-minute grid (ADR-0008). The working assumption, not contradicted by the client, is that the 30-minute minimum still applies to the rebooked remainder, so a 10:45–11:00 booking would not be allowed *(Source: client-qa-2026-09-29.md, Working assumptions)*.

## Decision Drivers

- Consistency with the general duration rule.
- Avoiding very short, fragmented bookings.

## Options Considered

### Option A: 30-minute minimum applies
- **Pros:** One consistent duration rule.
- **Cons:** Final short gaps stay unused.

### Option B: Allow 15-minute remainders
- **Pros:** Uses every freed gap.
- **Cons:** Creates an exception to the stated rule.

## Decision

**Proposed: the 30-minute minimum applies to rebooked remainders. Pending client confirmation.**

## Consequences

### Positive
- No special-case duration rule.

### Negative / Trade-offs
- Gaps shorter than 30 minutes cannot be rebooked.

### Risks / Follow-up Actions
- [ ] Confirm with the client alongside ADR-0011.
