# Security Assessment: Meeting Room Booking Pilot

**Version:** 1.0
**Date:** 2026-09-29
**Status:** Superseded by security-assessment-meeting-room-booking-v1.1.md (2026-09-29)
**Assessed by:** Solution Architect

## Scope

| Field | Value |
|-------|-------|
| Architecture document | arch-meeting-room-booking-v1.0.md |
| Integration assessments in scope | integration-microsoft-entra-id-v1.0.md (future phase); integration-spreadsheet-csv-import-v1.0.md |
| Regulatory constraints | None stated. Data protection posture per ADR-0015: EU Azure region and a simple privacy note. Whether any data-protection law or company policy imposes further duties on the stored names and emails is not established and is listed as an open question |
| Assessment scope | Design-level only. Code-level, runtime and infrastructure configuration hardening are out of scope |

Headline: the pilot's dominant risk is the ADR-0005 decision to run without login. The architecture cannot remove it inside the application; it can only bound it with network or platform gating and make the impersonation traceable. Everything else in the design is small in scale and carries ordinary web-application risks.

## Threat Actors

| Threat Actor | Goals | Access Level |
|--------------|-------|--------------|
| External opportunist (anyone who finds the URL) | Read bookings and names, vandalise or cancel bookings, use the site as an admin | Unauthenticated, if the site is publicly reachable |
| Curious or mischievous colleague | See other people's bookings, act as an admin, cancel a rival's bookings, hoard rooms as someone else | Inside the trusted group; effectively authenticated because they may pick any user |
| Well-meaning admin | Accidental mass cancellation or wrong import | Privileged |
| Former employee or contractor with the URL | Continued access after leaving | Depends on gating; possible if only obscurity protects the site |
| Malicious or careless file supplier | Craft a CSV that injects formulas or exhausts the importer | Authenticated as admin (or an impersonator) |
| Compromised administrator machine (later phase) | Token or session theft | Authenticated |

## Attack Surface Map

| Surface | Type | Authentication | Data Sensitivity | Notes |
|---------|------|----------------|-----------------|-------|
| Internet edge to the web application | External boundary | None in pilot (pick-a-user); access gating decision open (ADR-0021) | PII (names, emails), booking history | The dominant exposure |
| Web application (Razor Pages UI and handlers) | Internal container | Signed cookie holding the selected person id | PII, booking data | Authorisation is a server-side check against the selected identity |
| Pick-a-user cookie | Session token | Signed and encrypted; carries only a person id | Low, but grants identity | Forgeable if signing key is exposed or if the cookie is unsigned |
| Admin pages (rooms, people, bookings, import) | Internal function | Role from database | PII, integrity of all bookings | One click from any user under pick-a-user |
| CSV upload channel | External boundary (file) | Admin role via selected identity | PII (emails), bookings | Untrusted input; formula-injection and resource concerns |
| Status sweeper (hosted service) | Internal container | None (in-process) | Booking status | No inbound surface |
| Azure SQL Database | Internal data store | Managed identity, TLS | PII, all bookings | Not internet-exposed if firewall is restricted to Azure services or the app |
| Application Insights and logs | Internal platform service | Instrumentation key/connection string | Operational data; possible PII | Retention and content need control |
| Cookie key ring storage | Internal secret | Database access | Signing key | Protects cookie integrity |
| Microsoft Entra ID | External boundary, future | OpenID Connect | Identity claims | Not in pilot; assessed for the design path |
| Deployment pipeline and infrastructure code | Supply chain | Delivery team platform | Source and deploy rights | Out of scope for design assessment; noted below |

## Attack Vectors

### Internet edge and web application

| Vector | Description | STRIDE | Threat Actor | Likelihood |
|--------|-------------|--------|--------------|------------|
| Direct access with no authentication | Anyone reaching the site opens the picker, chooses an admin and has full control of rooms, people, bookings and import | S, E | External opportunist; former employee | High if gating is absent; Low if gating is in place |
| Impersonation of any colleague | A user picks another person and books, cancels or reads that person's bookings | S | Curious colleague | High (accepted by ADR-0005) |
| Cookie tampering | User edits the cookie to select another person or role | S, E | Curious colleague | Low if signed and role is loaded server-side; High if the app trusts the cookie's role |
| Cross-site request forgery | A page on another site causes a signed-in browser to submit a cancel, deactivate or import request | T | External opportunist | Medium |
| Reading other users' bookings by changing an identifier | A booking id in a URL is changed to another person's booking | I | Curious colleague | Medium |
| Booker exposed in availability results | Availability view returns owner details that employees must not see | I | Curious colleague | Medium |
| Unattributable admin actions | An admin action is performed while impersonating; the audit trail shows the selected identity, not the real person | R | Curious colleague | High |
| Request flooding or heavy queries | Repeated availability searches or uploads exhaust a small App Service and database tier | D | External opportunist | Low to Medium |
| Test-only clock control reaching production | A debug endpoint that moves the clock is deployed by mistake and used to force no-shows or bypass the 2-hour rule | T, E | External opportunist; colleague | Low |

### CSV upload channel

| Vector | Description | STRIDE | Threat Actor | Likelihood |
|--------|-------------|--------|--------------|------------|
| Formula injection | A cell such as `=HYPERLINK(...)` is echoed to the rejection report or downloaded CSV and executes when opened in a spreadsheet | T | Malicious file supplier | Medium |
| Oversized or malformed file | Very large file, deeply repeated rows or invalid encoding exhaust memory or CPU | D | Malicious or careless file supplier | Low |
| Import by an impersonator | An unauthorised person runs the import to fill rooms with bookings | T, E | Curious colleague; external opportunist | Medium (linked to the first vector) |
| Raw file retention | The uploaded file with names and emails is stored beyond need | I | Any reader of storage | Low |
| Unrecorded import | Import runs without a record of who and when | R | Any | Low (audit event and ImportRun recorded) |

### Web application to Azure SQL

| Vector | Description | STRIDE | Threat Actor | Likelihood |
|--------|-------------|--------|--------------|------------|
| Injection through search or import input | Crafted room name or email in a query | T | Any | Low (parameterised queries through EF Core) |
| Database credential theft | Connection string leaked from configuration | S, I | Insider or attacker with app access | Low (managed identity, no password stored) |
| Over-privileged database identity | App identity able to alter schema or read other databases | E | Attacker who compromises the app | Low to Medium (mitigate with data-reader/writer roles only) |
| Public database endpoint | SQL server firewall open to the internet | I, D | External opportunist | Medium if defaults are left; Low if restricted |
| Cookie key ring read | Anyone with database read access can read the key ring and forge cookies | I, S | Insider | Low |

### Application Insights and logging

| Vector | Description | STRIDE | Threat Actor | Likelihood |
|--------|-------------|--------|--------------|------------|
| Emails and names written to logs | Telemetry captures request bodies or full emails | I | Anyone with log access | Medium |
| Log injection or telemetry spoofing | Crafted input creates misleading log lines | T, R | Any | Low |

### Microsoft Entra ID (future phase)

| Vector | Description | STRIDE | Threat Actor | Likelihood |
|--------|-------------|--------|--------------|------------|
| Account matching by unverified email | A user in another tenant or with an unverified email claim signs in and is matched to an existing person | S, E | External opportunist | Medium if tenant restriction and object-identifier binding are omitted |
| Token validation errors | Wrong issuer, audience or signature handling accepted | S | External opportunist | Low with standard library |
| Identity provider outage | Sign-in unavailable | D | Platform | Low |
| Redirect URI misconfiguration | Tokens returned to an unintended site | I | External opportunist | Low |

### STRIDE Coverage Matrix

| Trust Boundary | S — Spoofing | T — Tampering | R — Repudiation | I — Information Disclosure | D — Denial of Service | E — Elevation of Privilege |
|----------------|-------------|--------------|----------------|---------------------------|----------------------|---------------------------|
| Internet edge to web application | Covered (SEC-01, SEC-02) | Covered (SEC-03, SEC-11) | Covered (SEC-05) | Covered (SEC-04) | Covered (SEC-08) | Covered (SEC-01, SEC-02) |
| CSV upload channel | Covered (SEC-01) | Covered (SEC-06) | Covered (audit event and ImportRun; no finding) | Covered (raw file discarded; see integration assessment) | Covered (SEC-08) | Covered (SEC-01) |
| Web application to Azure SQL | Covered (managed identity; no finding) | Covered (parameterised queries; no finding) | Covered (SEC-05) | Covered (SEC-10) | Covered (SEC-08) | Covered (least-privilege database role, listed in mitigations) |
| Web application to Application Insights | Gap, accepted: spoofed telemetry has negligible impact | Gap, accepted: log injection is low impact | Gap, accepted: platform logs are not the audit record (audit events are in the database) | Covered (SEC-09) | Gap, accepted: telemetry is sampled and cannot block the app | Gap, accepted: not applicable |
| Web application and Microsoft Entra ID (future) | Covered (SEC-07) | Covered (SEC-07: token validation) | Covered (audit of sign-in linking) | Gap, accepted: only minimal claims are read | Gap, accepted: identity outage blocks sign-in only; users regain access when restored | Covered (SEC-07) |

## Vulnerability Findings

| # | Finding | Location | Severity | CWE | Mitigation | ADR |
|---|---------|----------|----------|-----|------------|-----|
| SEC-01 | Pick-a-user provides no authentication. If the site is reachable outside the trusted group, anyone can act as an admin and read, change or cancel all bookings, manage people and rooms, and run the import | Internet edge, all containers | High | CWE-306, CWE-284, CWE-287 | Enforce the trusted-group boundary outside the app: IP allow-list on the App Service (preferred where a stable office or VPN address exists) or tenant-only platform sign-in; do not expose the production URL until one is configured; disable the import page after the pilot start | ADR-0021 (Open) |
| SEC-02 | The identity cookie could be forged or its role trusted, allowing selection of any person or role beyond what the picker offers | Pick-a-user cookie, web app | Medium | CWE-565, CWE-345, CWE-602 | Signed and encrypted cookie holding only the person id; on each request load person and role from the database; picker lists only active people; Secure, HttpOnly, SameSite attributes; short lifetime | ADR-0022 |
| SEC-03 | Cookie-based identity without request forgery protection allows another site to trigger cancellations, deactivations or imports | Web application | Medium | CWE-352 | Anti-forgery tokens on every state-changing request; SameSite=Lax or Strict; state changes only via POST | — |
| SEC-04 | Booking and person data could leak across users: booking-id guessing, or availability responses that include owner details | Web application read paths | Medium | CWE-639, CWE-200, CWE-285 | Owner-or-admin check in every booking handler; separate read models so availability never selects the owner; boundary tests for NFR-010 and NFR-012 | — |
| SEC-05 | Audit trail cannot prove who acted because identity is picked, not proven; late and admin-cancellation records are only as trustworthy as the selected identity | Audit events, booking records | Medium | CWE-778, CWE-345 | Record selected identity, time and origin on every state change and admin action (NFR-020); log picker switches; state the limitation in the pilot brief; real sign-in in next phase | — |
| SEC-06 | CSV values that begin with `=`, `+`, `-` or `@` are echoed into the rejection report and downloadable CSV and could execute when opened in a spreadsheet | Import service | Medium | CWE-1236 | Prefix or escape such cells in HTML and CSV output; validate every field against strict patterns before echoing | — |
| SEC-07 | Future Entra ID linking by email alone could let an unrelated or unverified account take over a person record | Entra ID integration (future) | Medium | CWE-290, CWE-287 | Single-tenant registration; link once on verified email, then bind to immutable object identifier and tenant; refuse unmatched accounts; standard token validation | ADR-0022 |
| SEC-08 | No rate limiting or size limits on searches and uploads on a small hosting tier | Web application, import | Low | CWE-770, CWE-400 | Request size limits, upload size and row caps, basic rate limiting middleware; access gating reduces exposure | — |
| SEC-09 | Emails or names may appear in logs and telemetry, spreading personal data to more people and longer retention | Application Insights | Low | CWE-532 | Log person id rather than email; do not log request bodies; sampling and a short retention period | — |
| SEC-10 | Cookie key ring stored in the database unprotected: anyone with database read can forge cookies | Azure SQL | Low | CWE-312, CWE-321 | Restrict database access to the app identity and named administrators; consider protecting the key ring with a Key Vault key when Key Vault is introduced with Entra ID | — |
| SEC-11 | A test-only clock control needed for E2E tests could be enabled in production | Web application, deployment | Low | CWE-489, CWE-16 | Compile the feature out of release builds or gate it on the test environment name; verify in pipeline that production has it disabled | — |

Severity totals: Critical 0, High 1, Medium 6, Low 4 (11 findings).

The database firewall and TLS settings are configuration hardening (out of scope for this skill) but the design requires: SQL server not open to the internet (only the app's identity and named administrators), minimum TLS 1.2 and HTTPS-only on the App Service.

## Risk Register

| # | Risk | Severity | Likelihood | Status | Owner |
|---|------|----------|------------|--------|-------|
| 1 | Unauthenticated pilot exposed beyond the trusted group (SEC-01) | High | Medium | Open, mitigation undecided (ADR-0021) | Client IT and Office Operations, with delivery lead |
| 2 | Colleagues can impersonate any user including admin (SEC-01, SEC-02, SEC-05) | Medium | High | Accepted for the pilot (ADR-0005) | Office Operations |
| 3 | Request forgery and data leakage across users (SEC-03, SEC-04) | Medium | Medium | Open, mitigated by design and tests | Delivery team |
| 4 | CSV formula injection (SEC-06) | Medium | Low | Open, mitigated by design | Delivery team |
| 5 | Future Entra ID account matching (SEC-07) | Medium | Low | Open until sign-in phase | Solution Architect |
| 6 | Availability and abuse limits on small tier (SEC-08) | Low | Low | Open | Delivery team |
| 7 | Personal data in logs and key material handling (SEC-09, SEC-10) | Low | Low | Open | Delivery team |
| 8 | Debug clock in production (SEC-11) | Low | Low | Open | Delivery team |

## ADR Triggers

| ADR | Finding | Status |
|-----|---------|--------|
| ADR-0021 — Pilot Access Gating for the Pick-a-user Deployment | SEC-01 (High) | Open (mitigation recommended, decision needs client IT and Office Operations input) |
| ADR-0022 — Current-user Seam and Entra ID Migration Path (Proposed, created as an architecture decision) | SEC-02, SEC-07 | Proposed |

No Critical findings exist. The only High finding (SEC-01) has ADR-0021, created before saving this assessment. It is Open rather than Accepted because the mitigation choice depends on facts only the client can supply; the backlog must include a story for it, and the production deployment must not go live before it is resolved.

## Open Questions

- [ ] Does the office or VPN have a stable egress IP range, and must phones work off-network? Client IT and Office Operations. Blocks SEC-01 resolution and production exposure.
- [ ] Is there a company data-protection policy or legal duty (for example a data-processing register) that applies to storing names and emails? Office Operations; does not block build, but affects the privacy note.
- [ ] How long should logs and telemetry be kept during the pilot? Project lead (default suggestion: 30 to 90 days).
- [ ] Deferred to infrastructure review: SQL firewall rules, TLS versions, HTTP security headers, Key Vault use, deployment pipeline permissions and branch protection. Recommended step: infrastructure-as-code review before go-live.
- [ ] Deferred to CI: dependency vulnerability scanning and static analysis (in the test strategy).
- [ ] Deferred to pre-go-live: baseline web vulnerability scan against the test environment (in the test strategy).

## Sources
- *Source: arch-meeting-room-booking-v1.0.md, System Context, Component Design, Identity Design, Import Flow, Deployment*
- *Source: integration-microsoft-entra-id-v1.0.md; integration-spreadsheet-csv-import-v1.0.md*
- *Source: ADR-0005, ADR-0006, ADR-0015, ADR-0021, ADR-0022*
- *Source: nfrs.md, NFR-009 to NFR-014, NFR-019, NFR-020*
