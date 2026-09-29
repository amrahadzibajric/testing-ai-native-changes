# E-14: Tenant Access Gate

**EPIC ID:** E-14
**Status:** Deferred — Post-POC (ADR-0024)
**Delivery Phase:** Post-POC (hosted pilot; must be verified before any hosted deployment with real data)
**Dependencies:** E-12 (App Service); client IT permission for the Entra app registration
**ADR References:** ADR-0005, ADR-0021 (Accepted), ADR-0022
**Architecture References:** arch v1.2 Target State (Target Flow: First Visit Through the Access Gate); integration-microsoft-entra-id-v1.0.md; security-assessment v1.2 (SEC-01, SEC-05, SEC-12)

## Description

This deferred EPIC places the tenant-only Microsoft sign-in gate (Azure App Service Easy Auth, single-tenant, company Entra tenant) in front of the pick-a-user application so that only authenticated tenant members can reach it from any network, including phones. The gate is access gating only; pick-a-user remains the in-app identity. It also captures the gate identity read-only on every audit event so admin actions become attributable. Security findings SEC-01, SEC-05 and SEC-12 re-activate the moment hosting begins, so this EPIC is a precondition for exposing real data. It depends on client IT approving the app registration, which is the main schedule risk.

## Scope Boundaries

- **Includes:** app registration and Easy Auth as code; securing other App Service endpoints; automated reachability check; gate identity reader and audit capture; mismatch indicator; off-network and Conditional Access verification.
- **Excludes:** application-level Entra sign-in (E-16); hosting resources (E-12); pipeline gate tokens for tests (E-15).

## Key Technical Components

Easy Auth configuration, Gate identity reader, audit gate fields, oversight visibility of mismatch.

## Key User Roles Affected

Client IT (approval), all pilot users (Microsoft sign-in step), Admins (mismatch review).

## Done Criteria

- Unauthenticated requests to production and test are redirected to Microsoft sign-in; multi-tenant sign-in is impossible.
- Only assigned pilot users can pass when user assignment is required.
- Audit events carry the gate sign-in name and object id; a mismatch is informational and never blocks.
- Phones off the office network can sign in; Conditional Access does not block them.

## Features

| Feature | Description | Stories | Surfaces |
|---------|-------------|---------|----------|
| [F-14.01: Easy Auth Gate Configuration](../features/_deferred/F-14.01-easy-auth-gate-configuration.md) | App registration, gate as code, endpoint hardening, reachability check | 4 | Azure |
| [F-14.02: Gate Identity Audit Capture](../features/_deferred/F-14.02-gate-identity-audit-capture.md) | Reader, audit fields, mismatch indicator | 3 | Web, Engineering |
| [F-14.03: Conditional Access and Off-network Verification](../features/_deferred/F-14.03-conditional-access-and-off-network-verification.md) | Phone off-network checks and client communication | 2 | Azure |
