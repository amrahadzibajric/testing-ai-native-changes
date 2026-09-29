---
id: M-05
name: Admin Booking Oversight and Reporting
phase: 1
status: defined
epic: TBD
adr-refs: [ADR-0005, ADR-0009]
---

# Module: Admin Booking Oversight and Reporting

**Last updated:** 2026-09-29

## Purpose

This module gives admins a single place to see every booking, filter it, and review who cancels late or does not show up. It supports the goal of reducing hoarded and unused rooms by making behaviour visible, without imposing automatic penalties. *(Source: room-booking-spec.md, Cancelling; For admins; client-qa-2026-09-29.md, Answer 6; ADR-0009)*

## Users and Roles

- **Admin:** full access.
- **Employee:** no access. Employees see only their own bookings, through M-01. *(Source: room-booking-spec.md, Privacy)*

## Key Capabilities

1. **All-bookings list.** Shows every booking with room, person, date, time, attendees and status. *(Source: room-booking-spec.md, For admins)*
2. **Filters** by room, person and status (Confirmed, Checked in, Cancelled, Cancelled by admin, No-show, Done). A "late cancellation" filter and a date range are recommended additions. *(Source: room-booking-spec.md, For admins; the last two are Product Strategist recommendations)*
3. **Late cancellation review.** A view or filter showing cancellations flagged late, with the person, so admins can see who does this often. Counts per person over a chosen period are included. *(Source: room-booking-spec.md, Cancelling)*
4. **No-show review.** A view or filter of no-show bookings, with counts per person. *(Source: room-booking-spec.md, Checking in; client-qa-2026-09-29.md, Answer 6)*
5. **Act from the list.** From any booking an admin can cancel it, or open it for editing, using M-02.

## Business Rules

| # | Rule | Value | Source |
|---|------|-------|--------|
| AO-01 | Admin-only | Only admins can see other people's bookings and this module | room-booking-spec.md, Privacy |
| AO-02 | Review only | Reports are informational. No automatic blocking, limit reduction or notification arises from late cancellations or no-shows | client-qa-2026-09-29.md, Answer 6; ADR-0009 |
| AO-03 | Late flag source | The late flag is set at cancellation time (less than 2 hours before start) and never changes afterward | room-booking-spec.md, Cancelling |
| AO-04 | Edits invisible to reports | Edits are not counted or flagged, so last-minute edits do not show up as late behaviour (accepted trade-off) | client-qa-2026-09-29.md, Answer 10; ADR-0007 |
| AO-05 | Cancelled by admin | Bookings cancelled by an admin action (room or person deactivation) are shown with their own status and are excluded from late-cancellation counts | Assumption; see M-02 open items. Confirmed by ADR-0015 (admin cancellations are never late-flagged) |

## Interfaces

- **Depends on:** Booking Lifecycle (M-02) for status and flags; Room Management (M-03) and People and Identity (M-04) for filter values.
- **Consumed by:** none (end of the chain, used by admins directly).

## Out of Scope

Export to Excel or PDF, dashboards and charts, utilisation analytics, scheduled report emails, automatic warnings, penalties, retention of historical data beyond the pilot, notification to employees about their own statistics.

## Related ADRs

ADR-0005 (role enforcement in the pilot), ADR-0009.

## Open Items

- [ ] What reporting period and threshold do admins consider "a lot" of late cancellations? Assumed: none in the system; admins judge from the counts.
- [ ] Is an export (for example CSV) of the list wanted for Office Operations reviews? Not requested; deferred.
- [ ] Should employees be able to see their own late-cancellation and no-show counts? Not stated; assumed no.
