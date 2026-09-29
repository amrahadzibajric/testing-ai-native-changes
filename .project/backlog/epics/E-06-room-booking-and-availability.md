# E-06: Room Booking and Availability

**EPIC ID:** E-06
**Delivery Phase:** Phase 1 — POC (build) — module M-01
**Dependencies:** E-02, E-03, E-04 (identity and role), E-05 (active rooms). Its rules engine and write service are consumed by E-07 and E-09.
**ADR References:** ADR-0001, ADR-0003, ADR-0005, ADR-0008, ADR-0013 (Proposed), ADR-0014, ADR-0019, ADR-0020 (Proposed), ADR-0025
**Architecture References:** arch v1.2 (Booking rules, Booking write service, Availability pages, Concurrency); key flow 1 (create a booking under concurrency); data-model bookings.md; security-assessment v1.2 (SEC-04)

## Description

This EPIC delivers module M-01, the heart of the product: finding a free room for a date, time and group size, and booking it under the rules that stop rooms being double booked or hogged. It builds the pure booking rules engine, the single booking-write service that guarantees no double booking under concurrency on SQLite, the availability search and timeline board (web) and per-room list (phone), the create booking flows (self, on behalf of another person, admin for self), rebooking of the freed no-show remainder, and the My bookings list. Before this EPIC the application can only manage people and rooms; afterwards an employee can find and book a room in under a minute and the room is never double booked.

## Scope Boundaries

- **Includes:** rules engine (grid, duration, hours, past, capacity, overlap, 3-booking limit, named refusals); booking-write service and concurrency control; availability search and read model; timeline board and phone list; create booking for self and on behalf; remainder rebooking; My bookings list and presentation.
- **Excludes:** edit, cancel, check-in, sweeper and cascade (E-07); admin oversight lists (E-08); import (E-09); recurring bookings, notifications, holidays, named attendees (out of scope).

## Key Technical Components

Booking rules component, Booking write service, Availability and booking pages, My bookings page, availability read model, effective status.

## Key User Roles Affected

Employee (searches, books, sees own bookings), Admin (also books on behalf of others).

## Done Criteria

- All booking rules are enforced with a message naming the broken rule; the same rules serve manual booking, edit, remainder rebooking and import.
- N simultaneous identical requests for one slot result in exactly one booking.
- Availability shows free or busy per room and never reveals who booked.
- An employee cannot exceed 3 upcoming bookings; admin bookings on behalf count toward the person; admins booking for themselves have no limit.
- The freed remainder of a no-show can be booked on the 15-minute grid, with the 30-minute minimum applied pending ADR-0013.

## Features

| Feature | Description | Stories | Surfaces |
|---------|-------------|---------|----------|
| [F-06.01: Booking Rules Engine](../features/F-06.01-booking-rules-engine.md) | Pure rule set with named refusals | 5 | Engineering |
| [F-06.02: Booking Write Service and Concurrency Control](../features/F-06.02-booking-write-service-and-concurrency-control.md) | Single write path, immediate SQLite transaction, no double booking | 4 | Engineering |
| [F-06.03: Availability Search and View](../features/F-06.03-availability-search-and-view.md) | Availability query, privacy read model, timeline board and phone list | 4 | Web, Phone browser |
| [F-06.04: Create Booking](../features/F-06.04-create-booking.md) | Book for self, on behalf, admin for self, drawer and phone flow | 5 | Web, Phone browser |
| [F-06.05: Rebook Freed No-show Remainder](../features/F-06.05-rebook-freed-no-show-remainder.md) | Bookable remainder on the 15-minute grid, 30-minute minimum pending | 3 | Web, Phone browser |
| [F-06.06: My Bookings List](../features/F-06.06-my-bookings-list.md) | Own upcoming and past bookings with status | 3 | Web, Phone browser |
