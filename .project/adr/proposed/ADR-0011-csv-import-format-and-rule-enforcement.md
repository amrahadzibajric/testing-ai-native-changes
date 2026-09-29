# ADR-0011: CSV Import Format and Rule-enforcing Import Behaviour

**Date:** 2026-09-29
**Status:** Proposed
**Deciders:** Office Operations (client stakeholder) to confirm; proposed by the delivery team

## Source

**Type:** Stakeholder Interview
**Reference:** client-qa-2026-09-29.md, Follow-up Answer 12 and "Proposed import format"
**Participants / Authors:** Delivery team (proposal); client stakeholder (asked delivery team to propose); recorded by Amra Hadzibajric

---

## Context

The client asked the delivery team to propose the import format *(Source: client-qa-2026-09-29.md, Answer 12)*. The proposal is a single UTF-8, comma-separated CSV with a header row and columns `room`, `person_email`, `date`, `start`, `end`, `attendees`. Room must match an active room name exactly; person must already exist; date is `YYYY-MM-DD`, Monday to Friday within the import week; times are `HH:MM` on the 15-minute grid within 08:00–20:00 and 30 minutes to 4 hours long; attendees is a whole number from 1 to room capacity. Every booking rule (overlap, hours, duration, capacity, 3-booking limit) is applied; violating rows are rejected into a report for the admin, and nothing is silently changed *(Source: client-qa-2026-09-29.md, Proposed import format)*.

## Decision Drivers

- Simple for Office Operations to export from a spreadsheet.
- Import must not bypass business rules.
- Transparent handling of failures.

## Options Considered

### Option A: Single CSV with strict validation and rejection report
- **Pros:** Simple, safe, auditable.
- **Cons:** Rejected rows must be fixed and re-imported.

### Option B: Lenient import that auto-corrects rows
- **Pros:** Fewer rejections.
- **Cons:** Silent changes; explicitly ruled out by the proposal.

## Decision

**Proposed: we will use Option A as described. Pending client confirmation.**

## Consequences

### Positive
- Bookings imported are valid by construction.

### Negative / Trade-offs
- Historical double-bookings in the spreadsheet will surface as rejected rows.

### Risks / Follow-up Actions
- [ ] Obtain client confirmation of the format.
- [ ] Confirm whether the date range is a single fixed week chosen at import time.
- [ ] Decide whether imported bookings count towards the 3-booking limit in the order rows appear (the proposal says the limit is enforced).
