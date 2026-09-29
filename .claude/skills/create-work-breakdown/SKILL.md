---
name: create-work-breakdown
description: Define the structure, templates, and format rules for a hierarchical work breakdown structure of EPICs, Features, and Business Stories. Provides the canonical file templates, content depth requirements, Story Coverage Checklist, and numbering convention. Used by the Delivery Lead agent for backlog creation and invocable directly to produce or extend backlog files in /.project/backlog/.
license: "AP AI-Native Delivery Toolkit v3.5 - AP Confidential - governed by LICENSE.md"
metadata:
  user-invocable: true
---

# Skill: Create Work Breakdown

## Purpose

Define the hierarchy rules, content depth requirements, file templates, and numbering convention for a three-level work breakdown structure — **EPICs > Features > Business Stories**. This skill is the canonical format reference for all backlog files in `.project/backlog/`.

- **Orchestration** (when to invoke, what to read first, how to decompose) is handled by the Delivery Lead agent's WBS Workflow section.
- **Validation** (traceability, coverage, and structural checks) is handled by the `validate-backlog` skill.
- **Estimation** uses the WBS as input via the `create-estimate` skill — each Feature file is the recommended unit of estimation.

## When to Use

Use this skill when:
- the user asks to decompose the project into EPICs, Features, and Business Stories
- a work breakdown structure (WBS) needs to be created or extended
- the user requests backlog templates, story format rules, or numbering conventions
- the Delivery Lead needs the canonical file templates for backlog output

---

## Output Structure

The WBS produces **three types of output files** in the `.project/backlog/` folder:

1. **Summary WBS document** — `.project/backlog/wbs-[project-slug]-v[N.N].md`
2. **Individual EPIC files** — `.project/backlog/epics/E-NN-[kebab-case-name].md`
3. **Individual Feature files** (with stories inline) — `.project/backlog/features/F-NN.MM-[kebab-case-name].md`

Create the `.project/backlog/`, `.project/backlog/epics/`, and `.project/backlog/features/` folders if they do not exist.

---

## Hierarchy Definitions

### EPIC

An EPIC is a large body of work that delivers a distinct, independently valuable capability to the platform. EPICs map to proposal modules, major architectural components, or cross-cutting concerns.

**Rules for EPICs:**

- Each EPIC has a clear business outcome — "when this EPIC is done, [capability] is available"
- EPICs are sized for weeks-to-months of team effort, not days
- EPICs should be deliverable in a sequence that respects dependencies
- An EPIC is NOT a phase — multiple EPICs may run in parallel within a phase
- Include cross-cutting EPICs for concerns that span the project: infrastructure, security/compliance, testing/quality, DevOps/CI-CD, onboarding/training, etc. — derive these from the architecture document and proposal, do not assume a fixed list

### Feature

A Feature is a coherent slice of an EPIC that delivers a user-visible or system-visible capability. Features are the primary unit for estimation.

**Rules for Features:**

- Each Feature belongs to exactly one EPIC
- A Feature is estimable — its scope is clear enough to size using the `create-estimate` skill
- Features within an EPIC can be prioritised and sequenced independently
- Integration with each external system is a separate Feature (not part of "backend development")
- Each ADR that requires implementation work maps to at least one Feature

### Business Story

A Business Story is a concrete, implementable unit of work within a Feature. Stories are written from the perspective of a user, system, or operator.

### Option-Dependent Stories

When a pending ADR introduces a platform consolidation alternative, some stories will differ between Option A and Option B. Handle these as follows:

- **Write the Feature description option-neutral** where possible — describe the capability the story delivers, not the vendor that delivers it.
- **Where the implementation differs materially**, add an `> Option dependency` callout block under the story:
  ```
  > **Option dependency — ADR-NNNN:** Option A: [brief description of Option A implementation]. Option B: [brief description of Option B implementation]. Gate: [which gate must clear].
  ```
- **Do not write two parallel versions** of the same story — one story with the option dependency callout is sufficient.
- **Mark gate-blocked stories** with `[BLOCKED — pending Gate N (ADR-NNNN)]` in the story title until the gate clears.
- **When the ADR resolves**, remove the option dependency callouts and gate-blocked markers from all affected stories and update the implementation description to match the confirmed stack.

**Rules for Business Stories:**

- Each Story belongs to exactly one Feature
- A Story follows the format: **"As a [role], I want [capability] so that [benefit]"** — or for technical stories: **"The system must [requirement] to satisfy [constraint/ADR]"**
- A Story includes **acceptance criteria** — a testable definition of done
- Stories should be small enough for one developer to complete in 1–5 days
- Include stories for: happy path, error handling, edge cases, accessibility, compliance, and documentation where applicable. Do not write "testing" stories — QA derives tests from each story's acceptance criteria (`test-standards` skill)

---

## Story Coverage Checklist

For every Feature, write Stories that cover the following dimensions. Not every dimension applies to every Feature — use judgement, but omit only where genuinely inapplicable.

- **Core functionality** — the happy path
- **Edge cases and error handling** — what happens when things go wrong
- **Access control** — who can do this, entitlement checks
- **Multi-surface delivery** — applicable delivery surfaces (derive from proposal scope)
- **Compliance** — regulatory requirements that apply to this feature (derive from ADRs and proposal)
- **Accessibility** — applicable accessibility standards
- **Performance** — NFR targets that apply to this feature
- **Monitoring and observability** — logging, metrics, alerting for this feature
- **Documentation** — API docs, runbooks, user guides if applicable

---

## Content Depth Requirements

**Every EPIC, Feature, and Story must include substantive content — not just titles.** The reader must be able to review any individual file and understand what is covered, what is missing, and what needs clarification without cross-referencing other documents.

#### EPIC content must include:

- **Description:** 3–5 sentences explaining what this EPIC delivers, why it matters to the business, and what the platform looks like before vs. after this EPIC is complete
- **Scope boundaries:** What is explicitly included and what is NOT part of this EPIC (to prevent scope ambiguity between EPICs)
- **Key technical components:** Which architecture containers/services are built or modified
- **Key user roles affected:** Which user types interact with capabilities from this EPIC
- **Done criteria:** What must be true for this EPIC to be considered complete

#### Feature content must include:

- **Description:** 2–3 sentences explaining the capability this feature delivers, including what user or system behaviour changes when this feature is complete
- **Scope detail:** Specific capabilities included — enumerate them, do not summarise
- **Technical approach summary:** 1–2 sentences on how this will be built
- **Surfaces:** Which delivery surfaces this feature applies to — be explicit
- **Dependencies:** Other features or EPICs that must be complete before this feature can start or function
- **Open questions / risks:** Any unresolved items from ADRs or source documents that affect this feature

#### Story content must include:

- **Full story statement:** "As a [role], I want [specific capability] so that [concrete benefit]" — or for technical stories: "The system must [specific requirement] to satisfy [constraint/ADR reference]"
- **Acceptance criteria:** 2–5 bullet points, each testable (can be verified as pass/fail). Use "Given / When / Then" format where clarity benefits from it.
- **Notes:** Any implementation hints, edge cases, or references to specific ADR decisions or NFR targets that the developer needs to know

---

## EPIC File Template

Save each EPIC to: `.project/backlog/epics/E-NN-[kebab-case-name].md`

Example: `.project/backlog/epics/E-01-project-discovery-and-architecture.md`

```markdown
# E-[NN]: [EPIC Name]

**EPIC ID:** E-[NN]
**Delivery Phase:** [Which proposal phase(s) this maps to]
**Dependencies:** [Other EPICs that must complete first or run in parallel]
**ADR References:** [Relevant ADR numbers]
**Architecture References:** [Relevant containers/services from the architecture document]

## Description

[3–5 sentences: what this EPIC delivers, why it matters, before/after state of the platform]

## Scope Boundaries

- **Includes:** [explicit list of what is in this EPIC]
- **Excludes:** [what is NOT part of this EPIC — handled by other EPICs or out of scope]

## Key Technical Components

[Architecture containers/services built or modified]

## Key User Roles Affected

[List of user types that interact with capabilities from this EPIC]

## Done Criteria

- [Criterion 1 — testable statement of what "done" means]
- [Criterion 2]
- ...

## Features

| Feature | Description | Stories | Surfaces |
|---------|-------------|---------|----------|
| [F-NN.01: Name](../features/F-NN.01-kebab-name.md) | [1-sentence description] | N | [Surfaces] |
| [F-NN.02: Name](../features/F-NN.02-kebab-name.md) | [1-sentence description] | N | [Surfaces] |
| ... | ... | ... | ... |
```

---

## Feature File Template

Save each Feature to: `.project/backlog/features/F-NN.MM-[kebab-case-name].md`

Example: `.project/backlog/features/F-01.01-product-discovery-workshop.md`

Stories are included inline within the Feature file.

```markdown
# F-[NN].[MM]: [Feature Name]

**Feature ID:** F-[NN].[MM]
**EPIC:** [E-NN: EPIC Name](../epics/E-NN-kebab-name.md)
**Surfaces:** [Web | iOS | Android | Admin Portal | Editorial Backend — list all that apply]
**Dependencies:** [Other features/EPICs required before this can start]
**ADR References:** [If applicable]
**Integration points:** [External systems involved, if any]
**Open questions / risks:** [Unresolved items affecting this feature, if any]

## Description

[2–3 sentences: what capability this delivers, what changes for the user or system]

## Scope Detail

- [Specific capability 1]
- [Specific capability 2]
- [Specific capability 3]
- ...

## Technical Approach

[1–2 sentences on implementation approach]

## Stories

---

### S-[NN].[MM].01: [Short story title]

**Estimated Hours:** TBD

As a [role], I want [specific capability] so that [concrete benefit].

_Acceptance criteria:_

- [ ] [Testable criterion 1]
- [ ] [Testable criterion 2]
- [ ] [Testable criterion 3]

_Notes:_ [Implementation hints, edge cases, .project/adr/NFR references. Omit if none.]

---

### S-[NN].[MM].02: [Short story title]

**Estimated Hours:** TBD

[Same structure]

---

[Continue for all stories in this feature]
```

---

## Summary WBS Template

Save to: `.project/backlog/wbs-[project-slug]-v[N.N].md`

Example: `.project/backlog/wbs-cnbc-dach-v1.0.md`

The summary document follows the same structure as the detailed WBS but with **summarised** EPIC and Feature content. Its purpose is to provide a single-document overview of the entire backlog with traceability matrices, while the detailed content lives in the individual files.

```markdown
# Work Breakdown Structure: [Project Name]

**Version:** [N.N]
**Date:** [YYYY-MM-DD]
**Status:** Draft
**Source documents:**

- Proposal: [filename and version]
- Architecture: [filename and version]
- ADRs: [count] accepted, [count] proposed, [count] open
- Formal documents: [list]

---

## Summary

| #         | EPIC              | Features             | Stories             | Delivery Phase(s) | Key Dependencies |
| --------- | ----------------- | -------------------- | ------------------- | ----------------- | ---------------- |
| E-01      | ...               | N                    | N                   | ...               | ...              |
| ...       | ...               | ...                  | ...                 | ...               | ...              |
| **Total** | **[count] EPICs** | **[count] Features** | **[count] Stories** |                   |                  |

---

## Dependency Map

[Mermaid diagram showing EPIC-level dependencies — which EPICs block other EPICs]

---

## EPIC Details

### E-[NN]: [EPIC Name]

**Description:** [1–2 sentence summary of what this EPIC delivers]
**Delivery Phase:** [Phase(s)]
**Dependencies:** [Other EPICs]

> Full detail: [E-NN-kebab-name.md](epics/E-NN-kebab-name.md)

| Feature | Description | Stories | Surfaces |
|---------|-------------|---------|----------|
| [F-NN.01: Name](features/F-NN.01-kebab-name.md) | [1-sentence description] | N | [Surfaces] |
| [F-NN.02: Name](features/F-NN.02-kebab-name.md) | [1-sentence description] | N | [Surfaces] |

---

[Repeat for all EPICs]

---

## Traceability Matrix

### Proposal Module → EPIC Mapping

| Proposal Module | EPIC(s)    | Coverage notes                                       |
| --------------- | ---------- | ---------------------------------------------------- |
| M1: ...         | E-01, E-02 | [Brief note on how the module is split across EPICs] |
| ...             | ...        | ...                                                  |

### ADR → Feature Mapping

| ADR      | Title | Status   | Feature(s)       | Implementation notes                                    |
| -------- | ----- | -------- | ---------------- | ------------------------------------------------------- |
| ADR-0001 | ...   | Accepted | F-08.01, F-08.02 | [How this ADR's decision drives work in these features] |
| ...      | ...   | ...      | ...              | ...                                                     |

### Integration → Feature Mapping

| External System | Integration type   | Feature(s)                | Readiness          |
| --------------- | ------------------ | ------------------------- | ------------------ |
| ...             | ...                | ...                       | ...                |

### NFR → Story Mapping

| NFR                   | Target        | Story(s)               | Verification approach         |
| --------------------- | ------------- | ---------------------- | ----------------------------- |
| ...                   | ...           | ...                    | ...                           |

---

## Coverage Notes

[Any gaps, assumptions, or items that could not be fully decomposed. Flag estimation blockers. List any scope items from the proposal or architecture that do not yet have corresponding work items, and explain why.]
```

---

## Estimated Hours Field

Every story template includes an `**Estimated Hours:** TBD` field. This field is populated by the `create-estimate` skill after estimates are produced — it should not be filled in manually during WBS creation.

The WBS Excel export (`_export_wbs.py`) reads this field and includes it as an "Estimated Hours" column in the Stories sheet. Leave the value as `TBD` until estimates are finalised.

---

## Timeline Planning Rules

When the Delivery Lead produces a phased delivery timeline or sprint plan alongside the WBS, apply these rules:

### EPIC-Level Buffer Weeks

Insert one **empty buffer week** for every 10 active working weeks within each EPIC. Label these "Buffer / Vacation / Sick Leave" — no stories, milestones, or demos are scheduled into buffer weeks.

**Rule:** buffer week falls at position `N × 10 + N` for N = 1, 2, 3, … (i.e. after every 10th active week). Apply until the EPIC ends.

| Active weeks elapsed | Buffer week inserted at |
|----------------------|-------------------------|
| 10 | Week 11 |
| 20 | Week 22 |
| 30 | Week 33 |
| 40 | Week 44 |
| … | … |

Scale to the actual EPIC length — do not assume a fixed total.

### Programme-Level Buffer Weeks (multi-EPIC engagements)

For programmes running 3 or more EPICs concurrently, add at least **4 dedicated programme-level buffer weeks** in addition to each EPIC's own buffers. Place them at key transition points:

- After each major EPIC closes — allows integration stabilisation before the next phase
- Before the final deployment phase — last opportunity for cross-workstream regression testing

Programme buffer weeks are separate from EPIC-level buffers and must appear explicitly in the programme Gantt. They represent programme-wide contingency, not individual EPIC risk. A programme shorter than 20 weeks can use 2 programme buffer weeks; scale up proportionally for longer engagements.

---

## Story Persona Naming Rules

Stories are written from the perspective of project roles. Use only the following personas — do not invent custom role names based on project terminology or org-chart titles.

| Persona | When to use |
|---------|-------------|
| **Program Architect** | Architecture design, integration design, security patterns, configuration specifications, test strategy, deployment planning, coordination plans, design documentation. All work that would traditionally be attributed to "Solution Architect", "Security Architect", "Technical Architect", or "Delivery Lead" at the story level maps to this persona. The Program Architect's 40 h/wk allocation covers all of these stories — they are not separate billed roles. |
| **Software Engineer** | All implementation stories — code changes, configuration implementation, script writing, tooling. |
| **QA Engineer** | Test strategy documents, test case creation, test automation implementation, verification and validation execution. |
| **Program Manager** | Process coordination artefacts, stakeholder communication plans. Rarely a story persona — programme management is overhead, not story-level work. |

**Do not use as story personas:** Solution Architect, Security Architect, Delivery Lead, Technical Architect, Systems Analyst, Business Analyst, DevOps Engineer, or any project-specific role name. Collapse all design/architecture/planning work into Program Architect stories; all code/config/script work into Software Engineer stories.

---

## Phase Staffing Guidelines

Apply these rules when producing a team model or resource plan alongside the WBS. They define *who joins when* across a standard delivery lifecycle.

### When each role joins each phase

| Phase | Roles active | Guidance |
|-------|-------------|----------|
| **Discovery** | Program Architect (lead), 1 SE (from week 2 of discovery) | PA drives discovery. One SE joins from the second week to review codebase structure, map dependencies, and prepare the technical baseline in parallel. Additional SEs have no concrete implementation work yet — adding them creates idle time, not speed. |
| **Implementation** | PA (full), all SEs (full from start), QA (joins at ~60% through implementation) | All SEs should be onboarded by the end of discovery. QA joins during implementation — not at V&V — to begin test case creation and automation scaffolding while code is still being written. Recommended: QA starts at 20 h/wk when approximately 60% of implementation stories are in progress, scaling to 40 h/wk as the full test suite forms. |
| **V&V / UAT** | PA (reduced), SEs (reduced), QA (full) | V&V is QA-led. PA and SEs drop to 20 h/wk for defect triage and remediation support. |
| **Deployment** | PA (reduced), 1–2 SEs (reduced), QA (reduced) | Deployment execution and environment activation. Most SEs wind down. Retain one SE at 20–40 h/wk for deployment support and hotfix capacity. |
| **Hypercare** *(optional)* | PA (reduced), 1 SE (reduced), QA (reduced) | Monitoring, triage, and minor remediation. All roles at 20 h/wk. Gradually reduces to zero as the system stabilises. **Include Hypercare when the proposal or contract commits to post-go-live monitoring and triage by the delivery team. Exclude it when support transitions to a separate support contract, client ops team, or managed service — and remove the Hypercare column from all tables below.** |

### Standard hours per role per phase

Hours scale with project complexity — the table below shows typical values. Always document deviations in the resource plan. Omit the Hypercare column when Hypercare is excluded from scope.

| Role | Discovery | Implementation | V&V | Deployment | Hypercare *(if in scope)* |
|------|-----------|----------------|-----|------------|-----------|
| Engagement Lead (L) | 4–8 h/wk | 4–8 h/wk | 4–8 h/wk | 4–8 h/wk | 4–8 h/wk |
| Delivery Solution Lead (L) | 8–16 h/wk | 8–16 h/wk | 8–16 h/wk | 8–16 h/wk | 8 h/wk |
| Program Architect (S2) | 40 h/wk | 40 h/wk | 20 h/wk | 20 h/wk | 20 h/wk |
| Program Manager (S2) | 16–40 h/wk | 16–40 h/wk | 16–20 h/wk | 16–20 h/wk | 16 h/wk |
| Software Engineer (S1–S2) | 40 h/wk (from week 2) | 40 h/wk | 20 h/wk | 20 h/wk | 20 h/wk |
| QA Engineer (S1) | — | 20 h/wk (from ~60% through impl.) | 40 h/wk | 20 h/wk | 20 h/wk |

These are defaults. Override in the resource plan with a note when a specific EPIC or project context warrants a different shape.

---

## SE Track Parallelization Patterns

When an EPIC has enough independently deliverable work, multiple SE tracks can run in parallel. Use these patterns — named by purpose, not by person — to structure parallel delivery.

### Track types

| Track type | Duration | Purpose |
|------------|----------|---------|
| **Main** | Full EPIC duration | Core application layer — the anchor SE who starts in discovery and remains through Hypercare (at reduced hours in later phases). Every EPIC has exactly one Main track. |
| **Domain** | Ends when its domain is complete | Focused on a self-contained technical concern that the Main track builds on top of. Exits cleanly when its stories are done. |
| **Burst** | Short window during peak implementation | Absorbs parallel capacity during the highest-intensity implementation window. Prefer rotating a person from a finishing EPIC or track over adding new headcount. Exits when the peak window passes. |

Name each track after the work it owns (e.g. "Backend", "Data Layer", "Batch"), not a letter.

### When to add a track

- Add a **Domain** track when a self-contained technical concern can be isolated and completed independently, freeing the Main track to build on top of it.
- Add a **Burst** track when removing it would materially extend the EPIC and a person can rotate in from a finishing track without overlap.

### Track duration rule

Never extend a track past the point where its assigned stories are complete. Idle tracks are waste. Only Track A typically continues through V&V and Hypercare.

### Cross-EPIC SE rotation

When multiple EPICs run concurrently in a programme, the same person can rotate across EPICs as peaks shift. In the programme resource model, show one row per person — not one row per EPIC role. A person who finishes on one EPIC and then moves to another is a single continuous row in the programme summary, not two separate rows.

### Peak concurrent SE guideline

A team of 6 concurrent SEs is a reasonable upper bound for most programmes. Above 6, coordination overhead grows non-linearly. If more than 6 SEs would be needed simultaneously, consider sequencing EPICs more aggressively before growing the team. Count distinct people, not track rows — two tracks held by the same person count as one.

---

## Staffing Allocation Rules

When the Delivery Lead produces a team model or resource plan alongside the WBS, apply these constraints. Hours scale with project complexity — justify deviations in the team model.

**Role responsibilities:**
- **Engagement Lead (EL):** Project budget, commercial governance, resource allocation. Default seniority: L. Increase to SL for very large or high-risk commercial engagements.
- **Delivery Solution Lead (DSL):** Technical leadership, milestone ownership, overall delivery on time and on schedule. Default seniority: L. Increase to SL for very high technical complexity or multiple concurrent workstreams.
- **Program Architect (PA):** Programme quality, architecture design, internal demos (last gate before client), documentation, solutioning. Minimum seniority: S2. Increase to L for highly complex or multi-system architecture.
- **Program Manager (PM):** Project management + Scrum Master — sprint planning, ceremonies, invoicing, cross-team coordination. Minimum seniority: S2. Increase to L for large teams or heavy governance programmes.

| Role | Typical weekly hours | Notes |
|------|---------------------|-------|
| Engagement Lead (L) | 4–8 h/wk | Scale up during active budget or resource negotiations. |
| Delivery Solution Lead (L) | 8–16 h/wk | Higher end when actively unblocking decisions or managing multiple EPICs. |
| Program Architect (S2) | 20–40 h/wk | Minimum 50% on all projects. 100% during discovery, complex design phases, or pre-client demos. |
| Program Manager (S2) | 16–40 h/wk | 16h for small teams; 40h for large or ceremony-heavy programmes. Includes SM ceremonies. |
| Software Engineer (L) | 40 h/wk; 20 h/wk only if phase wind-down warrants it | Track lead across a full EPIC. Maximise allocation. |
| Software Engineer (S2) | 40 h/wk; 20 h/wk only if phase wind-down warrants it | Domain or main track lead. Full allocation. |
| Software Engineer (S1) | 40 h/wk; 20 h/wk only if phase wind-down warrants it | Bounded domain or burst track. Full allocation. |
| Software Engineer (J3) | 40 h/wk; 20 h/wk only if phase wind-down warrants it | Must be paired with S1+. Can own well-defined stories independently. |
| Software Engineer (J2) | 40 h/wk; 20 h/wk only if phase wind-down warrants it | Must be paired with S1+. Works on clearly scoped stories with regular review. |
| Software Engineer (J1) | 40 h/wk; 20 h/wk only if phase wind-down warrants it | Must be paired with S1+. Close guidance required; never sole engineer in a phase. |
| QA Engineer (S1+) | 40 h/wk; 20 h/wk only if phase wind-down warrants it | QAs own test strategy, test plans, test design, and test automation from day 1 of discovery. They are the gate for all QA test artifacts. QA capacity must never fall below 25% of dev capacity on any active feature stream. Test case execution is a minor part of this role. |
| QA Engineer (J2–J3) | 40 h/wk; 20 h/wk only if phase wind-down warrants it | Must be paired with S1+ QA. Executes test cases under supervision; test strategy, test plan, and automation ownership always stays with S1+. |

**Allocation principles:**
- Prefer fewer people at high allocation over many people at low allocation.
- **Concentrate capacity — do not spread it.** Prefer a shorter engagement at 40 h/wk over a longer engagement at 20 h/wk. Two weeks at full capacity delivers faster and maintains sharper context than four weeks at half capacity.
- **Do not staff to a blocked dependency.** If a person cannot start meaningful work until another EPIC, track, or story completes, either start them later (after the dependency clears) or assign them to parallel work elsewhere — never model them as allocated but idle. Idle allocation is not a valid cost; it is a planning mistake.
- **All delivery roles (SE, QA) are programme-scoped, not EPIC-scoped or feature-scoped.** People move between EPICs and features as work completes — they are never assigned to a single EPIC or feature. One QA Engineer covers all EPICs sequentially. Never assign one QA Engineer per EPIC.
- **Role names use the seniority title in parentheses, not the role abbreviation.** Write `Engagement Lead (L)`, not `Engagement Lead (EL)`. Write `QA Engineer (S1)`, not `QA Engineer (PA)`. EL, DSL, PA, PM are role abbreviations — the parenthetical slot is reserved for the seniority ladder (J1–J3, S1–S2, L, SL).
- **Standard team shape:** 1 EL (L) + 1 DSL (L) + 1 PA (S2) + 1 PM (S2) + 2–3 SE tracks + 1 QA Engineer (S1, shared). DevOps is optional — where IaC and CI/CD can be absorbed by the PA and SE Backend Lead, no dedicated DevOps role is required.
- Use only clean values: 4, 8, 16, 20, 40 hrs/week. Never 32, 6, 12, 14, 18, 22, 24, 28 or similar.
- EL and DSL are leadership roles, not execution — justify any allocation above the typical range with a specific reason.
- PA below 50% becomes a rubber-stamp reviewer — either justify at 50% or increase to 100% where complexity demands it.

---

## Numbering Convention

- EPICs: `E-01`, `E-02`, ... `E-NN`
- Features: `F-[EPIC#].[SEQ]` — e.g., `F-03.01` is the first feature of EPIC 3
- Stories: `S-[EPIC#].[FEATURE-SEQ].[SEQ]` — e.g., `S-03.01.04` is the fourth story of Feature 1 of EPIC 3

### File naming convention

- EPIC files: `E-NN-[kebab-case-name].md` — e.g., `E-01-project-discovery-and-architecture.md`
- Feature files: `F-NN.MM-[kebab-case-name].md` — e.g., `F-01.01-product-discovery-workshop.md`

Derive the kebab-case name from the EPIC or Feature title: lowercase, hyphens for spaces, drop articles and conjunctions.
