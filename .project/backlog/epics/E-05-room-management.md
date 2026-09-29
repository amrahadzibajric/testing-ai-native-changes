# E-05: Room Management

**EPIC ID:** E-05
**Delivery Phase:** Phase 1 — POC (build) — module M-03
**Dependencies:** E-02, E-03, E-04 (admin role and policies). F-05.02 depends on F-07.05 (cascade service in E-07).
**ADR References:** ADR-0005, ADR-0014 (pattern), ADR-0024 (seed)
**Architecture References:** arch v1.2 (Room service, admin room pages); data-model rooms.md

## Description

This EPIC delivers module M-03 so that admins can maintain the list of bookable rooms: add rooms, rename them, change capacity, and take a room out of use with a clear and safe effect on future bookings. Before it, the rooms exist only as seed data; afterwards an admin controls the room list through the app, deactivation cancels future bookings as cancelled by admin, and inactive rooms disappear from booking and import. The specification's rooms (Focus 2, Team 6, Board 12, Old Room 4 inactive) are seeded in E-02.

## Scope Boundaries

- **Includes:** add, rename, capacity change, room list, deactivate with warning and cascade, reactivate, phone and web screens.
- **Excludes:** room deletion, scheduled future deactivation, equipment, floor plans, room-level opening hours, restoring bookings after reactivation (all out of scope in M-03).

## Key Technical Components

Room service, admin room pages, use of the cascade service (F-07.05).

## Key User Roles Affected

Admin (manages rooms), Employee (sees only active rooms).

## Done Criteria

- Admins can add, rename and re-size rooms; names are unique ignoring case.
- Lowering capacity leaves existing bookings unchanged; new and edited bookings respect the new capacity.
- Deactivating a room warns with the count, cancels future Confirmed bookings as cancelled by admin (never late), leaves in-progress bookings untouched and removes the room from booking and import.
- Reactivation makes the room bookable again without restoring cancelled bookings.

## Features

| Feature | Description | Stories | Surfaces |
|---------|-------------|---------|----------|
| [F-05.01: Room Maintenance](../features/F-05.01-room-maintenance.md) | Add, rename, change capacity, list, screens | 4 | Web, Phone browser, Admin |
| [F-05.02: Room Deactivation and Reactivation](../features/F-05.02-room-deactivation-and-reactivation.md) | Take a room out of use with cascade and reactivate | 4 | Web, Phone browser, Admin |
