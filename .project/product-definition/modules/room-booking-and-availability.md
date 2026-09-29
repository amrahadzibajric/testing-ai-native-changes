---
id: M-01
name: Room Booking and Availability
phase: 1
status: defined
epic: TBD
adr-refs: [ADR-0001, ADR-0003, ADR-0005, ADR-0007, ADR-0013, ADR-0014]
---

# Module: Room Booking and Availability

**Last updated:** 2026-09-29

## Purpose

This module lets people find a free room for the day, time and group size they need, and create a booking. It replaces the shared spreadsheet and Slack messages that currently cause double bookings and rooms that nobody can see are free. It also owns the core booking rules that every other module relies on: time grid, duration, office hours, capacity, overlap and the 3-upcoming-booking limit. *(Source: room-booking-spec.md, Background; Finding and booking a room)*

## Users and Roles

- **Employee:** searches availability, books a room for themselves, sees only their own bookings. *(Source: room-booking-spec.md, Who will use it; Privacy)*
- **Admin:** does everything an employee can, and can also book on behalf of another person. *(Source: room-booking-spec.md, Stopping people from hogging rooms)*

## Key Capabilities

1. **Availability search.** The user chooses a date and time range (and optionally a minimum number of people) and sees which active rooms are free. Rooms whose capacity is below the requested number of people are filtered out. *(Source: room-booking-spec.md, Finding and booking a room)*
2. **Availability view.** A room is shown as free or busy only. The view never reveals who booked a busy slot. *(Source: client-qa-2026-09-29.md, Working assumptions; room-booking-spec.md, Privacy)*
3. **Create booking.** The user picks a room, date, start time, end time and the number of attendees. Attendees are a head count, not a list of people. *(Source: room-booking-spec.md, Finding and booking a room; client-qa-2026-09-29.md, Working assumptions)*
4. **Book on behalf of another person (admin).** An admin selects the person the booking is for. It counts toward that person's 3-booking limit. *(Source: room-booking-spec.md, Stopping people from hogging rooms)*
5. **Book the freed remainder of a no-show.** When a booking is marked no-show, the time from the no-show moment to the original end is available for booking again, with start times on the 15-minute grid. *(Source: client-qa-2026-09-29.md, Answers 4 and 11; ADR-0008)*
6. **My bookings.** A user sees their own upcoming and past bookings with status. Editing and cancelling are provided by the Booking Lifecycle module (M-02).

## Business Rules

| # | Rule | Value | Source |
|---|------|-------|--------|
| BR-01 | Minimum duration | 30 minutes | room-booking-spec.md, Finding and booking a room |
| BR-02 | Maximum duration | 4 hours | room-booking-spec.md, Finding and booking a room |
| BR-03 | Time grid | Start and end times fall on 15-minute marks (:00, :15, :30, :45) | room-booking-spec.md, Finding and booking a room |
| BR-04 | Office hours | 08:00 to 20:00, Monday to Friday; the whole booking must fit inside | room-booking-spec.md, Finding and booking a room |
| BR-05 | No past bookings | A booking cannot start in the past | room-booking-spec.md, Finding and booking a room |
| BR-06 | No overlap | A room cannot be booked twice for the same time; back-to-back is allowed (one ends at 10:00, the next starts at 10:00). Only active bookings (confirmed or checked in) block a room; cancelled and no-show bookings do not, except that a no-show still occupies the time up to the no-show moment | room-booking-spec.md; client-qa-2026-09-29.md, Answer 4 |
| BR-07 | Capacity | Attendees must be at least 1 and no more than the room's capacity | room-booking-spec.md; client-qa-2026-09-29.md, Proposed import format |
| BR-08 | Booking limit | An employee may have at most 3 upcoming bookings. "Upcoming" means confirmed bookings that have not started | room-booking-spec.md; client-qa-2026-09-29.md, Working assumptions |
| BR-09 | Admin on behalf | An admin booking for another person counts toward that person's 3 | room-booking-spec.md |
| BR-10 | Admin for self | An admin booking for themselves has no limit | room-booking-spec.md |
| BR-11 | Admins follow time rules | Admins follow all time, duration, capacity and overlap rules; only the 3-booking limit is waived for admins booking for themselves | client-qa-2026-09-29.md, Working assumptions |
| BR-12 | Remainder rebooking | A rebooked no-show remainder must start on the 15-minute grid, at or after the no-show moment, and still meets BR-01 to BR-08. The 30-minute minimum applying here is pending client confirmation (ADR-0013, Proposed) | ADR-0008; ADR-0013 |
| BR-13 | Inactive rooms | Inactive rooms are not offered for booking | room-booking-spec.md, For admins |
| BR-14 | Single time zone | All times are in one office time zone; no time-zone selection | room-booking-spec.md, Not needed for now |
| BR-15 | No holidays calendar | Public holidays are not modelled; a weekday is bookable | client-qa-2026-09-29.md, Working assumptions |

A booking that fails any rule is refused with a message that names the rule broken. Nothing is silently adjusted.

## Interfaces

- **Depends on:** Room Management (M-03) for the list of active rooms and capacities; People and Identity (M-04) for the current user, role and person records; Booking Lifecycle (M-02) for status transitions that free or block time.
- **Consumed by:** Booking Lifecycle (M-02), Admin Booking Oversight (M-05), Booking Import (M-06, which reuses the same rule set).

## Out of Scope

Recurring meetings; email, calendar or Slack notifications; booking across multiple time zones; native mobile app; room equipment or catering; approval workflows; named attendee lists. *(Source: room-booking-spec.md, Not needed for now)*

## Related ADRs

ADR-0001, ADR-0003, ADR-0005, ADR-0007, ADR-0008, ADR-0013 (Proposed, pending client confirmation), ADR-0014.

## Open Items

- [ ] Confirm the 30-minute minimum for rebooked remainders (ADR-0013, Proposed). Consequence if not confirmed: a 15-minute gap could be bookable.
- [ ] Confirm the office time zone by name (single time zone is assumed, the zone itself is not stated). **Resolved by ADR-0015:** Central European Time with daylight saving (CET/CEST, e.g. Europe/Berlin-equivalent); rules evaluate in office local time. The exact IANA zone name remains an assumption to confirm with the client.
- [ ] Confirm "start in the past" means start earlier than the current time, so a booking starting exactly on the current grid time is refused only if that moment has passed. Assumption: evaluated to the minute.
- [ ] Confirm whether an admin booking on behalf of a person may exceed the person's limit (assumed no, per BR-09).
