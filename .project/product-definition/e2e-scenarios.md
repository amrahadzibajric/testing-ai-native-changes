# End-to-End Scenarios

**Last updated:** 2026-09-29
**Source:** room-booking-spec.md; client-qa-2026-09-29.md; ADR-0001 to ADR-0014; module files M-01 to M-06

Primary input for the QA agent's E2E test design. Each scenario describes a business workflow that must succeed end to end. Rule values are defined in the module files. Scenarios that rely on a Proposed ADR (0011 to 0013) are marked *pending client confirmation*.

Module IDs: M-01 Room Booking and Availability; M-02 Booking Lifecycle; M-03 Room Management; M-04 People and Identity; M-05 Admin Booking Oversight and Reporting; M-06 Booking Import.

---

## E2E-001: Employee finds and books a room

**Modules involved:** M-04, M-01
**Trigger:** An employee needs a room for 5 people next Tuesday.
**Preconditions:** Rooms Focus (2), Team (6), Board (12) active; employee is an active person with fewer than 3 upcoming bookings.

### Flow
1. Employee picks their name on the pick-a-user screen (only active people listed).
2. Employee searches Tuesday 10:00 to 11:30 for 5 people.
3. Availability shows Team and Board as free; Focus is excluded because it fits 2; busy rooms show no booker name.
4. Employee books Team for 10:00 to 11:30 with 5 attendees.

### End State
A Confirmed booking exists; it appears in the employee's My Bookings; Team shows busy for that period for everyone. The employee's upcoming count rises by 1.

### Failure Paths
- Attendees 7 for Team: refused, capacity 6.
- Duration 20 minutes or 4 h 15 min: refused, outside 30 min to 4 h.
- Start 10:10 or end 19:50: refused, not on the 15-minute grid.
- Ends after 20:00, or Saturday: refused, outside office hours.
- Start in the past: refused.
- Another user books the same room and time first: the second request is refused, no double booking.

### Source
room-booking-spec.md, Finding and booking a room; NFR-005.

---

## E2E-002: Back-to-back bookings and overlap boundary

**Modules involved:** M-01
**Trigger:** Two employees book the same room consecutively.
**Preconditions:** Team is free 09:00 to 12:00.

### Flow
1. Employee A books Team 09:00 to 10:00.
2. Employee B books Team 10:00 to 11:00.
3. Employee C tries Team 09:45 to 10:15.

### End State
A and B hold confirmed bookings; C is refused for overlap.

### Failure Paths
- C's overlapping request states the room is not free at that time.

### Source
room-booking-spec.md, Finding and booking a room.

---

## E2E-003: Three-booking limit, edit and cancellation

**Modules involved:** M-01, M-02
**Trigger:** An employee with 3 upcoming bookings tries to book another.
**Preconditions:** Employee E has exactly 3 Confirmed, not-started bookings.

### Flow
1. E tries to book a fourth room: refused, limit of 3 reached.
2. E edits one existing booking to a different time: accepted (all rules re-run); the count stays at 3 because an edit is not a new booking.
3. E cancels one booking more than 2 hours before start: not flagged late; count drops to 2.
4. E books a new room: accepted; count is 3.

### End State
E holds 3 upcoming bookings; one cancellation exists, not flagged late.

### Failure Paths
- The edit results in an overlap or off-grid time: refused, original booking unchanged.
- A booking that has started is not counted as upcoming.

### Source
room-booking-spec.md, Stopping people from hogging rooms; ADR-0007; ADR-0014, rule 2.

---

## E2E-004: Admin books for others, and for self

**Modules involved:** M-04, M-01
**Trigger:** An admin helps a colleague and books for themselves.
**Preconditions:** Colleague P has 2 upcoming bookings. Admin A is active.

### Flow
1. A (acting as admin) books Board for P. It is accepted; P now has 3.
2. A tries a second booking for P: refused, P is at the limit of 3.
3. A books four rooms for A on different days: all accepted, no limit.
4. A tries a 5-hour booking for A: refused, time rules still apply to admins.

### End State
P has 3 upcoming bookings; A has four.

### Source
room-booking-spec.md, Stopping people from hogging rooms; client-qa-2026-09-29.md, Working assumptions.

---

## E2E-005: Check in and completion

**Modules involved:** M-01, M-02
**Trigger:** The booker arrives for a meeting.
**Preconditions:** Confirmed booking Team 14:00 to 15:00 today.

### Flow
1. At 13:45 the booker opens the app on their phone and checks in (10 minutes before start is the earliest: 13:50, so 13:45 is refused; the booker retries at 13:50).
2. The booking status becomes Checked in.
3. At 15:00 the system marks the booking Done.

### End State
Booking status Done; visible in history.

### Failure Paths
- Check-in attempt at 14:16 (more than 15 minutes after start): refused, and the booking is already No-show (E2E-006).
- Employee tries to check in to someone else's booking: refused (assumed; see M-02 open items).

### Source
room-booking-spec.md, Checking in; client-qa-2026-09-29.md, Answer 5.

---

## E2E-006: No-show releases the room and the remainder is rebooked

**Modules involved:** M-02, M-01
**Trigger:** Nobody checks in within 15 minutes of the start.
**Preconditions:** Confirmed booking Team 10:00 to 11:30. *Pending client confirmation of the 30-minute minimum (ADR-0013).*

### Flow
1. No check-in is recorded by 10:15. The system marks the booking No-show and frees the room.
2. The booking appears in the admin no-show view (E2E-009).
3. At 10:20 another employee looks up Team and books 10:30 to 11:30 (start on the 15-minute grid).

### End State
Original booking is No-show; a new Confirmed booking holds the remainder.

### Failure Paths
- Start 10:20: refused, not on the grid.
- Start 10:45, end 11:15 is allowed (30 minutes); a booking 11:15 to 11:30 is refused because it is under 30 minutes (ADR-0013, Proposed).
- The original booker checks in at 10:16: refused.

### Source
room-booking-spec.md, Checking in; client-qa-2026-09-29.md, Answers 4 and 11; ADR-0008; ADR-0013.

---

## E2E-007: Late and timely cancellation

**Modules involved:** M-02, M-05
**Trigger:** Employees cancel bookings at different times before the start.
**Preconditions:** Bookings start today at 15:00.

### Flow
1. Employee X cancels at 12:30 (2.5 hours before): cancelled normally, no flag.
2. Employee Y cancels at 13:30 (1.5 hours before): cancelled and flagged as a late cancellation.
3. Employee Z edits the booking at 14:55 to a later time: accepted, not flagged (edits are never late).
4. An admin opens the late-cancellation view and sees Y's cancellation.

### End State
Two cancellations, one flagged; no automatic consequence for Y.

### Failure Paths
- A cancellation attempt after 15:00 (started) or after the end: refused.
- Exactly 2 hours before: not flagged (M-02 LC-06; boundary to confirm).

### Source
room-booking-spec.md, Cancelling; client-qa-2026-09-29.md, Answers 6 and 10; ADR-0007; ADR-0009.

---

## E2E-008: Started bookings cannot be edited or cancelled

**Modules involved:** M-02
**Trigger:** A user tries to change a booking after it starts.
**Preconditions:** Confirmed booking today 10:00 to 11:00, no check-in; current time 10:05.

### Flow
1. The owner tries to edit the end time to 11:30: refused, booking has started.
2. The owner tries to cancel: refused, booking has started.
3. After check-in the same attempts are also refused.

### End State
Booking unchanged.

### Source
room-booking-spec.md, Cancelling; ADR-0014, rule 1.

---

## E2E-009: Admin oversight of bookings and behaviour

**Modules involved:** M-05, M-02, M-04
**Trigger:** An admin reviews the week.
**Preconditions:** Mixed bookings exist: Confirmed, Done, No-show, Cancelled (some late), Cancelled by admin.

### Flow
1. Admin opens all bookings and filters by room Team, then by person, then by status No-show.
2. Admin opens the late-cancellation view and sees counts per person.
3. Admin cancels a booking from the list on behalf of an employee.
4. Employee views their own bookings: sees only their own, and sees the admin origin on a booking cancelled by admin.

### End State
Admin has full visibility; employees see only their own data; no automatic penalties were applied.

### Failure Paths
- An employee opens the admin bookings screen or requests another person's bookings: refused.

### Source
room-booking-spec.md, For admins; Privacy; ADR-0009.

---

## E2E-010: Take a room out of use

**Modules involved:** M-03, M-02, M-01, M-05
**Trigger:** Team is going into renovation.
**Preconditions:** Team has three future bookings for different people, plus one booking in progress.

### Flow
1. Admin deactivates Team, after seeing that three bookings will be cancelled.
2. The three future bookings become Cancelled by admin; the in-progress one is untouched.
3. Team no longer appears in availability or booking.
4. Affected employees see their bookings cancelled by an admin, not by themselves, and the cancellations are not late-flagged.
5. Later, the admin reactivates Team; it is bookable again; cancelled bookings stay cancelled.

### End State
Team bookable only after reactivation; each affected user's upcoming count drops accordingly.

### Source
room-booking-spec.md, For admins; client-qa-2026-09-29.md, Working assumptions.

---

## E2E-011: Add a room and change capacity

**Modules involved:** M-03, M-01
**Trigger:** A new small room opens and Board is reconfigured.
**Preconditions:** Admin logged in via picker.

### Flow
1. Admin adds "Huddle" with capacity 3; it appears in availability for 3 or fewer people.
2. Admin lowers Board from 12 to 8. An existing booking with 10 attendees stays valid.
3. An employee edits that booking (say, changing the time): refused because 10 exceeds the new capacity of 8, until attendees are reduced.
4. A new booking for 10 people in Board is refused.

### End State
Capacity rules apply to new and edited bookings only.

### Source
room-booking-spec.md, For admins; client-qa-2026-09-29.md, Working assumptions.

---

## E2E-012: People management and deactivation

**Modules involved:** M-04, M-02, M-01
**Trigger:** An employee leaves the company.
**Preconditions:** Person J is active with 2 future bookings and 1 past booking.

### Flow
1. Admin deactivates J after seeing that 2 future bookings will be cancelled.
2. J's 2 future bookings become Cancelled by admin, freeing the rooms; the past booking remains.
3. J no longer appears on the pick-a-user screen.
4. Admin filters bookings by person J and sees the history.
5. Admin tries to deactivate the only remaining admin: refused (assumed protection).

### End State
J has no upcoming bookings; history intact.

### Source
ADR-0004; ADR-0014, rules 3 and 5.

---

## E2E-013: First-week import at go-live

**Modules involved:** M-06, M-03, M-04, M-01
**Trigger:** The pilot goes live; the spreadsheet is retired. *Pending client confirmation (ADR-0011, ADR-0012).*
**Preconditions:** Rooms and people already created; Office Operations has cleaned and exported a CSV; go-live date is Monday.

### Flow
1. Admin uploads the CSV covering Monday to Friday of go-live week.
2. The tool processes rows per person in date/time order and applies all booking rules.
3. Valid rows become Confirmed bookings. Rejected rows appear in the report with reason: unknown room, unknown person, date outside the 5-working-day window, off-grid time, wrong duration, capacity exceeded, overlap with an earlier row, or beyond the person's 3rd upcoming booking.
4. Office Operations fixes the rejected rows and the admin re-imports; already imported rows are not duplicated.

### End State
The first week's valid bookings exist; every rejection is accounted for in the report; nothing was changed silently.

### Failure Paths
- A person has 5 rows in the window: the first 3 in date/time order are imported, rows 4 and 5 are rejected.
- An unreadable file (wrong header): whole file refused with a message.

### Source
client-qa-2026-09-29.md, Answers 7 and 12, Proposed import format; ADR-0010; ADR-0014, rules 2 and 4.

---

## E2E-014: Privacy and role enforcement in the pilot

**Modules involved:** M-04, M-01, M-05
**Trigger:** Users switch identities.
**Preconditions:** Several active people exist.

### Flow
1. An employee sees availability with busy rooms but not who booked them.
2. The employee's booking list contains only their own bookings.
3. The employee cannot open room management, people management, import or the all-bookings view.
4. The same user switches to an admin identity through the picker and gains access.

### End State
Rules follow the selected identity. The last step demonstrates the accepted pilot limitation (ADR-0005): the picker is not a security control.

### Source
room-booking-spec.md, Privacy; ADR-0005.

---

*Maintained by the Product Strategist. Consumed by QA for E2E test design.*
