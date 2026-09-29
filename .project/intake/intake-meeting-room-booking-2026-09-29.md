# Intake Record: Meeting Room Booking

**Date:** 2026-09-29
**Project slug:** meeting-room-booking
**Client / organisation:** Office Operations
**Prepared by:** Product Strategist (recorded with Amra Hadzibajric)
**Sources read:** room-booking-spec.md (formal business request), client-qa-2026-09-29.md (client Q&A, follow-up answers, proposed import format, working assumptions)

## 1. Project Fundamentals

1. **Project and client:** Meeting Room Booking for Office Operations. *(Source: room-booking-spec.md)*
2. **Problem and audience:** Rooms are booked through a shared spreadsheet and Slack messages. Rooms get double booked, people hold rooms "just in case" and never show up, and nobody knows which rooms are free. The tool serves employees, who book and manage their own bookings, and office admins, who look after rooms and help with bookings. *(Source: room-booking-spec.md, Background; Who will use it)*
3. **New or replacement:** Greenfield build that replaces the spreadsheet process; the spreadsheet is retired after a one-week import. *(Source: client-qa-2026-09-29.md, Working assumptions)*
4. **Delivery format:** Web application for desktop and phone browsers, with check-in from the user's own device. Native mobile app is not needed. *(Sources: client-qa-2026-09-29.md, Answer 5, Working assumptions; room-booking-spec.md, Not needed for now)*
5. **Personas:** Employees (up to about 50 users in total) and office admins. Admins are added by admins; users are managed through a people-management screen. *(Source: client-qa-2026-09-29.md, Answer 2)*

## 2. Scope and Constraints

**Stage and timeline.** The first version is a pilot with no fixed go-live date. *(Source: client-qa-2026-09-29.md, Answer 1)*

**Technology and vendor constraints.** Azure resources can be used. Future sign-in provider is expected to be Microsoft company accounts; login itself is out of scope for the pilot, which uses a "pick a user" mechanism. Identity should be designed around email so Microsoft sign-in can be added later. *(Sources: client-qa-2026-09-29.md, Answers 8–9; room-booking-spec.md, Not needed for now)*

**Regulatory, compliance and security.** No regulatory requirements are stated. Privacy rules are stated: employees see only their own bookings; only admins manage rooms or see everyone's bookings. Because the pilot has no real login, these rules are not securely enforceable and are acceptable only for a trusted pilot group. *(Source: room-booking-spec.md, Privacy; ADR-0005)*

**In scope (as stated by sources).**
- Room availability search by date, time and required capacity.
- Booking, editing and cancelling by employees; admins can act for anyone.
- Booking rules: 30 min to 4 h, 15-minute blocks, 08:00–20:00 Monday to Friday, no past bookings, no overlaps (back-to-back allowed), capacity limit, and a maximum of 3 upcoming bookings per employee (admin bookings on behalf of someone count; admins booking for themselves are unlimited).
- Late-cancellation flag (cancellation under 2 hours before start), no cancellation once started.
- Check-in from 10 minutes before to 15 minutes after start; automatic no-show and room release; automatic done marking; rebooking of the freed remainder.
- Admin functions: room management (add, rename, change capacity, take out of use and reactivate, cancel future bookings for a removed room with the cancellation shown as admin-made), view and filter all bookings by room, person or status, and people management.
- One-off CSV import of upcoming-week bookings with a rejection report.

**Out of scope for now.** Recurring meetings, email or calendar notifications, proper login, multiple time zones, native mobile app, public-holiday calendar, automatic penalties. *(Sources: room-booking-spec.md, Not needed for now; client-qa-2026-09-29.md, Answer 6, Working assumptions)*

**Rooms.** Focus (2), Team (6), Board (12), Old Room (4, not currently in use). *(Source: room-booking-spec.md)*

**Milestones.** None stated; no fixed go-live date.

## 3. Decisions Confirmed (ADRs Accepted)

| ADR | Decision |
|-----|----------|
| ADR-0001 | Pilot for up to 50 users, no fixed go-live date |
| ADR-0002 | Host on Azure |
| ADR-0003 | Web app; check-in from own laptop or phone |
| ADR-0004 | Admins add and maintain people manually |
| ADR-0005 | No real login in the pilot ("pick a user") |
| ADR-0006 | Identity keyed on email for future Microsoft sign-in |
| ADR-0007 | Employees can edit bookings; edits re-run all rules and are never flagged late |
| ADR-0008 | No-show remainder rebookable, starts on the 15-minute grid |
| ADR-0009 | Late cancellations and no-shows are admin review only |
| ADR-0010 | One-off import of upcoming-week bookings |

## 4. Items Pending Client Confirmation (ADRs Proposed)

| ADR | Item |
|-----|------|
| ADR-0011 | CSV import format and rule-enforcing behaviour with rejection report |
| ADR-0012 | Office Operations owns spreadsheet clean-up before export |
| ADR-0013 | 30-minute minimum applies to a rebooked no-show remainder |

These are proposals or working assumptions that the client has not contradicted. Downstream phases may proceed on them; they should be confirmed before the import is built and before the rules are finalised.

## 5. Conflicts

None found. The original request lists a "mobile app" as not needed, while the Q&A assumes a web app usable on phone browsers; these are compatible, since one is a native app and the other a browser experience. The Q&A answers add detail to the original request (editing, remainder rebooking, hosting, import) and supersede its silence on those points without contradicting it.

## 6. Gaps and Open Questions

No information gap was found that blocks downstream work, so no Open ADRs were created. The following detailed points are noted for resolution in the product definition phase, using the working assumptions listed in the sources or standard defaults:

- Whether a booking that has already started can be edited (assumed no, consistent with the cancel rule).
- How the 3-booking limit is evaluated when an edit moves a booking or when the import loads multiple rows for one person.
- Treatment of a deactivated person's future bookings (people removal behaviour).
- Which week the import covers and when it runs relative to pilot start, given there is no fixed go-live date.
- Whether the "pick a user" mechanism lists all people or only those flagged active.

### Resolution note (2026-09-29, Phase 3)

The project lead (Amra Hadzibajric) confirmed rules for all five points above in a working session on 2026-09-29. They are recorded in ADR-0014 (Accepted). Each item is now resolved:

- **Editing a started booking:** RESOLVED. Not allowed, same as the cancellation rule (ADR-0014, rule 1).
- **Edits and imports against the 3-booking limit:** RESOLVED. An edit does not count as a new booking; on import, rows are processed in date/time order per person and rows beyond the 3rd upcoming booking are rejected into the report (ADR-0014, rule 2).
- **Deactivated person's future bookings:** RESOLVED. Cancelled and marked "cancelled by admin" (ADR-0014, rule 3).
- **Import week:** RESOLVED. The 5 working days starting on the go-live date (ADR-0014, rule 4).
- **"Pick a user" list:** RESOLVED. Active people only (ADR-0014, rule 5).

These were confirmed by the project lead rather than the client stakeholder and should be shown to Office Operations. Residual detail points are carried as open items in the product definition modules.

## 7. Working Assumptions Carried Forward

From the client Q&A, not contradicted by the client: single office and time zone; availability shows a room as busy without revealing who booked it; "upcoming" means confirmed bookings not yet started; no-show, release and done marking are automatic; the booker checks in and attendees are a head count only; admins follow the same time rules as employees, with only the 3-booking limit waived for their own bookings; rooms taken out of use can be reactivated; lowering capacity leaves existing bookings unchanged; no public-holiday calendar. *(Source: client-qa-2026-09-29.md, Working assumptions)*

## 8. Actions Completed in This Session

- Intake record written (this file).
- Project context populated: .project/context/project-context.md.
- ADR-0001 to ADR-0010 created as Accepted; ADR-0011 to ADR-0013 created as Proposed.

## 9. Recommended Next Step

Proceed to Phase 3 (product definition) when instructed. Recommend obtaining client confirmation of ADR-0011 to ADR-0013 in parallel.
