# ADR-0002: Host the Solution on Azure

**Date:** 2026-09-29
**Status:** Accepted
**Deciders:** Office Operations (client stakeholder); client IT (as source of the constraint)

## Source

**Type:** Stakeholder Interview
**Reference:** client-qa-2026-09-29.md, Answer 8
**Participants / Authors:** Client stakeholder (Office Operations); recorded by Amra Hadzibajric

---

## Context

The original request contains no hosting or technology constraints *(Source: room-booking-spec.md)*. When asked about hosting constraints from IT, the client answered that Azure resources can be used *(Source: client-qa-2026-09-29.md, Answer 8)*. The same client expects Microsoft accounts to be the future sign-in provider *(Source: client-qa-2026-09-29.md, Answer 9)*.

## Decision Drivers

- Client IT makes Azure resources available.
- Fit with the client's Microsoft-based environment and the future Microsoft sign-in.

## Options Considered

### Option A: Azure
- **Pros:** Available to the client, aligned with Microsoft identity, no new vendor onboarding.
- **Cons:** Ties the deployment to one cloud provider.

### Option B: Other cloud or on-premises hosting
- **Pros:** None identified from the sources.
- **Cons:** Not offered by the client; would need new approval.

## Decision

**We will host the solution on Azure.**

Specific Azure services are not decided here; they belong to the architecture phase.

## Consequences

### Positive
- Hosting is unblocked without further IT negotiation.

### Negative / Trade-offs
- Cloud portability is not a goal for the pilot.

### Risks / Follow-up Actions
- [ ] Architecture phase to select Azure services and confirm subscription/access details with client IT.
