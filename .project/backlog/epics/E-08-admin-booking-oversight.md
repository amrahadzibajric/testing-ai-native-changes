# E-08: Admin Booking Oversight

**EPIC ID:** E-08
**Delivery Phase:** Phase 1 — POC (build) — module M-05
**Dependencies:** E-04 (admin policy), E-06 (bookings), E-07 (status, flags, cancel and edit)
**ADR References:** ADR-0005, ADR-0009, ADR-0015 (decision 3)
**Architecture References:** arch v1.2 (Oversight queries, admin bookings pages); data-model bookings.md (review indexes)

## Description

This EPIC delivers module M-05 so that admins have one place to see every booking, filter it and review who cancels late or does not show up. It supports the goal of reducing hoarded and unused rooms by making behaviour visible, without any automatic penalty. Before this EPIC only owners see their own bookings; afterwards admins see all bookings with filters by room, person and status, review late cancellations and no-shows with per-person counts over a period, and can act on a booking from the list through the lifecycle features.

## Scope Boundaries

- **Includes:** all-bookings list and filters; late-cancellation and no-show review with counts; admin actions from the list; web and phone screens.
- **Excludes:** export to Excel or PDF, dashboards, charts, analytics, scheduled report emails, automatic warnings or penalties, employee-facing statistics (M-05 out of scope).

## Key Technical Components

Oversight queries (admin-only read models), admin bookings and review pages.

## Key User Roles Affected

Admin only. Employees have no access.

## Done Criteria

- Admins can list all bookings and filter by room, person, status, date range and late cancellation.
- The late-cancellation and no-show views show per-person counts and exclude admin-origin cancellations from late counts.
- Reports are informational: no code path blocks, limits or notifies anyone based on them.
- Admins can cancel or open for editing a booking from the list.
- An employee cannot open any admin oversight page or query another person's bookings.

## Features

| Feature | Description | Stories | Surfaces |
|---------|-------------|---------|----------|
| [F-08.01: All Bookings List and Filters](../features/F-08.01-all-bookings-list-and-filters.md) | Admin list of every booking with filters | 4 | Web, Phone browser, Admin |
| [F-08.02: Late Cancellation and No-show Review](../features/F-08.02-late-cancellation-and-no-show-review.md) | Review views with per-person counts | 4 | Web, Phone browser, Admin |
| [F-08.03: Admin Actions from the Booking List](../features/F-08.03-admin-actions-from-the-booking-list.md) | Cancel or edit from the list | 3 | Web, Phone browser, Admin |
