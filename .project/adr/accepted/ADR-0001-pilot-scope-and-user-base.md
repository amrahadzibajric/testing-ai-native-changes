# ADR-0001: First Release Is a Pilot for Up to 50 Users

**Date:** 2026-09-29
**Status:** Accepted
**Deciders:** Office Operations (client stakeholder)

## Source

**Type:** Stakeholder Interview
**Reference:** client-qa-2026-09-29.md, Answers 1 and 2
**Participants / Authors:** Client stakeholder (Office Operations); recorded by Amra Hadzibajric

---

## Context

The business request describes a small internal tool to replace a shared spreadsheet and Slack messages for booking meeting rooms *(Source: room-booking-spec.md, Background)*. It did not say whether the first release is company-wide or a trial, nor how many people would use it. The client confirmed the first version is a pilot with no fixed go-live date, and that no more than 50 users will be supported *(Source: client-qa-2026-09-29.md, Answers 1–2)*.

## Decision Drivers

- Keep the first release small and low risk while the booking rules are proven in practice.
- Avoid over-engineering for scale that is not needed.
- No committed date, so scope rather than schedule drives planning.

## Options Considered

### Option A: Pilot for a small user group
- **Pros:** Small scope, fast feedback, lets rules be tuned before wider rollout.
- **Cons:** Some capabilities (login, notifications) are deferred and will need later work.

### Option B: Company-wide tool from day one
- **Pros:** Single rollout.
- **Cons:** Requires real authentication, user provisioning and scale planning that the client has not asked for.

## Decision

**We will build the first version as a pilot supporting no more than 50 users, with no fixed go-live date.**

This matches the client's answers and keeps the pilot aligned with the "not needed for now" list in the original request.

## Consequences

### Positive
- Sizing and non-functional targets can assume a small user base (up to 50).
- No date pressure on scope decisions.

### Negative / Trade-offs
- Work needed for a company-wide rollout is deferred and will need a later decision.

### Risks / Follow-up Actions
- [ ] Confirm the criteria for moving from pilot to wider rollout when that becomes relevant.
