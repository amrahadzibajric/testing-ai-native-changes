---
name: backlog-status
allowed-tools: Read, Glob, Grep
description: Aggregate and report the current state of the feature backlog from feature-file front-matter. Scans .project/backlog/features/ (active root + _cancelled/_deferred/_superseded subfolders), rolls up counts and PD totals by status / EPIC / phase, lists blockers, flags stale entries, and cross-checks against the committed estimate. Read-only — never modifies files. Sprint execution is the external tracker's responsibility (typically Azure DevOps in AP engagements); this skill reports design-time backlog state only.
license: "AP AI-Native Delivery Toolkit v3.5 - AP Confidential - governed by LICENSE.md"
metadata:
  user-invocable: true
---

# Skill: Backlog Status

## Purpose

Produce a consolidated status report of the feature backlog by scanning the front-matter on every feature file. Answers questions like:

- How many features are Planned / InProgress / Done / Blocked?
- What's committed vs deferred vs cancelled?
- Which EPICs still have unstarted features?
- Which features are Blocked and why?
- Does the sum of active feature PDs still reconcile to the committed estimate?

This skill is **read-only** — it never modifies backlog files, feature files, or estimates. It reports; it does not fix.

**Scope boundary:** this skill reports the design-time backlog state. Sprint-level execution — story assignments, daily progress, in-sprint burndown — lives in the external sprint-tracking tool (typically Azure DevOps in AP engagements), not in this repository. The `Sprint Range` front-matter field is advisory only (WBS design-time intent); the report treats it as reference metadata, not as operational sprint status.

---

## Invocation

Use this skill when:

- The user asks "what's the backlog status?", "how many features are done?", "which features are blocked?"
- Before a governance readout — snapshot the current commitment state
- After a scope change (feature added / cancelled / deferred) — confirm the new totals reconcile
- Regularly (weekly or per fortnight) as a health-check on backlog hygiene

Invocable directly as `/backlog-status` (whole backlog) or `/backlog-status <EPIC-ID>` (scope report to one EPIC).

---

## Data Sources

The skill reads (never writes):

| Location | What it reads | Why |
|---|---|---|
| `.project/backlog/features/*.md` | Front-matter of every active feature | Primary status source |
| `.project/backlog/features/_cancelled/*.md` | Cancelled features | Included in aggregate counts (Cancelled bucket) |
| `.project/backlog/features/_deferred/*.md` | Deferred features | Included in aggregate counts (Deferred bucket) |
| `.project/backlog/features/_superseded/*.md` | Superseded features | Included in aggregate counts (Superseded bucket) |
| `.project/backlog/epics/E-*.md` | EPIC titles + sprint ranges | For per-EPIC rollup labels |
| Latest `.project/backlog/wbs-*.md` | Summary metrics for cross-check | Compare to feature-level rollup |
| Latest `.project/estimates/estimate-*.md` | Committed PD total | Reconciliation check |

---

## Front-Matter Fields Consumed

For every `F-NN.MM-*.md` file, extract these fields from the header block:

| Field | Required? | How it's used |
|---|---|---|
| `**Feature ID:**` | Yes | Identity |
| `**EPIC:**` | Yes | Group-by dimension (extract `E-NN` from link) |
| `**Status:**` | Yes | Primary status enum |
| `**Phase:**` | Yes | Group-by dimension (Phase 2 / 2.x / 3) |
| `**Sprint Range:**` | No (advisory) | Reported for context only, NOT used as an operational status |
| `**PD Estimate:**` | Yes | Sum for total PD calculation; `—` treated as zero |
| `**Last Updated:**` | Yes | For stale-entry flagging |
| `**Status Reason:**` | Yes | Included verbatim in Blocked / Cancelled / Deferred / Superseded detail sections |
| `**Cancelled Reason:**` | Conditional | Detail for cancelled features |
| `**Superseded By:**` | Conditional | Detail for superseded features |
| `**Deferred To:**` | Conditional | Detail for deferred features |
| `**Runtime Pipeline:**` | Conditional | Link to `.project/features/<slug>/` if pipeline has started |
| `**PD Actual:**` | Conditional | Reported alongside PD Estimate for closed-out features |

If a file is missing required front-matter fields, report it as a data-quality error rather than silently excluding it.

---

## Report Structure

### 1. Headline Summary

Report at the top:

- Total feature count (all subfolders included)
- Active feature count (Status = Planned / Discovery / Spec / InProgress / InReview / Blocked / Done)
- Inactive feature count (Cancelled / Deferred / Superseded)
- Total committed PD (sum of active features)
- Total delivered PD (sum of PD Actual for Done features, if populated)
- Reconciliation vs the latest estimate's committed PD total (delta if any)

### 2. Status Distribution Table

| Status | Count | Total PD | % of active PD |
|---|---:|---:|---:|
| Planned | ... | ... | ... |
| Discovery | ... | ... | ... |
| Spec | ... | ... | ... |
| InProgress | ... | ... | ... |
| InReview | ... | ... | ... |
| Blocked | ... | ... | ... |
| Done | ... | ... | ... |
| **Active total** | **...** | **...** | **100%** |
| Cancelled *(inactive)* | ... | — | — |
| Deferred *(inactive)* | ... | *(retained sizing for re-add reference)* | — |
| Superseded *(inactive)* | ... | — | — |
| **Grand total** | **N** | | |

### 3. Per-EPIC Rollup

For each EPIC (grouped by presentation group if the WBS defines one):

| EPIC | Title | Active | Planned | InProgress | Done | Blocked | PD |
|---|---|---:|---:|---:|---:|---:|---:|
| E-01 | ... | ... | ... | ... | ... | ... | ... |
| ... | | | | | | | |

Highlight rows where:
- Blocked count > 0 (warning icon)
- All features Done (checkmark — EPIC complete)
- No features Planned or higher (attention — EPIC has no active work)

### 4. Per-Phase Rollup

Simple table by Phase field:

| Phase | Feature count | PD |
|---|---:|---:|
| Phase 2 | ... | ... |
| Phase 2.x | ... | ... |
| Phase 3 setup | ... | ... |

### 5. Blockers

For every feature with `Status: Blocked`, list:

- Feature ID + title
- EPIC
- `Status Reason` (verbatim — this is why it's blocked)
- `Last Updated` (how long has it been blocked?)
- Recommended action (if inferable from Status Reason — e.g., "waiting on ADR-XXXX resolution")

If no blockers: report "No blocked features."

### 6. Stale Entries

Flag features that haven't been updated in more than 14 days if their status is `InProgress`, `Discovery`, `Spec`, or `InReview` (transitional states shouldn't sit still for long). List:

- Feature ID + title
- Status
- Last Updated
- Days since update
- Suggested action (nudge status, re-verify, or update)

### 7. Recently Changed

Features updated in the last 7 days. Useful for governance-cadence readouts:

- Feature ID
- Prior status → current status (if inferable from git log)
- Last Updated
- Status Reason (summary)

### 8. Cross-Reference Reconciliation

Compare feature-level rollup to authoritative source figures:

- Sum of active feature PDs vs `estimate-*.md` v-latest committed PD → report delta
- Feature count in root vs count in WBS Summary section → report delta
- Active features count vs "Active features" number in WBS → report delta
- Deferred/Cancelled/Superseded counts vs the exclusion notes in the WBS → confirm consistent

Any delta > 5 PD or > 2 features triggers a **RECONCILIATION WARNING** in the report.

### 9. Runtime Pipeline Coverage

If any features have a `Runtime Pipeline:` field populated, report:

- Count of features with pipeline started
- Which features have started but not completed the pipeline
- Cross-check against `/feature-status <slug>` output (do the runtime artifacts exist?)

If no features have started their pipeline yet: report "No features have entered the feature pipeline yet — this is expected pre-Sprint 0."

---

## Output Format

Produce a Markdown report. Structure:

```markdown
# Backlog Status — YYYY-MM-DD

## Headline
[key metrics]

## Status Distribution
[table]

## Per-EPIC Rollup
[table]

## Per-Phase Rollup
[table]

## Blockers
[list or "No blocked features"]

## Stale Entries
[list or "No stale entries"]

## Recently Changed (last 7 days)
[list or "No recent changes"]

## Reconciliation
[cross-checks with delta reporting]

## Runtime Pipeline Coverage
[stats or "Pre-Sprint 0 — pipeline not started"]

## Recommended Next Actions
[bullet list of 1–3 items based on the report — e.g. "Address 2 blockers", "Reconciliation delta of 15 PD — review estimate against feature files", "Nudge 3 stale entries"]
```

Do not save the report to disk — return it as the skill's output text. If the user wants a persisted snapshot, they can ask explicitly (in which case save to `.project/state/backlog-status-YYYY-MM-DD.md`).

---

## Failure Modes

If the skill encounters:

- **Missing front-matter on a feature file** — report the file as a data-quality error with the missing field, continue with the rest
- **Malformed PD Estimate** — treat as zero, flag as error
- **No feature files found** — report "No feature files under .project/backlog/features/" and exit
- **Conflict between root and subfolder** (e.g., a feature exists in both root and _superseded) — report as a critical error, do not attempt to resolve

---

## Relationship to Other Skills

- **`/project-status`** — engagement-level view; call `/backlog-status` for backlog-specific detail
- **`/feature-status <slug>`** — single-feature pipeline detail; call this for one feature; call `/backlog-status` for the whole backlog
- **`/validate-backlog`** — structural + traceability validation of the backlog against source artifacts; `/backlog-status` reports *state*, `/validate-backlog` reports *correctness*
- **`create-work-breakdown`** — creates/extends backlog files; not consumed by `/backlog-status`
- **`run-feature-pipeline`** — advances a feature through Discovery / Spec / InProgress; each transition updates the feature front-matter that `/backlog-status` then reads

---

## Non-Goals (What This Skill Does NOT Do)

- **No sprint-level operational tracking.** Sprint execution — story-level assignments, daily standups, sprint burndown — happens in the external sprint-tracking tool (typically Azure DevOps in AP engagements), not in this repository. The `Sprint Range` field in feature front-matter is advisory only; this skill reports it as metadata, never as operational status.
- **No file modifications.** Read-only.
- **No feature pipeline execution.** For that use `/run-feature-pipeline`.
- **No backlog validation.** For that use `/validate-backlog`.
- **No estimate revision.** For that use `create-estimate` skill or update the estimate file directly.
- **No project-management-tool integration.** The external tracker's state (in-sprint story states) is not read back into this skill.
