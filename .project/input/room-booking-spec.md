# Meeting Room Booking – Business Request

**From:** Office Operations
**Subject:** We need a simple way to book meeting rooms

## Background

Right now people book meeting rooms through a shared spreadsheet and a lot of Slack messages. Rooms get double booked, people hold rooms "just in case" and never show up, and nobody really knows which rooms are free. We want a small internal tool that fixes this.

## Who will use it

- **Employees** – book rooms for their meetings and manage their own bookings.
- **Office admins** – look after the rooms and help people with bookings.

## What we need

### Finding and booking a room
- People should see which rooms are free for the day and time they want.
- It should be possible to filter by how many people the room needs to fit.
- When booking, they pick the room, the date, start and end time, and how many people are coming.
- A room obviously can't be booked twice for the same time. Back-to-back meetings are fine though (one ends at 10:00, the next starts at 10:00).
- Meetings should be at least 30 minutes and no longer than 4 hours. Times should be in 15-minute blocks, like we do in calendars.
- The office is open 08:00–20:00, Monday to Friday, so bookings have to fit inside that. No bookings in the past.
- You can't book a room for more people than it fits.

### Stopping people from hogging rooms
- An employee can have a maximum of 3 upcoming bookings at a time.
- If an admin books a room for someone, it still counts toward that person's 3.
- Admins booking for themselves don't have a limit.

### Cancelling
- People can cancel their own bookings. Admins can cancel anyone's.
- If someone cancels less than 2 hours before the meeting, we want to mark it as a late cancellation so we can see who does this a lot.
- Once a meeting has started or is finished, it can't be cancelled anymore.

### Checking in
- When people arrive, they check in to the booking. They can do this from 10 minutes before the start until 15 minutes after.
- If nobody checks in within 15 minutes of the start, the booking should be marked as a no-show and the room should become free again.
- After the meeting ends, the booking is simply marked as done.

### For admins
- Add new rooms, change their name or how many people they fit.
- Take a room out of use (for example, when it's being renovated). Any future bookings for that room should be cancelled and the people who booked it should be able to see it was cancelled by an admin, not by them. That room shouldn't show up for booking anymore.
- See all bookings, and filter them by room, person or status.

### Privacy
- Employees should only see their own bookings, not what others booked.
- Only admins should be able to manage rooms or see everyone's bookings.

## Rooms we have today

| Room | Fits |
|------|------|
| Focus | 2 |
| Team | 6 |
| Board | 12 |
| Old Room | 4 (currently not in use) |

## Not needed for now

- Recurring meetings
- Email or calendar notifications
- Proper login (we can pick a user for now)
- Different time zones
- Mobile app
