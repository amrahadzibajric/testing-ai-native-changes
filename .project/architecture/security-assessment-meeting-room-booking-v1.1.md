# Security Assessment: Meeting Room Booking Pilot

**Version:** 1.1
**Date:** 2026-09-29
**Status:** Superseded by security-assessment-meeting-room-booking-v1.2.md (2026-09-29)
**Assessed by:** Solution Architect

## Change Log

| Version | Date | Change |
|---------|------|--------|
| 1.0 | 2026-09-29 | Initial assessment. SEC-01 (High) open pending ADR-0021 |
| 1.1 | 2026-09-29 | ADR-0021 accepted (tenant-only Easy Auth gate; IP allow-list rejected because the pilot must work off-network). SEC-01 mitigation set to the accepted gate and residual risk re-rated High to Medium (Low if user assignment is required). SEC-05 re-rated Medium to Low, conditional on capturing the gate identity in the audit trail. New finding SEC-12 (gate misconfiguration or bypass). New surface, threat and STRIDE rows for the gate. Register, ADR triggers and open questions updated. No High or Critical findings remain |

## Scope

| Field | Value |
|-------|-------|
| Architecture document | arch-meeting-room-booking-v1.1.md |
| Integration assessments in scope | integration-microsoft-entra-id-v1.0.md (sign-in for the gate in the pilot; application sign-in in a later phase); integration-spreadsheet-csv-import-v1.0.md |
| Regulatory constraints | None stated. Data protection posture per ADR-0015: EU Azure region and a simple privacy note. Whether any data-protection law or company policy imposes further duties on the stored names and emails is not established and is listed as an open question |
| Assessment scope | Design-level only. Code-level, runtime and infrastructure configuration hardening are out of scope |

Headline: the pilot's dominant risk remains the ADR-0005 decision to run without in-app login, but its external half is now controlled. Since ADR-0021 a Microsoft sign-in gate restricted to the company tenant sits in front of the app, so an outsider cannot reach the picker. What remains is the internal half: any person who passes the gate can still pick any user, including an admin. That is the ADR-0005 accepted limitation, now confined to authenticated tenant members and made attributable through the audit trail. It is bounded, not removed.

## Threat Actors

| Threat Actor | Goals | Access Level |
|--------------|-------|--------------|
| External opportunist (anyone who finds the URL) | Read bookings and names, vandalise or cancel bookings, use the site as an admin | Unauthenticated; blocked by the gate unless it is misconfigured or has no tenant account |
| Curious or mischievous colleague | See other people's bookings, act as an admin, cancel a rival's bookings, hoard rooms as someone else | Passes the gate with their own tenant account, then may pick any user |
| Well-meaning admin | Accidental mass cancellation or wrong import | Privileged |
| Former employee or contractor with the URL | Continued access after leaving | Blocked when the tenant account is disabled; possible only if the account stays active |
| Tenant guest or other non-pilot tenant member | Reach the pilot because they hold a tenant account | Passes the gate unless user assignment is required |
| Malicious or careless file supplier | Craft a CSV that injects formulas or exhausts the importer | Authenticated as admin (or an impersonator) |
| Compromised administrator machine (later phase) | Token or session theft | Authenticated |

## Attack Surface Map

| Surface | Type | Authentication | Data Sensitivity | Notes |
|---------|------|----------------|-----------------|-------|
| Internet edge to the web application | External boundary | App Service authentication (Easy Auth) against the company Entra tenant; then pick-a-user inside the app (ADR-0005, ADR-0021) | PII (names, emails), booking history | Dominant exposure, now gated |
| Easy Auth gate and Microsoft Entra ID sign-in (pilot) | External boundary and platform component | OpenID Connect handled by the platform; single-tenant app registration | Identity claims; sign-in name and object identifier passed to the app | Configuration correctness is the control (SEC-12); client IT dependency |
| Web application (Razor Pages UI and handlers) | Internal container | Signed cookie holding the selected person id | PII, booking data | Authorisation is a server-side check against the selected identity |
| Pick-a-user cookie | Session token | Signed and encrypted; carries only a person id | Low, but grants identity | Forgeable if signing key is exposed or if the cookie is unsigned |
| Admin pages (rooms, people, bookings, import) | Internal function | Role from database | PII, integrity of all bookings | One click from any gated user under pick-a-user |
| CSV upload channel | External boundary (file) | Admin role via selected identity | PII (emails), bookings | Untrusted input; formula-injection and resource concerns |
| Status sweeper (hosted service) | Internal container | None (in-process) | Booking status | No inbound surface |
| Azure SQL Database | Internal data store | Managed identity, TLS | PII, all bookings | Not internet-exposed if firewall is restricted to Azure services or the app |
| Application Insights and logs | Internal platform service | Instrumentation key/connection string | Operational data; possible PII | Retention and content need control |
| Cookie key ring storage | Internal secret | Database access | Signing key | Protects cookie integrity |
| Microsoft Entra ID as application sign-in (future) | External boundary, future | OpenID Connect | Identity claims | Not in the pilot application; assessed for the design path |
| Deployment pipeline and infrastructure code | Supply chain | Azure DevOps Pipelines (ADR-0023); service connection to the client subscription | Source and deploy rights | Out of scope for design assessment; noted in open questions |

## Attack Vectors

### Internet edge, gate and web application

| Vector | Description | STRIDE | Threat Actor | Likelihood |
|--------|-------------|--------|--------------|------------|
| Direct access with no authentication | Anyone reaching the site opens the picker, chooses an admin and has full control | S, E | External opportunist; former employee | Low with the gate in place and correctly configured; High if the gate is missing or misconfigured (see SEC-12) |
| Gate misconfiguration or bypass | "Allow unauthenticated" left on, multi-tenant issuer chosen, a path excluded, or a second ingress or the test environment left ungated, exposing the picker | S, E | External opportunist | Low to Medium (verified by a pre-go-live reachability check) |
| Access by tenant members outside the pilot group | Any tenant member or guest signs in through the gate, because the gate is tenant-wide unless user assignment is required | S, I | Tenant guest; curious colleague | Medium tenant-wide; Low with user assignment required |
| Impersonation of any colleague | A gated user picks another person and books, cancels or reads that person's bookings | S | Curious colleague | High (accepted by ADR-0005) |
| Cookie tampering | User edits the picker cookie to select another person or role | S, E | Curious colleague | Low if signed and role is loaded server-side; High if the app trusts the cookie's role |
| Cross-site request forgery | A page on another site causes a signed-in browser to submit a cancel, deactivate or import request | T | External opportunist | Medium (the gate does not replace anti-forgery tokens) |
| Reading other users' bookings by changing an identifier | A booking id in a URL is changed to another person's booking | I | Curious colleague | Medium |
| Booker exposed in availability results | Availability view returns owner details that employees must not see | I | Curious colleague | Medium |
| Admin action attributable only to the picked identity | An admin action is performed while impersonating; without gate identity capture the audit trail shows only the selected identity | R | Curious colleague | High without gate identity capture; Low with it |
| Request flooding or heavy queries | Repeated availability searches or uploads exhaust a small App Service and database tier | D | External opportunist; gated user | Low |
| Test-only clock control reaching production | A debug endpoint that moves the clock is deployed by mistake and used to force no-shows or bypass the 2-hour rule | T, E | External opportunist; colleague | Low |
| Identity provider outage | Microsoft sign-in unavailable, so nobody can reach the app | D | Platform | Low |

### CSV upload channel

| Vector | Description | STRIDE | Threat Actor | Likelihood |
|--------|-------------|--------|--------------|------------|
| Formula injection | A cell such as `=HYPERLINK(...)` is echoed to the rejection report or downloaded CSV and executes when opened in a spreadsheet | T | Malicious file supplier | Medium |
| Oversized or malformed file | Very large file, deeply repeated rows or invalid encoding exhaust memory or CPU | D | Malicious or careless file supplier | Low |
| Import by an impersonator | A gated person picks an admin and runs the import to fill rooms with bookings | T, E | Curious colleague | Medium (linked to impersonation) |
| Raw file retention | The uploaded file with names and emails is stored beyond need | I | Any reader of storage | Low |
| Unrecorded import | Import runs without a record of who and when | R | Any | Low (audit event and ImportRun recorded, including the gate identity) |

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
| Emails, names and gate sign-in names written to logs | Telemetry captures request bodies, full emails or the gate identity headers | I | Anyone with log access | Medium |
| Log injection or telemetry spoofing | Crafted input creates misleading log lines | T, R | Any | Low |

### Microsoft Entra ID as application sign-in (future phase)

| Vector | Description | STRIDE | Threat Actor | Likelihood |
|--------|-------------|--------|--------------|------------|
| Account matching by unverified email | A user in another tenant or with an unverified email claim signs in and is matched to an existing person | S, E | External opportunist | Medium if tenant restriction and object-identifier binding are omitted |
| Token validation errors | Wrong issuer, audience or signature handling accepted | S | External opportunist | Low with standard library |
| Redirect URI misconfiguration | Tokens returned to an unintended site | I | External opportunist | Low |

### STRIDE Coverage Matrix

| Trust Boundary | S — Spoofing | T — Tampering | R — Repudiation | I — Information Disclosure | D — Denial of Service | E — Elevation of Privilege |
|----------------|-------------|--------------|----------------|---------------------------|----------------------|---------------------------|
| Internet edge to web application | Covered (SEC-01, SEC-02, SEC-12) | Covered (SEC-03, SEC-11) | Covered (SEC-05) | Covered (SEC-04) | Covered (SEC-08) | Covered (SEC-01, SEC-02) |
| Easy Auth gate and Entra ID sign-in (pilot) | Covered (SEC-01, SEC-12) | Covered (SEC-12: configuration integrity) | Covered (SEC-05: gate identity recorded) | Covered (SEC-09: gate identity in logs) | Gap, accepted: an Entra outage blocks access only; data is unaffected and access returns when restored | Covered (SEC-01: in-tenant impersonation) |
| CSV upload channel | Covered (SEC-01) | Covered (SEC-06) | Covered (audit event and ImportRun; no finding) | Covered (raw file discarded; see integration assessment) | Covered (SEC-08) | Covered (SEC-01) |
| Web application to Azure SQL | Covered (managed identity; no finding) | Covered (parameterised queries; no finding) | Covered (SEC-05) | Covered (SEC-10) | Covered (SEC-08) | Covered (least-privilege database role, listed in mitigations) |
| Web application to Application Insights | Gap, accepted: spoofed telemetry has negligible impact | Gap, accepted: log injection is low impact | Gap, accepted: platform logs are not the audit record (audit events are in the database) | Covered (SEC-09) | Gap, accepted: telemetry is sampled and cannot block the app | Gap, accepted: not applicable |
| Web application and Microsoft Entra ID as application sign-in (future) | Covered (SEC-07) | Covered (SEC-07: token validation) | Covered (audit of sign-in linking) | Gap, accepted: only minimal claims are read | Gap, accepted: identity outage blocks sign-in only; users regain access when restored | Covered (SEC-07) |

## Vulnerability Findings

| # | Finding | Location | Severity | CWE | Mitigation | ADR |
|---|---------|----------|----------|-----|------------|-----|
| SEC-01 | Pick-a-user provides no authentication. Without a gate, anyone who can reach the site can act as an admin. **Now gated:** the only External path is a Microsoft sign-in restricted to the company tenant. **Residual:** any person who passes the gate can still pick any user, including an admin, and read, change or cancel all bookings, manage people and rooms, and run the import | Internet edge, all containers | **Medium residual** (High before ADR-0021; Low external exposure if user assignment is required) | CWE-306, CWE-284, CWE-287 | **Accepted mitigation:** tenant-only App Service Authentication (Easy Auth) in front of production and test, access gating only (ADR-0021). Recommended hardening: "user assignment required" for the pilot users. Do not expose with real data until the gate is verified; disable the import page after the pilot start. In-tenant impersonation is the accepted ADR-0005 limitation | ADR-0021 (Accepted) |
| SEC-02 | The identity cookie could be forged or its role trusted, allowing selection of any person or role beyond what the picker offers | Pick-a-user cookie, web app | Medium | CWE-565, CWE-345, CWE-602 | Signed and encrypted cookie holding only the person id; on each request load person and role from the database; picker lists only active people; Secure, HttpOnly, SameSite attributes; short lifetime | ADR-0022 |
| SEC-03 | Cookie-based identity without request forgery protection allows another site to trigger cancellations, deactivations or imports | Web application | Medium | CWE-352 | Anti-forgery tokens on every state-changing request; SameSite=Lax or Strict; state changes only via POST | — |
| SEC-04 | Booking and person data could leak across users: booking-id guessing, or availability responses that include owner details | Web application read paths | Medium | CWE-639, CWE-200, CWE-285 | Owner-or-admin check in every booking handler; separate read models so availability never selects the owner; boundary tests for NFR-010 and NFR-012 | — |
| SEC-05 | Audit trail cannot prove who acted because identity is picked, not proven. **Improved:** the gate supplies an authenticated tenant identity, so each action can be traced to a real account. **Residual:** the gate identity proves who signed in, not that the picked person is who they claim to be; shared or lent tenant accounts remain outside the design | Audit events, booking records | **Low** (was Medium; conditional on the capture being built) | CWE-778, CWE-345 | Record on every audit event the gate sign-in name and immutable object identifier (read-only, never used for authorisation) next to the picked person, time and origin (NFR-020); store a mismatch indicator so admins can review cases where the gate account and picked person differ, without blocking them; log picker switches; state the limitation in the pilot brief; real sign-in in next phase | ADR-0021 |
| SEC-06 | CSV values that begin with `=`, `+`, `-` or `@` are echoed into the rejection report and downloadable CSV and could execute when opened in a spreadsheet | Import service | Medium | CWE-1236 | Prefix or escape such cells in HTML and CSV output; validate every field against strict patterns before echoing | — |
| SEC-07 | Future Entra ID linking by email alone could let an unrelated or unverified account take over a person record | Entra ID integration (future) | Medium | CWE-290, CWE-287 | Single-tenant registration; link once on verified email, then bind to immutable object identifier and tenant; refuse unmatched accounts; standard token validation | ADR-0022 |
| SEC-08 | No rate limiting or size limits on searches and uploads on a small hosting tier | Web application, import | Low | CWE-770, CWE-400 | Request size limits, upload size and row caps, basic rate limiting middleware; the gate reduces exposure to authenticated users | — |
| SEC-09 | Emails, names or gate sign-in names may appear in logs and telemetry, spreading personal data to more people and longer retention | Application Insights | Low | CWE-532 | Log person id rather than email; keep gate identity in the audit table, not in telemetry; do not log request bodies or identity headers; sampling and a short retention period | — |
| SEC-10 | Cookie key ring stored in the database unprotected: anyone with database read can forge cookies | Azure SQL | Low | CWE-312, CWE-321 | Restrict database access to the app identity and named administrators; consider protecting the key ring with a Key Vault key when Key Vault is introduced with Entra ID | — |
| SEC-11 | A test-only clock control needed for E2E tests could be enabled in production | Web application, deployment | Low | CWE-489, CWE-16 | Compile the feature out of release builds or gate it on the test environment name; verify in pipeline that production has it disabled | — |
| SEC-12 | The gate is only as strong as its configuration: allowing unauthenticated requests, a multi-tenant issuer, an excluded path, an ungated test environment or another ingress would reopen the ADR-0005 exposure. Easy Auth also protects only the web application, not other App Service endpoints such as the deployment console | Easy Auth configuration, App Service, test environment | Medium | CWE-1188, CWE-16, CWE-306 | Define the gate as code (Bicep) for production and test; require authentication with a single-tenant issuer; the only excluded path is the platform health check, if used; the application uses identity headers as audit metadata only, never for authorisation, so a spoofed header cannot grant anything; secure deployment endpoints separately; automated and manual reachability check that unauthenticated requests are redirected, run in the pipeline and before go-live (NFR-009) | ADR-0021 |

Severity totals (residual): Critical 0, High 0, Medium 7 (SEC-01 residual, 02, 03, 04, 06, 07, 12), Low 5 (SEC-05, 08, 09, 10, 11); 12 findings. Inherent severity of SEC-01 without the gate remains High.

The database firewall and TLS settings are configuration hardening (out of scope for this skill) but the design requires: SQL server not open to the internet (only the app's identity and named administrators), minimum TLS 1.2 and HTTPS-only on the App Service.

## Residual Risk After Gating

| Question | Answer |
|----------|--------|
| Can an outsider with no tenant account reach the picker? | No, if the gate is configured as designed (SEC-12 guards this) |
| Can any tenant member reach it? | Yes unless "user assignment required" is switched on for the pilot users; guests are tenant members. The hardening reduces this to the up to 50 pilot users |
| Can a gated user act as any other person, including an admin? | Yes. This is the ADR-0005 accepted limitation. Consequences: employee privacy and owner-only check-in can be circumvented; an admin's rights are one click away |
| Is the impersonation traceable? | Yes once the gate identity is captured in the audit record (SEC-05): each action shows the tenant account that performed it and the identity it claimed. It is deterrence and after-the-fact evidence, not prevention |
| What removes the remaining risk? | Real in-app sign-in bound to the Entra object identifier (ADR-0022, next phase) |

## Risk Register

| # | Risk | Inherent | Residual | Likelihood | Status | Owner |
|---|------|----------|----------|------------|--------|-------|
| 1 | Unauthenticated pilot exposed beyond the trusted group (SEC-01, SEC-12) | High | Medium tenant-wide; Low with user assignment required | Low to Medium | Mitigated by design, verification pending; client IT must permit the app registration (ADR-0021) | Client IT, with delivery lead |
| 2 | Gated colleagues can impersonate any user including admin (SEC-01, SEC-02) | Medium | Medium | High | Accepted for the pilot (ADR-0005); attributable via audit | Office Operations |
| 3 | Audit cannot prove real actor (SEC-05) | Medium | Low | Medium | Mitigated when gate identity capture is built (ADR-0021) | Delivery team |
| 4 | Request forgery and data leakage across users (SEC-03, SEC-04) | Medium | Medium | Medium | Open, mitigated by design and tests | Delivery team |
| 5 | CSV formula injection (SEC-06) | Medium | Medium | Low | Open, mitigated by design | Delivery team |
| 6 | Future Entra ID account matching (SEC-07) | Medium | Medium | Low | Open until sign-in phase | Solution Architect |
| 7 | Availability and abuse limits on small tier (SEC-08) | Low | Low | Low | Open | Delivery team |
| 8 | Personal data in logs and key material handling (SEC-09, SEC-10) | Low | Low | Low | Open | Delivery team |
| 9 | Debug clock in production (SEC-11) | Low | Low | Low | Open | Delivery team |
| 10 | Tenant Conditional Access or IT approval blocks the gate or personal phones (dependency, not a vulnerability) | n/a | n/a | Medium | Open, client IT | Project lead |

## ADR Triggers

| ADR | Finding | Status |
|-----|---------|--------|
| ADR-0021 — Pilot Access Gating for the Pick-a-user Deployment | SEC-01 (High inherent), SEC-05, SEC-12 | Accepted (2026-09-29); precondition: client IT permits the app registration |
| ADR-0022 — Current-user Seam and Entra ID Migration Path | SEC-02, SEC-07 | Proposed |

No Critical or High residual findings exist. SEC-01 was the only High finding; its ADR is now Accepted. The backlog must include stories for gate configuration and verification (SEC-01, SEC-12) and audit capture of the gate identity (SEC-05), and the production deployment must not go live before the gate is configured and verified.

## Open Questions

- [ ] Can client IT create the single-tenant app registration and grant consent, which tenant is the company tenant, and can "user assignment required" be enabled for the pilot users? Client IT. Blocks production exposure and SEC-01 closure.
- [ ] Would tenant Conditional Access (MFA, compliant device) block pilot users' personal phones? Client IT. Affects usability of the gate off-network.
- [ ] How should automated tests (E2E, ZAP, k6) pass the gate on the test environment? Recommended in the test strategy: pipeline service principal bearer token; confirm feasibility with client IT. Delivery lead.
- [ ] Should a mismatch between the gate account and the picked person ever be blocked rather than only recorded? That would amount to real authentication; project lead to decide when the sign-in phase is planned.
- [ ] Is there a company data-protection policy or legal duty (for example a data-processing register) that applies to storing names, emails and now the sign-in name in the audit trail? Office Operations; does not block build, but affects the privacy note.
- [ ] How long should logs and telemetry be kept during the pilot? Project lead (default suggestion: 30 to 90 days).
- [ ] Deferred to infrastructure review: SQL firewall rules, TLS versions, HTTP security headers, Key Vault use, deployment endpoint protection, Azure DevOps service connection scope and branch protection. Recommended step: infrastructure-as-code review before go-live.
- [ ] Deferred to CI: dependency vulnerability scanning and static analysis (in the test strategy).
- [ ] Deferred to pre-go-live: baseline web vulnerability scan against the test environment (in the test strategy).

## Sources
- *Source: arch-meeting-room-booking-v1.1.md, System Context, Component Design, Identity Design, Import Flow, Deployment*
- *Source: integration-microsoft-entra-id-v1.0.md; integration-spreadsheet-csv-import-v1.0.md*
- *Source: ADR-0005, ADR-0006, ADR-0015, ADR-0021 (Accepted), ADR-0022, ADR-0023*
- *Source: nfrs.md, NFR-009 to NFR-014, NFR-019, NFR-020*
- *Source: project lead answers, working session 2026-09-29 (off-network requirement)*
- *Note: statements about Easy Auth behaviour and identity headers are general platform knowledge to verify against current Microsoft documentation*
