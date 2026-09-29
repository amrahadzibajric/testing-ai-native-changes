# Meeting Room Booking Pilot: Architecture Document

**Version:** 1.1
**Date:** 2026-09-29
**Status:** Superseded by arch-meeting-room-booking-v1.2.md (2026-09-29)

## Change Log

| Version | Date | Change |
|---------|------|--------|
| 1.0 | 2026-09-29 | Initial architecture. Access gating (ADR-0021) open; stack and CI/CD platform proposed or assumed |
| 1.1 | 2026-09-29 | Project lead answers of 2026-09-29 applied. ADR-0021 Accepted: tenant-only Microsoft sign-in gate (App Service Easy Auth) as access gating only; IP allow-list rejected because the pilot must work off-network. ADR-0016 Accepted on the .NET basis. ADR-0023 Accepted: Azure DevOps Pipelines as the CI/CD platform (new CI/CD section). Gate identity recorded in audit events. Context, System Context, Component Design, Identity Design, Data Model, Integration Points, Deployment, NFR mapping, Key Decisions, Open Questions and Risks updated. Hosting region and subscription remain Proposed (ADR-0017) |

## Context

Office Operations books meeting rooms today through a shared spreadsheet and Slack messages, which leads to double bookings, rooms held "just in case" and no visibility of free rooms *(Source: room-booking-spec.md, Background)*. The first version is a pilot for up to 50 users with no fixed go-live date *(Sources: client-qa-2026-09-29.md, Answers 1 and 2; ADR-0001)*, hosted on Azure *(Source: ADR-0002)*, delivered as a web application usable in desktop and phone browsers *(Source: ADR-0003)*.

This document turns the six product modules (M-01 to M-06), the non-functional requirements and the ADRs into a technical design. The design principle throughout is to keep the pilot as small and cheap as its scale allows: one deployable application, one database, no messaging or caching infrastructure, and a clean seam where Microsoft sign-in can be added later.

Status of decisions used here: ADR-0001 to ADR-0010, ADR-0014, ADR-0015, ADR-0016, ADR-0021 and ADR-0023 are Accepted. ADR-0011 to ADR-0013 (import format, clean-up ownership, 30-minute remainder minimum) are Proposed by the product definition and are treated as working assumptions. The remaining technology choices (ADR-0017 hosting shape and region, ADR-0018 background processing, ADR-0019 time handling, ADR-0020 concurrency, ADR-0022 current-user seam) are recommendations recorded as Proposed ADRs. Nothing below should be read as confirmed by the client unless it cites an Accepted ADR; ADR-0016, ADR-0021 and ADR-0023 were confirmed by the project lead, and ADR-0021 still needs client IT to permit the Entra app registration.

Key inputs: `.project/product-definition/` (modules, scope, NFRs, E2E scenarios, integration points), `.project/context/project-context.md`, and the source documents in `.project/input/`.

## System Context

The system is used by two kinds of people, employees and office admins, from their own laptops and phones, on or off the office network. Every request first passes a Microsoft sign-in gate restricted to the company tenant (platform-level App Service authentication, ADR-0021). Inside the application there is no login: a user picks their name *(Source: ADR-0005)*. The application makes no calls to any other system. Its touchpoints are the sign-in gate operated by the Azure platform against Microsoft Entra ID, a one-off CSV file exported from the old spreadsheet by Office Operations and uploaded by an admin *(Source: ADR-0010, ADR-0011)*, and, in a later phase, Microsoft company-account sign-in inside the application *(Sources: client-qa-2026-09-29.md, Answer 9; ADR-0006)*. Email, calendar, Slack, HR directory and room-side devices are explicitly not integrated *(Source: integration-points.md)*.

The trust boundary that matters most is the edge of the trusted pilot group. Because pick-a-user is not authentication, that boundary is enforced by the platform gate, not by the application (ADR-0021, security assessment SEC-01). The gate works on any network, so phones need no VPN. Residual risk: anyone who passes the gate can still pick any user, including an admin; this is the ADR-0005 accepted limitation, now confined to authenticated tenant members and traceable through the audit trail.

```mermaid
flowchart TD
    subgraph Users["Users (company tenant members, pilot group up to 50)"]
        P1([Employee])
        P2([Office Admin])
    end

    subgraph Ops["Office Operations"]
        P3([Spreadsheet owner])
    end

    subgraph Gate["Access gate - ADR-0021"]
        G1["App Service authentication\nEasy Auth, company tenant only"]
    end

    SYS[[Meeting Room Booking Pilot\nAzure EU region]]

    subgraph External["External systems"]
        E1[Spreadsheet CSV export\none-off file]
        E2["Microsoft Entra ID\ncompany tenant"]
    end

    P1 -->|Search, book, edit, cancel, check in| G1
    P2 -->|Manage rooms and people, oversight, import| G1
    G1 -->|Signed-in requests only| SYS
    G1 <-->|OpenID Connect sign-in| E2
    P3 -->|Exports cleaned CSV, hands to admin| E1
    E1 -.->|CSV file, uploaded by admin| SYS
    SYS -.->|In-app OIDC sign-in, later phase| E2
```

## Component Design

### Container view

The system is a single web application (one deployable unit) backed by one relational database. Everything else is a managed Azure platform service, not a component of the design.

```mermaid
flowchart TD
    subgraph Client["Client devices"]
        BR[Browser\ndesktop or phone]
    end

    subgraph Azure["Azure EU region"]
        subgraph APP["Web application: ASP.NET Core on App Service"]
            AUTH[App Service authentication\nEasy Auth gate, company tenant]
            UI[Razor Pages UI\nresponsive]
            SVC[Application modules\nM-01 to M-06]
            BG[Status sweeper\nhosted service]
        end
        DB[(Azure SQL Database)]
        AI[Application Insights\nand Log Analytics]
    end

    ENTRA[Microsoft Entra ID\ncompany tenant]

    BR -->|HTTPS| AUTH
    AUTH <-->|OpenID Connect| ENTRA
    AUTH -->|Authenticated requests plus identity headers| UI
    UI --> SVC
    BG --> SVC
    SVC -->|EF Core, managed identity| DB
    APP -->|logs and metrics| AI
```

### Technology recommendations (sized for up to 50 users)

Status column: Accepted where an Accepted ADR exists, otherwise Proposed.

| Concern | Recommendation | Rationale | ADR | Status |
|---------|---------------|-----------|-----|--------|
| Application | ASP.NET Core (current LTS), Razor Pages, server-rendered, small amount of JavaScript for the availability grid | One project, one deploy, phone-friendly, anti-forgery built in, easiest route to Entra ID later; delivery team works in .NET | ADR-0016 | Accepted |
| Data access | Entity Framework Core with migrations | Fast to build and evolve a small schema | ADR-0016 | Accepted |
| Database | Azure SQL Database, smallest tier that passes the performance test | Cheapest managed relational option; point-in-time restore satisfies NFR-008; managed identity avoids stored database passwords | ADR-0016, ADR-0017 | Accepted (stack); tier Proposed |
| Hosting | Azure App Service (Linux, Basic B1, one instance, Always On) in one EU region | Managed, low cost | ADR-0017 | Proposed |
| Access gating | App Service authentication (Easy Auth), Microsoft provider, single-tenant, require authentication; gate identity recorded in audit | Works off-network with no VPN; only company account holders reach the app; no sign-in code | ADR-0021 | Accepted |
| Background work | In-process hosted service every ~30 seconds, idempotent updates, effective status computed on read | No extra resource, correct even if a run is late | ADR-0018 | Proposed |
| Time | UTC storage, configured office zone (assumed Europe/Berlin), injected clock | Correct across daylight saving and testable | ADR-0019 | Proposed |
| Concurrency | Per-room lock inside a transaction, single booking-write service | Guarantees NFR-005 simply | ADR-0020 | Proposed |
| Identity | `ICurrentUser` seam; picker now, Entra ID later; signed cookie holds only the person id | Sign-in is additive | ADR-0022 | Proposed |
| Observability | Application Insights with sampling; structured logs; audit events in the database | NFR-019, NFR-020 | ADR-0017 | Proposed |
| Infrastructure as code | Bicep, including the gate configuration | Repeatable, reviewable test and production environments | ADR-0017 | Proposed |
| CI/CD | Azure DevOps Pipelines (YAML, multi-stage) | Team platform; see CI/CD section | ADR-0023 | Accepted |

Not needed at this scale and deliberately excluded: API gateway, message queue, cache, search index, CDN, container orchestration, separate front-end application, microservices. Each can be added later without reshaping the module boundaries.

Indicative running cost (to be verified in the Azure pricing calculator for the chosen region; these are order-of-magnitude figures, not quotes): App Service Basic B1 in the low tens of dollars per month, Azure SQL Database Basic tier in single-digit dollars per month, Application Insights within or near the free monthly ingestion allowance at this volume. Easy Auth adds no charge on the App Service plan (to verify). Two environments (test and production) therefore land in the range of a few tens of dollars per month, with the test environment able to be stopped outside working hours.

### Component view (inside the web application)

```mermaid
flowchart TD
    subgraph WEB["Presentation"]
        PU[Pick-a-user and role header]
        PB[Availability and booking pages]
        PM[My bookings, edit, cancel, check in]
        PA[Admin pages:\nrooms, people, bookings, import]
        PP[Privacy note page]
    end

    subgraph CORE["Application core"]
        CU["ICurrentUser\nseam ADR-0022"]
        GI["Gate identity reader\naudit metadata only"]
        AZ[Authorisation policies\nemployee vs admin]
        BW[Booking write service\nlock, validate, persist]
        RULES[Booking rules\ngrid, duration, hours, capacity,\noverlap, limit, past]
        LC[Lifecycle service\nedit, cancel, check in,\nlate flag, transitions]
        RM[Room service]
        PS[People service]
        IMP[Import service]
        RPT[Oversight queries]
        CLK["Office clock\nTimeProvider plus office zone"]
        AUD[Audit recorder]
    end

    subgraph JOBS["Background"]
        SW[Status sweeper\nno-show and done]
    end

    subgraph DATA["Data access"]
        EF[EF Core repositories]
    end

    PU --> CU
    PB & PM & PA --> AZ --> CU
    PB & PM --> BW
    PM --> LC
    PA --> RM & PS & IMP & RPT
    IMP --> BW
    LC --> RULES
    BW --> RULES
    RULES --> CLK
    LC --> CLK
    SW --> LC
    BW & LC & RM & PS & IMP & RPT --> EF
    BW & LC & RM & PS & IMP --> AUD
    GI --> AUD
    AUD --> EF
```

Why these components exist as separate concerns:

- **Booking rules** is a pure, stateless rule set (grid, duration 30 min to 4 h, office hours, capacity, no past, overlap, 3-booking limit) so that manual booking, edit, remainder rebooking and import all enforce identical rules *(Source: booking-import.md, IM-03)*. Every refusal returns the name of the broken rule *(Source: nfrs.md, NFR-017)*.
- **Booking write service** is the only code path that creates or moves a booking. It takes the per-room lock, re-checks the rules against current data and commits (ADR-0020). This is what makes NFR-005 hold for concurrent users and for imports.
- **Lifecycle service** owns state changes. The late flag is set only when the person cancelling is the booking owner and the cancellation is less than 2 hours before start; an admin cancelling another person's booking, and room or person deactivation, never set it *(Source: ADR-0015, decision 3)*. Check-in is permitted only when the acting person is the booking owner *(Source: ADR-0015, decision 4)*. Edits and cancellations are refused once the booking has started *(Source: ADR-0014, rule 1)*.
- **Office clock** is the only source of "now" and of local-time conversion (ADR-0019). Tests substitute a fake clock.
- **ICurrentUser** is the single place the acting person and role are established (ADR-0022). No other component reads the picker.
- **Gate identity reader** reads the sign-in name and object identifier that the platform gate passes to the application and hands them to the audit recorder. It has no authorisation role: no rule, policy or page decision reads it (ADR-0021).
- **Audit recorder** records who did what, whether an admin acted, and the gate identity, on every state change and admin action *(Source: nfrs.md, NFR-019, NFR-020; ADR-0021)*.

### Module-to-component traceability

| Module | Components |
|--------|-----------|
| M-01 Room Booking and Availability | Availability and booking pages, booking write service, booking rules |
| M-02 Booking Lifecycle | My bookings page, lifecycle service, status sweeper |
| M-03 Room Management | Room service, admin room pages |
| M-04 People and Identity | Pick-a-user, `ICurrentUser`, authorisation policies, people service, gate identity reader |
| M-05 Admin Booking Oversight | Oversight queries, admin bookings pages |
| M-06 Booking Import | Import service, admin import page |

## CI/CD (Azure DevOps)

*(Source: ADR-0023; test-strategy-meeting-room-booking-v1.1.md)*

Azure DevOps Pipelines is the CI/CD platform. Pipelines are YAML in the repository.

| Stage | Trigger | What happens |
|-------|---------|--------------|
| Pull-request validation | Every pull request | Restore, build, unit tests (developer-owned), integration and API/contract tests with SQL Server containers, dependency vulnerability scan |
| Merge to main | Merge | Same checks, publish artifact, deploy Bicep and the application to the test environment, run the E2E smoke set through the gate |
| Nightly | Schedule | Full E2E, performance and security baseline against the test environment |
| Production | Manual approval on an Azure DevOps environment | Deploy the tested artifact outside office hours; verify that the production clock control is disabled and that unauthenticated requests are redirected to sign-in |

Deployment to Azure uses a service connection to the client's subscription, preferably with workload identity federation so no long-lived secret exists. Where the Azure DevOps organisation and the source repository are hosted is not stated and is an open question. Hosting region and subscription remain Proposed (ADR-0017).

## Time Handling (CET/CEST)

*(Source: ADR-0015, decision 1; ADR-0019)*

The office observes Central European Time with daylight saving; the working assumption is the IANA zone Europe/Berlin, held as configuration so it can be corrected without code change.

- **Storage.** Booking start and end are stored as UTC instants. Comparing "now" to a start or end is a plain instant comparison.
- **Rule evaluation.** Anything defined on the wall clock is evaluated after converting to office local time: the 08:00 to 20:00 office hours, Monday to Friday, the 15-minute grid, the "today" boundary, and the import window. Durations defined in elapsed time (2-hour late rule, 10 minutes before and 15 minutes after start for check-in) are computed on instants.
- **Daylight-saving transitions.** The next transitions are Sunday 25 October 2026 (clocks go back) and Sunday 28 March 2027 (clocks go forward). Both fall on a Sunday morning outside office hours, so no valid booking can sit in a skipped or repeated hour, but weekday tests, "start of day" and displayed times must still be right across the change (for example, a booking made for Monday 08:00 has a different UTC offset before and after the change). Any user input that lands on a nonexistent or ambiguous local time is refused with a clear message.
- **Display.** All screens show office local time; no time-zone selector exists *(Source: room-booking-spec.md, Not needed for now)*.
- **Testability.** The application never calls the system clock directly; it uses an injectable clock, so DST scenarios and the 10-minute, 15-minute and 2-hour boundaries are tested deterministically.

## Scheduled and Background Processing

*(Source: booking-lifecycle.md, LC-08 to LC-10; nfrs.md, NFR-006; ADR-0018)*

A hosted service in the web application wakes about every 30 seconds and applies two idempotent, set-based updates:

1. Confirmed bookings whose start plus 15 minutes has passed become No-show, with the no-show moment recorded. The remaining time then becomes bookable (the availability query already treats a no-show as occupying only the time up to the no-show moment).
2. Checked-in bookings whose end time has passed become Done.

Design points:

- **Correct even when late.** Availability, overlap and check-in logic compute an effective status from the clock: a Confirmed booking with start plus 15 minutes already passed is treated as a No-show whether or not the job has yet persisted the change. The sweeper therefore keeps stored state and reports current, but correctness of booking rules does not depend on it. This meets the 1-minute target of NFR-006 by a wide margin.
- **Safe on restart or repeat.** Updates have a status guard (`WHERE status = 'Confirmed' AND start <= threshold`), so running twice, or two instances overlapping briefly during a deployment, causes no harm.
- **No missed events.** The sweeper is state-based, not event-based; after downtime the first run catches up.
- **Always On** must be enabled so the process is not unloaded.
- **Unaffected by the gate.** The sweeper is in-process and receives no web requests, so the sign-in gate does not apply to it.
- A Confirmed booking that starts and is never checked in is never Done; a checked-in booking that is not ended manually becomes Done at its end time *(Source: booking-lifecycle.md, status diagram)*.

## Identity Design

*(Source: ADR-0005, ADR-0006, ADR-0014 rule 5, ADR-0021, ADR-0022)*

**Two layers in the pilot.** The first layer is the platform gate (ADR-0021): a Microsoft sign-in restricted to the company tenant that decides who may reach the application at all. It is configuration, not application code, and it authorises nothing inside the application. The second layer is pick-a-user (ADR-0005), which decides which person the application treats the caller as.

**Pilot: pick a user.** A start page lists active people (name and role); choosing one stores the person's identifier in a signed, encrypted cookie. On every request the server loads the person and role from the database and derives permissions from that; a role or identifier sent by the browser is never trusted for authorisation. The header always shows who the user is acting as and their role, with a "Switch user" link. All authorisation policies (employee sees only own bookings, admin-only pages, owner-only check-in) read from `ICurrentUser`.

**Gate identity is audit metadata only.** When the platform supplies the authenticated sign-in name and Entra object identifier to the application (as request headers; exact names to be verified against Microsoft documentation at build time), the gate identity reader passes them to the audit recorder, which stores them with every audit event next to the picked person. A mismatch between the gate account and the picked person is stored as an indicator that admins can review; it never blocks an action, because blocking would be real authentication and belongs to the Entra ID phase. Locally and in tests, where no gate exists, the values are empty.

**Email is the key.** Person email is unique and case-insensitive *(Source: nfrs.md, NFR-023)*, matches the import `person_email` column and will match the Microsoft account later.

**Path to Microsoft Entra ID sign-in inside the application (later phase, not built now).** Replace the picker implementation of `ICurrentUser` with an OpenID Connect implementation (single-tenant to the company directory). On first sign-in the verified email claim is matched to the existing person and the Entra object identifier and tenant are stored on the person record; subsequent sign-ins match on that immutable identifier rather than on email alone, to avoid account-matching weaknesses. Roles remain in the application database (employee or admin) so no Entra group design is needed. The picker is removed or limited to a test environment, and the platform gate can then be retired or kept. No data migration is required. See `integration-microsoft-entra-id-v1.0.md`.

**Pilot limitation, residual after gating.** The picker is not authentication. Anyone who passes the gate can act as any person, including an admin. This is accepted for a trusted group *(Source: ADR-0005)*; the gate confines it to company tenant members (to the pilot users only if "user assignment required" is enabled) and the audit trail makes it attributable. Owner-only check-in and employee privacy are enforced against the selected identity and can be circumvented by switching user.

## Data Model (conceptual, for reasoning only)

The full conceptual model is in `.project/data-model/` (Phase 5). The entities below are the minimum needed to reason about flows and storage.

| Entity | Key attributes | Notes |
|--------|---------------|-------|
| Person | id, name, email (unique, case-insensitive), role (employee or admin), active, later: external identity reference | Owned by M-04; at most about 50 rows |
| Room | id, name (unique, case-insensitive), capacity, active | Owned by M-03; seeded with Focus, Team, Board active and Old Room inactive |
| Booking | id, room, owner person, created-by person, start (UTC), end (UTC), attendees, status, checked-in at, no-show at, late-cancellation flag, cancelled-by person, cancellation origin (owner, admin action, room deactivated, person deactivated), source (manual or import), row version | Owned by M-01 and M-02; the late flag is true only when the owner cancelled less than 2 hours before start |
| AuditEvent | id, time, acting person, gate sign-in name and object identifier (from the platform gate, nullable), action, target, details | Supports NFR-019, NFR-020; gate fields added by ADR-0021 |
| ImportRun | id, admin, go-live date, window, counts, time, and rejected rows retained for the report | Supports M-06 repeatability and audit |

Ownership: one application, one database; the application owns all data. The spreadsheet is retired after import and is not a data source afterwards.

### Key flow 1: create a booking under concurrency

```mermaid
sequenceDiagram
    participant U as Employee browser
    participant W as Web app
    participant CU as ICurrentUser
    participant BW as Booking write service
    participant DB as Azure SQL

    U->>W: POST book room, date, start, end, attendees (already through the sign-in gate)
    W->>CU: Who is acting, and role?
    CU-->>W: Person and role from database
    W->>BW: Create booking request
    BW->>DB: BEGIN transaction, lock room row
    Note over BW,DB: A second simultaneous request for the same room waits here
    BW->>DB: Read overlapping active bookings and person's upcoming count
    BW->>BW: Validate grid, duration, hours, capacity, past, overlap, limit in office time
    alt All rules pass
        BW->>DB: Insert booking, audit event (with gate identity)
        BW->>DB: COMMIT
        BW-->>W: Confirmed
        W-->>U: Booking confirmed
    else A rule fails, including "room already booked"
        BW->>DB: ROLLBACK
        BW-->>W: Refused, names the broken rule
        W-->>U: Message naming the rule
    end
```

The second of two simultaneous requests for the same slot waits for the lock, then finds the overlap and is refused with a clear message, satisfying NFR-005 and E2E-001.

### Key flow 2: check in, no-show and remainder rebooking

```mermaid
sequenceDiagram
    participant Owner as Booking owner phone
    participant W as Web app
    participant LC as Lifecycle service
    participant SW as Status sweeper
    participant DB as Azure SQL
    participant Other as Another employee

    Owner->>W: Check in (10:00 booking, at 09:52)
    W->>LC: Check in request, acting person
    LC->>LC: Acting person is owner? Now within start-10 min to start+15 min?
    LC->>DB: Set status Checked in
    LC-->>Owner: Checked in

    Note over SW,DB: Alternative path when nobody checks in
    SW->>DB: Every ~30 s: Confirmed with start+15 min passed, set No-show
    Other->>W: Search Team, 10:20
    W->>DB: Availability (no-show occupies only until 10:15)
    W-->>Other: Team free from next grid point 10:30
    Other->>W: Book 10:30 to 11:30
    W-->>Other: Confirmed
    SW->>DB: Every ~30 s: Checked in with end passed, set Done
```

### Key flow 3: first visit through the access gate

```mermaid
sequenceDiagram
    participant U as Colleague browser or phone
    participant G as Easy Auth gate
    participant E as Microsoft Entra ID
    participant W as Web app
    participant DB as Azure SQL

    U->>G: GET site (any network)
    G-->>U: Not signed in, redirect to Microsoft sign-in
    U->>E: Sign in with company account
    E-->>G: Token for company tenant
    G-->>U: Session established, back to the site
    U->>G: GET site
    G->>W: Request plus gate identity headers
    W-->>U: Pick-a-user page (active people)
    U->>W: Choose a person
    W->>DB: Load person and role
    W-->>U: App as that person, audit events record gate account and picked person
```

An account outside the tenant, or not assigned when user assignment is required, is refused at Microsoft sign-in and never reaches the application.

## Import Flow (M-06)

*(Source: booking-import.md; ADR-0010, ADR-0011, ADR-0012, ADR-0014, ADR-0015 decision 5)*

```mermaid
flowchart TD
    A[Admin enters go-live date and uploads CSV] --> B{"File acceptable?\nUTF-8, header row,\nsize and row cap"}
    B -->|No| X[Refuse whole file with message]
    B -->|Yes| C["Compute window:\nstart = go-live if weekday,\nelse following Monday;\n5 working days Mon to Fri"]
    C --> D[Parse rows; sort by start time,\nties by file order]
    D --> E[For each row:\nmatch room and person by exact name and email]
    E --> F{"All booking rules pass?\nwindow, grid, hours, duration,\ncapacity, overlap, past, 3rd booking"}
    F -->|Yes| G{Already exists?\nsame room, person, start, end}
    G -->|Yes| H[Report as duplicate]
    G -->|No| I[Write via booking write service\nas Confirmed, source import]
    F -->|No| J[Add to rejection report\nwith line, values, reason]
    I --> K[Next row]
    H --> K
    J --> K
    K --> L[Show summary and rejection report,\noffer downloadable CSV of rejected rows]
    L --> M[Office Operations fixes rows;\nadmin re-imports corrected file]
    M --> A
```

Design points:

- **Nothing is changed silently.** A row is imported exactly as given or rejected with a reason *(Source: booking-import.md, IM-06)*.
- **Same write path as manual booking.** Each row goes through the booking write service (ADR-0020), so overlap and the 3-booking limit hold even if someone books manually during the import.
- **Ordering.** Rows are processed in start-time order (ties by file order). This satisfies "date/time order per person" for the limit and gives a deterministic winner where two rows overlap *(Source: booking-import.md, IM-04, IM-05)*.
- **Window.** The admin enters the go-live date; the window is 5 working days from that date, moving to the following Monday if it is a weekend *(Source: ADR-0015, decision 5)*. Rows outside the window are rejected. Rows dated earlier on the go-live day itself are rejected by the no-past rule; import before opening time to avoid this.
- **Repeatable.** Re-importing a corrected file reports already-imported identical rows as duplicates rather than creating them twice *(Source: booking-import.md, IM-09, assumption)*.
- **Size.** Files of a few hundred rows finish within the 1-minute target (NFR-004) because each row is a short transaction; a row and size cap protects the app.
- **Report safety.** Cell values echoed in the report and in the downloadable CSV are escaped against spreadsheet formula injection (security assessment SEC-06).
- **One-off.** The import page is admin-only and can be disabled by configuration after the pilot start *(Source: booking-import.md, IM-10, IM-11)*.

## Integration Points

Three touchpoints exist, all low-risk. Azure is the platform, not an integration.

```mermaid
flowchart LR
    P[[Meeting Room Booking Pilot]]

    F["Spreadsheet CSV export\nOffice Operations"] -->|"File upload via admin browser\nUTF-8 CSV, one-off"| P
    G["Easy Auth gate\nplatform component"] <-->|"OpenID Connect sign-in\npilot"| E["Microsoft Entra ID"]
    G -->|"Authenticated requests\nplus identity headers"| P
    P -.->|"OpenID Connect in the app, later phase"| E
    P -->|"Telemetry SDK, platform service"| AI[Application Insights]
```

| System | Direction | Protocol and format | Authentication | Priority | Assessment |
|--------|-----------|---------------------|----------------|----------|-----------|
| Spreadsheet CSV export | Inbound, one-off | File upload over HTTPS, UTF-8 CSV with header | Uploaded by an admin acting through the app (pilot: selected identity behind the gate) | P1 | `integration-spreadsheet-csv-import-v1.0.md` (Ready with caveats) |
| Microsoft Entra ID, platform gate (pilot) | Sign-in, performed by the Azure platform in front of the app | OpenID Connect, single-tenant app registration | App registration in the company tenant; client IT permission required | P1 | `integration-microsoft-entra-id-v1.0.md` (addendum 2026-09-29; Ready with caveats) |
| Microsoft Entra ID, in-app sign-in (later) | Inbound identity, later phase | OpenID Connect, JSON web tokens | Application registration (single tenant) | P2 | `integration-microsoft-entra-id-v1.0.md` (Ready with caveats) |
| Azure platform services (App Service, Azure SQL, Application Insights) | Platform | Azure managed services, TLS | Managed identity where supported | P1 | Covered in this document; not an integration assessment |

Explicitly not integrated: email, calendar, Slack, HR directory, room-side devices, payment or facilities systems *(Source: integration-points.md)*.

## Deployment

```mermaid
flowchart TD
    Internet([Company tenant members:\nlaptops and phones, any network])

    subgraph Region["Azure EU region - to confirm with client IT"]
        subgraph RG_PROD["Production resource group"]
            GP["Easy Auth gate\ncompany tenant only"]
            APPP["App Service Linux B1\n1 instance, Always On\nASP.NET Core app + hosted sweeper"]
            SQLP[("Azure SQL Database\nsmallest passing tier\nPITR backups, TDE")]
            AIP[Application Insights\nLog Analytics]
        end
        subgraph RG_TEST["Test resource group"]
            GT["Easy Auth gate\nsame configuration"]
            APPT["App Service B1\nstop outside hours"]
            SQLT[("Azure SQL Database\nBasic tier")]
        end
    end

    ENTRA[Microsoft Entra ID\ncompany tenant]
    ADO[Azure DevOps Pipelines\nADR-0023]

    Internet -->|HTTPS only, TLS 1.2+| GP
    GP <-->|OIDC sign-in| ENTRA
    GP --> APPP
    APPP -->|"Managed identity, TLS"| SQLP
    APPP -->|telemetry| AIP
    Internet --> GT --> APPT --> SQLT
    SQLP -.->|"Automated backups:\npoint-in-time restore"| SQLP
    ADO -->|"Bicep and app deploy via service connection"| RG_TEST
    ADO -->|"Approved deploy"| RG_PROD
```

Deployment notes:

- **Region.** An EU Azure region *(Source: ADR-0015, decision 2)*. Candidates are West Europe or Germany West Central; the choice is confirmed with client IT (ADR-0017, Proposed).
- **Availability.** Single instance, best effort during office hours *(Source: nfrs.md, NFR-007)*. There is no zone redundancy or failover; a platform restart causes a brief outage, which is acceptable. Deployment slots are not needed; deploy outside office hours.
- **Backup and recovery.** Azure SQL automated backups with point-in-time restore satisfy RPO 24 hours and RTO one business day (NFR-008), with the restore procedure documented and rehearsed once before go-live. The environment is recreated from Bicep in the same time frame.
- **Transport and storage.** HTTPS only, minimum TLS 1.2, HTTP redirected; the database encrypts data at rest by default *(Source: nfrs.md, NFR-011)*.
- **Secrets.** The application reaches the database using managed identity, so there is no database password. The cookie signing key ring is persisted in the database, so restarts and redeployments do not log everyone out. The Easy Auth provider configuration is defined in Bicep; a client secret for the sign-in app registration, if the platform requires one, is held as an App Service setting or Key Vault reference (workload-identity alternatives to be checked at build time). No other secrets exist in the pilot.
- **Access gating.** The Easy Auth gate (ADR-0021) is configured on both production and test, defined as code, with authentication required on every path except a platform health check if one is used. Anything outside the tenant, or unassigned when user assignment is required, is refused at Microsoft sign-in. Other App Service endpoints such as the deployment console are secured separately. Until the gate is configured and verified, the deployment must not be exposed for real data.
- **Privacy note.** A static page states what is stored (name, email, role, booking history, and the sign-in name recorded in the audit trail), why, who can see it (admins see all bookings; employees see their own), how long (for the pilot duration, NFR-021), and whom to contact. Text to be agreed with Office Operations *(Source: ADR-0015, decision 2)*.
- **Environments.** Test (smallest tiers; may be stopped outside working hours) and production. Test-only clock control (needed for check-in and no-show tests) is compiled out or disabled in production. Automated tests reach the gated test environment with a pipeline service principal token (see test strategy).

## Non-Functional Requirements Mapping

| NFR | Requirement | Design response | Verified by |
|-----|-------------|-----------------|-------------|
| NFR-001 | Up to 50 people | Single instance and smallest SQL tier are ample; nothing user-count sensitive | Performance test at 20 concurrent users |
| NFR-002 | A few thousand bookings | Simple indexed tables (room, start, owner); no partitioning | Performance test with seeded 5,000 bookings |
| NFR-003 | Typical page under 2 s | Server-rendered pages, indexed availability query for one day | k6 p95 threshold |
| NFR-004 | Import of a few hundred rows under 1 min | Row-by-row short transactions on the shared write path; row cap | Timed integration test with 500 rows |
| NFR-005 | No double booking, ever | Per-room lock plus transactional re-check in a single write service (ADR-0020) | Parallel-request integration test |
| NFR-006 | Automatic status changes within 1 min | 30-second sweeper plus effective status on read (ADR-0018) | Fake-clock integration test; staging observation |
| NFR-007 | Available in office hours, best effort | Single App Service instance, Always On, deploy outside hours; sign-in depends on Entra ID availability | Uptime probe (optional) |
| NFR-008 | RPO 24 h, RTO 1 business day | Azure SQL PITR; Bicep recreate | Restore rehearsal before go-live |
| NFR-009 | Not exposed beyond trusted group | Tenant-only Easy Auth gate, works off-network (ADR-0021, Accepted); residual in-tenant impersonation accepted (ADR-0005) | Pipeline and pre-go-live check that unauthenticated requests are redirected to sign-in on production and test |
| NFR-010 | Server-side role enforcement | Authorisation policies on every page and handler; role loaded from database on each request | Role-boundary integration tests |
| NFR-011 | HTTPS only | HTTPS-only setting, TLS 1.2 minimum, HSTS | Security scan |
| NFR-012 | Privacy: own bookings only; availability without booker | Separate read models: availability never selects owner; my bookings filtered by owner | Boundary tests on availability and my-bookings |
| NFR-013 | Minimal personal data | Name, email, role, booking history, and the gate sign-in name in audit events | Design review |
| NFR-014 | Data protection | EU region and privacy note (ADR-0015) | Privacy note reviewed by Office Operations |
| NFR-015 | Single office time zone with DST correctness | UTC storage, configured zone, injected clock (ADR-0019) | DST tests around 25 Oct 2026 and 28 Mar 2027 |
| NFR-016 | Desktop and phone browsers | Responsive server-rendered pages; check-in screen designed for phones | E2E on desktop and mobile viewports |
| NFR-017 | Rule failures name the rule | Rules engine returns named results | API-level tests per acceptance criterion |
| NFR-018 | Reasonable accessibility (WCAG 2.1 AA aim) | Semantic HTML, keyboard operation, contrast | Automated accessibility check in E2E |
| NFR-019 | Basic logging | Application Insights, structured logs, audit events | Log review in test |
| NFR-020 | Audit of state changes | Audit events plus actor, origin and gate identity fields on booking and audit records | Integration tests on audit content |
| NFR-021 | Retain history for the pilot | No deletion; no archival | Design review |
| NFR-022 | Azure hosting | App Service, Azure SQL | Deployment |
| NFR-023 | Email as unique case-insensitive key | Unique index with case-insensitive collation | Integration test |
| NFR-024 | Low cost | Smallest suitable tiers, no extra infrastructure | Cost check after first month |

## Key Decisions

| Decision | Status | Reference |
|----------|--------|-----------|
| Pilot, up to 50 users | Accepted | ADR-0001 |
| Azure hosting | Accepted | ADR-0002 |
| Web app; check-in from own device | Accepted | ADR-0003 |
| No login in pilot; email as key | Accepted | ADR-0005, ADR-0006 |
| Office time zone CET/CEST, EU region and privacy note, admin cancellations not late-flagged, owner-only check-in, weekend go-live rule | Accepted (confirmed by project lead) | ADR-0015 |
| ASP.NET Core modular monolith with Azure SQL | Accepted (delivery team works in .NET, project lead 2026-09-29) | ADR-0016 |
| App Service B1, Azure SQL tier, EU region, Bicep | Proposed (region and subscription await client IT) | ADR-0017 |
| In-process sweeper with effective status on read | Proposed | ADR-0018 |
| UTC storage, office-zone evaluation | Proposed | ADR-0019 |
| Per-room lock concurrency control | Proposed | ADR-0020 |
| Pilot access gating: tenant-only Easy Auth, IP allow-list rejected | Accepted (project lead 2026-09-29; client IT to permit app registration) | ADR-0021 |
| Current-user seam and Entra ID path | Proposed | ADR-0022 |
| Azure DevOps Pipelines as CI/CD platform | Accepted (project lead 2026-09-29) | ADR-0023 |
| CSV format, clean-up ownership, 30-minute remainder minimum | Proposed (product definition) | ADR-0011, ADR-0012, ADR-0013 |

## Key Architectural Trade-offs

- **Single instance versus resilience.** One App Service instance is cheapest and simplest; the cost is no failover. Accepted because availability is best effort in office hours (NFR-007) and recovery within one working day is sufficient (NFR-008).
- **In-process scheduler versus separate job.** Running the sweeper in the web process avoids a second resource. The cost is coupling; mitigated by idempotent updates and effective status on read (ADR-0018).
- **Lock discipline versus database constraint.** The per-room lock gives correctness with minimal machinery but relies on one write path; a slot table could be added if more write paths appear (ADR-0020).
- **Pick-a-user behind a tenant gate versus real security.** Speed and scope against a real exposure. The gate keeps outsiders out on any network and makes actions attributable, but a gated user can still act as anyone. The architecture bounds the risk with access gating (ADR-0021) and prepares the seam for real sign-in (ADR-0022). Adding the gate costs a client IT task and a Microsoft sign-in step for users.
- **Gate identity recorded but not enforced.** Recording the gate account beside the picked person gives evidence for later review; enforcing a match would turn the pilot into a login system and is deferred to the Entra ID phase.
- **Server-rendered versus SPA.** Fewer moving parts and no token handling now; less interactive. Adequate for the scale and an easy fit for the later sign-in.

## Open Questions

- [ ] Confirm the exact IANA time zone name (assumed Europe/Berlin) with Office Operations. Owner: project lead.
- [ ] Choose the Azure EU region and confirm subscription, policies and cost centre with client IT (ADR-0017). Owner: project lead with client IT.
- [ ] Can client IT create the single-tenant Entra app registration for the gate (ADR-0021), which tenant is the company tenant, and can user assignment be required for the pilot users? Would Conditional Access block personal phones? Owner: client IT. Blocks production exposure.
- [ ] Where do the Azure DevOps organisation and source repository live (ADR-0023), and who provides the deployment service connection? Owner: project lead with client IT.
- [ ] Client confirmation of ADR-0011 to ADR-0013 and of ADR-0015. Owner: project lead.
- [ ] Agree the privacy note wording, including the sign-in name recorded in the audit trail, and whether any company data policy requires more. Owner: Office Operations.
- [ ] Availability and backup assumptions (NFR-007, NFR-008) to be confirmed as acceptable. Owner: Office Operations.
- [ ] Is Microsoft Entra ID the exact future provider for in-app sign-in? Expected yes, since the gate now uses it (ADR-0021). Owner: client IT.
- [x] Confirm the delivery team's technology preference. Confirmed: .NET (ADR-0016).
- [x] Confirm the CI/CD platform. Confirmed: Azure DevOps (ADR-0023).
- [x] Decide access gating. Decided: tenant-only Easy Auth (ADR-0021).

## Risks

| # | Risk | Likelihood | Impact | Mitigation |
|---|------|------------|--------|------------|
| 1 | Gate missing or misconfigured, exposing the picker to non-staff | Low | High | Gate as code on production and test; reachability check in pipeline and before go-live; no real data until verified (ADR-0021, SEC-01, SEC-12) |
| 2 | Any gated colleague can impersonate an admin | High | Medium | Accepted by ADR-0005 for a trusted group; gate identity recorded in audit events makes it attributable; optional user assignment narrows the group; real sign-in in the next phase |
| 3 | Spreadsheet data is dirty and many rows are rejected | Medium | Medium | Rejection report; Office Operations owns clean-up (ADR-0012, Proposed); allow time for re-import |
| 4 | Time zone assumption or DST handling wrong | Low | Medium | Zone configurable; fake-clock DST tests; confirm zone name |
| 5 | Sweeper stops (process idle or crash) | Low | Low | Always On; effective status on read keeps rules correct; alert on missed heartbeat |
| 6 | Client IT delays or refuses the app registration, or Conditional Access blocks personal phones | Medium | High | Raise early; the off-network requirement rules out IP allow-list and VPN alternatives, so refusal would reopen ADR-0021 |
| 7 | Estimated Azure costs differ from expectations | Low | Low | Verify in pricing calculator; stop test environment out of hours |
| 8 | Go-live date slips, moving the import window | Medium | Low | Window is computed from the go-live date entered at import; import run just before go-live |
| 9 | Rules confirmed only by the project lead differ from client expectation | Low | Medium | Show ADR-0014 and ADR-0015 to Office Operations at next review |
| 10 | Automated tests cannot pass the gate on the test environment | Medium | Medium | Pipeline service principal token accepted by the gate; confirm with client IT; fall back to a gated-off test environment only if it is not internet-reachable |

## Sources

- *Sources: room-booking-spec.md; client-qa-2026-09-29.md*
- *Source: intake-meeting-room-booking-2026-09-29.md*
- *Sources: .project/product-definition/modules (M-01 to M-06), scope.md, nfrs.md, e2e-scenarios.md, integration-points.md*
- *Sources: ADR-0001 to ADR-0023 (ADR-0011 to ADR-0013, ADR-0017 to ADR-0020 and ADR-0022 Proposed)*
- *Source: project lead answers, working session 2026-09-29 (off-network requirement; .NET; Azure DevOps)*
