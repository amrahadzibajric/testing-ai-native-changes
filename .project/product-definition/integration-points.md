# Integration Points

**Last updated:** 2026-09-29
**Source:** room-booking-spec.md; client-qa-2026-09-29.md; ADR-0002, ADR-0006, ADR-0010, ADR-0011

Primary input for the Solution Architect's integration assessments. The pilot has very few external dependencies by design.

| System | Direction | Protocol | Modules | Priority | Assessment |
|--------|-----------|----------|---------|----------|------------|
| Microsoft identity (company Microsoft accounts) | Inbound (identity of the signed-in user) | To be selected in architecture (sign-in standard supported by Microsoft) | M-04 (and all through current user) | P2 (later phase; not in the pilot) | Assessed: integration-microsoft-entra-id-v1.0.md (Ready with caveats) |
| Azure hosting | Not an integration in the business sense; the platform the application runs on | Azure services | All | P1 | Covered in arch-meeting-room-booking-v1.0.md (platform, not an integration) |
| Spreadsheet CSV export (Office Operations) | Inbound, one-off, by file upload | CSV file: UTF-8, comma-separated, header row | M-06 | P1 (one-off at pilot start) | Assessed: integration-spreadsheet-csv-import-v1.0.md (Ready with caveats). No live connection. Format pending client confirmation (ADR-0011) |

## POC note (added 2026-09-29, ADR-0024)

In the local POC phase only the CSV import is live, using a synthetic or anonymised sample file. **Microsoft identity (Entra ID) and Azure hosting are deferred to the post-POC phase**: no Entra work is done, the tenant access gate (ADR-0021) is not configured, and nothing is deployed. The table above is unchanged and describes the pilot after the POC. The current-user seam stays in the code design so the later integration remains additive.

## Details

### Microsoft identity (future)
The client expects to use Microsoft company accounts for sign-in after the pilot. In the pilot, the picker replaces sign-in. The design obligation on the pilot is to key people on email (ADR-0006) and to isolate how the current person is determined. Nothing is integrated in the pilot. *(Source: client-qa-2026-09-29.md, Answer 9; ADR-0005, ADR-0006)*

### Azure hosting
Azure resources can be used; the specific services, subscription, region and network arrangements are not stated and are for the architecture phase. Network restriction of the pilot deployment is relevant because pick-a-user is not secure (ADR-0005). *(Source: client-qa-2026-09-29.md, Answer 8; ADR-0002)*

### CSV import from the spreadsheet
A single file is exported by Office Operations from the existing spreadsheet and uploaded by an admin, covering the 5 working days from the go-live date. There is no ongoing connection to the spreadsheet. Columns: `room`, `person_email`, `date`, `start`, `end`, `attendees`. *(Source: client-qa-2026-09-29.md, Proposed import format; ADR-0010, ADR-0011, ADR-0014)*

## Explicitly not integrated (out of scope)

- Email (no notifications). *(room-booking-spec.md, Not needed for now)*
- Calendar systems such as Outlook or Google Calendar.
- Slack or other chat tools.
- Company directory or HR system for people (people are added manually by admins). *(ADR-0004)*
- Room-side devices, displays or badge readers. *(client-qa-2026-09-29.md, Answer 5)*
- Payment, ticketing or facilities systems.

## Direction key
- **Inbound** — external system sends data to us
- **Outbound** — we send data to external system
- **Bidirectional** — both directions

## Priority key
- **P1** — required for MVP / Phase 1
- **P2** — required for a later phase
- **P3** — nice to have / future consideration

---

*Maintained by the Product Strategist (from source documents) and Solution Architect (from integration assessments). Integration assessment files saved to `.project/architecture/integration-*.md`.*
