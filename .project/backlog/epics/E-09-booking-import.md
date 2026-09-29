# E-09: Booking Import

**EPIC ID:** E-09
**Delivery Phase:** Phase 1 — POC (build) — module M-06 (complete, not a stub, per ADR-0024)
**Dependencies:** E-02, E-03, E-04, E-05, E-06 (rules engine and write service), E-07 (effective status), F-02.06 (seed people and rooms for the sample). Depends on the unconfirmed ADR-0011 to ADR-0013.
**ADR References:** ADR-0004, ADR-0010, ADR-0011 (Proposed), ADR-0012 (Proposed), ADR-0014 (rules 2 and 4), ADR-0015 (decision 5), ADR-0024
**Architecture References:** arch v1.2 (Import service, Import Flow, Report safety, POC data and sample file); integration-spreadsheet-csv-import-v1.0.md; data-model import.md; security-assessment v1.2 (SEC-06, SEC-08, SEC-14)

## Description

This EPIC delivers module M-06, the one-off import of the first working week's bookings from an Office Operations CSV export, at full behaviour in the POC: the ADR-0011 format, whole-file checks, per-row validation against the same rules as manual booking, the five-working-day window, per-person ordering and 3-booking limit, duplicate detection on re-import, a rejected-rows report with a downloadable CSV, formula-injection protection and purge of report rows once the import is settled. Before this EPIC, existing spreadsheet bookings would have to be entered twice; afterwards an admin uploads a file, sees exactly what was imported and what was rejected and why, corrects the rows and re-imports without duplicates. It is tested and demonstrated with a synthetic sample CSV prepared by the delivery team.

## Scope Boundaries

- **Includes:** import page and intake checks; window computation; field and reference validation; rules application, ordering and limit; execution through the write service; duplicate detection; ImportRun bookkeeping; report, downloadable rejected-rows CSV, escaping, settle and purge; import screens for web and phone; sample CSV and fixtures.
- **Excludes:** importing history or anything beyond the window, recurring imports, direct spreadsheet or SharePoint connections, automatic conflict resolution, creating people or rooms from the file, export back to the spreadsheet (M-06 out of scope); use of a real spreadsheet export (only anonymised, and only if Office Operations supplies one).

## Key Technical Components

Import service, admin import page, ImportRun and ImportReportRow entities, shared rules engine and write service, sample CSV generator.

## Key User Roles Affected

Admin (uploads, reviews, re-imports, settles); Office Operations (prepares the CSV, fixes rejected rows) as an external party.

## Done Criteria

- A valid file imports every row as a Confirmed booking with `source = Import`; counts equal imported plus duplicate plus rejected.
- A file with a wrong header, wrong encoding or over the size or row cap is refused whole with a message and writes nothing.
- Every rejected row appears in the report with its line number, raw values and every broken rule by name; nothing is changed silently.
- Re-importing a corrected file reports already-imported identical rows as duplicates.
- Formula-leading cells are escaped in the report and downloadable CSV; report rows are purged when the import is settled, leaving ImportRun counts and the audit event.
- 500 rows complete within 1 minute locally; the sample CSV exercises every rejection reason.

## Features

| Feature | Description | Stories | Surfaces |
|---------|-------------|---------|----------|
| [F-09.01: Import Page and File Intake](../features/F-09.01-import-page-and-file-intake.md) | Admin import page, whole-file acceptance checks, parsing | 4 | Web, Phone browser, Admin |
| [F-09.02: Import Window and Row Validation](../features/F-09.02-import-window-and-row-validation.md) | Window computation, field and reference validation, rules, ordering | 4 | Engineering |
| [F-09.03: Import Execution and Re-import](../features/F-09.03-import-execution-and-re-import.md) | Writing rows, duplicates, ImportRun, performance | 4 | Engineering |
| [F-09.04: Rejected-Rows Report and Downloadable CSV](../features/F-09.04-rejected-rows-report-and-downloadable-csv.md) | Report data, download, escaping, settle and purge | 4 | Web, Admin |
| [F-09.05: Import Screens (Web and Phone)](../features/F-09.05-import-screens-web-and-phone.md) | Upload, result and states screens | 3 | Web, Phone browser, Admin |
| [F-09.06: Sample Import CSV and Fixtures](../features/F-09.06-sample-import-csv-and-fixtures.md) | Synthetic sample covering every rejection reason | 3 | Engineering |
