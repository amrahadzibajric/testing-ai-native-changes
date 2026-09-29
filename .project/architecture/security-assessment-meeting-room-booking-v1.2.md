# Security Assessment: Meeting Room Booking

**Version:** 1.2
**Date:** 2026-09-29
**Status:** Draft
**Assessed by:** Solution Architect

## Change Log

| Version | Date | Change |
|---------|------|--------|
| 1.0 | 2026-09-29 | Initial assessment. SEC-01 (High) open pending ADR-0021 |
| 1.1 | 2026-09-29 | ADR-0021 accepted (tenant-only Easy Auth gate). SEC-01 residual re-rated High to Medium; SEC-05 re-rated Medium to Low; new finding SEC-12. No High or Critical findings remain |
| 1.2 | 2026-09-29 | Scope change: the current phase is a local proof of concept on localhost with SQLite and synthetic data (ADR-0024, ADR-0025). Findings gain a POC applicability column. SEC-01 and SEC-12 are not applicable while the application is not network-exposed and re-activate when hosting begins; SEC-09, SEC-10 and the Azure-specific vectors are deferred. Findings that concern application code (SEC-02, 03, 04, 06, 08, 11) still apply. New POC-specific findings SEC-13 (accidental exposure of the POC) and SEC-14 (sample and demonstration data). No High or Critical findings, so no new ADR is triggered. The v1.1 hosted-design assessment is retained below as the target state |
| 1.2 rev a | 2026-09-29 | Minor revision, append-only, same day, no version bump. Project lead decisions (ADR-0024): there is no demonstration clock control, so the "Demonstration clock control" surface, its attack vector and risk 7 are removed as features and SEC-11 is reframed as a guard that no clock control or time-override endpoint exists in the running application (the injectable clock is for automated tests only). Run instructions must forbid LAN binding and public tunnels (SEC-13, now a firm requirement). The delivery team demonstrates; there is no remote client access or live trial. A configured bootstrap admin is seeded on first run when no admin exists (arch Identity Design); it is a startup configuration item, not a new attack surface, and adds no finding. Severity totals are unchanged |

## Scope

| Field | Value |
|-------|-------|
| Architecture document | arch-meeting-room-booking-v1.2.md |
| Integration assessments in scope | integration-spreadsheet-csv-import-v1.0.md (applies in the POC); integration-microsoft-entra-id-v1.0.md (post-POC only) |
| Regulatory constraints | None stated. Data protection posture per ADR-0015: EU Azure region and a simple privacy note, both operationally deferred; the POC holds synthetic data only |
| Assessment scope | Design-level only. Code-level, runtime and infrastructure configuration hardening are out of scope |

### POC applicability summary

The POC runs on a developer machine, listens on localhost only and holds synthetic data (ADR-0024). Under those conditions there is no network attacker and no real personal data at stake, so the external-exposure half of the ADR-0005 risk does not exist. What remains in the POC is application behaviour that a demonstration audience or a colleague at the machine could exercise: role enforcement, privacy of other people's bookings, forgery protection and safe handling of the import file. The hosted-design findings are not discarded; they are labelled "post-POC" and become active when the application is deployed.

| Condition of the POC | Effect on the assessment |
|----------------------|--------------------------|
| Bound to localhost, no tunnel, no shared network binding | SEC-01 (unauthenticated exposure) and SEC-12 (gate misconfiguration) not applicable. They must be re-assessed before any hosted deployment. ADR-0021 stays Accepted and applies then |
| Synthetic seed data and an anonymised sample import file only | Privacy impact of any leak is negligible; SEC-09 (personal data in logs) and the GDPR-related open questions are deferred |
| SQLite file on the presenter's machine | Azure SQL vectors (public endpoint, managed identity, over-privileged identity) not applicable. The file is readable by anyone with the machine's file access; acceptable for synthetic data |
| No CI/CD, no cloud accounts | Supply-chain and service-connection concerns not applicable |
| Microsoft Entra ID not used | SEC-07 and the Entra vectors not applicable |

## Threat Actors

| Threat Actor | Goals | Access Level | Applies in POC? |
|--------------|-------|--------------|-----------------|
| External opportunist (anyone who finds the URL) | Read bookings and names, vandalise or cancel bookings, act as an admin | Unauthenticated | No: no network exposure. Yes if the POC is accidentally exposed (SEC-13) |
| Curious or mischievous colleague | See other people's bookings, act as an admin, cancel a rival's bookings | Passes the gate, then may pick any user | Post-POC. In the POC only someone at the demonstration machine |
| Well-meaning admin or presenter | Accidental mass cancellation or a wrong import during a demonstration | Privileged | Yes |
| Former employee or contractor with the URL | Continued access after leaving | Blocked when the tenant account is disabled | Post-POC |
| Tenant guest or non-pilot tenant member | Reach the pilot through a tenant account | Passes the gate unless user assignment is required | Post-POC |
| Malicious or careless file supplier | Craft a CSV that injects formulas or exhausts the importer | Admin role via selected identity | Yes (sample files are ours, but the import is real behaviour; the same code is hosted later) |
| Compromised administrator machine (later phase) | Token or session theft | Authenticated | Post-POC |

## Attack Surface Map

| Surface | Type | Authentication | Data Sensitivity | Notes | POC status |
|---------|------|----------------|-----------------|-------|-----------|
| Localhost listener of the web application | Local boundary | None; pick-a-user inside the app (ADR-0005) | Synthetic PII, booking data | Exposed only if bound beyond localhost (SEC-13) | Applies |
| Web application (Razor Pages UI and handlers) | Internal container | Signed cookie holding the selected person id | Synthetic PII, booking data | Authorisation is a server-side check against the selected identity | Applies |
| Pick-a-user cookie | Session token | Signed and encrypted; carries only a person id | Low, but grants identity | Key ring in a local file store in the POC | Applies |
| Admin pages (rooms, people, bookings, import) | Internal function | Role from database | Integrity of all bookings | One click from anyone at the machine | Applies |
| CSV upload channel | External boundary (file) | Admin role via selected identity | Synthetic emails, bookings | Untrusted input; formula-injection and resource concerns | Applies |
| Status sweeper (hosted service) | Internal container | None (in-process) | Booking status | No inbound surface | Applies |
| SQLite database file | Local data store | File system permissions | Synthetic PII, all bookings | Readable by anyone with file access on the machine | Applies (low impact) |
| Clock abstraction | Internal code seam | None; not reachable from HTTP | Integrity of time-based rules | Replaced by a fake clock in automated tests only; no endpoint, setting or UI may move the clock (SEC-11) | Applies |
| Console and file logs | Local | File system permissions | Operational data | Synthetic data only | Applies (low impact) |
| Easy Auth gate and Microsoft Entra ID sign-in | External boundary, platform component | OpenID Connect | Identity claims | Configuration correctness is the control (SEC-12) | Post-POC |
| Azure SQL Database | Internal data store | Managed identity, TLS | PII, all bookings | Not internet-exposed if firewall is restricted | Post-POC |
| Application Insights and logs | Platform service | Connection string | Operational data | Retention and content need control | Post-POC |
| Cookie key ring in the database | Internal secret | Database access | Signing key | Protects cookie integrity | Post-POC |
| Microsoft Entra ID as application sign-in (future) | External boundary, future | OpenID Connect | Identity claims | Design path only | Post-POC |
| Deployment pipeline and infrastructure code | Supply chain | Azure DevOps (ADR-0023); service connection | Source and deploy rights | Out of scope for design assessment | Post-POC |

## Attack Vectors

### Web application and localhost listener (POC and post-POC)

| Vector | Description | STRIDE | Threat Actor | Likelihood | POC status |
|--------|-------------|--------|--------------|------------|------------|
| Accidental exposure of the POC | The app is bound to all interfaces, shared through a public tunnel, or left running on a shared network, letting others reach an unauthenticated picker | S, E, I | External opportunist | Low if bound to localhost | Applies (SEC-13) |
| Impersonation of any person | Anyone at the machine picks another person, including an admin | S | Presenter, observer | High (accepted by ADR-0005) | Applies, low impact |
| Cookie tampering | User edits the picker cookie to select another person or role | S, E | Any | Low if signed and role is loaded server-side | Applies |
| Cross-site request forgery | A page in another browser tab causes a submit of cancel, deactivate or import | T | External opportunist | Low locally, Medium hosted | Applies |
| Reading other users' bookings by changing an identifier | A booking id in a URL is changed | I | Any | Medium | Applies |
| Booker exposed in availability results | Availability response includes owner details | I | Any | Medium | Applies |
| Admin action attributable only to the picked identity | No gate identity to prove the real actor | R | Any | Not meaningful locally | Post-POC (SEC-05) |
| Request flooding or heavy queries | Repeated searches or uploads exhaust the process | D | Any | Low | Applies |
| Clock control introduced into the running application | A debug endpoint, setting or query parameter moves the clock and forces no-shows or bypasses the 2-hour rule | T, E | Any | Low | Applies at build time (SEC-11); prevented by not building one |
| Direct access with no authentication (network) | Outsider reaches the picker and takes admin | S, E | External opportunist | Not applicable while localhost only | Post-POC (SEC-01) |
| Gate misconfiguration or bypass | Allow-unauthenticated left on, multi-tenant issuer, ungated test environment | S, E | External opportunist | Not applicable | Post-POC (SEC-12) |
| Access by tenant members outside the pilot group | Gate is tenant-wide unless user assignment is required | S, I | Tenant guest | Not applicable | Post-POC |
| Identity provider outage | Sign-in unavailable | D | Platform | Not applicable | Post-POC |

### CSV upload channel (POC and post-POC)

| Vector | Description | STRIDE | Threat Actor | Likelihood |
|--------|-------------|--------|--------------|------------|
| Formula injection | A cell such as `=HYPERLINK(...)` is echoed to the rejection report or downloaded CSV and executes when opened in a spreadsheet | T | Malicious file supplier | Medium; the POC sample includes such a cell to test the mitigation |
| Oversized or malformed file | Very large file or invalid encoding exhaust memory or CPU | D | Malicious or careless file supplier | Low |
| Import by an impersonator | Someone picks an admin and runs the import | T, E | Any | Medium (linked to impersonation) |
| Raw file and report retention | Uploaded file or rejected rows with emails kept beyond need | I | Any reader of storage | Low; the raw file is not stored and rejected-row data is purged after the import is settled (project lead, 2026-09-29) |
| Unrecorded import | Import runs without a record | R | Any | Low (audit event and ImportRun recorded) |

### Application to database (POC: SQLite)

| Vector | Description | STRIDE | Threat Actor | Likelihood |
|--------|-------------|--------|--------------|------------|
| Injection through search or import input | Crafted room name or email in a query | T | Any | Low (parameterised queries through EF Core; no raw SQL) |
| Database file copied or modified | Anyone with file access reads or edits the SQLite file | I, T | Anyone at the machine | Low impact: synthetic data |
| Divergent behaviour hides a defect | SQLite semantics (collation, locking) differ from Azure SQL, so a defect passes in the POC | T | Not an adversary | Medium; covered by re-testing on Azure SQL before hosting (SEC-14 related, ADR-0025) |

### Post-POC vectors retained from v1.1 (not active in the POC)

| Boundary | Vector | STRIDE | Note |
|----------|--------|--------|------|
| Web application to Azure SQL | Database credential theft; over-privileged database identity; public database endpoint; cookie key ring read | S, I, E | Managed identity, least-privilege roles, restricted firewall (configuration hardening, infrastructure review) |
| Application Insights and logging | Emails, names and gate sign-in names written to telemetry; log injection | I, T, R | SEC-09 |
| Microsoft Entra ID as application sign-in (future) | Account matching by unverified email; token validation errors; redirect URI misconfiguration | S, E, I | SEC-07 |

### STRIDE Coverage Matrix

Rows marked "post-POC" describe the target design and are not active in the POC.

| Trust Boundary | S | T | R | I | D | E |
|----------------|---|---|---|---|---|---|
| Localhost listener to web application (POC) | Covered (SEC-02, SEC-13) | Covered (SEC-03, SEC-11) | Gap, accepted: no gate identity locally; audit records the picked person only (SEC-05 post-POC) | Covered (SEC-04) | Covered (SEC-08) | Covered (SEC-02, SEC-13) |
| CSV upload channel (POC) | Covered (SEC-01 post-POC; impersonation accepted) | Covered (SEC-06) | Covered (audit event and ImportRun) | Covered (raw file not stored; rejected rows purged) | Covered (SEC-08) | Covered (admin role check; impersonation accepted) |
| Application to SQLite file (POC) | Gap, accepted: synthetic data | Covered (parameterised queries) | Covered (SEC-05 audit) | Gap, accepted: synthetic data | Gap, accepted: local only | Gap, accepted: file permissions of the machine |
| Internet edge to web application (post-POC) | Covered (SEC-01, SEC-02, SEC-12) | Covered (SEC-03, SEC-11) | Covered (SEC-05) | Covered (SEC-04) | Covered (SEC-08) | Covered (SEC-01, SEC-02) |
| Easy Auth gate and Entra ID sign-in (post-POC) | Covered (SEC-01, SEC-12) | Covered (SEC-12) | Covered (SEC-05) | Covered (SEC-09) | Gap, accepted: outage blocks access only | Covered (SEC-01) |
| Web application to Azure SQL (post-POC) | Covered (managed identity) | Covered (parameterised queries) | Covered (SEC-05) | Covered (SEC-10) | Covered (SEC-08) | Covered (least-privilege database role) |
| Web application to Application Insights (post-POC) | Gap, accepted | Gap, accepted | Gap, accepted | Covered (SEC-09) | Gap, accepted | Gap, accepted |
| Web application and Entra ID in-app sign-in (post-POC) | Covered (SEC-07) | Covered (SEC-07) | Covered | Gap, accepted | Gap, accepted | Covered (SEC-07) |

## Vulnerability Findings

| # | Finding | Location | Severity | CWE | Mitigation | ADR | POC applicability |
|---|---------|----------|----------|-----|------------|-----|-------------------|
| SEC-01 | Pick-a-user provides no authentication. When hosted, anyone who can reach the site can act as an admin. Gated by ADR-0021 (residual Medium: any gated person can pick any user) | Internet edge, all containers | High inherent; Medium residual (hosted) | CWE-306, CWE-284, CWE-287 | Tenant-only Easy Auth in front of production and test; user assignment required as hardening; no real data until the gate is verified | ADR-0021 (Accepted, deferred) | **Not applicable in the POC** (localhost only, synthetic data). Re-activates on hosting |
| SEC-02 | The identity cookie could be forged or its role trusted | Pick-a-user cookie, web app | Medium | CWE-565, CWE-345, CWE-602 | Signed and encrypted cookie holding only the person id; load person and role from the database each request; picker lists only active people; Secure, HttpOnly, SameSite attributes (Secure applies once HTTPS is used) | ADR-0022 | Applies |
| SEC-03 | Cookie identity without request forgery protection allows another site to trigger state changes | Web application | Medium | CWE-352 | Anti-forgery tokens on every state-changing request; SameSite; state changes only via POST | — | Applies |
| SEC-04 | Booking and person data could leak across users: booking-id guessing, or availability responses that include owner details | Web application read paths | Medium | CWE-639, CWE-200, CWE-285 | Owner-or-admin check in every booking handler; separate read models so availability never selects the owner; boundary tests for NFR-010 and NFR-012 | — | Applies |
| SEC-05 | Audit trail cannot prove who acted because identity is picked. Improved when the gate identity is captured | Audit events | Low (hosted, once gate capture is built) | CWE-778, CWE-345 | Gate sign-in name and object identifier recorded read-only on audit events; mismatch indicator; picker switches logged | ADR-0021 | Deferred: the fields exist and are empty in the POC. Picker switches are still logged |
| SEC-06 | CSV values beginning with `=`, `+`, `-` or `@` echoed into the report and downloadable CSV could execute in a spreadsheet | Import service | Medium | CWE-1236 | Escape such cells in HTML and CSV output; validate every field against strict patterns before echoing | — | Applies (import is fully in the POC; the sample file contains such a cell) |
| SEC-07 | Future Entra ID linking by email alone could let an unrelated account take over a person record | Entra ID integration (future) | Medium | CWE-290, CWE-287 | Single-tenant registration; link once on verified email, then bind to immutable object identifier and tenant | ADR-0022 | Not applicable (no Entra work in the POC) |
| SEC-08 | No rate limiting or size limits on searches and uploads | Web application, import | Low | CWE-770, CWE-400 | Request size limits, upload size and row caps, basic rate limiting | — | Applies (upload and row caps); rate limiting optional locally |
| SEC-09 | Emails, names or gate sign-in names may appear in logs and telemetry | Application Insights | Low | CWE-532 | Log person id rather than email; do not log request bodies or identity headers; short retention | — | Deferred (synthetic data; no Application Insights). Log discipline (no emails in logs) is cheap and should be built now |
| SEC-10 | Cookie key ring stored in the database unprotected | Azure SQL | Low | CWE-312, CWE-321 | Restrict database access; consider Key Vault protection | — | Deferred (POC key ring is a local file) |
| SEC-11 | A clock control (endpoint, setting or UI) could exist in the running application and be used to move time and defeat the time-based rules | Web application | Low | CWE-489, CWE-16 | None is built. Time-based rules run on the real clock. The injectable clock abstraction is used only by automated tests and is not registered as a controllable service or exposed over HTTP; a test asserts that no such endpoint exists | — | Applies: no demonstration clock control (ADR-0024, 2026-09-29) |
| SEC-12 | The gate is only as strong as its configuration | Easy Auth configuration, App Service, test environment | Medium (hosted) | CWE-1188, CWE-16, CWE-306 | Gate as code; single-tenant issuer; automated reachability check | ADR-0021 | **Not applicable in the POC**. Re-activates on hosting |
| SEC-13 | The POC could be exposed beyond the demonstration machine (bound to all interfaces, shared through a public tunnel, left running on a shared network), leaving an unauthenticated picker open | Web application host configuration | Low | CWE-668, CWE-1188 | Bind Kestrel to localhost only; run instructions forbid tunnels and LAN binding; synthetic data only, so impact stays limited; ADR-0024 rejected the tunnel option | ADR-0024 | Applies (new) |
| SEC-14 | Sample and demonstration data could contain real personal data (for example a real spreadsheet export used for the import demo) or be mistaken for production data | Seed data, sample CSV | Low | CWE-359, CWE-200 | Seed and sample data are synthetic or anonymised, using obviously fictional names and non-routable email addresses; no real export enters the POC; a note in the run instructions says the POC is not for real bookings | ADR-0024 | Applies (new) |

Severity totals (POC-applicable, 14 findings in total): Critical 0, High 0, Medium 4 (SEC-02, 03, 04, 06), Low 4 (SEC-08, 11, 13, 14; SEC-11 rises to Medium if it ever reaches a hosted build). Deferred to post-POC: SEC-01 (Medium residual), SEC-05 (Low), SEC-07 (Medium), SEC-09 (Low), SEC-10 (Low), SEC-12 (Medium). No Critical or High residual findings exist, so no ADR is triggered by this version.

## Residual Risk

| Question | Answer in the POC | Answer when hosted (unchanged from v1.1) |
|----------|-------------------|-------------------------------------------|
| Can an outsider reach the picker? | No: localhost only. Yes if the POC is exposed by mistake (SEC-13) | No, if the gate is configured as designed (SEC-12) |
| Can a user act as any other person, including an admin? | Yes, by design (ADR-0005); acceptable for a controlled demonstration | Yes, for gated users; the audit trail makes it attributable (SEC-05) |
| What data is at risk? | Synthetic data only | Real names, emails and booking history |
| What removes the remaining risk? | Nothing needed in the POC | Real in-app sign-in bound to the Entra object identifier (ADR-0022, later phase) |

## Risk Register

| # | Risk | Inherent | Residual | Likelihood | Status | Owner |
|---|------|----------|----------|------------|--------|-------|
| 1 | Unauthenticated app exposed beyond the trusted group (SEC-01, SEC-12) | High | Not applicable in the POC; Medium hosted | n/a in POC | Deferred to post-POC; must be re-assessed before hosting | Client IT with delivery lead (post-POC) |
| 2 | Anyone at the demonstration can act as any person (SEC-02) | Medium | Low | High | Accepted for the POC (ADR-0005) | Project lead |
| 3 | Request forgery and data leakage across users (SEC-03, SEC-04) | Medium | Medium | Medium | Open, mitigated by design and tests | Delivery team |
| 4 | CSV formula injection (SEC-06) | Medium | Medium | Low | Open, mitigated by design | Delivery team |
| 5 | POC accidentally exposed beyond localhost (SEC-13) | Low | Low | Low | Open, mitigated by configuration and instructions | Delivery team |
| 6 | Real personal data enters the POC (SEC-14) | Low | Low | Low | Open, mitigated by process | Project lead |
| 7 | A clock control is introduced into the running application (SEC-11) | Low | Low | Low | Open, mitigated by not building one and a test asserting its absence | Delivery team |
| 8 | Audit cannot prove real actor (SEC-05) | Medium | Low hosted | Medium | Deferred to hosting | Delivery team |
| 9 | Future Entra ID account matching (SEC-07), personal data in logs and key material (SEC-09, SEC-10) | Medium | Medium / Low | Low | Deferred to post-POC | Solution Architect |
| 10 | Tenant Conditional Access or IT approval blocks the gate (dependency) | n/a | n/a | Medium | Deferred; client IT | Project lead |

## ADR Triggers

| ADR | Finding | Status |
|-----|---------|--------|
| ADR-0021 Pilot Access Gating | SEC-01, SEC-05, SEC-12 | Accepted; deferred to post-POC by ADR-0024 |
| ADR-0022 Current-user Seam and Entra ID Path | SEC-02, SEC-07 | Proposed; seam kept, Entra work deferred |
| ADR-0024 Local POC first | SEC-13, SEC-14 (mitigations by configuration and process) | Accepted |

No Critical or High residual findings exist in this version. Before any hosted deployment, the backlog must include gate configuration and verification (SEC-01, SEC-12) and gate identity audit capture (SEC-05), and this assessment must be re-run against the hosted design.

## Open Questions

- [ ] Run instructions must forbid LAN binding and public tunnels for the POC (decided 2026-09-29, ADR-0024). Delivery team to write them; project lead to check before the first demonstration.
- [x] Resolved 2026-09-29 (ADR-0024): no demonstration clock control exists; nothing to confirm.
- [ ] Post-POC: can client IT create the single-tenant app registration, which tenant is the company tenant, can "user assignment required" be enabled, would Conditional Access block personal phones? Client IT; blocks hosting.
- [ ] Post-POC: how do automated tests pass the gate on the hosted test environment? Delivery lead.
- [ ] Post-POC: should a mismatch between the gate account and the picked person ever be blocked? Project lead, at sign-in phase planning.
- [ ] Post-POC: is there a company data-protection policy or legal duty covering names, emails and the sign-in name in the audit trail? Office Operations.
- [ ] Post-POC: log and telemetry retention period. Project lead.
- [ ] Deferred to infrastructure review: SQL firewall, TLS versions, HTTP security headers, Key Vault, deployment endpoint protection, Azure DevOps service connection scope and branch protection.
- [ ] Deferred to CI when it exists: dependency vulnerability scanning and static analysis. Until then, run `dotnet list package --vulnerable` manually before each demonstration build.
- [ ] Deferred to hosting: baseline web vulnerability scan against a deployed test environment.

## Sources
- *Source: arch-meeting-room-booking-v1.2.md, POC Scope, Local Run View, Identity Design, Import Flow, Target State*
- *Source: security-assessment-meeting-room-booking-v1.1.md (hosted-design findings carried forward)*
- *Source: integration-microsoft-entra-id-v1.0.md; integration-spreadsheet-csv-import-v1.0.md*
- *Source: ADR-0005, ADR-0015, ADR-0021, ADR-0022, ADR-0023, ADR-0024, ADR-0025*
- *Source: nfrs.md, NFR-009 to NFR-014, NFR-019, NFR-020*
- *Source: project lead, working session 2026-09-29 (local POC scope change; full functional scope clarification)*
- *Note: statements about Easy Auth behaviour and identity headers are general platform knowledge to verify against current Microsoft documentation when hosting begins*
