# ADR-0003: Web Application with Check-in from the User's Own Device

**Date:** 2026-09-29
**Status:** Accepted
**Deciders:** Office Operations (client stakeholder)

## Source

**Type:** Stakeholder Interview
**Reference:** client-qa-2026-09-29.md, Answer 5 and "Working assumptions carried forward"
**Participants / Authors:** Client stakeholder (Office Operations); recorded by Amra Hadzibajric

---

## Context

The request requires people to check in on arrival, from 10 minutes before the start until 15 minutes after *(Source: room-booking-spec.md, Checking in)*, and lists a mobile app as not needed *(Source: room-booking-spec.md, Not needed for now)*. It did not say where check-in happens. The client confirmed check-in is done from the user's own laptop or phone, with no devices at the rooms *(Source: client-qa-2026-09-29.md, Answer 5)*. The working assumption, not contradicted by the client, is a web app usable in desktop and phone browsers *(Source: client-qa-2026-09-29.md, Working assumptions)*.

## Decision Drivers

- No room-side hardware.
- Native mobile app is explicitly not needed.
- Phone usage is still required for check-in.

## Options Considered

### Option A: Responsive web app
- **Pros:** One codebase, works on laptop and phone, no app-store distribution.
- **Cons:** No native device features (not needed).

### Option B: Shared tablet or kiosk at each room
- **Pros:** Presence at the room.
- **Cons:** Rejected by the client; extra hardware.

### Option C: Native mobile app
- **Pros:** Native experience.
- **Cons:** Listed as not needed.

## Decision

**We will deliver a web application for desktop and phone browsers, and users will check in from their own laptop or phone.**

The web-app format is a working assumption carried forward and not contradicted by the client.

## Consequences

### Positive
- No hardware procurement or installation.

### Negative / Trade-offs
- Check-in cannot verify physical presence at the room.

### Risks / Follow-up Actions
- [ ] Design screens for phone-sized browsers.
