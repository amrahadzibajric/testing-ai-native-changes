---
name: test-standards
description: The single standard for test cases and test code — test level, volume, naming, comments, skips, traceability to acceptance criteria. Governs every QA-owned test (integration, API/contract, E2E/UI, messaging, performance, security) and the naming, comment and skip rules for developer-owned unit tests. Referenced by the QA, Implementation and Solution Architect agents; never restated elsewhere.
allowed-tools: Read, Glob, Grep
license: "AP AI-Native Delivery Toolkit v3.5 - AP Confidential - governed by LICENSE.md"
metadata:
  user-invocable: true
---

# Skill: Test Standards

## Purpose

One page that says what a good test is in this toolkit. Every test case in `test-cases.md`, every E2E scenario, and every line of test code is measured against it. Agents and skills **reference** this file; they do not copy its rules.

Which items are infrastructure and therefore never tested is decided in [`references/candidate-criteria.md`](references/candidate-criteria.md) (zone table, decision tree, manual designation). This file assumes that filter has already run.

## When to Use This Skill

- QA agent — before extracting test sources, before writing each test case, before writing test code, before producing a verification report
- Implementation agent — when writing unit tests (§3–§5 apply)
- Solution Architect — when writing coverage targets in `create-test-strategy`
- Reviewing an existing `test-cases.md` or test suite for cases that should not exist, wrong names, comment noise, or hidden skips

## 1. Level — a test states something Product would recognise

- Derive tests from the **numbered acceptance criteria** (`AC-n`) in `requirements.md` first, functional requirements (`FR-nnn`) second. `spec.md` supplies only the contract shape (route, payload, event name) needed to exercise an AC; it is never a test source on its own. `execution-plan.md` is never a test source.
- The assertion is an **outcome a user or downstream system observes**: the response the caller receives, state readable through the public API or UI, an event another service consumes, a threshold met.
- HTTP status codes are **assertions, not test subjects**. Frame the case as the user's experience and assert the code inside it: `VerifyUnauthenticatedUserCannotViewOrders` asserts 401; `VerifyOperatorCannotApproveClaimOutsideOwnRegion` asserts 403; `VerifyOrderIsCreatedWhenRequestIsValid` asserts 201 and the persisted order. A case that exists only because a status code exists is deleted.
- Never assert on repositories, aggregates, handlers, mappers, DI wiring, ORM, serialisation or layer boundaries. If the test would survive a rewrite of internals with the same public behaviour, it is at the right level.

## 2. Volume — one case per acceptance criterion, no more unless named

- **One test case per AC.** Add a case only for a failure or access condition that the AC or an FR **names**. Do not invent edge cases, permutations or negative paths that no requirement states.
- There is **no minimum**. Zero cases is the correct result for a story whose ACs describe no behaviour. Never add a case to fill a test level, a template section, or a summary-table row.
- If one AC seems to need several cases, the AC is too coarse. Flag it for the Requirements Writer to split; do not multiply cases.
- Variants of one rule are parameters of one case, not new cases.
- One behaviour appears **once** in `test-cases.md`, in a single flat list. No FR / AC / API / async / NFR sections; the `Source:` and `Level:` fields carry the type.
- E2E: one happy path per feature. A second scenario only for a failure path the requirements name.

## 3. Naming — behaviour, not mechanism

- Pattern: `Verify<Actor or domain subject><Business outcome>When<Condition>`. PascalCase, no underscores. Condition optional.
- The name contains a business verb (submit, reject, approve, cancel, notify, calculate, transition, publish, reserve, confirm, validate, route) and a domain term from **this project's** product definition. Use the client's terminology.
- The name never contains: HTTP status codes, method or handler names, layer names (repository, service, controller), framework words, ticket IDs. The status code belongs in the assertion (§1).
- The `test-cases.md` heading and the test method name are **identical**. The `TC-nnn` ID lives in the `Source:` line and the verification report, never in the method name.

| Good | Bad | Why bad |
|---|---|---|
| `VerifyOrderIsRejectedWhenPaymentMethodIsExpired` | `SubmitOrder_ThrowsException` | method + mechanism |
| `VerifyUnauthenticatedUserCannotViewOrders` | `VerifyGetOrdersReturns401` | endpoint + status code |
| `VerifyItemIsReservedWhenOrderIsConfirmed` | `HandleOrderConfirmedEvent_ReservesItem` | handler name |

## 4. Comments — none

- Test bodies contain no comments. No Arrange / Act / Assert markers, no step narration, no rationale, no copied requirement text.
- Traceability is one attribute or tag per test (`[Trait("TC", "TC-007")]`, `@tag TC-007`, `test.describe` annotation), not a comment.
- If a test needs a comment to be understood, rename it or split it. Explanations belong in `test-cases.md`.

## 5. Skips — a test that cannot run is not written

- Forbidden in test code: `[Ignore]`, `[Skip]`, `Assume.*`, `test.skip`, `it.skip`, `xit`, `xdescribe`, `.only`, `return` before the first assertion, `if (!<environment or dependency>) return`, try/catch that swallows a setup failure, and any equivalent.
- A test whose dependency is missing is **not written**. It is registered in `deferred-tests.md` with its trigger condition. When the trigger is met, the test is written then.
- A test that cannot reach its assertion **fails**. Failure is then either fixed or moved to the deferred registry and deleted from the code.
- Verification report: any runner-reported skipped test makes the run `Status: Fail` until the skip is removed. The report's `Skipped` column must read 0.

## 6. Traceability

- Every case carries `Source: AC-n` (or `FR-nnn` when no AC exists). Every AC has exactly one case unless §2 names a second. A case with no source is deleted.
- The verification report's traceability table is keyed by AC, one row per AC; FR rows only where no AC exists.

## 7. Manual and deferred

Manual designation, its justification, and the automate-vs-manual table are in [`references/candidate-criteria.md`](references/candidate-criteria.md) §Step 2. Deferred registration is the QA agent's Step 3.

## Related

- [`references/candidate-criteria.md`](references/candidate-criteria.md) — what is infrastructure and never tested; automate vs manual
- [`references/examples.md`](references/examples.md) — discard example strings per zone
- [`references/rationale.md`](references/rationale.md) — why each zone is discarded
- `create-test-strategy` — chooses frameworks, tools, CI stages; never restates these standards
- QA agent (`.claude/agents/qa.md`) — primary consumer
- Implementation agent (`.claude/agents/implementation.md`) — applies §3–§5 to unit tests
