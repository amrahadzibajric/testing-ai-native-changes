# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This repository is a **project elaboration workspace**. It contains uploaded source materials — RFPs, meeting notes, transcripts, reference documents — and is used to produce structured project documentation and proposals.

## Elaboration Workflow

Project elaboration follows a phased sequence. Each phase produces artifacts that the next phase consumes. Use `/project-status` at any time to see which phase the project is in and what should happen next.

| Phase | Name                    | Agent / Skill                               | Input | Output                                                              | Prerequisite |
|-------|-------------------------|---------------------------------------------|-------|---------------------------------------------------------------------|--------------|
| 0 | **Upload sources**      | —                                           | RFPs, transcripts, meeting notes, specs | Files in `.project/input/`                                               | — |
| 1 | **Review requirements** | `/requirements-reviewer`                    | Source documents | Client questions, safe assumptions, gap analysis                    | ≥1 source document |
| 2 | **Run intake**          | Product Strategist                          | Source documents | Intake record in `.project/intake/`, ADRs in `.project/adr/`, project context updated | ≥1 source document |
| 3 | **Define product**      | Product Strategist                          | Intake record, ADRs, project context | Product definition in `.project/product-definition/` (modules, scope, NFRs, E2E scenarios, integration points) | Intake complete |
| 4 | **Design architecture** | Solution Architect                          | Product definition, ADRs | Architecture doc in `.project/architecture/`, integration assessments, security assessment in `.project/architecture/`, test strategy in `.project/architecture/`, ADRs  | Product definition exists |
| 5 | **Create data model**   | Solution Architect / `/create-data-model`   | Product definition, architecture doc, ADRs, integration assessments | Domain models + ERDs + JSON Schema in `.project/data-model/`                 | Architecture exists |
| 6 | **Generate UI Designs** | Solution Architect / `/generate-ui-designs` | Product definition, architecture doc, data model, ADRs | Screen designs (web + mobile) in `.project/ui-designs/`                      | Data model exists |
| 7 | **Produce WBS**         | Delivery Lead                               | Product definition, architecture doc, data model, ADRs | WBS summary + EPICs + Features in `.project/backlog/`                        | Architecture exists |
| 8 | **Validate backlog**    | `/validate-backlog`                         | WBS, product definition, architecture, ADRs | Validation report (Blocking / Non-blocking findings)                | Backlog exists |
| 9 | **Estimate**            | Delivery Lead                               | Validated WBS (Feature files) | Estimates in `.project/estimates/`                                           | Validation passes (no Blocking findings) |
| 10 | **Plan delivery**       | Delivery Lead                               | Estimates, WBS | Delivery plan + team model in `.project/estimates/`; `team-allocation.yaml` in `.project/backlog/` | Estimates exist |
| 11 | **Compose proposal**    | Product Strategist → `create-presentation`  | Product definition, estimates, team model, delivery plan, architecture doc | Proposal in `.project/proposals/` (internal + client versions); presentation deck in `.project/presentations/` | Delivery plan exists |

**Iteration:** When source documents are updated or new meetings occur, re-enter at the earliest affected phase. The `validate-backlog` skill (Phase 8) detects gaps introduced by upstream changes.

**ADRs are continuous:** The `create-adr` skill runs throughout — any agent creates ADRs when decisions are confirmed, conflicts are found, or information gaps are identified. ADRs accumulate across all phases.

**Checking status:** Run `/project-status` to inventory existing artifacts, determine the current phase, flag blockers (conflict ADRs, missing prerequisites), and get the recommended next action.

## How to work with materials

Use the `read-documents` skill for all document access — it defines extraction rules per type, cross-referencing logic, and citation format. Key principles that apply workspace-wide:
- Preserve the client's language and terminology; do not substitute synonyms that may carry different meaning in their domain.
- Surface conflicts between documents explicitly — do not silently resolve them.
- Cite every sourced claim using the format defined in the skill.

## Agents

### Product Strategist (`.claude/agents/product-strategist.md`)
The primary agent for product-level documentation. Invoke it when the user wants to:
- Draft or update the project proposal
- Write or revise module descriptions
- Create or update an ADR
- Run an intake session for a new project or workstream

### Solution Architect (`.claude/agents/solution-architect.md`)
The technical design agent. Invoke it when the user wants to:
- Produce architecture documents (system context, component design, data flows)
- Assess an external integration (uses the `assess-integration` skill)
- Produce a design-level security assessment (uses the `assess-security` skill)
- Create or update the conceptual data model (uses the `create-data-model` skill)
- Generate UI designs in a claude.ai/design project (uses the `generate-ui-designs` skill)
- Conduct a technology evaluation with trade-off analysis
- Define non-functional requirements or module interfaces
- Review and strengthen the technical sections of a proposal

### Delivery Lead (`.claude/agents/delivery-lead.md`)
The delivery planning agent. Invoke it when the user wants to:
- Decompose the project into EPICs, Features, and Business Stories (uses `create-work-breakdown` for templates, `validate-backlog` as quality gate)
- Produce effort estimates for modules, features, or full projects (uses the `create-estimate` skill)
- Build a phased delivery plan with milestones and critical path
- Design a team model — roles, seniority, allocation hours, ramp-up, buffer weeks (uses the `create-team-model` skill)
- Map dependencies across modules, teams, and external parties
- Assess delivery risk

### Requirements Writer (`.claude/agents/requirements-writer.md`)
The feature-level requirements agent. Invoke it when the user wants to:
- Convert a user story, bug report, or feature request into a formal requirements document
- Produce testable functional requirements (FR-###) grounded in source materials and ADRs
- Create the initial `.project/features/<feature-slug>/requirements.md` for a feature

### Solution Spec Writer (`.claude/agents/solution-spec-writer.md`)
The feature-level technical design agent. Invoke it when the user wants to:
- Produce a technical solution spec from an existing requirements document
- Design the API, domain, persistence, and messaging changes for a specific feature
- Document the smallest change that satisfies requirements, grounded in the project's architecture

### Execution Planner (`.claude/agents/execution-planner.md`)
The feature-level implementation planning agent. Invoke it when the user wants to:
- Turn requirements + solution spec into a step-by-step developer-ready execution plan
- Produce an implementation checklist with verification steps at each stage
- Sequence work by dependency order across CLEAN Architecture layers

### Implementation (`.claude/agents/implementation.md`)
The code implementation agent. Invoke it when the user wants to:
- Execute a feature's execution plan by writing code step-by-step
- Scaffold a new backend service
- Add a frontend page or feature
- Implement domain, CQRS, saga, migration, or test changes following project conventions
- Track progress through the feature checklist

### QA (`.claude/agents/qa.md`)
The quality assurance agent. Unit testing is developer-owned and outside QA scope. Operates in four phases:
- **Feature Phase** (run after Solution Spec Writer, before story loop): Produces feature-scoped `test-plan.md` (test levels per story, known deferrals, cross-story coverage notes) and `e2e-scenarios.md`. Applies the Infrastructure-Only Gate — if the feature delivers no application behaviour, both files get a one-liner and the phase stops.
- **Phase 1 — Test Design** (run after Execution Planner, parallel to Implementation): Reads the feature `test-plan.md` and produces `test-cases.md` per story — one case per numbered acceptance criterion, per the `test-standards` skill. Infrastructure stories are skipped. (`test-plan.md` and `e2e-scenarios.md` are feature-scoped and never regenerated per story.)
- **Phase 2 — Test Automation** (run after Implementation completes): Reads the feature test plan, story test cases, and implemented code to write and execute automated tests (integration, API, saga, E2E, performance, security). Produces test code and a `verification-report.md` with acceptance-criteria traceability and zero skipped tests.
- **Phase 3 — Deferred Test Execution** (run when trigger conditions are met): Re-checks the deferred test registry and executes any tests whose trigger conditions are now satisfied (e.g., all stories done for E2E, infrastructure provisioned for performance tests).

### Feature implementation pipeline
The feature-level agents form a pipeline with hybrid granularity: requirements and spec are produced once at the feature level; execution, implementation, and QA run per story. Use `/run-feature-pipeline <slug>` to orchestrate end-to-end, or `/feature-status <slug>` to check progress and get the next action.

```
Feature phase (once):
  Requirements Writer  →  .project/features/<slug>/requirements.md
  Solution Spec Writer →  .project/features/<slug>/spec.md
  QA (feature phase)   →  .project/features/<slug>/test-plan.md      ← test levels per story, known deferrals
                          .project/features/<slug>/e2e-scenarios.md

Story loop (per story S-NN, in order):
  Execution Planner    →  .project/features/<slug>/stories/<S-NN-name>/execution-plan.md
                          .project/features/<slug>/stories/<S-NN-name>/checklist.md
  ┌─ Implementation    →  code changes                    ← parallel
  └─ QA (test design)  →  .project/features/<slug>/stories/<S-NN-name>/test-cases.md  ← story done
  QA (test automation) →  .project/features/<slug>/stories/<S-NN-name>/verification-report.md  [optional]

Feature close (optional, once after all stories done):
  QA (test automation) →  .project/features/<slug>/e2e-verification-report.md
```

Cross-cutting rules enforced by `/run-feature-pipeline`:
- **Infrastructure-first**: S-01 claims all migrations, aggregate roots, and shared message types; later stories must not redefine them
- **Story ordering**: each Execution Planner receives prior stories' plans as context
- **No test duplication**: QA Test Design for S-NN reads feature `test-plan.md` and prior stories' test cases before writing
- **Feature test plan + E2E scenarios from spec**: both produced after spec, before the story loop
- **QA automation is optional**: test cases defined alongside code is the done criterion; `verification-report.md` and `e2e-verification-report.md` are produced on request
- **E2E automation last**: if requested, runs only after all stories are done (implementation completed and QA design artifacts produced)

### Agent collaboration
Agents share the same document store (`.project/input/`), product definition (`.project/product-definition/`), project context (`.project/context/project-context.md`), and ADR registry (`.project/adr/`). They reference each other's outputs:
- The **Product Strategist** writes the product definition (modules, scope, NFRs, E2E scenarios, integration points) and composes proposals from it
- The **Solution Architect** reads `.project/product-definition/` and produces architecture documents and data models
- The **Delivery Lead** reads `.project/product-definition/modules/` and architecture documents to produce estimates
- The **Requirements Writer**, **Solution Spec Writer**, **Execution Planner**, **Implementation**, and **QA** agents read `.project/product-definition/`, architecture, and ADRs to produce feature-level artifacts in `.project/features/` and implement/verify code

## Document Store

Source materials live in `.project/input/`. Drop any source document directly into this folder — RFPs, meeting transcripts, emails, briefs, specs, notes, or any other input material. No subfolders or naming conventions are required. AI agents infer document type from content at read time.

## Output Store

Agent-produced documents are saved to dedicated folders:

| Folder | Contents | Naming convention | Primary agent |
|--------|----------|-------------------|---------------|
| `.project/intake/` | Intake session records | `intake-[project-slug]-[YYYY-MM-DD].md` | Product Strategist |
| `.project/product-definition/` | Live source of truth: modules, scope, NFRs, E2E scenarios, integration points | `modules/<slug>.md`, `scope.md`, `nfrs.md`, `e2e-scenarios.md`, `integration-points.md` | Product Strategist |
| `.project/proposals/` | Composed client deliverables (internal + client versions) derived from .project/product-definition/ + estimates + architecture | `proposal-[slug]-v[N.N].md` (internal), `proposal-[slug]-v[N.N]-client.md` (client) | Product Strategist |
| `.project/presentations/` | Branded .pptx presentation decks derived from the proposal | `ap-deck-[slug]-[YYYYMMDD]-v[N].pptx` | `create-presentation` skill |
| `.project/architecture/` | Architecture docs, integration assessments, security assessments, tech evaluations | `arch-*`, `integration-*`, `security-assessment-*`, `tech-eval-*` | Solution Architect |
| `.project/data-model/` | Conceptual data model: domain files, ERDs, JSON Schema | `[domain].md`, `data-model.json`, `README.md` | Solution Architect |
| `.project/ui-designs/` | Agreed design direction, design-system and screen briefs, plus the published Claude Design target record | `design-direction.md`, `[group]-web.md`, `[group]-mobile.md`, `README.md`, `.design-canvas.json` | Solution Architect |
| `.project/backlog/` | WBS summary, individual EPIC and Feature files, team allocation config for Excel export; inactive features organised into subfolders by status (see "Backlog State & Change Management" below) | `wbs-*.md`, `epics/E-NN-*.md`, `features/F-NN.MM-*.md`, `team-allocation.yaml`, `features/_cancelled/`, `features/_deferred/`, `features/_superseded/` | Delivery Lead |
| `.project/estimates/` | Effort estimates, delivery plans, team models, developer assignments | `estimate-*`, `delivery-plan-*`, `team-model-*`, `developer-assignments-*` | Delivery Lead |
| `.project/adr/` | Architecture Decision Records | `ADR-NNNN-short-hyphenated-title.md` | All agents |
| `.project/features/` | Feature-level requirements, specs, test plan, E2E scenarios; story-level execution plans, checklists, test artifacts | `<slug>/requirements.md`, `<slug>/spec.md`, `<slug>/test-plan.md`, `<slug>/e2e-scenarios.md`, `<slug>/stories/S-NN-<name>/execution-plan.md`, `<slug>/stories/S-NN-<name>/checklist.md`, `<slug>/stories/S-NN-<name>/test-cases.md`, `<slug>/stories/S-NN-<name>/verification-report.md` | Requirements Writer, Solution Spec Writer, Execution Planner, QA |
| `.project/context/` | Live project context (one file per project) | `project-context.md` | All agents |
| `.project/retrospectives/` | Pipeline audit and incident retrospective entries | `RETRO-NNNN-pipeline-audit-<slug>.md`, `RETRO-NNNN-incident-<slug>.md` | retrospect skill |

## Backlog State & Change Management

The backlog is a living artifact — features complete, get cancelled, get deferred, or get updated as the project runs. To keep the WBS, estimates, and delivery plan consistent as this evolution happens, the following conventions apply.

### Feature file front-matter (canonical source of feature state)

Every feature file (`.project/backlog/features/F-NN.MM-*.md`) carries a structured front-matter block immediately after the `**EPIC:**` line. This block is machine-parseable and is the canonical source of feature state:

```markdown
**Status:** Planned | Discovery | Spec | InProgress | InReview | Done | Blocked | Cancelled | Deferred | Superseded
**Phase:** Phase 2 | Phase 2.x | Phase 3 setup
**Sprint Range:** SNN-SMM   *(advisory — external tracker is authoritative for sprint execution)*
**PD Estimate:** <buffered PD from estimate>
**Last Updated:** YYYY-MM-DD
**Status Reason:** <free text — why the current status applies>
```

Conditional fields added when the status justifies them:
- `**Cancelled Reason:**` — added when Status = Cancelled
- `**Superseded By:**` — added when Status = Superseded (points to replacement feature[s] or governing ADR)
- `**Deferred To:**` — added when Status = Deferred (e.g., "Phase 2.x", "Phase 3")
- `**Runtime Pipeline:**` — added when `/run-feature-pipeline` first runs for this feature; links to the `.project/features/<slug>/` folder
- `**PD Actual:**` — added at feature closeout for retrospective accuracy

**Sprint execution is the external tracker's responsibility** (typically Azure DevOps in AP engagements). The `Sprint Range` field here captures the WBS design-time intent (for proposal + planning views); operational sprint tracking, story assignments, and daily progress live in the external tracker and are not mirrored back into this repository. The backlog files carry *what we said we'd build* and *what the current commitment state is*; the external tracker carries *what's happening this sprint*.

### Inactive-feature subfolder pattern

Features that are no longer active get moved into one of three subfolders under `.project/backlog/features/`, mirroring the ADR-folder convention:

| Subfolder | Contents | Status field value |
|---|---|---|
| `_cancelled/` | Will not be built; scope decision reversed or removed | `Cancelled` |
| `_deferred/` | Moved to Phase 2.x, Phase 3, or an indefinite later phase | `Deferred` |
| `_superseded/` | Replaced by another feature or moved out of AP scope (e.g., to a vendor) | `Superseded` |

Files in these subfolders are convention-excluded from active aggregation views (e.g., `/backlog-status`). They remain readable and searchable — the move preserves the content, just signals "not currently in scope".

Moves are done with `git mv` to preserve history. Cross-references from the WBS, EPIC files, and ADRs must be updated in the same commit as the move.

### Change management — where does each change land?

Any material change lands in exactly one of the following. If you're unsure which applies, prefer the higher-visibility option.

| Change type | Where it lands | Example |
|---|---|---|
| Architectural or scope-material decision | **New ADR** via `create-adr` skill | EPIC de-scope; feature addition; cutover date shift |
| New requirements from stakeholder input | **Intake record** in `.project/intake/` | Stakeholder Q&A email — creates intake doc + spawns ADRs |
| Feature status transition (Planned → InProgress → Done, or → Cancelled/Deferred/Superseded) | **Feature file front-matter** (`Status`, `Last Updated`, `Status Reason` fields) + optional file move to `_*` subfolder | Feature moves from Planned to InProgress when the pipeline starts |
| Story-level progress within a feature | **Story checklist** (`.project/features/<slug>/stories/<S-NN>/checklist.md`) — produced by the feature pipeline | Story tasks transitioning to done |
| Feature-level scope tweak (not big enough for an ADR) | **Feature file body** — extend `Scope Detail` / `Stories` sections; bump `Last Updated`; add a note in `Status Reason` if material | Adding a story to a feature during Discovery |
| Cross-cutting delivery replan (schedule / capacity / roles) | **Delivery plan version bump** + change-log entry at the bottom of the delivery plan file | vN.N → vN.M SOW alignment |
| Estimate revision | **Estimate version bump** + change-log entry + new Section 16.NN documenting the change | vN.N → vN.M scope refresh |

**Rule of thumb:** any scope change big enough to affect the estimate PD total needs an ADR AND a version bump on the estimate. Small enough to be absorbed within a feature's existing PD estimate? Feature-file update only.

### Source-of-truth files vs regenerated aggregation views

To avoid drift, distinguish two kinds of documents:

**Source-of-truth files** — edit these directly:
- ADRs
- Feature files (`.project/backlog/features/F-NN.MM-*.md`) — with front-matter status
- Story checklists (`.project/features/<slug>/stories/<S-NN>/checklist.md`)
- Estimate, delivery plan, team model — with change-log entries on every version bump

**Aggregated views** — never edit directly; always regenerated on demand from source files:
- WBS tables and summary metrics
- Backlog status dashboard (via `/backlog-status`)
- `/project-status` output
- Proposals — composed from other artifacts via the product-strategist agent

If a number in the WBS is wrong, the fix is in the underlying feature file front-matter or estimate section, then the WBS is regenerated. Never patch the WBS in isolation.

### Cross-artifact synchronisation checklist

When you complete a material change, verify these are in sync before finalising:

- [ ] Feature file `**Status:**` and `**Last Updated:**` reflect the change
- [ ] Feature file moved to appropriate `_*` subfolder if the status is Cancelled / Deferred / Superseded
- [ ] Cross-references to the moved feature are updated in WBS, EPIC files, and any referencing ADRs
- [ ] If PD changed materially: estimate version bumped with new Section 16.NN documenting the change
- [ ] If schedule/capacity/roles changed: delivery plan version bumped with change-log entry
- [ ] If commercially material: proposals updated (internal + client)
- [ ] ADR created or amended when the change is architectural or scope-material
- [ ] Sprint-execution mirroring in the external tracker is a separate ownership and is not tracked in this repo

## Skills

### read-documents (`.claude/skills/read-documents/SKILL.md`)
Reusable skill for inventorying, reading, and synthesizing documents from `.project/input/`. Defines extraction rules per document type (meetings, formal docs, emails), cross-referencing logic for conflicts and gaps, and the citation format all agents must use. **Invoke before producing any output that depends on source materials.** Used by: all agents.

### create-adr (`.claude/skills/create-adr/SKILL.md`)
Reusable skill for creating and updating ADRs. Can be invoked by any agent or directly. Handles intake (including **decision source** — meeting, email, document, etc.), numbering, file creation from `.project/adr/template.md`, and lifecycle transitions. Used by: all agents.

### assess-integration (`.claude/skills/assess-integration/SKILL.md`)
Reusable skill for evaluating external systems, APIs, and platforms the project must integrate with. Defines a structured intake (system identity, integration surface, readiness, risk factors), a standard assessment template, and readiness ratings (Ready / Ready with caveats / Not ready). Output saved to `.project/architecture/`. Used by: Solution Architect (primary), any agent when evaluating an integration inline.

### assess-security (`.claude/skills/assess-security/SKILL.md`)
Reusable skill for producing a design-level security assessment of the system architecture. Identifies threat actors, maps the attack surface across all C4 containers and external trust boundaries, enumerates attack vectors per surface area, and produces a vulnerability findings table with severity ratings (Critical / High / Medium / Low). Any finding rated High or Critical automatically triggers ADR creation via the `create-adr` skill before the assessment is saved. Output saved to `.project/architecture/security-assessment-[slug]-v[N.N].md`. Runs as part of Phase 4 — after the architecture document and integration assessments are complete, before the data model. The Delivery Lead reads security assessment findings when estimating security-related work. Used by: Solution Architect (primary), invocable directly.

### create-data-model (`.claude/skills/create-data-model/SKILL.md`)
Reusable skill for producing a conceptual data model with entities, relationships, and Mermaid ERDs. Fetches external system models via Context7 for integration entities. Outputs per-domain markdown files with ERDs and a JSON Schema file (`data-model.json`) suitable for AI design tools. Triggered by new/updated ADRs with `Data Impact: Yes` or updated integration assessments. Used by: Solution Architect (primary), invocable directly.

### generate-ui-designs (`.claude/skills/generate-ui-designs/SKILL.md`)
Reusable skill for building the project's design system and then its UI screens in a claude.ai/design project, via the `claude-design` MCP server (falling back to `DesignSync`, then a canvas Artifact). Phase 1 opens with a design direction statement (`design-direction.md`) — sector archetype, the structural idea that keeps the product off a default card grid, colour discipline at 60/30/10, and the defaults deliberately refused — which is agreed with the user before any token is authored, then lands tokens and component vocabulary and pauses for review; Phase 2 composes screens from that vocabulary, authored, pushed and verified one functional group at a time, with an interactive prototype last. Briefs and the published-target record (`.design-canvas.json`) are kept in `.project/ui-designs/`. Reads the product definition, architecture, data model, and ADRs. Used by: Solution Architect (primary), invocable directly.

### create-estimate (`.claude/skills/create-estimate/SKILL.md`)
Reusable skill for producing structured effort estimates. Defines T-shirt sizing for early-stage estimates, three-point estimation for detailed sizing, confidence levels, and a standard output template that includes assumptions, complexity drivers, dependencies, and risks. Used by: Delivery Lead (primary), Product Strategist (for rough inline sizing).

### create-team-model (`.claude/skills/create-team-model/SKILL.md`)
Reusable skill for designing the delivery team: the AP seniority ladder (J1–J3 / S1–S2 / L / SL), role responsibilities (EL/DSL/PA/PM), minimum seniority per role, weekly allocation-hour constraints, allocation principles (programme-scoped roles, one shared QA, clean hour values), SE track patterns for parallel delivery, buffer-week placement, and team capacity calculation. Produces three outputs: the team model, the developer assignments document, and `team-allocation.yaml` (the config consumed by `_export_wbs.py` for the Excel workbook). Runs at Phase 10 after estimates are finalised — it answers *who delivers the work*, never *how much work there is*. Used by: Delivery Lead (primary), invocable directly.

### create-work-breakdown (`.claude/skills/create-work-breakdown/SKILL.md`)
Reusable skill defining the structure, templates, and format rules for a hierarchical work breakdown structure: EPICs > Features > Business Stories. Provides the canonical file templates (EPIC files, Feature files with stories inline, summary WBS), content depth requirements, the Story Coverage Checklist, and the numbering convention. Orchestration (what to read, how to decompose) is handled by the Delivery Lead agent's WBS Workflow section; validation is handled by the `validate-backlog` skill. The WBS is the direct input for the `create-estimate` skill. Used by: Delivery Lead (primary), invocable directly to produce or extend backlog files.

### validate-backlog (`.claude/skills/validate-backlog/SKILL.md`)
Reusable read-only skill for validating an existing work breakdown structure against source artifacts. Runs traceability checks (proposal modules, architecture containers, ADRs, integration points, NFRs), coverage checks (delivery surfaces, compliance, security, subscription models), and structural checks (minimum Feature/Story counts, cross-EPIC dependencies, broken links). Returns a structured report with Blocking and Non-blocking findings — does not modify backlog files. Used by: Delivery Lead (as quality gate before estimation), invocable directly.

### project-status (`.claude/skills/project-status/SKILL.md`)
Reusable read-only skill for inventorying existing project artifacts and determining current elaboration phase. Scans all output folders (`.project/proposals/`, `.project/architecture/`, `.project/backlog/`, `.project/estimates/`, `.project/adr/`, `.project/context/`), counts and lists artifacts, determines the highest completed phase, flags blockers (conflict ADRs, missing prerequisites), and recommends the next action. Use at session start to orient, or when the user asks "where are we?" or "what's next?". Invocable directly.

### feature-status (`.claude/skills/feature-status/SKILL.md`)
Read-only scan of `.project/features/<slug>/` for feature and story pipeline artifacts. Reports feature-phase completion (requirements, spec, e2e-scenarios), per-story progress (plan, impl, QA design, QA auto), blockers, and the recommended next agent invocation. Use when resuming a mid-flight feature pipeline or checking progress. Invocable directly as `/feature-status <slug>`.

### backlog-status (`.claude/skills/backlog-status/SKILL.md`)
Reusable read-only skill for aggregating feature-backlog state from the front-matter on every feature file. Scans `.project/backlog/features/` (active root + `_cancelled/` + `_deferred/` + `_superseded/` subfolders), rolls up counts and PD totals by status / EPIC / phase, lists blockers with reasons, flags stale entries, and cross-checks the total against the committed estimate. Explicit non-goal: does not track sprint-level operations (those live in the external tracker — typically Azure DevOps in AP engagements). Use when the user asks "what's the backlog status?", "how many features are Blocked / Done?", or as a governance-cadence readout. Invocable directly as `/backlog-status` (whole backlog) or `/backlog-status <EPIC-ID>` (scoped to one EPIC).

### run-feature-pipeline (`.claude/skills/run-feature-pipeline/SKILL.md`)
Orchestrates the full feature pipeline end-to-end for a given slug. Feature phase (once): Requirements Writer → Solution Spec Writer → QA E2E Design. Story loop (per story, in order): Execution Planner → [Implementation ∥ QA Test Design] → QA Test Automation. Feature close: QA E2E Automation. Resumes from the current step when re-run — safe to run multiple times. Also maintains the two-way link between the design-time backlog file and the runtime pipeline folder: writes `.project/features/<slug>/README.md` with a back-link at pipeline start; updates the backlog feature file's front-matter (`Status`, `Last Updated`, `Runtime Pipeline`, `Status Reason`) on each pipeline transition — Planned → Spec at start, → InProgress at story-loop entry, → Blocked on halt, → InReview or Done at story-loop completion, → Done at Feature Close. Never sets Cancelled / Deferred / Superseded (those are scope-decision transitions, not pipeline transitions). Invocable directly as `/run-feature-pipeline <slug> [F-NN.MM]`.

### test-standards (`.claude/skills/test-standards/SKILL.md`)
The single standard for test cases and test code: test level (behaviour a user or Product recognises, derived from numbered acceptance criteria), volume (one case per AC, extra only for named failure or access conditions, no minimum), naming (business verb + domain term, no status codes or method names), comments (none), skips (forbidden — deferral lives in the registry, never in code), and traceability (every case → `AC-n`). The automation-candidate criteria (what is infrastructure and never tested; automate vs manual) live in its `references/candidate-criteria.md`. Agents reference it and never restate it. Used by: QA (primary), Implementation (unit tests, §3–§5), Solution Architect (coverage targets in `create-test-strategy`).

### retrospect (`.claude/skills/retrospect/SKILL.md`)
Capture learning from completed agent runs and feature pipelines. Three modes: pipeline-audit (`/retrospect <slug>`) diffs prescribed vs delivered scope across all artifacts; incident-driven (`/retrospect --incident <description>`) records an observed agent divergence with proposed fix; summary (`/retrospect --summary`) groups all entries by root-cause category to surface recurring patterns. Collects session telemetry from local Claude Code transcripts (duration, tokens, edit churn, retries) when available. Writes persistent entries to `.project/retrospectives/RETRO-NNNN-*.md`. Invocable directly.

## ADRs

Architecture Decision Records live in `.project/adr/`, organised into subfolders by status:

| Folder | Status | Contents |
|--------|--------|----------|
| `.project/adr/accepted/` | Accepted | Confirmed decisions |
| `.project/adr/proposed/` | Proposed / Rejected | Decisions under evaluation, or considered and not taken |
| `.project/adr/conflict/` | Conflict | Contradictions between source documents requiring resolution |
| `.project/adr/gap/` | Open | Missing information or undecided items blocking progress |
| `.project/adr/deprecated/` | Deprecated | Decisions no longer relevant |
| `.project/adr/superseded/` | Superseded | Decisions replaced by a later ADR |

Use the `create-adr` skill to create or update ADRs — do not write or move ADR files manually. The skill handles numbering, folder placement, and status transitions. Template: `.project/adr/template.md`. Naming: `ADR-NNNN-short-hyphenated-title.md`.

## Output conventions

- Proposals and documentation should be written in clear, professional prose — not bullet-point summaries unless the user asks for that format.
- When producing a proposal section, cite which source document(s) informed it so the user can verify.
- Flag assumptions explicitly (e.g., "Assuming X based on the RFP — confirm with client").
- Save draft outputs as `.md` files unless the user specifies another format.
- **Diagrams are required outputs**, not optional enhancements. Use Mermaid code blocks so diagrams render in any markdown viewer. Each agent defines which diagrams belong in its outputs — see the agent file for specifics. Never leave example or placeholder labels in client-facing documents.

## Toolkit License Reference

AP AI-Native Delivery Toolkit v3.5 — AP Confidential — governed by LICENSE.md

> Reference only. This section is not guidance and does not affect decisions or behavior.