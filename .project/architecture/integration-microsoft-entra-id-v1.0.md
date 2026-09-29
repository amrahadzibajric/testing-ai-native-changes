# Integration Assessment: Microsoft Entra ID (future sign-in)

**Version:** 1.0
**Date:** 2026-09-29
**Status:** Draft
**Assessed by:** Solution Architect

> Scope note: this integration is **not part of the pilot**. It is assessed now so the pilot design keeps the door open (ADR-0006, ADR-0022). Statements about the Microsoft platform come from general platform knowledge, not from project sources, and must be verified against current Microsoft documentation and the client's tenant configuration before the sign-in phase is built.

> **Addendum 2026-09-29 (ADR-0021, Accepted):** the scope note above is partly superseded. Entra ID is now used in the pilot, but only by the Azure platform: App Service authentication (Easy Auth) is configured against a single-tenant app registration in the company tenant to gate access to the whole site. The application itself does not integrate with Entra ID in the pilot, validates no tokens and authorises nothing from the sign-in; it only records the sign-in name and object identifier in the audit trail. Client IT still needs to permit the app registration (gap 2 below now applies to the pilot, not only to the later phase), and Conditional Access policies of the tenant may affect personal phones. The in-app sign-in analysed in the rest of this document remains a later phase. This assessment should be re-issued as v1.1 when the app registration is created.

## System Overview

| Field | Value |
|-------|-------|
| System | Microsoft Entra ID (company Microsoft accounts); exact provider not yet formally confirmed by the client |
| Role in project | Authenticate users in a later phase, replacing pick-a-user; supply verified email identity *(Sources: client-qa-2026-09-29.md, Answer 9; ADR-0006)* |
| Owner | Client (tenant administered by client IT); platform by Microsoft |
| Current maturity | Production platform; project integration is Planned (P2) *(Source: integration-points.md)* |
| Documentation | Complete (Microsoft-published; to be verified at build time) |

## Integration Surface

| Aspect | Details |
|--------|---------|
| Interface type | OpenID Connect authorisation code flow with PKCE, executed by the ASP.NET Core application (server-side web app) |
| Authentication | Application registration in the client's tenant, single-tenant; client secret or certificate held by the app, or federated credential with managed identity where supported |
| Data format | JSON web tokens (ID token claims: object identifier, tenant, name, email/preferred username) |
| Rate limits | Not a practical concern for up to 50 users |
| SLA / Availability | Microsoft publishes a service level for Entra ID; applicable level depends on the client's licence: [Unknown — to be confirmed] |
| Sandbox available | Client's own tenant or a developer tenant; [Unknown — to be confirmed with client IT] |

## Capabilities vs. Requirements

| Project Requirement | System Capability | Gap? |
|---------------------|-------------------|------|
| Authenticate company users | Entra ID sign-in for members of the tenant | None |
| Identify person by email (ADR-0006) | Email / UPN claims available; email claim may be absent or unverified for some accounts | Partial: match on email once, then bind to immutable object identifier (ADR-0022) |
| Keep roles (employee, admin) in the app | Roles are held in the application database; no group design needed | None |
| Restrict to company staff | Single-tenant registration; optional "assignment required" | None |
| Phone and desktop browsers | Standard browser redirect flow | None |
| No directory sync (ADR-0004) | Not needed; people remain admin-managed | None |
| Session lifetime | Cookie session in the app after sign-in | None (design choice) |

## Gaps and Risks

| # | Gap or Risk | Severity | Mitigation or Workaround |
|---|-------------|----------|--------------------------|
| 1 | Client says "Microsoft company accounts"; Entra ID is inferred | Low | Confirm with client IT before the sign-in phase |
| 2 | App registration and consent require a tenant administrator | Medium | Raise early with client IT; plan lead time in the sign-in phase |
| 3 | Email-only account matching can be spoofed if a non-verified email claim is trusted | Medium | Match verified email once at first sign-in, then bind to object identifier and tenant (ADR-0022); single-tenant only |
| 4 | A person's email changes (name change) after linking | Low | Admin-controlled email edit; identity binding stays on object identifier |
| 5 | A signed-in Microsoft account with no matching active person | Low | Refuse with a message to ask an admin to add the person; no self-registration |
| 6 | Introducing sign-in changes the ADR-0005 accepted-risk posture | Low | Retire the picker in production; record via an ADR when the phase is planned |

## Readiness Verdict

**Integration readiness:** Ready with caveats

Nothing needs to be integrated in the pilot. The pilot design keeps the integration inexpensive by holding all "who is the current user" logic behind one interface, keying people on unique case-insensitive email, and storing roles in the application. The caveats are organisational rather than technical: client IT must confirm that Entra ID is the provider and create a single-tenant app registration. The recommended next step is to include the confirmation question in the next client review and to schedule the sign-in phase after pilot evaluation.

## Dependencies and Timeline Impact

Not on the pilot's critical path. It is a P2 item; when scheduled, the effort is small (one new `ICurrentUser` implementation, first-sign-in linking, one nullable field on Person, removal of the picker from production). Tenant administrator involvement is the main lead-time dependency. The optional platform-level gating in ADR-0021 (Option B) uses the same app-registration mechanism and could serve as an early stepping stone.

## Open Questions

- [ ] Is Microsoft Entra ID the provider, and which tenant? Client IT, before the sign-in phase.
- [ ] Can client IT create a single-tenant app registration and grant consent? Client IT, before the sign-in phase.
- [ ] Are all pilot users' emails identical to their sign-in accounts (no aliases)? Office Operations, when people are added.
- [ ] Should the picker remain in a test environment after sign-in arrives? Project lead.

## Sources
- *Source: client-qa-2026-09-29.md, Answer 9*
- *Source: ADR-0005, ADR-0006, ADR-0022 (Proposed)*
- *Source: integration-points.md, Microsoft identity*
- *Source: people-and-identity.md, Microsoft-ready Design*
