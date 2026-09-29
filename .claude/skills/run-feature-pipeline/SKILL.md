---
name: run-feature-pipeline
description: Orchestrate the full feature implementation pipeline for a given feature (identified by slug or backlog code). Runs feature phase (Requirements Writer → Solution Spec Writer → QA E2E Design), then story loop per story in order (Execution Planner → [Implementation ∥ QA Test Design] → QA Test Automation), then feature close (QA E2E Automation). Resumes from the current step if the pipeline is already in progress — safe to re-run at any time. Triggers on "/run-feature-pipeline {slug-or-code}" or "run the pipeline for feature X" or "continue the pipeline for F-01.02".
allowed-tools: Read, Glob, Grep, Write, Edit
license: "AP AI-Native Delivery Toolkit v3.5 - AP Confidential - governed by LICENSE.md"
metadata:
  user-invocable: true
---

# Skill: Run Feature Pipeline

Orchestrate the end-to-end feature implementation pipeline for a single feature. Resume from the current step if already in progress.

## When to Use

Use this skill when:
- the user wants to run the full feature pipeline end-to-end for a given feature slug or backlog code
- resuming a partially completed feature pipeline from the current step
- the user runs `/run-feature-pipeline` with a slug, backlog code, or both
- orchestrating Requirements Writer → Solution Spec Writer → QA → Execution Planner → Implementation in sequence

## Inputs

Accept either form:

```
/run-feature-pipeline nomination-workflow          # slug only
/run-feature-pipeline F-01.02                     # backlog code only
/run-feature-pipeline nomination-workflow F-01.02  # both (slug + code)
/run-feature-pipeline F-01.02 nomination-workflow  # order doesn't matter
```

**Resolving the slug and backlog ref:**

1. If a backlog code is given (matches `F-\d+\.\d+`), scan `.project/backlog/features/` for a file whose name starts with that code (e.g. `F-01.02-nomination-workflow.md`). Extract the slug from the filename by stripping the leading `F-NN.MM-` prefix and `.md` suffix.
2. If a slug is given directly, use it as-is.
3. If both are given, verify they agree (the backlog file for the code must contain the slug); if they conflict, report the discrepancy and stop.
4. If neither is given, ask the user to provide one.
5. If a backlog code is given but no matching file exists in `.project/backlog/features/`, report the missing file and stop — do not guess.

The feature folder is always `.project/features/<slug>/`. The backlog ref, once resolved, is passed to the Requirements Writer as the seed item.

## Step 1: Determine Resume Point

Run the `feature-status` scan using the same canonical story list/order that the story loop will use. Determine story order from the backlog feature file when available (the resolved `.project/backlog/features/<code>-<slug>.md` file). If no backlog feature file is available, derive the story list/order from `.project/features/<slug>/requirements.md`. Do **not** determine story order by scanning `.project/features/<slug>/stories/` in alphanumeric order; use existing story folders only to check completion status for the canonical story IDs/order. Report the resume point before proceeding.

If the feature is already fully complete, report completion and stop.

**Also locate the backlog feature file** — this is the source-of-truth file the pipeline updates as it progresses. The backlog file may live in `.project/backlog/features/` (active) or in `.project/backlog/features/_cancelled/` / `_deferred/` / `_superseded/` (inactive). If the file is in an inactive subfolder, stop and report — the pipeline should not run on a cancelled / deferred / superseded feature without an explicit override. Ask the user to move the file back to the active folder first (typically after a new ADR reactivates the scope).

---

## Step 2: Initialise Runtime Feature Folder + Update Backlog Feature File

Before running Feature Phase steps, do two housekeeping actions to link the runtime pipeline with the design-time backlog file.

### Step 2a — Runtime README

Create `.project/features/<slug>/README.md` if it does not already exist. This file anchors the runtime pipeline artifacts and links back to the design-time backlog file. Use this template:

```markdown
# Feature: <slug>

**Backlog ref:** [<F-NN.MM>](../../backlog/features/<F-NN.MM>-<slug>.md) *(or the appropriate `_*/` subfolder path if inactive)*
**Runtime folder:** `.project/features/<slug>/`
**Pipeline started:** <YYYY-MM-DD>

## Artifacts (populated as the pipeline runs)

- Requirements: `requirements.md` — Requirements Writer output
- Spec: `spec.md` — Solution Spec Writer output
- Feature-level QA: `test-plan.md`, `e2e-scenarios.md` — QA (Feature Phase)
- Stories: `stories/S-NN-<name>/` — Execution Planner + Implementation + QA Test Design
- Feature close (optional): `e2e-verification-report.md` — QA (Feature Close)

## Status Log

*(Appended by /run-feature-pipeline as the feature transitions through states. Backlog feature file is the canonical source; this log is a convenience index.)*

- <YYYY-MM-DD> — pipeline started; backlog Status → Spec
```

If the README already exists, append a new dated entry to the Status Log rather than overwriting.

### Step 2b — Backlog Feature File Front-Matter

Update the backlog feature file (`.project/backlog/features/F-NN.MM-<slug>.md`) with:

- **Status:** — set to `Spec` (or `InProgress` if resuming past the story loop start)
- **Last Updated:** — set to today
- **Runtime Pipeline:** — add if not present, pointing to `../../features/<slug>/` (or `.project/features/<slug>/`)
- **Status Reason:** — set to a short current-state summary, e.g. "Pipeline started YYYY-MM-DD — Requirements Writer running"

Do these two housekeeping actions with the `Edit` tool. If the backlog feature file already has `Status: InProgress` or `Status: Done` (i.e., the pipeline is resuming), leave the fields alone at this step; per-stage transitions happen inside the Feature Phase / Story Loop below.

**Never overwrite `Status: Blocked` in Step 2b** — a Blocked status is a signal that a prior run stopped due to a blocker; the human needs to resolve it before the pipeline resumes. If Status is Blocked, stop and surface the `Status Reason`.

---

## Feature Phase

Run once per feature. Skip steps whose output already exists.

### Step F-1: Requirements

**Invoke**: Requirements Writer agent  
**Prompt**: "Write requirements for feature `<slug>`<, backlog item <ref>,> reading `.project/product-definition/`, architecture docs, and ADRs. Save to `.project/features/<slug>/requirements.md`."  
**Gate**: `requirements.md` exists and is non-empty. If missing, stop and report.

### Step F-2: Spec

**Invoke**: Solution Spec Writer agent  
**Prompt**: "Write a solution spec for feature `<slug>` from `.project/features/<slug>/requirements.md`. Save to `.project/features/<slug>/spec.md`."  
**Gate**: `spec.md` exists and is non-empty. If missing, stop and report.

### Step F-3: Feature Test Plan + E2E Scenarios

**Invoke**: QA agent (Feature Phase)  
**Prompt**: "Produce the feature test plan and E2E scenarios for feature `<slug>` from `.project/features/<slug>/requirements.md`, `.project/features/<slug>/spec.md`, and `.project/architecture/test-strategy-*.md`. The test plan maps test levels per story and documents known deferrals. The E2E scenarios cover full cross-story flows. Save `test-plan.md` and `e2e-scenarios.md` to `.project/features/<slug>/`. Do not produce story-level test cases at this stage."  
**Gate**: Both `.project/features/<slug>/test-plan.md` AND `.project/features/<slug>/e2e-scenarios.md` exist and are non-empty. If either is missing, stop and report.

---

## Story Loop

Before starting the loop, read the story list from the backlog feature file (`.project/backlog/features/<ref>-*.md` if ref was provided, otherwise from `.project/features/<slug>/requirements.md`, which should list stories). Treat that story list as the single source of truth for both inclusion and ordering. If no story list is found, ask the user to confirm the stories to run.

Normalize the stories into ordered `S-NN-...` story identifiers in the same order they appear in the backlog file or requirements. Ensure `.project/features/<slug>/stories/` exists, and create `.project/features/<slug>/stories/<S-NN-name>/` for each story if it does not already exist.

**Backlog transition — Story Loop entry:** before invoking Step S-1 for the first story (S-01), update the backlog feature file front-matter:

- **Status:** → `InProgress`
- **Last Updated:** → today
- **Status Reason:** → "Story loop started — S-01 execution planning in progress"

Append a matching entry to the runtime `README.md` Status Log: `<YYYY-MM-DD> — Story loop entered; backlog Status → InProgress`.

Repeat for each story from that ordered story list (not by scanning existing folders on disk). Process them in the .project/backlog/requirements order represented by the `S-NN-...` identifiers. Skip a story entirely if `.project/features/<slug>/stories/<S-NN-name>/test-cases.md` already exists and `checklist.md` has zero `[ ]` lines and zero `⚠️ BLOCKED` lines — that is the done criterion for a story.

### Step S-1: Execution Plan

**Invoke**: Execution Planner agent  
**Path**: `.project/features/<slug>/stories/<S-NN-name>/`

**For S-01:**  
"Write the execution plan for story `<S-01-name>` of feature `<slug>`. Read `.project/features/<slug>/requirements.md` and `.project/features/<slug>/spec.md`. **Infrastructure-first rule**: this is the first story — claim all shared infrastructure changes from the spec here: new database migrations, new aggregate roots, new shared message types. Later stories must not redefine these. Save `execution-plan.md` and `checklist.md` to `.project/features/<slug>/stories/<S-01-name>/`."

**For S-NN (N > 1):**  
"Write the execution plan for story `<S-NN-name>` of feature `<slug>`. Read `.project/features/<slug>/requirements.md`, `.project/features/<slug>/spec.md`, and the execution plans of all prior stories: `.project/features/<slug>/stories/<S-01-name>/execution-plan.md` [… through S-(NN-1)]. **Story ordering rule**: do not re-specify infrastructure or work already claimed by prior stories. Build on the confirmed foundation. Save `execution-plan.md` and `checklist.md` to `.project/features/<slug>/stories/<S-NN-name>/`."

**Gate**: `execution-plan.md` and `checklist.md` both exist. `checklist.md` contains no `⚠️ BLOCKED` items. If blocked, update the backlog feature file front-matter (**Status:** → `Blocked`; **Last Updated:** → today; **Status Reason:** → the blocker text verbatim) and append `<YYYY-MM-DD> — Blocked at S-NN execution plan: <reason>` to the runtime README Status Log. Surface the blocker text and stop — do not proceed to Step S-2.

### Step S-2: Implementation + QA Test Design (parallel)

Launch both agents concurrently.

**Implementation agent:**  
"Execute the plan in `.project/features/<slug>/stories/<S-NN-name>/execution-plan.md`. Work through `checklist.md` item by item, marking each `[x]` as complete. Follow project conventions in `CLAUDE.md` and architecture docs."

**QA agent (test design mode):**  
"Write test cases for story `<S-NN-name>` of feature `<slug>`. Read `.project/features/<slug>/test-plan.md` (feature test plan — test levels and known deferrals per story), `.project/features/<slug>/requirements.md`, `.project/features/<slug>/spec.md`, and `.project/features/<slug>/stories/<S-NN-name>/execution-plan.md`. <For S-NN (N > 1): Also read prior stories' test cases at `.project/features/<slug>/stories/<S-0*-name>/test-cases.md` — do not duplicate test cases already written for prior stories.> Apply the `test-standards` skill: one case per numbered acceptance criterion, behaviour-level, no invented edge cases. Save `test-cases.md` to `.project/features/<slug>/stories/<S-NN-name>/` (plus `deferred-tests.md` if any tests must be deferred)."

**Gate — story done when both pass:**  
- `checklist.md` has zero `[ ]` lines and zero `⚠️ BLOCKED` lines
- `test-cases.md` exists and is non-empty

> **Infrastructure-only stories:** the QA agent's Infrastructure-Only Gate writes a one-line marker `test-cases.md` ("No QA testing applies — this story delivers infrastructure only…"). That marker satisfies this gate — do not re-invoke QA or treat the story as incomplete.

If implementation is blocked (`⚠️ BLOCKED` in checklist), update the backlog feature file front-matter (**Status:** → `Blocked`; **Last Updated:** → today; **Status Reason:** → the blocker text verbatim) and append `<YYYY-MM-DD> — Blocked at S-NN implementation: <reason>` to the runtime README Status Log. Surface blocker text and stop.

**When the story-done gate passes**, append `<YYYY-MM-DD> — S-NN done` to the runtime README Status Log. Do not change the backlog feature Status yet unless this was the last story (see below).

**When all stories in the story list are done** (last story's gate has passed), update the backlog feature file front-matter:

- If the user requested optional QA test automation (Step S-3) or the optional Feature Close E2E automation, set **Status:** → `InReview` with **Status Reason:** → "All stories complete; QA verification in progress"
- Otherwise (no user-requested automation beyond story test-cases), set **Status:** → `Done` with **Status Reason:** → "All stories complete; feature delivery accepted at story test-cases level"

Append the matching entry to the runtime README Status Log.

### Step S-3: QA Test Automation _(optional)_

Run this step only if the user explicitly requests test automation, or if `verification-report.md` already exists for the story (in which case skip it — it is already done).

**Invoke**: QA agent (test automation mode)  
"Write automated tests for story `<S-NN-name>` of feature `<slug>`. Read `.project/features/<slug>/test-plan.md` (feature test plan), `.project/features/<slug>/stories/<S-NN-name>/test-cases.md`, and the implemented code. Apply the `test-standards` skill: method names equal case headings, no comments, no skipped or guarded tests. Save a `verification-report.md` with pass/fail status, zero skipped, and acceptance-criteria traceability to `.project/features/<slug>/stories/<S-NN-name>/`."

**Gate**: `verification-report.md` exists and is non-empty.

---

## Feature Close _(optional)_

Run only if the user explicitly requests E2E test automation. Requires all stories to be done (impl + test cases complete). Does not require per-story `verification-report.md`.

**Invoke**: QA agent (Phase 3 — deferred test execution)  
"Run the deferred test sweep for feature `<slug>`. Read the feature-level registry `.project/features/<slug>/deferred-tests.md` and all story-level `deferred-tests.md` files, and execute every entry whose trigger condition is now met — including the E2E scenarios in `.project/features/<slug>/e2e-scenarios.md` (trigger: feature-close). Save E2E test code and a pass/fail summary to `.project/features/<slug>/e2e-verification-report.md`, update the registries, and report any deferred tests that remain Pending with their blocking conditions."

**Gate**: `.project/features/<slug>/e2e-verification-report.md` exists and is non-empty. Report remaining Pending deferred tests (if any) in the progress output — they are not a gate failure, but must be surfaced.

**On successful Feature Close**, update the backlog feature file front-matter:

- **Status:** → `Done`
- **Last Updated:** → today
- **Status Reason:** → "Feature Close complete; E2E verification passed"

Append `<YYYY-MM-DD> — Feature Close complete; backlog Status → Done` to the runtime README Status Log.

Optionally, if the user provides a delivered-effort figure at close, populate `**PD Actual:**` in the backlog feature file for retrospective sizing accuracy. If not provided, leave the field absent.

---

## Backlog Feature File — Status Transition Matrix

Summary of when the pipeline modifies the backlog feature file's `Status` front-matter:

| Trigger | Backlog Status set to | Also updated |
|---|---|---|
| Pipeline starts (Step 2b) | `Spec` *(unless already past this stage)* | `Last Updated`, `Runtime Pipeline`, `Status Reason` |
| Story Loop entry (before S-01 Step S-1) | `InProgress` | `Last Updated`, `Status Reason` |
| Any step encounters `⚠️ BLOCKED` in checklist | `Blocked` | `Last Updated`, `Status Reason` (verbatim blocker text) |
| Last story done + user requested QA test automation OR feature close | `InReview` | `Last Updated`, `Status Reason` |
| Last story done + no automation requested | `Done` | `Last Updated`, `Status Reason` |
| Feature Close (E2E automation) complete | `Done` | `Last Updated`, `Status Reason`, optional `PD Actual` |

**Never write these transitions:**
- Never set `Status: Cancelled` — cancellation is a scope decision that goes through ADR + human-driven file move
- Never set `Status: Deferred` or `Status: Superseded` — same
- Never set `Status: Discovery` — Discovery is a per-group workshop external to the pipeline

**Never overwrite an existing `Blocked` status without human intervention.** If the pipeline resumes and finds `Status: Blocked`, stop and ask the user to confirm the blocker is resolved before proceeding.

---

## Progress Output

Report in-conversation as each step completes:

```
── Housekeeping ──────────────────────────────────────
[Setup] Backlog file:  .project/backlog/features/F-NN.MM-<slug>.md
[Setup] Runtime folder: .project/features/<slug>/ (README written)
[Setup] Backlog Status: Planned → Spec

── Feature Phase ─────────────────────────────────────
[Feature] Requirements Writer  → ✓ requirements.md
[Feature] Solution Spec Writer → ✓ spec.md
[Feature] QA — Feature Phase   → ✓ test-plan.md, e2e-scenarios.md

── Story Loop ────────────────────────────────────────
[Setup] Backlog Status: Spec → InProgress
[S-01] Execution Planner       → ✓ execution-plan.md + checklist.md
[S-01] Launching parallel:
        Implementation → (running...)
        QA Test Design → (running...)
[S-01] Implementation          → ✓ checklist complete (8/8)
[S-01] QA Test Design          → ✓ test-cases.md  ← story done
[S-01] QA Test Automation      → ✓ verification-report.md (Pass)  [optional]

── Feature Close (optional) ──────────────────────────
[Feature] QA — E2E Automation  → ✓ e2e-verification-report.md
[Setup] Backlog Status: InReview → Done

Feature `<slug>` complete.
Backlog: .project/backlog/features/F-NN.MM-<slug>.md (Status: Done)
Runtime: .project/features/<slug>/
```

## Failure Handling

- If any agent produces no output: report the step name, story, and failure reason. Stop the pipeline. Do not retry automatically.
- If `checklist.md` contains `⚠️ BLOCKED`: stop before QA automation, show the blocked item text, ask user to resolve before re-running.
- Re-running `/run-feature-pipeline <slug>` after a failure is always safe — the feature-status scan skips completed steps.

## Rules Summary

| Rule | Enforced at |
|------|------------|
| Infrastructure-first: migrations, aggregates, message types go to S-01 | S-01 Execution Planner prompt |
| Story ordering: pass all prior execution plans as context | S-NN (N > 1) Execution Planner prompt |
| No test duplication: read prior test-cases.md before writing | QA Test Design for S-NN (N > 1) |
| Feature test plan + E2E scenarios from spec, before story loop | Feature Phase Step F-3 |
| Story done = impl complete + test-cases.md written | Step S-2 gate |
| QA test automation is optional — run on request only | Step S-3 |
| E2E automation is optional — run on request only | Feature Close |
| Runtime README written at pipeline start with back-link to backlog file | Step 2a |
| Backlog feature front-matter updated on Status transitions (Planned/Spec → InProgress → InReview/Done, or → Blocked on halt) | Steps 2b, Story Loop entry, gate failures, feature close |
| Sprint execution not tracked here — external tracker (Azure DevOps in AP engagements) is authoritative | Global — no Sprint Range writes; only backlog Status |
| Never overwrite existing `Blocked` status without human intervention | Step 2b, Story Loop entry |
| Never set Cancelled / Deferred / Superseded from the pipeline — those are scope-decision transitions | Global |
