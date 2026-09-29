# ADR-0015: Office Time Zone, Data Protection Posture, Admin-action Late Flag, Check-in Ownership and Weekend Go-live Rule

**Date:** 2026-09-29
**Status:** Accepted
**Data Impact:** Yes
**Deciders:** Amra Hadzibajric (project lead)

## Source

**Type:** Internal Discussion
**Reference:** Project lead confirmation in working session, 2026-09-29, resolving open items carried in the product definition (M-01, M-02, M-06, nfrs.md NFR-014 and NFR-015)
**Participants / Authors:** Amra Hadzibajric (project lead); recorded by Solution Architect

---

## Context

The product definition (Phase 3) left several detailed points open. The office time zone was described only as "a single time zone" without a name *(Source: room-booking-spec.md, Not needed for now; nfrs.md, NFR-015; room-booking-and-availability.md, Open Items)*. No source stated a data protection requirement for the names and emails stored *(Source: nfrs.md, NFR-014)*. Two lifecycle questions were open: whether an admin cancelling someone else's booking shortly before the start is a late cancellation attributed to the owner, and whether an admin can check in on an owner's behalf *(Source: booking-lifecycle.md, Open Items)*. The go-live date is not fixed, and it was unclear what happens if it falls on a weekend *(Source: booking-import.md, Open Items; ADR-0014, rule 4)*.

The project lead confirmed the five decisions below in a working session on 2026-09-29. As with ADR-0014, these were confirmed by the project lead and not stated directly by the client stakeholder, so they should be shown to Office Operations at the next client review.

## Decision Drivers

- Correct, predictable evaluation of every time-based rule, including across daylight saving changes.
- Keep the pilot simple while being honest about personal data held.
- Do not penalise a person for something that was not their choice.
- Keep the check-in rule unambiguous ("the booker checks in").
- Close the last open detail of the import window.

## Options Considered

### Option A: Adopt the five decisions as confirmed by the project lead
- **Pros:** Closes every listed open item; reuses existing patterns (cancelled-by-admin, deactivation cascade); no new concepts.
- **Cons:** Not yet seen by the client stakeholder; the exact time zone name remains an assumption.

### Option B: Leave the points open until the client is asked
- **Pros:** Client-authoritative.
- **Cons:** Blocks architecture and data model on details that are unlikely to be contested.

## Decision

**We will apply the following five decisions.**

1. **Office time zone.** The office operates in Central European Time with daylight saving (CET in winter, CEST in summer), for example the IANA zone Europe/Berlin or an equivalent. All booking rules (office hours, 15-minute grid, check-in window, no-show, done, the 2-hour late-cancellation rule) are evaluated in office local time, and daylight-saving transitions must be handled correctly. The exact IANA zone name is an **assumption to confirm with the client**.
2. **Data protection.** The application is hosted in an EU Azure region, and a simple privacy note is provided for the names and email addresses stored. No stricter compliance regime is stated or required.
3. **Admin cancellations are never late-flagged.** When an admin cancels another person's booking less than 2 hours before the start, it is **not** flagged as a late cancellation, because the cancellation was not the person's choice. Cancellations caused by room deactivation or person deactivation are likewise never late-flagged.
4. **Only the owner checks in.** Only the booking owner can check in. Admins cannot check in on someone's behalf.
5. **Weekend go-live.** If the go-live date falls on a weekend, the 5-working-day import window starts on the following Monday (refines ADR-0014, rule 4).

## Consequences

### Positive
- Time rules have a defined reference clock, so tests and the import can be exact.
- Late-cancellation counts reflect only the owner's own behaviour, which keeps the admin review fair (ADR-0009).
- Check-in has a single, simple authorisation rule.
- Import window is fully defined for any go-live date.

### Negative / Trade-offs
- The zone name is an assumption; if the office is in a different CET-observing zone the rules still hold, but a non-CET office would require a change.
- Because pick-a-user is not authentication (ADR-0005), "only the owner can check in" is enforced against the selected identity and can be circumvented by switching user; this is the accepted pilot limitation, not a new one.
- The privacy note is a lightweight measure; whether company policy requires more (for example a legal review) is not established.

### Risks / Follow-up Actions
- [ ] Confirm the exact time zone name with Office Operations (working assumption: Europe/Berlin).
- [ ] Show these five decisions to Office Operations at the next client review; treat any objection as a change to this ADR.
- [ ] Draft the privacy note text (what is stored, why, who can see it, retention) with Office Operations.
- [ ] Reflect decision 3 in the data model: a late flag is set only when the actor is the booking owner.

## Change Log
- 2026-09-29: Decision 2 (EU Azure region and published privacy note) is deferred to the post-POC phase by ADR-0024, because the POC runs locally with synthetic data only. Decisions 1, 3, 4 and 5 are unaffected and remain in force. Status stays Accepted.
