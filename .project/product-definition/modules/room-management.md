---
id: M-03
name: Room Management
phase: 1
status: defined
epic: TBD
adr-refs: [ADR-0005, ADR-0014]
---

# Module: Room Management

**Last updated:** 2026-09-29

## Purpose

This module lets admins maintain the list of bookable rooms: add rooms, rename them, change how many people they fit, and take them out of use (for example during renovation) with a clear effect on future bookings. *(Source: room-booking-spec.md, For admins)*

## Users and Roles

- **Admin:** full access.
- **Employee:** no access to management; sees only active rooms in availability and booking screens. *(Source: room-booking-spec.md, Privacy)*

## Key Capabilities

1. **Add a room** with a name and a capacity.
2. **Rename a room** and **change its capacity.**
3. **Deactivate a room.** All future bookings of that room (Confirmed, not started) are cancelled and marked "cancelled by admin". The room no longer appears for booking. *(Source: room-booking-spec.md, For admins)*
4. **Reactivate a room** that was taken out of use. *(Source: client-qa-2026-09-29.md, Working assumptions)*
5. **Room list** showing name, capacity and active or inactive state.

## Business Rules

| # | Rule | Value | Source |
|---|------|-------|--------|
| RM-01 | Room attributes | Name, capacity (whole number of people, at least 1), active flag | room-booking-spec.md, Rooms we have today |
| RM-02 | Unique names | Room names are unique (case-insensitive); needed because the import matches rooms by exact name | client-qa-2026-09-29.md, Proposed import format (uniqueness is an assumption) |
| RM-03 | Deactivation effect | Future bookings are cancelled and marked "cancelled by admin"; the booker can see the admin origin | room-booking-spec.md, For admins |
| RM-04 | Bookings in progress | A booking that has already started when the room is deactivated is left as is | Assumption; ADR-0014 rule 1 pattern |
| RM-05 | Lowering capacity | Lowering capacity leaves existing bookings unchanged; new and edited bookings must respect the new capacity | client-qa-2026-09-29.md, Working assumptions |
| RM-06 | Reactivation | A reactivated room is bookable again; previously cancelled bookings are not restored | client-qa-2026-09-29.md, Working assumptions; restore-not-done is an assumption |
| RM-07 | Deactivation warning | Before confirming deactivation, the admin is told how many future bookings will be cancelled (recommended) | Product Strategist recommendation |

**Starting data:**

| Room | Capacity | Initial state |
|------|----------|---------------|
| Focus | 2 | Active |
| Team | 6 | Active |
| Board | 12 | Active |
| Old Room | 4 | Inactive ("currently not in use") |

*(Source: room-booking-spec.md, Rooms we have today)*

## Interfaces

- **Depends on:** People and Identity (M-04) for admin role.
- **Consumed by:** Room Booking and Availability (M-01), Booking Lifecycle (M-02, cancelled-by-admin transition), Booking Import (M-06, room name matching), Admin Booking Oversight (M-05, room filter).

## Out of Scope

Room equipment, floor plans, photos, room categories, room-level opening hours that differ from the office hours, scheduled future deactivation (an admin deactivates immediately), and restoring bookings after reactivation.

## Related ADRs

ADR-0005 (role enforcement against the selected user), ADR-0014 (person-deactivation pattern mirrors this module).

## Open Items

- [ ] Can a room be deleted, or only deactivated? Assumed: deactivation only, to preserve booking history.
- [ ] Should a room deactivation be schedulable for a future date (for a planned renovation)? Assumed no for the pilot.
- [ ] Confirm that a booking in progress is left untouched when its room is deactivated (RM-04).
