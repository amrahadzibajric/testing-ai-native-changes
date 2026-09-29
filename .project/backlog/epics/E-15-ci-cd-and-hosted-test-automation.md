# E-15: CI/CD and Hosted Test Automation

**EPIC ID:** E-15
**Status:** Deferred — Post-POC (ADR-0024)
**Delivery Phase:** Post-POC (hosted pilot)
**Dependencies:** E-12 (environments), E-13 (SQL Server fixture), E-14 (gate, for hosted tests); Azure DevOps organisation and repository location decided
**ADR References:** ADR-0017, ADR-0021, ADR-0023 (Accepted)
**Architecture References:** arch v1.2 Target State (Target CI/CD); test-strategy v1.2 Target state (pipeline stages, deployed test environment, passing the access gate); security-assessment v1.2 (Open Questions: deferred scanning)

## Description

This deferred EPIC introduces Azure DevOps Pipelines as the CI/CD platform decided in ADR-0023: pull-request validation, merge-to-main deployment to the test environment with gate-aware E2E smoke tests, a nightly full run with performance and security baselines, and a manually approved production stage. It also covers the Azure DevOps organisation, repository and service connection, and the hosted performance and security testing that the POC replaced with optional local runs. It depends on client IT for the service connection and for how automated tests obtain a token for the access gate.

## Scope Boundaries

- **Includes:** Azure DevOps organisation, repository, service connection; multi-stage YAML pipelines; gate access for tests; k6, ZAP and dependency scanning in the pipeline.
- **Excludes:** environment provisioning (E-12); the gate itself (E-14).

## Key Technical Components

Azure DevOps Pipelines, service connection (workload identity federation where allowed), Testcontainers for SQL Server, Playwright, k6, OWASP ZAP.

## Key User Roles Affected

Delivery team, client IT (service connection), project lead (approvals).

## Done Criteria

- Every pull request runs build, unit, integration and API tests with SQL Server containers and a dependency scan.
- A merge deploys to the test environment and runs the E2E smoke set through the gate.
- Nightly runs execute full E2E, k6 and ZAP; production deploys only by manual approval outside office hours.
- Pipeline verification confirms no clock control exists in the build and unauthenticated requests are redirected to sign-in.

## Features

| Feature | Description | Stories | Surfaces |
|---------|-------------|---------|----------|
| [F-15.01: Azure DevOps Organisation, Repository and Service Connection](../features/_deferred/F-15.01-azure-devops-organisation-repository-and-service-connection.md) | Platform setup and deployment permission | 3 | Azure DevOps |
| [F-15.02: Multi-stage Pipelines](../features/_deferred/F-15.02-multi-stage-pipelines.md) | PR validation, deploy to test, production approval, gate access for tests | 4 | Azure DevOps |
| [F-15.03: Hosted Performance and Security Testing](../features/_deferred/F-15.03-hosted-performance-and-security-testing.md) | Nightly k6, ZAP and scanning | 3 | Azure DevOps |
