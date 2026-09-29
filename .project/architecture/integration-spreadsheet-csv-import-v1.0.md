# Integration Assessment: Spreadsheet CSV Import (Office Operations)

**Version:** 1.0
**Date:** 2026-09-29
**Status:** Draft
**Assessed by:** Solution Architect

## System Overview

| Field | Value |
|-------|-------|
| System | Existing shared booking spreadsheet, exported as a CSV file by Office Operations |
| Role in project | One-off source of bookings for the 5 working days from the go-live date *(Sources: client-qa-2026-09-29.md, Answers 7 and 12; ADR-0010, ADR-0014, ADR-0015)* |
| Owner | Client (Office Operations) |
| Current maturity | Production spreadsheet; the import module is Planned. Format is Proposed (ADR-0011), not yet confirmed by the client |
| Documentation | Partial: the column format is defined in client-qa-2026-09-29.md; the spreadsheet's actual layout is not seen by the delivery team |

There is no live connection: a person exports the file, an admin uploads it once, and the spreadsheet is then retired.

## Integration Surface

| Aspect | Details |
|--------|---------|
| Interface type | File transfer: CSV upload through the admin web page over HTTPS |
| Authentication | None on the file; upload is performed by an admin acting in the app (pilot: selected identity, see ADR-0005) |
| Data format | CSV, UTF-8, comma-separated, header row; columns `room`, `person_email`, `date`, `start`, `end`, `attendees` *(Source: client-qa-2026-09-29.md, Proposed import format)* |
| Rate limits | Not applicable; application enforces a file size and row cap (proposed 1 MB and 2,000 rows) |
| SLA / Availability | Not applicable |
| Sandbox available | Yes: the test environment can be loaded with a sample file; a real anonymised export is needed [Unknown — to be confirmed] |

## Capabilities vs. Requirements

| Project Requirement | System Capability | Gap? |
|---------------------|-------------------|------|
| Bring across the first week's bookings | Spreadsheet can be exported to CSV by Office Operations | None, if columns can be produced (Partial until a sample is seen) |
| Rows must reference known rooms by exact name | Spreadsheet room names may be free text | Partial: typos will be rejected; Office Operations clean-up assumed (ADR-0012, Proposed) |
| Rows must reference known people by email | Spreadsheet may hold names or initials rather than email | Partial: a name-to-email mapping must be done during clean-up |
| Times on the 15-minute grid in HH:MM | Spreadsheet times may be free text or different formats | Partial: clean-up or export formatting required |
| Known double bookings resolved before export | Spreadsheet has known double bookings *(Source: room-booking-spec.md, Background)* | Partial: unresolved conflicts surface as rejected rows |
| Date within window, Monday to Friday | Depends on the export | None (validated on import) |
| No silent changes; rejection report | Provided by the import module | None |

## Gaps and Risks

| # | Gap or Risk | Severity | Mitigation or Workaround |
|---|-------------|----------|--------------------------|
| 1 | Format and behaviour (ADR-0011) and clean-up ownership (ADR-0012) not yet confirmed by the client | Medium | Confirm at next client review, before import build starts |
| 2 | Spreadsheet quality: free-text rooms, names instead of emails, off-grid times, double bookings | Medium | Rejection report lists row, values and reason; Office Operations fixes and re-imports; provide a template header and one worked example |
| 3 | More than 3 rows for a person in the window: rows beyond the 3rd rejected (ADR-0014) | Low | Expected behaviour; documented in the report reason |
| 4 | Go-live date not fixed; window computed from it (weekend moves to following Monday, ADR-0015) | Low | Admin enters go-live date at import; import run before opening time on the first day |
| 5 | Import run after start of the go-live day rejects earlier rows as past | Low | Import before 08:00 on go-live day |
| 6 | Formula injection: cell values starting with `=`, `+`, `-`, `@` echoed into the report or a downloaded CSV | Medium | Escape or prefix such values in the report and downloadable CSV (SEC-06) |
| 7 | Oversized or malformed file used to exhaust resources | Low | Size and row cap; refuse unreadable header with a message |
| 8 | Any reachable user acting as admin could run the import (pick-a-user) | Medium (covered by SEC-01) | Access gating (ADR-0021); disable the import page after the pilot start |
| 9 | Personal data (names by email) in the file is handled outside the tool | Low | Delete the uploaded file after processing; do not persist the raw file, keep only the report of rejected rows and counts |

## Readiness Verdict

**Integration readiness:** Ready with caveats

The integration is technically straightforward: one file, one strict format, and the same booking rules as manual booking. The caveats are business ones: the client has not yet confirmed the format (ADR-0011) or who cleans the spreadsheet (ADR-0012), and the quality of the real spreadsheet is unseen. The recommended next step is to obtain an anonymised sample export from Office Operations, run it through the validator during the build, and agree the clean-up responsibilities at the next client review.

## Dependencies and Timeline Impact

The import depends on rooms (M-03) and people (M-04) existing before it runs, and on the shared booking write service (ADR-0020). It is needed once, at go-live; it is on the critical path only for go-live readiness, not for the rest of the pilot. Because go-live is not fixed, the import window is entered at upload time.

## Open Questions

- [ ] Client confirmation of the CSV format and rejection behaviour (ADR-0011). Office Operations, before import build.
- [ ] Confirmation that Office Operations owns spreadsheet clean-up (ADR-0012). Office Operations, before import build.
- [ ] Can Office Operations supply an anonymised sample export? Office Operations, early in build.
- [ ] Should re-import skip or replace identical rows (assumed skip with a duplicate note)? Product Strategist and Office Operations.
- [ ] Should the raw uploaded file be discarded after processing (recommended)? Project lead.

## Sources
- *Source: client-qa-2026-09-29.md, Answers 7 and 12, Proposed import format*
- *Source: booking-import.md, IM-01 to IM-12*
- *Source: ADR-0010, ADR-0011 (Proposed), ADR-0012 (Proposed), ADR-0014, ADR-0015*
- *Source: room-booking-spec.md, Background*
