# ADR-0004: Admins Add and Maintain People Manually

**Date:** 2026-09-29
**Status:** Accepted
**Deciders:** Office Operations (client stakeholder)

## Source

**Type:** Stakeholder Interview
**Reference:** client-qa-2026-09-29.md, Answer 2
**Participants / Authors:** Client stakeholder (Office Operations); recorded by Amra Hadzibajric

---

## Context

The request names employees and office admins as user groups but does not say where the list of people comes from *(Source: room-booking-spec.md, Who will use it)*. The client answered that admins will add people, with no more than 50 users *(Source: client-qa-2026-09-29.md, Answer 2)*. The CSV import also depends on people already existing in the tool *(Source: client-qa-2026-09-29.md, Proposed import format)*.

## Decision Drivers

- Small user base (up to 50).
- No directory integration requested for the pilot.

## Options Considered

### Option A: Admin-managed people list in the tool
- **Pros:** Simple, no external dependency.
- **Cons:** Manual effort for admins.

### Option B: Sync from a company directory
- **Pros:** Automatic.
- **Cons:** Not requested; adds integration work and access needs.

## Decision

**We will provide a people-management screen where admins add and maintain users, including their role (employee or admin).**

## Consequences

### Positive
- No dependency on the company directory during the pilot.

### Negative / Trade-offs
- Manual upkeep as people join or leave.

### Risks / Follow-up Actions
- [ ] Define the person attributes (name, email, role, active flag) in the product definition.
- [ ] Decide how deactivated people's existing bookings are treated (product definition phase).
