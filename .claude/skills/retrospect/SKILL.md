---
name: retrospect
description: Capture learning from completed agent runs and feature pipelines. Diffs prescribed vs delivered scope, classifies root causes, collects session telemetry from local transcripts, and proposes targeted fixes with exact text. Use when reviewing a completed feature pipeline ("/retrospect {slug}"), recording an observed agent divergence ("/retrospect --incident {description}"), or surfacing recurring patterns ("/retrospect --summary"). Triggers on "/retrospect", "pipeline audit", "what went wrong", "root cause", "how can we improve this skill".
allowed-tools: Read, Glob, Grep, Bash
license: "AP AI-Native Delivery Toolkit v3.5 - AP Confidential - governed by LICENSE.md"
metadata:
  user-invocable: true
---

# Skill: Retrospect

## Purpose

Walk completed feature pipelines or agent behavior incidents to produce a persistent learning record (`.project/retrospectives/RETRO-NNNN-*.md`). Each entry captures what happened, what was expected, the divergence, a root-cause classification, and a targeted fix proposal with exact text.

Session telemetry is collected automatically from local Claude Code transcript files (JSONL) when they are present — duration, token use, tool call counts, edit churn, retries. Missing transcripts are never an error.

`Bash` is in `allowed-tools` solely for telemetry collection (inline-Python transcript parsing). The skill's primary read/scan logic uses Read/Glob/Grep only.

---

## When to Use

Use this skill when:
- reviewing a completed or in-progress feature pipeline for scope gaps and missed prescriptions
- recording an observed agent divergence or unexpected behavior as an incident entry
- surfacing recurring root-cause patterns across retrospectives with `/retrospect --summary`
- the user runs `/retrospect`, asks "what went wrong", "pipeline audit", or "root cause"

## Invocation

```
/retrospect <slug>                       # pipeline audit mode — slug or F-NN.MM backlog code
/retrospect --incident <description>     # incident mode — record an observed divergence
/retrospect --summary                    # summary mode — group all entries by root-cause category
```

---

## Mode 1: Pipeline Audit

Walk a completed or in-progress feature pipeline and diff prescribed scope against delivered scope.

**Algorithm:**

1. **Resolve slug** — accept slug or `F-NN.MM` code; scan `.project/backlog/features/` for a matching filename. Strip `F-NN.MM-` prefix and `.md` suffix to get slug. Error and stop if unresolvable.

2. **Build prescribed scope** — for each agent that participates in the feature pipeline (requirements-writer, solution-spec-writer, execution-planner, implementation, qa), read its `.claude/agents/<agent>.md` frontmatter `skills:` list. For each referenced skill, read its `SKILL.md` and extract checklist items and numbered bullets. Each prescription carries: `(skill, item, loaded-by-agent)`.

3. **Build delivered scope** — extract from feature artifacts:
   - FR-### identifiers and section headings from `requirements.md`
   - Section headings and key decisions from `spec.md`
   - Checklist items from each story's `checklist.md`
   - Test case identifiers from each story's `test-cases.md`
   - Verification entries from `verification-report.md` (if present)

4. **Diff** — produce two finding lists:
   - **Silent drops** — prescribed items not present (by keyword/concept match) in any downstream artifact
   - **Orphan additions** — delivered items not anchored to any source (skill / ADR / product-definition / requirements)

5. **Classify each silent drop by the phase where it dropped:**

   | Classification | What happened |
   |---|---|
   | `requirements-gate` | Skill not loaded by requirements-writer; prescription couldn't enter the FR list |
   | `spec-gate` | Was in requirements but not in spec |
   | `planning-gate` | Was in spec but not in execution-plan |
   | `implementation-gate` | Planned but no checklist tick or code artifact |
   | `verification-gate` | Implemented but no test case for its acceptance criterion |
   | `verification-surplus` | Test case with no `Source: AC-n` / `FR-nnn`, or more than one case per AC without a named failure condition (`test-standards` §2, §6) |

6. **Collect telemetry** — see Session Telemetry section.

7. **Propose targeted fix per finding:**

   | Classification | Fix target |
   |---|---|
   | `requirements-gate` | Add FR-NNN to requirements.md + consider adding `<skill>` to `<agent>`'s skill list |
   | `spec-gate` | Add section to spec.md |
   | `planning-gate` | Add step to execution-plan.md |
   | `implementation-gate` | Add or reopen checklist item |
   | `verification-gate` | Add test case to test-cases.md |
   | `verification-surplus` | Delete the case from test-cases.md and its test code; if behaviour is real, add the missing AC upstream instead |
   | `tooling-friction` | Fix target depends on telemetry signal — tool allowlist, agent prompt, skill load order, CLAUDE.md clarity |

8. **Write output** to `.project/retrospectives/RETRO-NNNN-pipeline-audit-<slug>.md` and return summary in conversation.

---

## Mode 2: Incident-Driven

Capture an observed agent behavior divergence with proposed fix.

**Algorithm:**

1. **Gather facts** — accept free-form description as argument. If absent, ask three structured questions:
   - Which agent ran?
   - What did you observe?
   - What did you expect?

2. **Detect recent writes** — run `git status --porcelain` if in a repo; list files written outside the agent's allowed phase outputs based on its `.claude/agents/<agent>.md`.

3. **Compare against contract** — read the relevant agent `.md` (allowed outputs, phase) and the CLAUDE.md phase table.

4. **Classify root cause:**

   | Category | Meaning |
   |---|---|
   | `prompt-override` | Orchestrator or user prompt explicitly asked for out-of-phase output |
   | `agent-definition-gap` | Agent file has only a soft default, no hard constraint |
   | `phase-contract-ambiguity` | CLAUDE.md phase table is unclear or missing for this case |
   | `skill-boundary-missing` | Skill doesn't guard its own write scope |
   | `tooling-friction` | Excessive duration, token use, retries, or edit churn (surfaceable from telemetry) |

5. **Collect telemetry** — see Session Telemetry section.

6. **Propose fix with exact text** — include file path, line anchor, and the exact text to add or change.

7. **Write output** to `.project/retrospectives/RETRO-NNNN-incident-<short-slug>.md` and return summary.

---

## Mode 3: Summary

Group all retrospective entries by root-cause category to surface recurring patterns. Output is in-conversation only — no file write.

**Algorithm:**

1. Glob `.project/retrospectives/RETRO-*.md`, read frontmatter only (id, date, mode, subject, root_cause_category, status).
2. Group by `root_cause_category`. Within each group, list entries (id, date, subject, status).
3. Flag agents or skills appearing in 2+ entries — recurring patterns warrant a structural fix.
4. Report total counts per category and overall open vs. resolved split.

---

## Session Telemetry (Transcript Analysis)

Attempted automatically in pipeline-audit and incident modes. Missing transcripts are never an error — the retrospective proceeds without them and records `telemetry: not-collected`.

**Discovery:**

1. Sanitize current working directory to the Claude Code project key: replace path separators and `:` with `-`, lowercase the drive letter (e.g., `c:\git\foo` → `c--git-foo`).
2. Glob `~/.claude/projects/<key>/*.jsonl`. If none exist, skip.
3. Map transcript to the feature run (heuristic):
   - For pipeline-audit: Grep each transcript for the feature slug in `tool_use` file paths or message text. Sessions with ≥3 matches are candidates; rank by recency.
   - For incident: use the most recent session with writes to the agent's expected output folder, or accept an approximate timestamp from the user.
4. Always record chosen sessions by uuid and timestamp range in the output so the reader can verify.

**Metrics extracted** (stream-parse JSONL via inline Python, one `json.loads` per line):

| Metric | Source | Signal |
|---|---|---|
| Wall-clock duration | `timestamp(last) − timestamp(first)` summed across sessions | Absolute latency |
| Per-agent phase time | Group `Agent` tool_use by `subagent_type`; diff adjacent assistant timestamps | Bottleneck agent identification |
| Token usage | Sum `usage.{input_tokens, output_tokens, cache_creation_input_tokens, cache_read_input_tokens}` | Cost and context-pressure |
| Tool call counts by name | Count `tool_use` blocks per `name` | Over/under-use of specific tools |
| Tool failure rate | Count `tool_result` records with `is_error: true` per tool | Permission or environment friction |
| Retry / course-correction count | User messages interleaved mid-run + assistant text containing "let me try", "actually", "wait" | Instruction churn |
| Edit churn per file | Count `Edit` / `Write` calls by `file_path`; rewrites of the same file | Flailing or unclear intent |
| Subagent fan-out | Count `Agent` tool_use by `subagent_type` | Delegation pattern check |
| Dead-end indicator | Last record's role + presence of mid-pipeline abort | Abandoned run detection |

**Tooling-friction thresholds** (defaults; record as configurable in the entry):

- Wall-clock > 2× median of comparable features → finding
- Tool failure rate > 10% on any single tool → finding
- Edit churn > 5 rewrites of the same file → finding
- Retry/course-correction count > 10 per session → finding

**Privacy rules:** aggregate metrics only — counts, durations, totals. Never copy raw user prompts or assistant text into the entry. Session uuid and timestamp range are recorded so a reader can open the source transcript for deeper inspection.

**Graceful degradation:** on parse error, file-too-large, or permission denied, record the failure reason in Section 8 and continue.

---

## Output Artifact Format

Write to `.project/retrospectives/RETRO-NNNN-<mode-short>-<slug>.md`:

```markdown
---
id: RETRO-NNNN
date: YYYY-MM-DD
mode: pipeline-audit | incident
subject: <feature-slug or agent-name or short description>
root_cause_category: prompt-override | agent-definition-gap | phase-contract-ambiguity | skill-boundary-missing | requirements-gate | spec-gate | planning-gate | implementation-gate | verification-gate | tooling-friction
status: Open | Fix Proposed | Fix Applied | Closed
related: [RETRO-NNNN, ...]
telemetry: collected | not-collected | failed
---

# Retrospective: <title>

## 1. What happened
<facts — agent, phase, artifacts produced, observable divergence>

## 2. What was expected
<contract — what the agent file / skill / phase table specifies>

## 3. Divergence
| | Expected | Actual | Delta |
|---|---|---|---|
| ... | ... | ... | ... |

## 4. Root cause
<category + 2–3 sentence narrative>

## 5. Proposed fix
<file path, line anchor, and exact text to add or change>

## 6. Fix status
<Open / Applied / Won't fix + rationale>

## 7. Related retrospectives
<links to prior RETRO-NNNN entries with the same root cause or subject>

## 8. Session telemetry
<one of:
 — "Not collected — no transcripts found at `~/.claude/projects/<key>/`."
 — "Not collected — no transcripts could be confidently mapped to this feature/incident."
 — "Collection failed: <reason>."
 — Aggregated metrics block (sessions by uuid + timestamp range, duration, tokens, tool calls,
    retries, edit churn, subagent fan-out). Metrics only — no raw text excerpts.>
```

---

## Numbering Convention

`RETRO-NNNN` — 4-digit sequential, zero-padded. Compute by globbing `.project/retrospectives/RETRO-*.md`, extracting the numeric portion from each filename, taking max + 1, zero-padding to 4 digits. Start at `0001` when none exist.

---

## Write Constraint

After any invocation, `git status` must show changes only under `.project/retrospectives/` (modes 1 and 2) or no changes (mode 3). Any write to any other path is a bug in this skill.
