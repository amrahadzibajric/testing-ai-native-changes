# ADR-0005: No Real Login in the Pilot (Pick a User)

**Date:** 2026-09-29
**Status:** Accepted
**Deciders:** Office Operations (client stakeholder)

## Source

**Type:** Document, confirmed by Stakeholder Interview
**Reference:** room-booking-spec.md, "Not needed for now"; client-qa-2026-09-29.md, Answer 9
**Participants / Authors:** Office Operations; recorded by Amra Hadzibajric

---

## Context

The request states proper login is not needed and that a user can be picked for now *(Source: room-booking-spec.md, Not needed for now)*. The client confirmed login remains out of scope for the pilot *(Source: client-qa-2026-09-29.md, Answer 9)*. The request also requires privacy: employees see only their own bookings and only admins manage rooms *(Source: room-booking-spec.md, Privacy)*.

## Decision Drivers

- Keep pilot scope small.
- Privacy rules still need a notion of the current user and their role.

## Options Considered

### Option A: "Pick a user" selection
- **Pros:** Fastest; matches the request.
- **Cons:** No real security; anyone can act as anyone (including an admin).

### Option B: Real authentication in the pilot
- **Pros:** Enforces privacy.
- **Cons:** Explicitly not needed; added scope.

## Decision

**We will use a "pick a user" mechanism instead of authentication in the pilot.**

The application must still behave according to the selected user's role, so privacy and admin rules are enforced against the selected identity.

## Consequences

### Positive
- Minimal pilot scope.

### Negative / Trade-offs
- Privacy and admin restrictions are not secure against a determined user; acceptable for a trusted pilot group only.

### Risks / Follow-up Actions
- [ ] Record this limitation in the NFRs and security assessment.
- [ ] Ensure the pilot deployment is not exposed beyond the trusted user group (architecture phase).
