# ADR-0021: Pilot Access Gating for the Pick-a-user Deployment

**Date:** 2026-09-29
**Status:** Accepted
**Data Impact:** Yes
**Deciders:** Amra Hadzibajric (project lead)

## Source

**Type:** Internal Discussion
**Reference:** Project lead answers in working session, 2026-09-29 (pilot must work outside the office network, no VPN dependency). Originates from security-assessment-meeting-room-booking-v1.0.md, finding SEC-01 (High); follow-up from ADR-0005 ("Ensure the pilot deployment is not exposed beyond the trusted user group"); nfrs.md NFR-009
**Participants / Authors:** Amra Hadzibajric (project lead); recorded by Solution Architect

---

## Context

The pilot has no authentication: anyone who can reach the application can select any person, including an admin, and then read every booking, edit rooms and people, cancel any booking and run the import *(Source: ADR-0005; people-and-identity.md, PI-08)*. On the public internet this would expose names, emails and booking history and allow tampering by any visitor. ADR-0005 accepted this only for a trusted group and deferred the access control to architecture.

On 2026-09-29 the project lead confirmed that the pilot must work outside the office network: phones are used off-network and there must be no dependency on a VPN. This settles the question that kept this ADR Open, because any control that depends on the caller's network location cannot meet it.

## Decision Drivers

- The trusted-group premise of ADR-0005 must be technically enforced, not assumed.
- No sign-in is to be built into the application in the pilot (ADR-0005).
- Must work from laptops and phones anywhere, with no VPN (project lead, 2026-09-29; ADR-0003).
- Low cost and low friction for up to 50 users.
- A stepping stone toward the later Microsoft Entra ID sign-in (ADR-0006, ADR-0022).

## Options Considered

### Option A: App Service access restrictions (IP allow-list) to the office network or company VPN egress addresses
- **Pros:** No cost, no code, no user friction inside the allow-listed network.
- **Cons:** Blocks phones on mobile data and any user working from home unless on VPN.
- **Outcome: Rejected.** The pilot must work off the office network with no VPN dependency.

### Option B: Platform-level Microsoft Entra ID sign-in (App Service Authentication, "Easy Auth") restricted to the company tenant, with pick-a-user retained inside the app
- **Pros:** Only company account holders can reach the app, from any network including phones; no application code for sign-in; the platform passes the authenticated identity to the app, enabling an audit trail (see Decision); same app-registration mechanism as the later Entra ID phase.
- **Cons:** Needs an Entra app registration and consent from client IT; adds a Microsoft sign-in step for users; does not stop one colleague from picking another user.
- **Outcome: Accepted.**

### Option C: Private endpoint plus VPN or private network only
- **Pros:** No public exposure at all.
- **Cons:** Requires VPN on phones, which the project lead ruled out; highest cost and setup.
- **Outcome: Rejected.**

### Option D: Obscure URL only (no gating)
- **Pros:** Zero effort.
- **Cons:** Not a control.
- **Outcome: Rejected.**

## Decision

**We will place a tenant-only Microsoft sign-in gate in front of the application using Azure App Service built-in authentication (Easy Auth), restricted to the company Microsoft Entra ID tenant. It is access gating only; pick-a-user remains the in-app identity per ADR-0005.**

Specifically:

1. **Gate.** Easy Auth is configured on the production and test App Services with the Microsoft identity provider, a single-tenant app registration in the company tenant, and "require authentication" for all requests (unauthenticated browsers are redirected to Microsoft sign-in). Anonymous access is not allowed on any path except a platform health-check path, if one is used. Multi-tenant or "any organisation" sign-in is not permitted.
2. **Not application login.** The application does not validate tokens, store passwords or authorise anything from the gate identity. The picker and `ICurrentUser` (ADR-0022) continue to establish the acting person and role, exactly as ADR-0005 describes. This is tenant-level access gating configured on the platform, not a login feature built into the application.
3. **Audit capture (addresses SEC-05).** Where the platform supplies the authenticated identity to the application (the sign-in name and the immutable Entra object identifier, which Easy Auth passes as request headers; header names to be verified against current Microsoft documentation at build time), the application records them read-only on every audit event, alongside the acting person chosen in the picker. The gate identity is never used for authorisation in the pilot. A mismatch between the gate account and the picked person is stored and can be shown to admins for review; it does not block the action, because blocking would amount to real authentication and belongs to the Entra ID phase.
4. **Recommended hardening (not required by this decision).** Ask client IT to switch on "user assignment required" for the enterprise application and assign the pilot users (or a group, subject to licensing), so that only the up to 50 pilot users, and not every tenant member, can pass the gate.
5. **Precondition.** Client IT must permit the app registration (and any required consent). Until the gate is configured and verified, the deployment must not be exposed with real data.

## Consequences

### Positive
- Turns the ADR-0005 trusted-group assumption into an enforced boundary that works on any network, including phones, with no VPN.
- Former employees lose access when their tenant account is disabled, with no action inside this application.
- Admin actions become attributable to a real, authenticated tenant account, not only to a picked name.
- The app registration and the identity headers are a direct stepping stone to the Entra ID phase (ADR-0022).

### Negative / Trade-offs
- **Residual risk: impersonation within the tenant.** Any person who passes the gate (every tenant member, unless user assignment is required) can still pick any active person, including an admin, and act as them. The gate identifies the caller; it does not constrain whom the caller may act as. This is the ADR-0005 accepted limitation, now bounded to authenticated tenant members and traceable through the audit record, but not removed.
- Tenant guest accounts count as tenant members unless user assignment is required.
- Users see a Microsoft sign-in step (typically once per session); the tenant's Conditional Access policies (for example MFA, compliant-device rules) apply and could block unmanaged personal phones.
- The pilot now depends on Microsoft Entra ID being available for sign-in (an outage blocks access only, not data).
- Easy Auth protects only this web application. Other App Service endpoints (for example the deployment console) are separate and must be secured separately.
- Adds a client IT task and a lead-time dependency. Client-side sign-in is a form of login, so the project lead should present it to Office Operations as tenant-level gating, not as a change to "no login".

### Risks / Follow-up Actions
- [ ] Client IT: create the single-tenant app registration in the company tenant and grant consent; confirm which tenant is the company tenant. Blocks production exposure. If client IT refuses, this ADR must be reopened (Option A and C are ruled out by the off-network requirement).
- [ ] Client IT: confirm whether Conditional Access would block personal phones, and whether "user assignment required" can be used for the pilot users.
- [ ] Verify the current Easy Auth configuration and identity header names against Microsoft documentation at build time (general platform knowledge was used here).
- [ ] Add the audit fields for the gate identity (data model: AuditEvent) and a privacy note line stating that the sign-in name is recorded in the audit trail.
- [ ] Add a backlog story for gate configuration, audit capture and the pre-go-live check that the URL is unreachable without a tenant sign-in (NFR-009).
- [ ] Test environment must also be gated (it holds a test-only clock control, SEC-11); the test strategy defines how automated tests pass the gate.

## Change Log
- 2026-09-29: Moved from Open (gap/) to Accepted. Option A (IP allow-list) rejected because the pilot must work off-network with no VPN (project lead). Chosen: Option B, tenant-only Easy Auth gate as access gating only; audit capture of the gate identity added; residual in-tenant impersonation risk recorded. Previously recommended Option A conditional on a stable egress IP range; that condition is now moot.
- 2026-09-29: Deferred to the post-POC phase by ADR-0024 (Accepted). The local POC runs on localhost only with synthetic data, so there is no network exposure to gate. Status stays Accepted and the decision stands for the hosted pilot: no hosted deployment may be exposed before the gate is configured and verified. Audit gate-identity fields remain in the model and are empty in the POC. The Entra app registration request to client IT is not needed during the POC.
