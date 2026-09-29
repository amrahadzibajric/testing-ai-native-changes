---
id: M-06
name: Booking Import
phase: 1
status: defined
epic: TBD
adr-refs: [ADR-0004, ADR-0010, ADR-0011, ADR-0012, ADR-0014]
---

# Module: Booking Import

**Last updated:** 2026-09-29

## Purpose

This module lets an admin bring bookings from the existing spreadsheet into the tool once, at the start of the pilot, so that the first working week is not entered twice. It is a one-off tool: after the import the spreadsheet is retired. *(Source: client-qa-2026-09-29.md, Answer 7, Working assumptions; ADR-0010)*

> **Pending client confirmation.** The file format and the rejection behaviour (ADR-0011) and the ownership of spreadsheet clean-up (ADR-0012) are Proposed, not yet confirmed by the client. The window and ordering rules from ADR-0014 were confirmed by the project lead and refine ADR-0011.

## Users and Roles

- **Admin:** uploads the file, reviews the report and re-imports.
- **Office Operations (client):** owns the spreadsheet clean-up and exports the CSV (ADR-0012, Proposed). *(Source: client-qa-2026-09-29.md, Proposed import format)*

## Key Capabilities

1. **Upload a CSV file** (UTF-8, comma-separated, header row) with columns `room`, `person_email`, `date`, `start`, `end`, `attendees`. *(Source: client-qa-2026-09-29.md, Proposed import format; ADR-0011)*
2. **Validate every row** against the same booking rules as manual booking (M-01).
3. **Import valid rows** as Confirmed bookings and **reject the rest**.
4. **Import report** listing each rejected row with its line number, the values and the reason(s), for the admin to fix and re-import. Nothing is silently changed. *(Source: client-qa-2026-09-29.md, Proposed import format)*
5. **Re-import** of a corrected file; rows already imported must not be duplicated (see Business Rules IM-09).

## File Format (proposed, ADR-0011)

| Column | Example | Rule |
|--------|---------|------|
| `room` | Team | Must match an active room name exactly |
| `person_email` | jane.doe@company.com | Must match a person already added by an admin |
| `date` | 2026-10-05 | `YYYY-MM-DD`, within the import window, Monday to Friday |
| `start` | 09:00 | `HH:MM`, 15-minute grid, at or after 08:00 |
| `end` | 10:30 | `HH:MM`, 15-minute grid, at or before 20:00, 30 minutes to 4 hours after start |
| `attendees` | 4 | Whole number, 1 to room capacity |

*(Source: client-qa-2026-09-29.md, Proposed import format)*

## Business Rules

| # | Rule | Value | Source |
|---|------|-------|--------|
| IM-01 | Window | The import covers the 5 working days starting on the go-live date. Rows dated outside the window are rejected | ADR-0014, rule 4 |
| IM-02 | Prerequisites | Rooms must exist and be active, and people must exist and be active, before the import | ADR-0004; client-qa-2026-09-29.md, Proposed import format |
| IM-03 | All rules apply | Overlap, office hours, duration, capacity, grid and the 3-booking limit are enforced on every row | ADR-0011 |
| IM-04 | Order of processing | Rows are processed in date/time order per person. Rows beyond that person's 3rd upcoming booking are rejected into the report | ADR-0014, rule 2 |
| IM-05 | Overlaps between rows | Where two rows overlap on the same room, the row processed first (earlier date/time; ties by file order) is accepted and the later row is rejected with a conflict reason. The tool never chooses on business grounds | ADR-0011 (no silent changes); tie-break is an assumption |
| IM-06 | No silent changes | Times, rooms and attendees are never adjusted; a row is imported exactly as given or rejected | client-qa-2026-09-29.md, Proposed import format |
| IM-07 | Past rows | Rows that would start in the past at the time of import are rejected by the no-past-bookings rule | M-01 BR-05 (consequence noted as an open item) |
| IM-08 | Status | Imported rows become Confirmed bookings and follow the lifecycle from that moment | Assumption |
| IM-09 | Repeatability | A corrected file can be re-imported. A row identical to an already-imported booking is reported as a duplicate, not created twice | Assumption |
| IM-10 | One-off | The module is used for the pilot start only; no scheduled or recurring imports | ADR-0010 |
| IM-11 | Access | Admin only | room-booking-spec.md, Privacy |
| IM-12 | Clean-up | Office Operations resolves known double bookings and typos in the spreadsheet before export | ADR-0012 (Proposed) |

## Interfaces

- **Inputs:** CSV file exported by Office Operations from the spreadsheet.
- **Depends on:** Room Booking and Availability (M-01) for validation; Room Management (M-03) for room names; People and Identity (M-04) for email matching.
- **Consumed by:** produces bookings that all other modules treat like any other booking.

## Out of Scope

Importing history, importing anything beyond the window, recurring imports, direct spreadsheet or SharePoint connections, automatic conflict resolution, creating people or rooms from the file, and export back to the spreadsheet.

## Related ADRs

ADR-0004, ADR-0010, ADR-0011 (Proposed), ADR-0012 (Proposed), ADR-0014.

## Open Items

- [ ] Client to confirm the CSV format and rejection behaviour (ADR-0011) and the clean-up ownership (ADR-0012).
- [ ] Go-live date is not fixed (ADR-0001). The window is defined relative to it, so the import must be run on or just before that day.
- [ ] "Working day" is taken as Monday to Friday with no public-holiday calendar; if the go-live date falls on a weekend, confirm whether the window begins on the next Monday. **Resolved by ADR-0015:** if go-live falls on a weekend, the 5-working-day window starts the following Monday.
- [ ] If the import runs after the start of the go-live day, earlier rows on that day are rejected as "past". Confirm this is acceptable, or that the import runs before opening time.
- [ ] Does the 3-booking limit count bookings already in the tool when the import runs (for example added manually in advance)? Assumed yes, since IM-04 refers to the "upcoming" count.
- [ ] Duplicate detection (IM-09) is an assumption; confirm whether re-import is expected to replace or skip.
