# ADR-0006: Design User Identity Around Email for Future Microsoft Sign-in

**Date:** 2026-09-29
**Status:** Accepted
**Deciders:** Office Operations (client stakeholder)

## Source

**Type:** Stakeholder Interview
**Reference:** client-qa-2026-09-29.md, Answer 9 and "Proposed import format" (email-as-key note)
**Participants / Authors:** Client stakeholder (Office Operations); recorded by Amra Hadzibajric

---

## Context

Login is out of scope for the pilot (ADR-0005), but the client stated the likely future sign-in provider is Microsoft, using company Microsoft accounts *(Source: client-qa-2026-09-29.md, Answer 9)*. The delivery team noted that email is the person key because it will match the future Microsoft sign-in *(Source: client-qa-2026-09-29.md, Proposed import format)*.

## Decision Drivers

- Avoid rework of user records when sign-in is introduced.
- Email is unique per person and present in Microsoft accounts.

## Options Considered

### Option A: Email as the unique person identifier
- **Pros:** Maps directly to Microsoft accounts; also used by the CSV import.
- **Cons:** Email changes need handling.

### Option B: Internal numeric ID only, no email
- **Pros:** Stable.
- **Cons:** Requires a later mapping step to Microsoft accounts and cannot be used by the import.

## Decision

**We will store each person with a unique email address as their key, so that Microsoft sign-in can be added later without changing user records.**

Microsoft sign-in itself is not built in the pilot.

## Consequences

### Positive
- Later Microsoft sign-in becomes an authentication addition, not a data migration.

### Negative / Trade-offs
- Email uniqueness must be enforced when admins add people.

### Risks / Follow-up Actions
- [ ] Confirm during architecture whether Microsoft Entra ID is the exact provider (client said "Microsoft company accounts").
