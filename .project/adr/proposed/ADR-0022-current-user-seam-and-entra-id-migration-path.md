# ADR-0022: Current-user Seam and Microsoft Entra ID Migration Path

**Date:** 2026-09-29
**Status:** Proposed
**Deciders:** Solution Architect (recommendation); project lead and client IT to confirm

## Source

**Type:** Internal Discussion
**Reference:** Phase 4 architecture design, 2026-09-29; ADR-0005, ADR-0006; people-and-identity.md "Microsoft-ready Design"; integration-microsoft-entra-id-v1.0.md
**Participants / Authors:** Solution Architect

---

## Context

The pilot uses pick-a-user (ADR-0005) but must be ready for Microsoft company-account sign-in later, with email as the person key (ADR-0006). The product definition requires that all authorisation decisions read the current person and role from one place *(Source: people-and-identity.md, Microsoft-ready Design)*. ADR-0006 left open whether Microsoft Entra ID is the exact provider.

## Decision Drivers

- Replacing the picker must change how the person is established, not how rules are applied.
- Avoid account-matching weaknesses when sign-in is added.
- No sign-in build in the pilot.

## Options Considered

### Option A: Single ICurrentUser abstraction with a pick-a-user implementation now and an Entra ID implementation later; link Entra accounts to people on first sign-in
- **Pros:** All rules depend on one interface; the swap is configuration plus one new implementation; linking on first sign-in allows matching by email once and then by the stable Entra object identifier.
- **Cons:** Requires a small amount of upfront discipline (no rule may read the picker directly).

### Option B: Build pick-a-user directly into controllers and pages
- **Pros:** Fewest lines now.
- **Cons:** The later change touches every module.

## Decision

**We propose Option A.**

The picker stores only the selected person identifier in a signed and encrypted cookie; the server always loads the person and role from the database on each request and never trusts a role sent by the browser. When Entra ID is introduced, sign-in is single-tenant to the company directory, the first sign-in matches the verified email claim to the existing person (email is unique and case-insensitive) and records the immutable Entra object identifier and tenant, and later sign-ins match on that identifier. The picker is then removed or limited to a test-only environment.

## Consequences

### Positive
- Sign-in becomes an additive change, in line with ADR-0006.
- Avoids the email-claim spoofing weakness of matching on email alone in every request.

### Negative / Trade-offs
- The person record gains one nullable field (external identity reference) later.

### Risks / Follow-up Actions
- [ ] Confirm with client IT that Microsoft Entra ID is the provider and single-tenant registration is possible (see integration assessment).
- [ ] Ensure no module reads identity outside the abstraction (checked in code review and an architecture test).

## Change Log
- 2026-09-29: ADR-0021 (Accepted) puts a tenant-only Easy Auth gate in front of the pilot. The gate identity is recorded read-only for audit and is not used for authorisation, so the seam in this ADR is unchanged. The same app registration mechanism eases the later sign-in phase. Status remains Proposed.
- 2026-09-29: The Entra ID sign-in path is deferred to post-POC by ADR-0024 (Accepted); no Entra work happens in the POC. The `ICurrentUser` seam and the pick-a-user implementation stay in the code design, so the deferral costs nothing later. Status remains Proposed.
