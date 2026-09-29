# ADR-0009: Late Cancellations and No-shows Are Admin Review Only

**Date:** 2026-09-29
**Status:** Accepted
**Deciders:** Office Operations (client stakeholder)

## Source

**Type:** Stakeholder Interview
**Reference:** client-qa-2026-09-29.md, Answer 6
**Participants / Authors:** Client stakeholder (Office Operations); recorded by Amra Hadzibajric

---

## Context

The request wants late cancellations marked so that admins can see who does it often, and no-shows recorded *(Source: room-booking-spec.md, Cancelling; Checking in)*. It did not say whether consequences follow. The client answered: admin review only, no automatic penalties *(Source: client-qa-2026-09-29.md, Answer 6)*.

## Decision Drivers

- Keep the pilot simple and non-punitive.
- Provide visibility to admins for follow-up.

## Options Considered

### Option A: Flag and report only
- **Pros:** Simple; matches the client.
- **Cons:** Relies on admins acting on the data.

### Option B: Automatic penalties (blocks, reduced limits)
- **Pros:** Enforces behaviour.
- **Cons:** Rejected by the client.

## Decision

**We will flag late cancellations and record no-shows for admins to review, with no automatic penalties.**

## Consequences

### Positive
- No penalty logic to build or dispute.

### Negative / Trade-offs
- Behaviour change depends on manual admin follow-up.

### Risks / Follow-up Actions
- [ ] Define the admin review view (filter by person and status) in the product definition.
