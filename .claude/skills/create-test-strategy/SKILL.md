---
name: create-test-strategy
description: Evaluate and recommend test frameworks, tools, and infrastructure for a project. Defines test levels (integration, API/contract, E2E/UI, performance, security), CI integration, coverage targets, and test data strategy. Produces the authoritative test automation strategy consumed by the QA agent.
allowed-tools: Read, Write, Edit, Glob, Grep
metadata:
  user-invocable: true
---

# Skill: Create Test Automation Strategy

## Purpose

Produce a **Test Automation Strategy** document that gives the QA agent and the delivery team an authoritative, project-specific playbook: which test levels apply, which frameworks and tools to use at each level, how tests integrate into CI/CD, and how test data is managed.

This document is the single source of truth for all QA tool and framework decisions. The QA agent reads it before writing any test plan or test code. Test-writing standards (level, volume, naming, comments, skips, traceability) live in the `test-standards` skill; this document never restates them.

**Scope of this strategy:** Integration, API/Contract, E2E/UI, Performance, and Security test levels. Unit testing is developer-owned and is not covered here — it is the implementation team's responsibility, not QA's.

---

## Invocation

Use this skill when:
- An architecture document exists and the QA pipeline is about to start
- The user asks to define, produce, or update the test automation strategy
- The project technology stack changes in a way that affects test tooling
- NFRs are updated with new measurable targets (performance, security)
- The user says "create the test strategy", "define test tooling", "what frameworks should QA use"

---

## Phase Position

This skill runs at **Phase 4** in the elaboration workflow — alongside or just after the architecture document — and before the feature pipeline begins. It requires:
- An architecture document in `.project/architecture/`
- NFRs in `.project/product-definition/nfrs.md`
- Integration assessments in `.project/architecture/integration-*` (if external systems are in scope)

---

## Owner

**Solution Architect** is the primary owner. The skill can also be invoked directly.

---

## Inputs

Before producing any output, read:

1. **Architecture document** — `.project/architecture/arch-*` — for technology stack, deployment topology, and integration points
2. **NFRs** — `.project/product-definition/nfrs.md` — for measurable performance, availability, and security targets that drive test coverage requirements
3. **Integration assessments** — `.project/architecture/integration-*` — for external systems that require contract or integration testing
4. **Project context** — `.project/context/project-context.md` — for team composition, CI/CD platform, and any existing tooling constraints
5. **Existing test strategy** — `.project/architecture/test-strategy-*` — if updating rather than creating

---

## Intake Questions

Ask only the questions not already answered by available context.

**Non-interactive fallback:** when invoked from a pipeline or the user is unavailable, do not block on unanswered questions — derive answers from the architecture document and project context where possible, choose stack-consistent defaults otherwise, and record every assumed answer in the Risks & Assumptions section of the output.

1. **What is the CI/CD platform?** — e.g. GitHub Actions, Azure DevOps, Jenkins. Drives pipeline stage definitions.
2. **What languages and runtimes are in use?** — Confirms framework compatibility (e.g. .NET → NUnit/xUnit, Node → Vitest/Jest, Python → pytest).
3. **Does the team have existing framework preferences or prior tooling?** — Avoids recommending a full replacement when adoption would be costly.
4. **Are there known performance NFR targets?** — Confirm the specific targets (latency p95, throughput RPS) to size performance test scope.
5. **Is there an existing test environment or must one be provisioned?** — Affects infrastructure recommendations.
6. **Are there any compliance or audit requirements that shape security test scope?** — e.g. OWASP Top 10 mandatory, PCI-DSS, SOC2.

---

## Test Levels

This strategy covers five QA-owned test levels. For each level, the strategy specifies the framework, the scope, and the trigger that runs it in CI/CD.

| Level | What it tests | When it runs |
|-------|--------------|--------------|
| **Integration** | Behaviour observable across a service boundary — persisted outcome read back through the public API, event consumed by another service, adapter contract honoured | PR validation |
| **API / Contract** | Endpoint behaviour as the user experiences it — success outcome, named failure conditions, access boundaries; status codes are assertions, not test subjects | PR validation |
| **E2E / UI** | Full user journeys through the running application — browser-driven or API-chain flows | Merge to main / nightly |
| **Performance** | Load, stress, and soak testing against NFR throughput and latency targets | Nightly / scheduled |
| **Security** | OWASP baseline scans, dependency vulnerability scanning, auth and authorisation boundary checks | Nightly / scheduled |

---

## Output

Save to `.project/architecture/test-strategy-[project-slug]-v[N.N].md`.

### Template

```markdown
# Test Automation Strategy: [Project Name]

**Version:** x.x
**Date:** YYYY-MM-DD
**Status:** Draft | In Review | Approved

## Context

[Why this strategy exists. Link to the architecture document and NFRs that drive testing requirements. Summarize the application's technical stack so framework choices are grounded in the project's technology decisions.]

**Unit testing:** Unit tests are developer-owned and outside the scope of this strategy. Framework selection for unit tests is an implementation team decision.

---

## Test Levels

| Level | Scope | Framework / Tool | Rationale |
|-------|-------|------------------|-----------|
| Integration | Outcomes observable across a service boundary (persisted state via API, consumed events, adapter contracts) | [e.g. NUnit + TestContainers, pytest + testcontainers-python] | [Why this tool] |
| API / Contract | Endpoint behaviour per acceptance criterion, including named access boundaries | [e.g. Playwright API mode, Pact, REST Assured, SuperTest] | [Why] |
| E2E / UI | Full user journeys through the running application | [e.g. Playwright, Cypress] | [Why] |
| Performance | Load, stress, soak testing against NFR targets | [e.g. k6, Artillery, JMeter, Locust] | [Why] |
| Security | OWASP baseline, dependency scanning, auth boundary checks | [e.g. OWASP ZAP, Snyk, Trivy, npm audit] | [Why] |

---

## Framework Evaluation

[For each non-obvious framework choice, document the evaluation. At minimum include:]

### [Framework Name] for [Test Level]

- **Alternatives considered:** [list]
- **Key criteria:** [e.g. language alignment, CI speed, community support, parallel execution, reporting quality]
- **Decision:** [chosen tool and why]

---

## Test Infrastructure

### Local Development

[How developers and QA engineers run tests locally — commands, prerequisites, containers needed.]

```bash
# Integration tests (requires Docker)
[command]

# E2E tests (requires running app)
[command]
```

### CI/CD Integration

[Where tests run in the pipeline, which levels gate which stages, parallelism strategy, artifact collection.]

| Pipeline Stage | Test Levels Run | Gate (pass required?) | Timeout |
|---------------|----------------|----------------------|---------|
| PR validation | Integration, API / Contract | Yes | [e.g. 10 min] |
| Merge to main | Integration, API / Contract, E2E / UI | Yes | [e.g. 30 min] |
| Nightly | E2E / UI, Performance, Security | No (alert) | [e.g. 60 min] |

### Test Environments

[What environments exist for testing, how they are provisioned, and data isolation approach per level.]

| Level | Environment | Provisioning | Isolation |
|-------|------------|--------------|-----------|
| Integration | Local + CI (containers) | [e.g. TestContainers on-demand] | Per-test teardown |
| API / Contract | Dedicated test instance | [e.g. docker-compose] | Shared, reset between runs |
| E2E / UI | Staging / QA environment | [e.g. deployed per branch] | Tenant isolation |
| Performance | Isolated load environment | [e.g. provisioned on schedule] | No shared traffic |
| Security | Staging | [e.g. scan on deploy] | Read-only scan |

---

## Test Data Strategy

[How test data is created, seeded, and cleaned up per level.]

| Level | Strategy |
|-------|---------|
| Integration | [e.g. Seed via fixture builders in test setup; rollback after each test] |
| API / Contract | [e.g. Isolated test tenant; state reset via API or DB script before each suite] |
| E2E / UI | [e.g. Dedicated test account with known data state; seeded via admin API] |
| Performance | [e.g. Bulk seed via data generation script; isolated environment with production-scale data volume] |
| Security | [e.g. Scan-only — no data mutation; uses staging data] |

---

## Coverage Targets

Coverage is measured against acceptance criteria, not against code structure or endpoint matrices (`test-standards` §2, §6).

| Level | Coverage Target | Measurement | Rationale |
|-------|----------------|-------------|-----------|
| Integration + API | Every numbered acceptance criterion has exactly one passing test case (plus one per failure or access condition the requirements name) | Acceptance Criteria Traceability table in each story's `verification-report.md` | Product-recognisable behaviour is what ships |
| E2E / UI | One happy-path scenario per feature; a failure-path scenario only where the requirements name one | Scenario traceability in `e2e-verification-report.md` | Business-critical flows must be verified end-to-end |
| Performance | All NFR targets in `.project/product-definition/nfrs.md` with a measurable threshold | [e.g. k6 threshold assertions] | Non-negotiable SLA compliance |
| Security | OWASP Top 10 baseline; access boundaries named in acceptance criteria | [e.g. ZAP scan report + Snyk report] | Compliance and risk mitigation |

---

## Reporting & Observability

[How test results surface to the team.]

- **CI dashboards:** [e.g. GitHub Actions test summary, Azure DevOps Test Plans]
- **Failure notifications:** [e.g. Slack alert on nightly failure, PR comment on gate failure]
- **Trend tracking:** [e.g. flaky test detection via retry count reporting]
- **Artifact retention:** [e.g. test reports stored as CI artifacts for N days]

---

## Risks & Assumptions

| # | Risk / Assumption | Impact | Mitigation |
|---|-------------------|--------|------------|
| 1 | [e.g. E2E test environment not yet provisioned] | Blocks E2E test execution until environment is ready | Defer E2E tests in the deferred test registry; unblock when environment is available |
| 2 | | | |

---

## Key Decisions

[Summarise framework and tool choices made, with brief rationale. Link to full technology evaluations in `.project/architecture/tech-eval-*` where they exist.]

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Integration test framework | [tool] | [why] |
| API / Contract test framework | [tool] | [why] |
| E2E / UI framework | [tool] | [why] |
| Performance tool | [tool] | [why] |
| Security scanning | [tool] | [why] |

---

## Open Questions

- [ ] [Question — owner, due date if known]
```

---

## Updating an Existing Strategy

When updating rather than creating from scratch:

1. Read the existing `.project/architecture/test-strategy-*` document
2. Identify what changed (new NFR, new integration, technology stack update, team feedback)
3. Update only the affected sections — do not regenerate the full document unless the stack changed materially
4. Increment the version number and update the date
5. Flag any changes that break existing QA test plans or automation code — coordinate with the QA agent

---

## Validation Checklist

Before finalising output, verify:

- [ ] All five test levels are addressed (or explicitly excluded with justification)
- [ ] Framework choices are consistent with the architecture document's technology stack
- [ ] Every measurable NFR from `.project/product-definition/nfrs.md` maps to a coverage target and a performance test entry
- [ ] Every external system in integration assessments has a corresponding integration or contract test level entry
- [ ] CI/CD pipeline stages are defined with gate conditions and timeouts
- [ ] Test data strategy is defined for each level
- [ ] Unit testing is explicitly noted as developer-owned and out of scope
- [ ] The document is saved to `.project/architecture/test-strategy-[project-slug]-v[N.N].md`