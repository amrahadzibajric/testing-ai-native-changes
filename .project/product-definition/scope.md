# Project Scope

**Last updated:** 2026-09-29
**Source:** room-booking-spec.md; client-qa-2026-09-29.md; intake-meeting-room-booking-2026-09-29.md; ADR-0001 to ADR-0014

The project is a pilot of a small internal meeting room booking tool for Office Operations, for up to about 50 users, hosted on Azure, replacing a shared spreadsheet and Slack messages. Items marked (pending) rely on Proposed ADRs 0011 to 0013 and are not yet confirmed by the client.

## POC versus post-POC (added 2026-09-29)

*(Source: project lead, working session 2026-09-29; ADR-0024, ADR-0025)*

The current phase is a **local proof of concept**: the application is built to run on a developer machine (localhost, SQLite, synthetic data) and is demonstrated to the client by the delivery team. **All functional scope below is in the POC**: modules M-01 to M-06, including the full M-06 Booking Import (ADR-0011 CSV format, per-row rule validation, rejected-rows report, five-working-day window rules), tested with a synthetic or anonymised CSV sample. Rooms are seeded exactly as in the specification (Focus 2, Team 6, Board 12 active; Old Room 4 inactive; no other rooms). The only exclusion is infrastructure.

Deferred to the post-POC phase (not cancelled): Azure hosting and the EU-region and privacy-note operational items, the tenant access gate, Azure DevOps CI/CD pipelines, infrastructure as code, Application Insights, managed identity, and the Microsoft Entra ID sign-in path (the current-user seam stays in the code design). Where the sections below say "hosted on Azure", read it as the target for the pilot after the POC. Assumption to confirm: the client does not need to use the POC remotely.

## In Scope

**Booking (M-01, M-02)**
- Availability search by date, time and minimum capacity; availability shown without revealing who booked. *(room-booking-spec.md; client-qa-2026-09-29.md, Working assumptions)*
- Creating bookings with room, date, start, end and attendee count, subject to: 30 minutes to 4 hours; 15-minute grid; 08:00 to 20:00 Monday to Friday; no past bookings; no overlap (back-to-back allowed); attendees within capacity.
- Maximum 3 upcoming bookings per employee; admin bookings on behalf of someone count; admins booking for themselves are unlimited.
- Editing bookings that have not started (all rules re-run, never flagged late, does not count as a new booking). *(ADR-0007, ADR-0014)*
- Cancelling bookings that have not started; late cancellation flag when less than 2 hours before start.
- Check-in from the user's own laptop or phone, from 10 minutes before to 15 minutes after the start. *(ADR-0003)*
- Automatic no-show marking and room release; rebooking of the freed remainder on the 15-minute grid (30-minute minimum pending, ADR-0013); automatic Done marking. *(ADR-0008)*

**Administration (M-03, M-04, M-05)**
- Room management: add, rename, change capacity, deactivate and reactivate; deactivation cancels future bookings as "cancelled by admin".
- People management: add, edit, deactivate and reactivate people with role (employee or admin); deactivation cancels future bookings as "cancelled by admin". *(ADR-0004, ADR-0014)*
- Pick-a-user selection listing active people only; role-based screens and privacy rules against the selected identity. *(ADR-0005, ADR-0014)*
- Email as the unique person key so Microsoft sign-in can be added later. *(ADR-0006)*
- Admin view of all bookings with filters by room, person and status; late cancellation and no-show review. *(ADR-0009)*

**Migration (M-06)**
- One-off CSV import of bookings for the 5 working days starting on the go-live date, with a rejection report (pending, ADR-0011, ADR-0012). *(ADR-0010, ADR-0014)*

**Delivery**
- Web application usable in desktop and phone browsers; hosted on Azure. *(ADR-0002, ADR-0003)*
- Initial data: rooms Focus (2), Team (6), Board (12) active; Old Room (4) inactive.

## Out of Scope

- Recurring meetings. *(room-booking-spec.md, Not needed for now)*
- Email or calendar notifications (no email, calendar or Slack integration). *(room-booking-spec.md)*
- Proper login and authentication in the pilot. *(ADR-0005)*
- Multiple time zones. *(room-booking-spec.md)*
- Native mobile app. *(room-booking-spec.md)*
- Automatic penalties for late cancellations or no-shows. *(ADR-0009)*
- Room-side check-in devices or displays. *(client-qa-2026-09-29.md, Answer 5)*
- Public holiday calendar. *(client-qa-2026-09-29.md, Working assumptions)*
- Importing booking history or anything beyond the import window. *(ADR-0010)*
- Directory synchronisation of people. *(ADR-0004)*
- Named attendee lists (attendees are a head count only).
- Multiple offices or buildings.

## Deferred (possible future phase)

- Microsoft company-account sign-in (P2), replacing pick-a-user. *(client-qa-2026-09-29.md, Answer 9; ADR-0006)*
- Enforced security for privacy and admin restrictions (arrives with real sign-in). *(ADR-0005)*
- Email and calendar notifications (for example reminders and cancellation notices).
- Recurring bookings.
- Export and analytics of utilisation, late cancellations and no-shows.
- Public holiday handling.
- Going from pilot to company-wide use (more than 50 users, fixed rollout date). *(ADR-0001)*

## Pending Client Confirmation

| Item | ADR | Affects |
|------|-----|---------|
| CSV format and rule-enforcing import with rejection report | ADR-0011 (Proposed) | M-06 |
| Office Operations owns spreadsheet clean-up | ADR-0012 (Proposed) | M-06 |
| 30-minute minimum applies to rebooked no-show remainder | ADR-0013 (Proposed) | M-01, M-02 |
| Five rules recorded in ADR-0014 were confirmed by the project lead; to be shown to Office Operations | ADR-0014 (Accepted) | M-02, M-04, M-06 |

---

*Maintained by the Product Strategist. All scope changes go here first — then the proposal is recomposed.*
