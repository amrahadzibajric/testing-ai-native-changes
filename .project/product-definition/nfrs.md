# Non-Functional Requirements

**Last updated:** 2026-09-29
**Source:** room-booking-spec.md; client-qa-2026-09-29.md; ADR-0001 to ADR-0014. The sources state few non-functional requirements. Entries marked **Assumption** are proposed defaults appropriate to a pilot and must be confirmed with the client; they are not client commitments.

| ID | Category | Requirement | Target | Applies To | Basis / Source ADR |
|----|----------|-------------|--------|------------|--------------------|
| NFR-001 | Scalability | Support the pilot user base | Up to 50 people in the directory; concurrent use expected to be a small fraction of that | All | Stated: client-qa-2026-09-29.md, Answer 2; ADR-0001 |
| NFR-002 | Scalability | Data volume | A few thousand bookings over the pilot is sufficient; no large-scale design needed | All | Assumption |
| NFR-003 | Performance | Availability search and booking respond quickly enough to feel immediate | Typical page response under 2 seconds under pilot load | M-01, M-02 | Assumption |
| NFR-004 | Performance | Import of one week of bookings completes in one interaction | File of up to a few hundred rows processed in under 1 minute; report shown on completion | M-06 | Assumption |
| NFR-005 | Correctness | A room can never be double booked, including when two users submit at nearly the same moment | Zero overlapping active bookings per room; the second concurrent request is refused with a clear message | M-01, M-02, M-06 | Stated as a rule: room-booking-spec.md; concurrency is an inference |
| NFR-006 | Correctness | Automatic status changes (no-show, room release, done) happen close to their trigger time | Within 1 minute of the trigger moment | M-02 | Assumption |
| NFR-007 | Availability | The service is available during office hours | 08:00 to 20:00 Monday to Friday, best effort. No formal uptime SLA; brief maintenance outside office hours is acceptable | All | Assumption to confirm |
| NFR-008 | Availability | Recovery after failure | Data restorable from a backup taken at least daily; recovery time within one working day (RTO 1 business day, RPO 24 hours) | All | Assumption to confirm |
| NFR-009 | Security | Pick-a-user is not authentication. Privacy and admin restrictions are enforced against the selected identity but are not secure against a determined user. The application must not be exposed beyond the trusted pilot group (for example network-restricted) | Accepted pilot risk; access limited to trusted staff | M-04, all | Stated: ADR-0005 (risk noted, mitigation left to architecture) |
| NFR-010 | Security | Role rules are enforced by the server, not just hidden in the interface: an employee cannot retrieve another person's bookings or perform admin actions when acting as an employee | Enforced for every request against the selected identity | All | ADR-0005; room-booking-spec.md, Privacy |
| NFR-011 | Security | Data in transit protected | HTTPS only | All | Assumption |
| NFR-012 | Privacy | Employees see only their own bookings; availability shows busy without the booker; only admins see everyone's bookings | As stated | M-01, M-05 | Stated: room-booking-spec.md, Privacy |
| NFR-013 | Privacy | Personal data held is limited to name, email, role and booking history; no other personal data collected | Minimal data set | M-04 | Assumption |
| NFR-014 | Compliance | No regulatory requirement was stated. Whether company policy or local data protection rules apply to storing employee names and emails on Azure is not addressed | To be confirmed | All | Gap: no source states requirements. **Resolved by ADR-0015:** host in an EU Azure region and provide a simple privacy note for the names and emails stored; no stricter compliance regime stated |
| NFR-015 | Time handling | A single office time zone applies to all times, including office hours, the 2-hour late rule and the check-in window; no time-zone conversion or selection | One configured zone | All | Stated: room-booking-spec.md, Not needed for now; zone name is a gap. Daylight-saving changes must not shift booking times displayed. **Resolved by ADR-0015:** Central European Time with daylight saving (CET/CEST, e.g. Europe/Berlin-equivalent); all rules evaluate in office local time; exact IANA zone name still to be confirmed with the client |
| NFR-016 | Usability | Works in current desktop and phone browsers; phone layout supports availability search, booking, and especially check-in | Latest two versions of Chrome, Edge, Safari and Firefox on desktop; Chrome and Safari on phones | M-01, M-02 | Delivery format stated (ADR-0003); browser list is an assumption |
| NFR-017 | Usability | Rule failures give a message that names the broken rule (for example "maximum booking length is 4 hours") | Every refusal carries a specific reason | M-01, M-02, M-06 | Assumption |
| NFR-018 | Accessibility | Reasonable accessibility (keyboard operation, sufficient contrast) | No formal WCAG level required for the pilot; WCAG 2.1 AA as a design aim | All | Assumption |
| NFR-019 | Observability | Basic application logging of errors and admin actions (room and person changes, cancellations by admin, import runs) for troubleshooting | Logs retained for the pilot duration | All | Assumption |
| NFR-020 | Auditability | The booking record keeps who made it, who cancelled or edited it, and whether an admin acted, so "cancelled by admin" and late flags are trustworthy | Recorded on every state change | M-02, M-05 | Inferred from room-booking-spec.md, For admins; Cancelling |
| NFR-021 | Data retention | Booking history retained for the pilot; no archival or deletion policy defined | Keep all records during the pilot | All | Assumption to confirm |
| NFR-022 | Hosting | Hosted on Azure resources made available by the client | Azure | All | Stated: ADR-0002 |
| NFR-023 | Portability of identity | Person identity is keyed on email so Microsoft sign-in can be added without data migration | Email unique and case-insensitive | M-04 | Stated: ADR-0006 |
| NFR-024 | Cost | Pilot-scale, low-cost Azure footprint | Smallest suitable tiers | All | Assumption |

## Categories

Performance, Scalability, Availability, Security, Privacy, Observability, Accessibility, Data retention, Time handling, Correctness.

## Open questions

- Availability target and backup expectations (NFR-007, NFR-008) are assumptions.
- Company data policy for hosting employee names and emails on Azure (NFR-014). **Resolved by ADR-0015:** EU Azure region plus a simple privacy note.
- The office time zone by name (NFR-015). **Resolved by ADR-0015:** CET/CEST (Europe/Berlin-equivalent); exact IANA name to be confirmed with the client.
- Whether the pilot needs to be restricted to the corporate network (NFR-009).

## POC applicability note (added 2026-09-29, ADR-0024, ADR-0025)

*Append-only note. No requirement above has been changed or removed; the table describes the requirements as they apply to the hosted pilot.* The current phase is a local POC on a developer machine with SQLite and synthetic data, and all functional scope is included. Infrastructure and hosting NFRs are deferred to the post-POC phase and apply again when hosting begins.

| NFR | POC status | Note |
|-----|-----------|------|
| NFR-001, NFR-002 | Applies | Data volumes still apply and are checked locally |
| NFR-003, NFR-004 | Applies, indicative | Measured locally; Azure figures are measured post-POC |
| NFR-005, NFR-006 | Applies | NFR-005 is met in the POC by SQLite's single-writer serialisation through the single booking-write service; the per-room lock on Azure SQL applies when hosted (ADR-0020, ADR-0025) |
| NFR-007 (availability in office hours) | Deferred to post-POC | No hosting in the POC |
| NFR-008 (RPO 24 h, RTO 1 business day) | Deferred to post-POC | POC data is synthetic and disposable; reset by re-seeding |
| NFR-009 (not exposed beyond the trusted group) | Met by not exposing: localhost only | The tenant access gate (ADR-0021) is deferred and applies when hosted |
| NFR-010, NFR-012 | Applies | Server-side role enforcement and privacy are functional and are built and tested |
| NFR-011 (HTTPS only) | Deferred to post-POC | HTTP on localhost is acceptable |
| NFR-013 | Applies | Synthetic data only in the POC |
| NFR-014 (data protection, EU region, privacy note) | Deferred to post-POC | No real personal data in the POC |
| NFR-015 to NFR-018, NFR-023 | Applies | NFR-023 uses SQLite `NOCASE` locally (ASCII folding); Azure SQL collation applies when hosted |
| NFR-019 (basic logging) | Applies, local logs | Application Insights is post-POC |
| NFR-020, NFR-021 | Applies | Gate identity fields exist and are empty in the POC. Import report rows are purged after the import is settled (accepted 2026-09-29), a refinement of NFR-021 for that one table |
| NFR-022 (Azure hosting) | Deferred to post-POC | Target platform remains Azure (ADR-0002) |
| NFR-024 (low cost) | Applies to the target; the POC has no running cost | |

---

*Maintained by the Product Strategist (from source documents) and Solution Architect (from architecture design). Read by the QA agent for NFR test cases.*
