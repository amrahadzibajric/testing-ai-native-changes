# E-07: Booking Lifecycle

**EPIC ID:** E-07
**Delivery Phase:** Phase 1 — POC (build) — module M-02
**Dependencies:** E-02, E-03, E-04, E-06 (rules engine and write service). F-07.05 (cascade) is consumed by E-04 (person deactivation) and E-05 (room deactivation).
**ADR References:** ADR-0003, ADR-0007, ADR-0008, ADR-0009, ADR-0013 (Proposed), ADR-0014, ADR-0015 (decisions 3 and 4), ADR-0018 (Proposed), ADR-0019 (Proposed)
**Architecture References:** arch v1.2 (Lifecycle service, Status sweeper, Scheduled and Background Processing); key flow 2 (check in, no-show, remainder rebooking); data-model bookings.md (status lifecycle, cancellation origin)

## Description

This EPIC delivers module M-02, which governs what happens to a booking after it is created: edit, cancel with the late-cancellation flag, check in, automatic no-show marking with room release, automatic done marking, and the deactivation cascade that cancels future bookings as "cancelled by admin". It is what turns the tool from a booking list into a room-hogging deterrent: unused bookings are visible and their time is released automatically without any penalty. Before this EPIC a booking can only be created; afterwards its full lifecycle runs on the real clock, with correct behaviour even if the background sweeper is late.

## Scope Boundaries

- **Includes:** edit; cancel and late flag (owner and admin); check-in (owner only); sweeper and shared effective-status computation; shared deactivation cascade and the cancelled-by-admin visibility; related UI (edit drawer and sheet, cancel confirm, phone check-in).
- **Excludes:** admin oversight lists and reports (E-08); notifications and automatic penalties (out of scope, ADR-0009); recurring bookings.

## Key Technical Components

Lifecycle service, Status sweeper (hosted service), effective-status computation, cascade service, My bookings actions.

## Key User Roles Affected

Employee (edit, cancel, check in), Admin (edit and cancel anyone's booking, cascade through deactivation), System (sweeper).

## Done Criteria

- Only Confirmed bookings that have not started can be edited or cancelled; edits re-run all rules, are never late and do not count as new bookings.
- Cancellation under 2 hours before start by the owner is late-flagged; admin cancellations and deactivations are never late-flagged.
- Only the owner can check in, from 10 minutes before start until 15 minutes after.
- No-show and done transitions occur within 1 minute of their trigger and are correct even before the sweeper has run.
- Deactivating a room or person cancels future bookings as "cancelled by admin" and leaves in-progress bookings untouched.

## Features

| Feature | Description | Stories | Surfaces |
|---------|-------------|---------|----------|
| [F-07.01: Edit Booking](../features/F-07.01-edit-booking.md) | Edit room, date, time, attendees with full re-validation | 4 | Web, Phone browser |
| [F-07.02: Cancel Booking and Late Cancellation Flag](../features/F-07.02-cancel-booking-and-late-cancellation-flag.md) | Owner and admin cancellation, late flag, refusals, UI | 5 | Web, Phone browser |
| [F-07.03: Check-in](../features/F-07.03-check-in.md) | Owner-only check-in window and phone-first UI | 4 | Web, Phone browser |
| [F-07.04: Status Sweeper (No-show and Done)](../features/F-07.04-status-sweeper-no-show-and-done.md) | Background transitions and effective status | 4 | Engineering |
| [F-07.05: Deactivation Cascade Cancellation](../features/F-07.05-deactivation-cascade-cancellation.md) | Shared cascade for room and person deactivation | 4 | Engineering, Web |
