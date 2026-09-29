# ADR-0010: One-off Import of Upcoming-week Bookings from the Spreadsheet

**Date:** 2026-09-29
**Status:** Accepted
**Deciders:** Office Operations (client stakeholder)

## Source

**Type:** Stakeholder Interview
**Reference:** client-qa-2026-09-29.md, Answer 7 and "Working assumptions carried forward"
**Participants / Authors:** Client stakeholder (Office Operations); recorded by Amra Hadzibajric

---

## Context

Bookings currently live in a shared spreadsheet *(Source: room-booking-spec.md, Background)*. The client asked to bring across only the bookings for the upcoming week rather than start fresh or migrate history *(Source: client-qa-2026-09-29.md, Answer 7)*. The spreadsheet is retired after the one-week import *(Source: client-qa-2026-09-29.md, Working assumptions)*.

## Decision Drivers

- Avoid double entry during the switch.
- Limit migration effort.

## Options Considered

### Option A: Import upcoming week only
- **Pros:** Small, matches the client.
- **Cons:** Requires a prepared file and cleanup.

### Option B: Start fresh
- **Pros:** No migration.
- **Cons:** Not chosen by the client.

### Option C: Import full history
- **Pros:** Complete record.
- **Cons:** Not requested.

## Decision

**We will provide a one-off import of bookings for the upcoming week; earlier history is not migrated.**

The file format and the ownership of cleanup are recorded separately (ADR-0011, ADR-0012, both Proposed).

## Consequences

### Positive
- Continuity for the first week.

### Negative / Trade-offs
- Import tooling is used once.

### Risks / Follow-up Actions
- [ ] Confirm the import week and go-live timing relationship, since there is no fixed go-live date (ADR-0001). The import should be run close to the actual pilot start.
