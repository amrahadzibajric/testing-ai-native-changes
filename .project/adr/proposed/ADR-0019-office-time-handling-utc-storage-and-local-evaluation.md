# ADR-0019: Office Time Handling: UTC Storage, Office-zone Evaluation

**Date:** 2026-09-29
**Status:** Proposed
**Deciders:** Solution Architect (recommendation); project lead to confirm

## Source

**Type:** Internal Discussion
**Reference:** Phase 4 architecture design, 2026-09-29; ADR-0015 (decision 1); nfrs.md NFR-015; room-booking-and-availability.md BR-14
**Participants / Authors:** Solution Architect

---

## Context

All booking rules must be evaluated in office local time, Central European Time with daylight saving, and daylight-saving transitions must be handled correctly *(Source: ADR-0015)*. The next transitions are Sunday 25 October 2026 (clocks go back) and Sunday 28 March 2027 (clocks go forward). Office hours are Monday to Friday, so transitions fall on a weekend outside office hours, but date arithmetic, weekday tests and displayed times must still be right.

## Decision Drivers

- Correctness across daylight saving with minimal complexity.
- Testability: the clock must be controllable in tests.
- Zone name is still an assumption, so it must be configuration, not code.

## Options Considered

### Option A: Store instants in UTC; keep the office time zone as configuration; convert to office local time for every rule; inject a clock abstraction
- **Pros:** Unambiguous storage; comparison of "now" to start is a simple instant comparison; wall-clock rules (08:00 to 20:00, weekday, grid) use the configured zone; a fake clock makes DST tests trivial.
- **Cons:** Requires disciplined conversion at the boundaries (input, display, import).

### Option B: Store local date and time without offset
- **Pros:** Simple to read in the database.
- **Cons:** Ambiguous during the repeated hour in autumn; "now" comparisons need conversion anyway; breaks if the zone ever changes.

### Option C: Store local time plus offset (datetimeoffset)
- **Pros:** Unambiguous and readable.
- **Cons:** Offset baked into rows; date-only rules still need zone lookup; less clean than UTC.

## Decision

**We propose Option A.**

Bookings store start and end as UTC instants. The office zone identifier (assumed Europe/Berlin, to be confirmed) is application configuration. Office hours, the 15-minute grid, the weekday test, the 2-hour late rule, the check-in window and the import window are all computed by converting to the office zone. Nonexistent or ambiguous local times cannot occur inside office hours; input that lands on them is rejected with a clear message. The application reads time only through an injectable clock (TimeProvider).

## Consequences

### Positive
- DST-correct behaviour and straightforward automated tests around 25 October 2026 and 28 March 2027.
- Changing the zone identifier needs no data migration.

### Negative / Trade-offs
- Database values are UTC and need conversion when read directly (for example in ad-hoc SQL).

### Risks / Follow-up Actions
- [ ] Confirm the IANA zone name with Office Operations.
- [ ] Ensure the production hosting image includes time zone data (Linux App Service does).
