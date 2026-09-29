# Automation Candidate Criteria

Reference for the `test-standards` skill (`../SKILL.md`). Answers two questions before any test case is written: does this item belong in the test plan at all, and if it does, should it be automated or designated manual? Level, volume, naming, comments, skips and traceability are governed by `../SKILL.md`, not here.

**Unit tests are developer-owned and outside QA scope.** These criteria apply to QA-owned test levels: integration, API/contract, E2E/UI, performance, and security.

## When to Use

- Building the Feature Phase test plan — apply the Infrastructure zone classification to each story when constructing the "Test Levels by Story" table
- Beginning Phase 1 Test Design for any story — apply before extracting testable items from `requirements.md`
- A test case candidate is ambiguous — run it through the Automation Candidate Decision Tree
- Deciding whether a hard-to-automate test should be discarded entirely or designated manual
- Reviewing an existing `test-cases.md` for test cases that should not exist

## Automate: Functional Guarantees

Testable business outcomes. These add real value and are automated.

| Category | What it covers | Example |
|---|---|---|
| **Domain rules** | Business logic constraints that must be enforced | "Resource cannot be modified if status is immutable" |
| **State transitions** | Valid and invalid paths through application state | "Request moves to Approved after validation succeeds" |
| **Business events** | What the system publishes as a consequence of actions | "Compensation event is sent when workflow step fails" |
| **API contracts** | Endpoint behaviour for the acceptance criteria it serves — success outcome, named failure conditions, access boundaries. Never a status-code matrix | "Unauthenticated user cannot view orders" (asserts 401) |
| **Async workflows** | Saga steps, message routing, compensation, timeouts | "Rollback handler runs when failure event is received" |
| **Cross-service interactions** | Contracts and data consistency at integration boundaries | "Event published by one service is consumed by another as specified" |
| **Non-functional thresholds** | Measurable performance or scalability targets | "Response time ≤ 200ms p95" |

## Do Not Automate: Technical Verification & Setup

These do not add functional value — they verify that the environment is correct, not that the system works. The **Infrastructure zone classification** table below is the authoritative list. Full example strings per zone are in [`examples.md`](examples.md); the reasoning behind each discard is in [`rationale.md`](rationale.md). Read the rationale when a discard is contested — do not re-derive it.

**Rule of thumb:** if the test would still pass after replacing all your business logic with a stub that returns a hardcoded value, the test is not testing your domain — discard it.

## Automation Candidate Decision Tree

Apply each step in order. Stop at the first disqualifying condition.

```
Step 1: Does the test verify application BEHAVIOUR
        (domain rules, state transitions, API contracts, events, async flows)?
  ├─ NO  → Discard entirely. Do not write a test case. Do not mark manual.
  └─ YES → Step 2

Step 2: Can an automated assertion verify an observable outcome?
        (response status/shape, persisted state, published event, rule enforcement)
  ├─ NO, and the reason is fundamental (visual layout, hardware, cost-prohibitive) → Designate MANUAL
  └─ YES, or NO only because a dependency is not yet available → Automate (or Defer)
```

### Step 1 — Infrastructure zone classification

| Zone | Examples | Disposition |
|------|----------|-------------|
| Infrastructure / setup | DB startup, container init, migration runner, test harness setup, scaffolding / package configuration | Discard — passing test run proves setup worked |
| File / asset presence | static files, config location, health endpoints | Discard — deployment concern, not functional |
| Static analysis / linting | unused imports, undefined variables, naming conventions | Discard — linters already enforce this |
| Schema / type validation | field exists on entity, interface implemented, DI-resolvable handler | Discard — compiler proves this |
| Framework integration | ORM persists, broker publishes, serializer converts | Discard — tests third-party code, not your domain |
| DI / composition | container resolves service, factory returns type | Discard — application startup proves this |
| One-off data / seed operations | admin user created at init, reference data seeded, defaults populated | Discard — dependent business-logic tests prove it worked |
| Pattern repetition | N identical handlers or runners following the same structural pattern | Discard N-1 — test one representative instance |

Identifying the zone is not a second decision — the discard is already decided. The zone provides the justification when documenting why no test cases exist for a story.

**This table is the single source of truth for infrastructure classification.** The QA agent's Infrastructure-Only Gate (Tier 1/2) and its feature test plan "Test Levels by Story" table both defer to this classification — do not maintain parallel lists elsewhere.

### Step 2 — Automate vs. Manual

Default: **automate**. Designate manual only when a documented reason prevents automation:

| Situation | Disposition | Justification |
|---|---|---|
| Visual layout or rendering correctness | Manual | Automation tools cannot reliably assert subjective visual quality |
| Physical device or hardware interaction | Manual | Hardware dependency makes automation impossible or cost-prohibitive |
| Cost-prohibitive environment (full-scale load infra, no sandbox for licensed external system) | Deferred or Manual | Note the blocking constraint; prefer Deferred if condition will be met |
| One-off data migration correctness | Manual | Non-repeatable; automate the equivalent domain logic instead |
| Exploratory or heuristic testing | Manual | Non-repeatable by definition; record findings, not a test case |
| Rapidly changing UI under active development | Deferred | Defer until UI stabilises; do not substitute manual testing for this |

Every manual designation requires a written justification in the test case. "Too hard to automate" is not a justification.

## Related

- [`../SKILL.md`](../SKILL.md) — level, volume, naming, comments, skips, traceability
- [`examples.md`](examples.md) — full discard example strings per zone
- [`rationale.md`](rationale.md) — why each zone is discarded
