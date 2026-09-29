# ADR-0020: Booking Concurrency Control: Per-room Lock with Transactional Overlap Check

**Date:** 2026-09-29
**Status:** Proposed
**Deciders:** Solution Architect (recommendation); project lead to confirm

## Source

**Type:** Internal Discussion
**Reference:** Phase 4 architecture design, 2026-09-29; nfrs.md NFR-005; room-booking-and-availability.md BR-06; booking-import.md IM-03, IM-05
**Participants / Authors:** Solution Architect

---

## Context

A room must never be double booked, including when two users submit at nearly the same moment; the second request must be refused with a clear message *(Source: nfrs.md, NFR-005)*. The same rule applies to edits, remainder rebooking and every import row. Azure SQL has no declarative time-range exclusion constraint.

## Decision Drivers

- Guaranteed correctness at the data layer, not just in application code.
- Simplicity at low contention (50 users, three active rooms).

## Options Considered

### Option A: Per-room row lock (UPDLOCK on the room row) inside a transaction, then overlap check, then insert
- **Pros:** Simple; serialises writes per room only; correct under concurrency; easy to test.
- **Cons:** Correctness lives in the transaction discipline; every write path must use the same code.

### Option B: Slot table with a unique index on (room, 15-minute slot)
- **Pros:** The database itself enforces no-overlap; strong defence in depth.
- **Cons:** More rows and more bookkeeping on edit, cancel and no-show release.

### Option C: Serializable isolation on the overlap query
- **Pros:** No explicit lock statement.
- **Cons:** Deadlock-prone under contention; harder to reason about.

## Decision

**We propose Option A, implemented in a single booking-write service used by manual booking, edit, remainder rebooking and import.**

All writes that create or move a booking acquire the room lock (both rooms when an edit changes room, in a fixed order to avoid deadlocks), re-check every rule against current data and commit. A concurrency integration test is required. Option B remains available as an added safeguard if testing shows gaps.

## Consequences

### Positive
- NFR-005 is met with minimal machinery.
- Import and manual booking share one enforcement path.

### Negative / Trade-offs
- Only as strong as the discipline of using the single write service; no database-level backstop in Option A.

### Risks / Follow-up Actions
- [ ] Parallel-request integration test proving exactly one of N simultaneous identical requests succeeds.
- [ ] Re-evaluate Option B if additional write paths appear.

## Change Log
- 2026-09-29: For the local POC on SQLite (ADR-0025, ADR-0024) the single booking-write service is kept, but the guarantee comes from SQLite's single-writer serialisation: the overlap check and insert run in one immediate write transaction. The UPDLOCK per-room lock described here applies when the application moves to Azure SQL. Status remains Proposed.
