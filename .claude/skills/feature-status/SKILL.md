---
name: feature-status
description: Scan .project/features/{slug}/ and report feature pipeline progress. Shows feature-level artifact completion (requirements, spec, e2e-scenarios), per-story progress table (plan, impl, QA design, QA auto), current step, blockers, and recommended next action. Use when resuming a mid-flight feature pipeline or checking what has been done and what is next. Accepts a slug or backlog code (e.g. F-01.02) — resolves the slug automatically from .project/backlog/features/ if a code is given. Triggers on "/feature-status {slug-or-code}" or "what is the status of feature X" or "where are we on F-01.02".
allowed-tools: Read, Glob, Grep
license: "AP AI-Native Delivery Toolkit v3.5 - AP Confidential - governed by LICENSE.md"
metadata:
  user-invocable: true
---

# Skill: Feature Status

Scan `.project/features/<slug>/` and report progress through the feature pipeline. Output is in-conversation only — never written to a file.

## When to Use

Use this skill when:
- the user asks about the status or progress of a feature pipeline
- checking which artifact is missing or which story is next in the pipeline
- the user runs `/feature-status` with a slug or backlog code
- resuming a mid-flight feature implementation and needing to know the next step

## Inputs

Accept either form:

```
/feature-status nomination-workflow   # slug
/feature-status F-01.02              # backlog code
```

**Resolving the slug:**

1. If the argument matches `F-\d+\.\d+`, scan `.project/backlog/features/` for a file whose name starts with that code (e.g. `F-01.02-nomination-workflow.md`). Extract the slug by stripping the `F-NN.MM-` prefix and `.md` suffix.
2. Otherwise treat the argument as a slug directly.
3. If no argument is given, attempt to infer the slug from the path of the currently open file. Ask the user if it cannot be determined.
4. If a backlog code is given but no matching file exists in `.project/backlog/features/`, report this and stop.

## Artifact Scan

### Feature-level — `.project/features/<slug>/`

| Artifact | Agent | Prerequisite for |
|----------|-------|-----------------|
| `requirements.md` | Requirements Writer | everything else |
| `spec.md` | Solution Spec Writer | test-plan, e2e-scenarios, story execution plans |
| `test-plan.md` | QA (feature phase) | per-story test case writing |
| `e2e-scenarios.md` | QA (feature phase) | QA automation in feature close |
| `deferred-tests.md` | QA (feature phase) | _(optional — feature-scoped deferred test registry)_ |
| `e2e-verification-report.md` | QA (test automation, feature close) | _(optional)_ |

### Story-level — `.project/features/<slug>/stories/S-NN-<name>/`

Scan each story folder in alphanumeric order for status reporting. (Canonical story execution order is determined by the backlog feature file or `requirements.md` — see `run-feature-pipeline` skill. Folder scan order here is for display only.) For each story:

| Step | Artifact(s) | Agent | Complete when |
|------|-------------|-------|---------------|
| 1 — Plan | `execution-plan.md` + `checklist.md` | Execution Planner | both files exist |
| 2a — Impl | all items in `checklist.md` | Implementation | zero `[ ]` items, zero `⚠️ BLOCKED` items |
| 2b — QA Design | `test-cases.md` | QA (test design) | file exists and is non-empty (a one-line "No QA testing applies — infrastructure only" marker counts as complete) |
| 3 — QA Auto _(optional)_ | `verification-report.md` | QA (test automation) | file exists |

Step 2 (parallel) is the done gate for a story — complete when both 2a and 2b are complete. Step 3 is optional and shown for informational purposes only.

### Deferred test scan

Scan the feature-level `.project/features/<slug>/deferred-tests.md` and each story's `deferred-tests.md` (any may be absent). Count entries by status (Pending / Resolved / Abandoned). Report the pending count in the output; if any Pending entry's trigger condition appears to be met (e.g. `Feature-close: all stories complete` and all stories are done), flag it as an actionable item — QA Phase 3 should be run.

## Step Determination Logic

- **Feature phase complete**: `requirements.md`, `spec.md`, `test-plan.md`, and `e2e-scenarios.md` all exist
- **Story done**: Step 2a complete AND `test-cases.md` exists and is non-empty
- **Current story**: first story folder where Step 2 is not done
- **Current step within story**: first missing artifact in the sequence Plan → Impl → QA Design (stop here for done; QA Auto is optional)
- **Step 2a complete**: `checklist.md` exists AND has zero `[ ]` lines AND zero `⚠️ BLOCKED` lines
- **Step 2 complete**: 2a complete AND `test-cases.md` exists and is non-empty
- **Feature complete**: feature phase complete AND all stories done

## Blockers to Flag

Report as blockers (bold, before the story table):

- `spec.md` is missing but one or more story execution plans exist — plans may be based on an incomplete spec
- `checklist.md` contains `⚠️ BLOCKED` items — implementation is halted, user intervention needed
- `checklist.md` has unchecked `[ ]` items but `test-cases.md` already exists — QA test design ran before implementation finished
- `test-plan.md` is missing but `e2e-scenarios.md` exists — Feature Phase ran incomplete; re-invoke QA in Feature Phase
- `execution-plan.md` exists but `checklist.md` is missing — Execution Planner produced incomplete output
- Story S-NN has an execution plan but story S-(NN-1) is not done (impl + test cases incomplete) — story ordering violated

## Output Format

```
## Feature Status: <slug>

**Phase:** <Feature phase in progress | Feature phase complete — Story loop, S-NN in progress | Complete>
**Next action:** <exact agent to invoke and what to tell it>

---

### Feature Artifacts
- [x] requirements.md     ✓
- [x] spec.md             ✓
- [ ] test-plan.md        ✗  (produced by QA in Feature Phase after spec)
- [ ] e2e-scenarios.md    ✗  (produced by QA in Feature Phase after spec)

### Story Progress

| Story | Plan | Impl | QA Design | Done | QA Auto* |
|-------|:----:|:----:|:---------:|:----:|:--------:|
| S-01 — <name> |  ✓  |  ✓  |     ✓     |  ✓  |    ✓    |
| S-02 — <name> |  ✓  | ...  |    ...    |     |    ✗    |  ← current
| S-03 — <name> |  ✗  |  ✗  |     ✗     |     |    ✗    |

_* QA Auto (verification-report.md) is optional — not required for Done._

### Deferred Tests

<N pending / M resolved / K abandoned — or "No deferred test registries found.">
<⚠️ if any pending trigger condition appears met: "Trigger met for <Test ID> (<condition>) — run QA Phase 3.">

---

### Recommended Next Step

**<Story or phase>** — <brief status>
> "<Exact prompt the user should send to the appropriate agent>"
```

Use `...` for a step in progress (plan exists, impl partially checked off). Use `✓` for complete, `✗` for not started. Use `⚠️` in the Impl column when `checklist.md` contains `⚠️ BLOCKED` items.

If there are no story folders yet, omit the story table and set Phase to "Feature phase in progress".

If all stories are done (impl + test cases complete), set Phase to "Complete" and omit the Recommended Next Step section. Optionally note that QA automation steps (Step S-3 per story, Feature Close E2E) are available if the team wants them.

If `e2e-verification-report.md` exists, show it under Feature Artifacts as a bonus indicator — it does not change the Complete status.
