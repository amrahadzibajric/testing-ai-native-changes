# E-04: People and Identity

**EPIC ID:** E-04
**Delivery Phase:** Phase 1 — POC (build) — module M-04
**Dependencies:** E-02 (persistence, audit, bootstrap), E-03 (UI foundation). F-04.01 story S-04.01.03 (deactivate person) also depends on F-07.05 (cascade service in E-07).
**ADR References:** ADR-0001, ADR-0004, ADR-0005, ADR-0006, ADR-0014 (rules 3 and 5), ADR-0015 (decision 4), ADR-0022 (seam only)
**Architecture References:** arch v1.2 (People service, ICurrentUser, Authorisation policies, Pick-a-user); Identity Design; security-assessment v1.2 (SEC-02, SEC-04); data-model people-and-identity.md

## Description

This EPIC delivers module M-04: the admin-managed people directory, the pick-a-user entry screen with the `ICurrentUser` seam, and the server-side role and privacy enforcement. Before it, the application has no notion of who is acting. Afterwards a presenter can pick any active person, every request loads that person and role from the database, admins manage up to about 50 people keyed by a unique case-insensitive email, and employees are kept away from admin pages and other people's data. It also keeps the door open for Microsoft sign-in: all rules read identity from one place, so a later Entra ID implementation replaces how the person is established, not how rules apply.

## Scope Boundaries

- **Includes:** people management (add, edit, deactivate, reactivate, last-admin protection, cap message); picker page, cookie identity, switch user, stale identity handling; authorisation policies, ownership and privacy enforcement, refusal experience; audit for person and picker events.
- **Excludes:** passwords, sign-in, sessions and tokens, directory sync, groups (M-04 out of scope); the Entra ID implementation and the gate identity reader (E-14, E-16); the cascade service itself (F-07.05); the admin-only page content of other modules.

## Key Technical Components

People service, `ICurrentUser` and picker implementation, Authorisation policies, signed encrypted cookie, admin people pages, phone picker.

## Key User Roles Affected

Employee (picks own name), Admin (manages people), presenter switching identities.

## Done Criteria

- Admins can add, edit, deactivate and reactivate people; email is unique ignoring case; the last active admin cannot be deactivated or demoted.
- The picker lists active people only and the header always shows who is acting and their role.
- Every page and handler is deny-by-default with server-side role checks; an employee cannot reach admin pages or another person's bookings.
- Deactivating a person cancels their future bookings as cancelled by admin and never late-flags them.
- The pick-a-user limitation (not authentication) is documented and visible in the design of the demonstration (E2E-014).

## Features

| Feature | Description | Stories | Surfaces |
|---------|-------------|---------|----------|
| [F-04.01: People Management](../features/F-04.01-people-management.md) | Admin add, edit, deactivate, reactivate people | 5 | Web, Phone browser, Admin |
| [F-04.02: Pick-a-User and Current-User Seam](../features/F-04.02-pick-a-user-and-current-user-seam.md) | `ICurrentUser`, picker page, header, switch, stale identity, cookie hardening | 5 | Web, Phone browser |
| [F-04.03: Role-based Authorisation and Privacy Enforcement](../features/F-04.03-role-based-authorisation-and-privacy-enforcement.md) | Deny-by-default policies, ownership checks, privacy read models, refusal experience | 4 | Web, Phone browser |
