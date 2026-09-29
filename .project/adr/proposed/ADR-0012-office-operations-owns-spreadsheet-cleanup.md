# ADR-0012: Office Operations Owns Spreadsheet Clean-up Before Import

**Date:** 2026-09-29
**Status:** Proposed
**Deciders:** Office Operations (client stakeholder) to confirm

## Source

**Type:** Stakeholder Interview
**Reference:** client-qa-2026-09-29.md, "Proposed import format" (final bullet)
**Participants / Authors:** Delivery team (assumption); recorded by Amra Hadzibajric

---

## Context

The existing spreadsheet suffers from double bookings *(Source: room-booking-spec.md, Background)*. The import rejects rows that violate rules (ADR-0011). The delivery team assumed Office Operations owns cleaning the spreadsheet, for example resolving known double bookings, before export *(Source: client-qa-2026-09-29.md, Proposed import format)*. The client has not yet confirmed this.

## Decision Drivers

- Only the client knows which of two conflicting bookings is correct.
- Keeps the import tool free of conflict-resolution logic.

## Options Considered

### Option A: Office Operations cleans the data before export
- **Pros:** Correct decisions made by the business.
- **Cons:** Effort on the client side.

### Option B: Delivery team cleans the data
- **Pros:** Less client effort.
- **Cons:** Delivery team cannot judge which booking should stand.

## Decision

**Proposed: Office Operations will own spreadsheet clean-up and export. Pending client confirmation.**

## Consequences

### Positive
- Clear responsibility.

### Negative / Trade-offs
- Import timing depends on the client's preparation.

### Risks / Follow-up Actions
- [ ] Confirm ownership with Office Operations together with ADR-0011.
