---
name: project-status
description: Inventory existing project artifacts across all output folders and report current elaboration phase, completed work, missing artifacts, and recommended next steps. Use when the user asks "where are we?", "what's the project status?", "what should I do next?", or at the start of a new session to orient.
license: "AP AI-Native Delivery Toolkit v3.5 - AP Confidential - governed by LICENSE.md"
metadata:
  user-invocable: true
---

# Skill: Project Status

## Purpose

Scan the workspace to determine the current state of project elaboration. Report which artifacts exist, which are missing, what phase the project is in, and what the recommended next action is. This skill is read-only — it never creates or modifies files.

---

## Invocation

Use this skill when:
- The user asks "where are we?", "what's the status?", "what do I do next?", or similar orientation questions
- A new session starts and the user wants to pick up where they left off
- Before starting a major phase of work, to confirm prerequisites are in place
- After completing a phase, to confirm outputs were produced and identify the next step

---

## Artifact Inventory

Scan the following locations and count files matching the expected patterns:

| Category | Location | Pattern | What to report |
|----------|----------|---------|----------------|
| Source documents | `.project/input/` | `*` excluding `README.md` | Count of source files |
| Project context | `.project/context/` | `project-context.md` | Exists / missing |
| Intake records | `.project/intake/` | `intake-*.md` | Count, latest filename |
| Product definition — modules | `.project/product-definition/modules/` | `*.md` (excluding README) | Count of modules |
| Product definition — scope | `.project/product-definition/` | `scope.md` | Exists / missing, populated / empty |
| Product definition — NFRs | `.project/product-definition/` | `nfrs.md` | Exists / missing, populated / empty |
| Product definition — E2E scenarios | `.project/product-definition/` | `e2e-scenarios.md` | Exists / missing, populated / empty |
| Product definition — integrations | `.project/product-definition/` | `integration-points.md` | Exists / missing, populated / empty |
| Proposals | `.project/proposals/` | `proposal-*-v*.md` | Count, latest version |
| Architecture docs | `.project/architecture/` | `arch-*.md` | Count, latest filename |
| Integration assessments | `.project/architecture/` | `integration-*.md` | Count, list system names |
| Tech evaluations | `.project/architecture/` | `tech-eval-*.md` | Count, list topics |
| Test strategy | `.project/architecture/` | `test-strategy-*.md` | Exists / missing, version |
| ADRs — Accepted | `.project/adr/accepted/` | `ADR-*.md` | Count |
| ADRs — Proposed | `.project/adr/proposed/` | `ADR-*.md` | Count |
| ADRs — Conflict | `.project/adr/conflict/` | `ADR-*.md` | Count (flag if > 0) |
| ADRs — Open | `.project/adr/gap/` | `ADR-*.md` | Count (flag if > 0) |
| ADRs — Deprecated | `.project/adr/deprecated/` | `ADR-*.md` | Count |
| ADRs — Superseded | `.project/adr/superseded/` | `ADR-*.md` | Count |
| Data model | `.project/data-model/` | `*.md`, `data-model.json` | Exists / missing, domain count, JSON Schema present |
| UI designs | `.project/ui-designs/` | `*-web.md`, `*-mobile.md`, `assets/**` | Exists / missing, group count, web+mobile pairs, downloaded assets |
| WBS summary | `.project/backlog/` | `wbs-*.md` | Exists / missing, version |
| EPICs | `.project/backlog/epics/` | `E-*.md` | Count |
| Features | `.project/backlog/features/` | `F-*.md` | Count |
| Estimates | `.project/estimates/` | `estimate-*.md` | Count, list scopes |
| Delivery plan | `.project/estimates/` | `delivery-plan-*.md` | Exists / missing, version |
| Team model | `.project/estimates/` | `team-model-*.md` | Exists / missing, version |

---

## Phase Determination

Based on the artifact inventory, determine which elaboration phase the project is in. Phases are cumulative — later phases require earlier phases to be complete.

| Phase | Name | Complete when | Typical agent |
|-------|------|---------------|---------------|
| 0 | **Not started** | No source documents uploaded, excluding folder template READMEs | — |
| 1 | **Documents uploaded** | ≥1 real source document in `.project/input/` | — |
| 2 | **Intake complete** | Intake record exists in `.project/intake/` | Product Strategist |
| 3 | **Product defined** | ≥1 module file in `.project/product-definition/modules/` + `scope.md` populated | Product Strategist |
| 4 | **Architecture designed** | ≥1 architecture document in `.project/architecture/` + test strategy in `.project/architecture/test-strategy-*.md` + security assessment in `.project/architecture/security-assessment-*.md` | Solution Architect |
| 5 | **Data model created** | ≥1 domain file + `data-model.json` in `.project/data-model/` | Solution Architect |
| 6 | **UI designs generated** | ≥1 design in `.project/ui-designs/` | Solution Architect |
| 7 | **WBS produced** | WBS summary + EPIC + Feature files in `.project/backlog/` | Delivery Lead |
| 8 | **WBS validated** | Validation has been run (check Coverage Notes in WBS for validation reference) | Delivery Lead |
| 9 | **Estimated** | ≥1 estimate in `.project/estimates/` | Delivery Lead |
| 10 | **Delivery planned** | Delivery plan + team model in `.project/estimates/` | Delivery Lead |
| 11 | **Proposal composed** | ≥1 proposal version in `.project/proposals/` referencing estimates, team model, and delivery plan | Product Strategist |

Report the **highest completed phase** and the **current phase** (the next one to work on).

---

## Technology Stack Decision Status

When reporting project status, check for pending platform consolidation ADRs (Proposed ADRs that replace multiple other ADRs). If found:

1. **Report the pending decision prominently** under Blockers & Warnings:
   - ADR number and title
   - Which other ADRs it supersedes or affects (e.g., ADR-0018, ADR-0035, ADR-0039, ADR-0057)
   - Validation gates remaining and their target resolution date
   - Impact: which EPICs have option-dependent estimates

2. **Include a Technology Stack Decision section** in the status output:

```markdown
### Technology Stack Decision (Pending)

| ADR | Title | Status | Gates remaining | Target |
|-----|-------|--------|-----------------|--------|
| ADR-NNNN | [title] | Proposed | Gate N, Gate N | YYYY-MM-DD |

**Impact:** Option A and Option B estimates exist. EPICs [list] have gate-blocked stories.
**Next action:** [Gate owner] to [specific action] by [date].
```

3. **Suppress the "stack decision pending" warning once all gates are cleared** and the ADR moves to Accepted, Rejected, or a hybrid outcome is recorded.

---

## Blockers and Warnings

Flag these conditions prominently:

- **Conflict ADRs > 0** — Contradictions between source documents need resolution before dependent work proceeds
- **Open/Gap ADRs > 0** — Missing information that may block estimation or architectural decisions
- **Product definition missing but architecture exists** — Architecture may have been designed without a confirmed product definition; check alignment
- **Product definition missing but WBS exists** — WBS may be based on proposal prose rather than structured module definitions
- **Architecture missing but WBS exists** — WBS may be based on assumptions rather than confirmed architecture
- **Architecture exists but no test strategy** — Phase 4 incomplete; QA agent will lack framework/tooling guidance when writing test plans
- **Architecture exists but no security assessment** — Phase 4 incomplete; High/Critical design-level findings will not have been captured or converted to ADRs before downstream phases begin
- **WBS exists but no validation reference** — Quality gate may not have been run
- **Estimates exist but no delivery plan** — Estimates need to be rolled into a delivery plan

---

## Output Format

Return the status report in the conversation as structured markdown. Do not write it to a file.

```markdown
## Project Status: [Project Name]

**Date:** YYYY-MM-DD
**Current phase:** [Phase N — Name]
**Next action:** [What should happen next]

---

### Artifact Inventory

| Category | Count | Latest / Notes |
|----------|-------|----------------|
| Source documents | N | [list filenames or "(none)"] |
| Project context | ✓ / ✗ | |
| Intake records | N | [latest filename] |
| Product definition — modules | N | [module names] |
| Product definition — scope | ✓ / ✗ | populated / empty |
| Product definition — NFRs | ✓ / ✗ | populated / empty |
| Product definition — E2E scenarios | ✓ / ✗ | populated / empty |
| Product definition — integrations | ✓ / ✗ | populated / empty |
| Proposals | N | [latest version] |
| Architecture docs | N | [latest filename] |
| Integration assessments | N | [system names] |
| Tech evaluations | N | [topics] |
| Test strategy | ✓ / ✗ | [version] |
| Data model | ✓ / ✗ | [N domains, JSON Schema ✓/✗] |
| UI designs | ✓ / ✗ | [N screens] |
| ADRs | N total | N accepted, N proposed, N conflict, N open, N deprecated, N superseded |
| WBS summary | ✓ / ✗ | [version] |
| EPICs | N | |
| Features | N | |
| Estimates | N | [scopes] |
| Delivery plan | ✓ / ✗ | [version] |
| Team model | ✓ / ✗ | [version] |

---

### Phase Progress

- [x] Phase 1: Documents uploaded
- [x] Phase 2: Intake complete
- [x] Phase 3: Product defined
- [ ] Phase 4: Architecture designed ← **current**
- [ ] Phase 5: Data model created
- [ ] Phase 6: UI designs generated
- [ ] Phase 7: WBS produced
- [ ] Phase 8: WBS validated
- [ ] Phase 9: Estimated
- [ ] Phase 10: Delivery planned
- [ ] Phase 11: Proposal composed

---

### Blockers & Warnings

- [Any blockers or warnings from the checks above, or "None" if clean]

---

### Recommended Next Steps

1. [Most important next action — which agent/skill to invoke and why]
2. [Secondary action if applicable]
3. [Any housekeeping — resolve conflicts, close open ADRs, etc.]
```

---

## When No Project Exists

If the workspace has no source documents, project context, or any output artifacts, report:

> **No project found.** Upload source documents to `.project/input/` and run an intake session with the Product Strategist to begin.
