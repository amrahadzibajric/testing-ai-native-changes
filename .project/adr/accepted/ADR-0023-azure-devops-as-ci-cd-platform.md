# ADR-0023: Azure DevOps as the CI/CD Platform

**Date:** 2026-09-29
**Status:** Accepted
**Deciders:** Amra Hadzibajric (project lead)

## Source

**Type:** Internal Discussion
**Reference:** Project lead answers in working session, 2026-09-29 (Azure DevOps is the CI/CD platform). Closes the CI/CD follow-up in ADR-0017 and the "assumed Azure DevOps" assumption in arch-meeting-room-booking-v1.0.md and test-strategy-meeting-room-booking-v1.0.md
**Participants / Authors:** Amra Hadzibajric (project lead); recorded by Solution Architect

---

## Context

The architecture and test strategy assumed Azure DevOps Pipelines as the CI/CD platform and asked the delivery lead to confirm it *(Sources: ADR-0017, follow-up; arch-meeting-room-booking-v1.0.md, Open Questions; test-strategy-meeting-room-booking-v1.0.md, Risk 2)*. The project lead confirmed on 2026-09-29 that Azure DevOps is the CI/CD platform and that the delivery team works in .NET (ADR-0016). This ADR records that platform choice only. The Azure hosting region and subscription remain undecided and stay in ADR-0017 (Proposed), pending client IT.

## Decision Drivers

- Team already uses it (project lead).
- Native fit with .NET builds, Azure deployment and test-result reporting.
- Low cost at pilot scale.

## Options Considered

### Option A: Azure DevOps Pipelines (YAML, multi-stage)
- **Pros:** Confirmed team platform; first-class .NET and Azure tasks; test and code-coverage reporting built in; environments with approvals for production deployment; Microsoft-hosted agents can run Docker, which the Testcontainers-based integration tests need.
- **Cons:** Hosted parallel-job capacity for a new organisation may need a free-tier request or a purchase (to be verified); deployment into the client's Azure subscription needs a service connection granted by client IT.

### Option B: GitHub Actions
- **Pros:** Equivalent capability.
- **Cons:** Not the team's platform.

### Option C: Jenkins or other self-hosted tooling
- **Pros:** Full control.
- **Cons:** Operating burden with no benefit at this scale.

## Decision

**We will use Azure DevOps Pipelines as the CI/CD platform (Option A).**

Pipelines are defined as YAML in the repository, with the stages already defined in the test strategy: pull-request validation, merge to main with deployment to the test environment, nightly runs, and a manual pre-go-live gate. Production deployment goes through an Azure DevOps environment with approval. Deployments to Azure use a service connection based on workload identity federation where the client's tenant allows it, so no long-lived secret is stored. Infrastructure is deployed from Bicep (ADR-0017).

## Consequences

### Positive
- Removes an assumption from the architecture and test strategy; no change to the stage definitions is needed.
- Build, test, scan and deploy live in one place familiar to the team.

### Negative / Trade-offs
- The pipeline needs permission to deploy into the client's Azure subscription (client IT dependency, same conversation as region and subscription in ADR-0017).
- Automated tests against the deployed test environment must pass the Easy Auth gate (ADR-0021); the test strategy defines how.

### Risks / Follow-up Actions
- [ ] Decide where the Azure DevOps organisation and project live (delivery team's or the client's) and where the source repository is hosted (Azure Repos or another host). Not stated in the sources. Owner: project lead.
- [ ] Confirm hosted-agent parallelism (free-tier grant or purchase) for the organisation. Owner: delivery lead.
- [ ] Client IT to provide the Azure service connection or equivalent deployment permission (with ADR-0017 region and subscription).

## Change Log
- 2026-09-29: Deferred to the post-POC phase by ADR-0024 (Accepted). The local POC needs no pipeline; tests run on developer machines. Status stays Accepted and the platform choice stands for when hosting begins. The follow-up actions above are not needed during the POC.
