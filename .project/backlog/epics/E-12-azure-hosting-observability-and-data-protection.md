# E-12: Azure Hosting, Observability and Data Protection

**EPIC ID:** E-12
**Status:** Deferred — Post-POC (ADR-0024)
**Delivery Phase:** Post-POC (hosted pilot)
**Dependencies:** POC complete (E-11); post-POC trigger, owner and date to be decided by the project lead; client IT (subscription, region, policies)
**ADR References:** ADR-0002, ADR-0015 (decision 2, operational part), ADR-0016, ADR-0017 (Proposed), ADR-0024
**Architecture References:** arch v1.2 Target State (Target Container View, Target Deployment Topology, Deployment notes); security-assessment v1.2 (SEC-09, SEC-10); nfrs.md NFR-007, NFR-008, NFR-011, NFR-014, NFR-019, NFR-022, NFR-024

## Description

This deferred EPIC hosts the application on Azure in an EU region as designed in the target state: an App Service (Linux, Basic B1, one instance, Always On) with Azure SQL, defined as code (Bicep) for test and production, with Application Insights, HTTPS-only transport, managed identity, backups and the privacy note. It is not built in the POC (ADR-0024) and remains traceable so that the parked decisions are not lost. Estimation is not possible until client IT confirms subscription, region and policy (ADR-0017 is Proposed).

## Scope Boundaries

- **Includes:** environment confirmation and Bicep; App Service configuration; managed identity and database access; observability; privacy note and EU region; backup and recovery.
- **Excludes:** the access gate (E-14); database provider switch and locking (E-13); pipelines (E-15); Entra sign-in (E-16); go-live and hypercare (E-17).

## Key Technical Components

App Service, Azure SQL Database, Application Insights and Log Analytics, Bicep templates, Key Vault (if chosen), privacy note page.

## Key User Roles Affected

Client IT (subscription, policy), Office Operations (privacy note wording), delivery team.

## Done Criteria

- Test and production environments can be created from Bicep in the agreed EU region.
- HTTPS only with TLS 1.2 minimum; managed identity connects to Azure SQL; no database password.
- Application Insights is wired with log discipline and an uptime probe.
- A privacy note page with agreed wording is published; backup and recovery meet RPO 24 hours and RTO one business day.

## Features

| Feature | Description | Stories | Surfaces |
|---------|-------------|---------|----------|
| [F-12.01: Azure Environment and Infrastructure as Code](../features/_deferred/F-12.01-azure-environment-and-infrastructure-as-code.md) | Subscription, region, Bicep, App Service, identity | 4 | Azure |
| [F-12.02: Application Insights and Hosted Logging](../features/_deferred/F-12.02-application-insights-and-hosted-logging.md) | Telemetry, log discipline, uptime probe | 3 | Azure |
| [F-12.03: EU Region and Privacy Note](../features/_deferred/F-12.03-eu-region-and-privacy-note.md) | Privacy note text, page and policy confirmation | 3 | Web, Azure |
| [F-12.04: Backup and Recovery Configuration](../features/_deferred/F-12.04-backup-and-recovery-configuration.md) | Backups and recovery runbook | 2 | Azure |
