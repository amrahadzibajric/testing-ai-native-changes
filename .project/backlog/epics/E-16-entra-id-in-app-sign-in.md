# E-16: Entra ID In-App Sign-in

**EPIC ID:** E-16
**Status:** Deferred — Post-POC (ADR-0024); later phase after the hosted pilot
**Delivery Phase:** Post-POC (sign-in phase; not part of the hosted pilot itself)
**Dependencies:** E-14 (same app registration mechanism), E-12; the `ICurrentUser` seam from E-04 (kept in the POC)
**ADR References:** ADR-0005, ADR-0006, ADR-0022 (Proposed)
**Architecture References:** arch v1.2 (Identity Design — Path to Microsoft Entra ID sign-in); integration-microsoft-entra-id-v1.0.md; security-assessment v1.2 (SEC-07)

## Description

This deferred EPIC replaces pick-a-user with Microsoft company-account sign-in inside the application by adding an OpenID Connect implementation of `ICurrentUser` (single tenant), linking accounts to people first by verified email and afterwards by the immutable Entra object identifier. Roles remain in the application database and no data migration is needed because email was the key from the start. The privacy and admin restrictions that are enforced against the selected identity in the pilot then become genuinely enforced. The seam is built in the POC (E-04), so this EPIC adds an implementation, not a redesign.

## Scope Boundaries

- **Includes:** OIDC implementation of `ICurrentUser`; account linking; picker retirement or test-only restriction; verification with a stubbed provider and a developer tenant.
- **Excludes:** directory synchronisation, multi-factor logic beyond tenant policy, groups (out of scope for M-04).

## Key Technical Components

OIDC authentication, `ICurrentUser` Entra implementation, Person external identity fields, authorisation policies (unchanged).

## Key User Roles Affected

All users (sign-in), Admins (link status), client IT (app registration).

## Done Criteria

- Users sign in with company accounts; the acting person and role come from the database via `ICurrentUser`.
- The first sign-in links by verified email and later sign-ins match on object id and tenant; an unrelated account cannot take over a person record.
- The picker is removed or limited to a test-only environment.

## Features

| Feature | Description | Stories | Surfaces |
|---------|-------------|---------|----------|
| [F-16.01: Entra ID Current-User Implementation](../features/_deferred/F-16.01-entra-id-current-user-implementation.md) | OIDC implementation, account linking, role handling | 3 | Web, Phone browser |
| [F-16.02: Sign-in Cutover and Picker Retirement](../features/_deferred/F-16.02-sign-in-cutover-and-picker-retirement.md) | Configuration switch, verification and retirement | 2 | Web, Phone browser |
