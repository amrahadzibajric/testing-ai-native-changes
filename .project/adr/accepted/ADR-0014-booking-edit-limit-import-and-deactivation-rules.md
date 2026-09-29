# ADR-0014: Booking Edit, Limit, Import Window, Person Deactivation and User-list Rules

**Date:** 2026-09-29
**Status:** Accepted
**Deciders:** Amra Hadzibajric (project lead)

## Source

**Type:** Internal Discussion
**Reference:** Project lead confirmation in working session, 2026-09-29, resolving the open points in intake-meeting-room-booking-2026-09-29.md, section 6
**Participants / Authors:** Amra Hadzibajric (project lead); recorded by Product Strategist

---

## Context

The intake record listed five detailed rules that no source document settled *(Source: intake-meeting-room-booking-2026-09-29.md, section 6)*. Several accepted or proposed ADRs explicitly deferred them to the product definition phase: ADR-0007 (editing and the 3-booking limit, editing a started booking), ADR-0004 (treatment of deactivated people), ADR-0010 and ADR-0011 (which week the import covers, and how the limit applies to imported rows). The project lead confirmed the five rules below in a working session. This decision was made by the project lead, not stated directly by the client stakeholder; it is consistent with the working assumptions in client-qa-2026-09-29.md and with the room-deactivation pattern in room-booking-spec.md ("Take a room out of use"), and should be shown to Office Operations at the next client review.

## Decision Drivers

- Consistency with existing rules (the cancellation rule, the room-deactivation pattern).
- Keeping the 3-booking limit meaningful without penalising employees for changing plans.
- A small, predictable pilot import.
- Avoiding orphaned bookings for people who leave.

## Options Considered

### Option A: Adopt the five rules as confirmed by the project lead
- **Pros:** Resolves every open point; reuses existing patterns; no new concepts.
- **Cons:** Not yet seen by the client stakeholder.

### Option B: Leave the points open until the client is asked
- **Pros:** Client-authoritative.
- **Cons:** Blocks the product definition on details the client is unlikely to disagree with.

## Decision

**We will apply the following five rules.**

1. **Started bookings cannot be edited.** A booking that has started cannot be edited, the same as the cancellation rule (ADR-0007 follow-up resolved).
2. **Edits do not count as new bookings; import order per person.** Editing a booking does not count as a new booking toward the 3-upcoming-booking limit. On import, rows are processed in date/time order per person; rows beyond that person's 3rd upcoming booking are rejected into the import report (ADR-0011 follow-up resolved).
3. **Deactivating a person cancels their future bookings.** When an admin deactivates a person, that person's future bookings are cancelled and marked "cancelled by admin", following the same pattern as room deactivation (ADR-0004 follow-up resolved).
4. **Import window.** The import covers the 5 working days starting on the go-live date (ADR-0010 and ADR-0011 follow-up resolved).
5. **Pick-a-user list.** The "pick a user" list shows active people only (ADR-0005 follow-up resolved).

## Consequences

### Positive
- All five intake gaps are closed; the product definition can state precise rules.
- Deactivation and room removal behave identically for the affected booker.

### Negative / Trade-offs
- A person who deactivates a colleague cannot see the consequence in advance unless the screen warns about the number of bookings to be cancelled (recommended in the People Management module).
- The import window is tied to the go-live date, which is not yet fixed (ADR-0001).

### Risks / Follow-up Actions
- [ ] Show these five rules to Office Operations at the next client review; treat any objection as a change to this ADR.
- [ ] ADR-0011 (CSV format) remains Proposed; rule 2 and rule 4 refine it and depend on its confirmation.
- [ ] Clarify details this ADR does not settle (listed as open items in the product definition): whether such cancellations are ever flagged late, the meaning of "working day" without a holiday calendar, and how rows dated earlier on the go-live day itself are treated.
