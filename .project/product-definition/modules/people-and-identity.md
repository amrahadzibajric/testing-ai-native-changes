---
id: M-04
name: People and Identity
phase: 1
status: defined
epic: TBD
adr-refs: [ADR-0004, ADR-0005, ADR-0006, ADR-0014]
---

# Module: People and Identity

**Last updated:** 2026-09-29

## Purpose

This module keeps the list of people who use the tool and tells the rest of the system who the current user is and what role they hold. In the pilot there is no login: a user "picks a user" from the list. The module is designed around each person's email address so that Microsoft company-account sign-in can replace the picker later without reshaping data. *(Source: room-booking-spec.md, Not needed for now; client-qa-2026-09-29.md, Answers 2 and 9; ADR-0004, ADR-0005, ADR-0006)*

## Users and Roles

- **Admin:** adds, edits, deactivates and reactivates people; assigns the role.
- **Employee:** picks their own name to use the tool; cannot manage people.

Two roles exist: **employee** and **admin**. *(Source: ADR-0004)*

## Key Capabilities

1. **Add a person** with name, email and role. Up to about 50 people. *(Source: client-qa-2026-09-29.md, Answer 2)*
2. **Edit a person** (name, role; email change treated as a controlled edit because it is the identity key).
3. **Deactivate a person.** Their future bookings are cancelled and marked "cancelled by admin", following the room-deactivation pattern. Their past bookings remain for history and reporting. *(Source: ADR-0014, rule 3)*
4. **Reactivate a person.** Cancelled bookings are not restored.
5. **Pick a user.** A start screen lists active people; choosing one makes the app behave as that person, with that role. Inactive people are not listed. *(Source: ADR-0014, rule 5)*
6. **Role-based access.** The app applies the privacy rules against the selected identity: employees see only their own bookings; only admins see everyone's bookings and manage rooms and people. *(Source: room-booking-spec.md, Privacy; ADR-0005)*
7. **Switch user.** The current user can return to the picker to act as someone else (needed for the pilot; it is also a visible symptom of the security limitation below).

## Business Rules

| # | Rule | Value | Source |
|---|------|-------|--------|
| PI-01 | Person attributes | Name, email, role (employee or admin), active flag | ADR-0004 (attributes defined here) |
| PI-02 | Email is the key | Email is the unique identity key (case-insensitive), matching future Microsoft accounts and the import column `person_email` | ADR-0006; client-qa-2026-09-29.md, Proposed import format |
| PI-03 | User cap | About 50 people at most | client-qa-2026-09-29.md, Answer 2; ADR-0001 |
| PI-04 | Picker lists active only | Inactive people cannot be picked | ADR-0014, rule 5 |
| PI-05 | Deactivation cascade | Future (not started) Confirmed bookings are cancelled as "cancelled by admin"; past and in-progress bookings are unchanged | ADR-0014, rule 3; in-progress handling is an assumption |
| PI-06 | Deactivation warning | The admin sees how many future bookings will be cancelled before confirming (recommended) | Product Strategist recommendation |
| PI-07 | At least one admin | The system refuses to deactivate or demote the last active admin (assumed protection) | Assumption |
| PI-08 | Pilot security limitation | Picking a user is not authentication. Anyone with access to the app can act as any person, including an admin. Privacy and admin restrictions are enforced against the selected identity but are not secure against a determined user. Acceptable for a trusted pilot group only | ADR-0005 |
| PI-09 | Single role per person | A person holds exactly one role at a time | Assumption |

## Microsoft-ready Design

The future sign-in provider is Microsoft company accounts. This module commits to: (a) email as the unique user key; (b) all authorisation decisions read "current person and role" from one place, so replacing the picker with sign-in changes how the current person is established, not how rules are applied. Actual sign-in, single sign-on, token handling and directory sync are deferred. *(Source: client-qa-2026-09-29.md, Answer 9; ADR-0006)*

## Interfaces

- **Depends on:** none in the pilot. Future: Microsoft identity (see integration-points.md).
- **Consumed by:** every other module (current user, role); Booking Import (M-06, matching `person_email`); Admin Booking Oversight (M-05, person filter); Booking Lifecycle (M-02, cancelled-by-admin transition).

## Out of Scope

Passwords, sign-in, single sign-on, sessions and token lifetimes, self-registration, directory synchronisation, multi-factor authentication, groups or teams, delegation of booking rights.

## Related ADRs

ADR-0004, ADR-0005, ADR-0006, ADR-0014.

## Open Items

- [ ] Deployment access control: because pick-a-user is not secure, the pilot should be reachable only by the trusted group (network restriction). To be decided in the architecture phase (ADR-0005 follow-up).
- [ ] Is a change of email address allowed after bookings exist? Assumed yes as an admin-only edit; the past bookings follow the person.
- [ ] Confirm PI-07 (last-admin protection) and PI-09 (single role) as acceptable.
- [ ] Do employees need to be told, on the picker, which role they are acting under? Assumed yes: the role is shown in the app header.
