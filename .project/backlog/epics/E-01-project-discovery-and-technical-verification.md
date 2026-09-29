# E-01: Project Discovery and Technical Verification

**EPIC ID:** E-01
**Delivery Phase:** Phase 1 — POC (Discovery, before any feature-build work)
**Dependencies:** None (entry EPIC). Blocks E-02 and every later EPIC.
**ADR References:** ADR-0011, ADR-0012, ADR-0013 (Proposed, confirmation needed); ADR-0015 (zone name); ADR-0016, ADR-0018, ADR-0019, ADR-0020, ADR-0022 (Proposed, disposition needed); ADR-0024, ADR-0025
**Architecture References:** arch-meeting-room-booking-v1.2.md (Open Questions, POC Scope, Concurrency); security-assessment v1.2 (Open Questions); test-strategy v1.2 (Open Questions); integration-spreadsheet-csv-import-v1.0.md (Open Questions)

## Description

This EPIC is the standalone Discovery phase of the local POC. The requirements and architecture are already elaborated, so Discovery here does not analyse an existing system; it retires the small number of technical unknowns that the architecture flagged (the SQLite immediate-transaction API, time zone data on the demonstration machine, the UI approach for the timeline board) and it turns the open and Proposed decisions into recorded dispositions so the build starts on stable ground. When it is complete the team has a verified technical baseline, a recorded answer or an explicit working assumption for every open build-start item, and a client confirmation pack for the Proposed import and remainder rules (ADR-0011 to ADR-0013). No production code is written in this EPIC beyond throwaway spike code.

## Scope Boundaries

- **Includes:** technical spikes listed in the architecture Open Questions; architecture baseline note; confirmation pack for ADR-0011 to ADR-0015; disposition of Proposed ADR-0018, ADR-0019, ADR-0020, ADR-0022; resolution of build-start open items (IANA zone name, bootstrap admin settings, user cap behaviour, meaning of "import settled"); the CSV template and worked example for Office Operations; the request for an anonymised spreadsheet export.
- **Excludes:** any hosting, Azure, client IT or Entra investigation (deferred, see E-12 to E-17); product feature construction (E-02 onward); creation of the sample import CSV itself (F-09.06).

## Key Technical Components

Spike code only (throwaway). Outputs are decision notes and ADR dispositions that feed the Persistence, Booking Write Service, Office Clock, UI and Identity components.

## Key User Roles Affected

Project lead (decisions), Office Operations (confirmation of ADR-0011 to ADR-0015, template review), delivery team.

## Done Criteria

- The SQLite immediate-transaction approach, WAL and busy-timeout settings are proven and written down for F-06.02.
- The office zone resolves with correct offsets on the demonstration machine across the 25 October 2026 and 28 March 2027 transitions.
- The UI approach for the timeline board and the phone list is confirmed feasible.
- Every Proposed ADR the POC builds on has a recorded disposition (accepted, amended, or explicit working assumption), and the confirmation pack for ADR-0011 to ADR-0015 exists.
- Every open build-start item has a recorded answer or an explicit working assumption.

## Features

| Feature | Description | Stories | Surfaces |
|---------|-------------|---------|----------|
| [F-01.01: Technical Spikes and Stack Verification](../features/F-01.01-technical-spikes-and-stack-verification.md) | Proves the SQLite concurrency approach, time zone data and the timeline UI approach before build | 4 | Engineering |
| [F-01.02: Proposed ADR and Open Item Resolution](../features/F-01.02-proposed-adr-and-open-item-resolution.md) | Confirmation pack for ADR-0011 to ADR-0015 and dispositions for Proposed technical ADRs and open items | 3 | Engineering |
| [F-01.03: Import Format Template and Sample Request](../features/F-01.03-import-format-template-and-sample-request.md) | CSV template and worked example for Office Operations and a request for an anonymised export | 2 | Engineering |
