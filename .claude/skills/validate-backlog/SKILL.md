---
name: validate-backlog
description: Run traceability, coverage, and structural checks against an existing work breakdown structure in /.project/backlog/. Verifies that every proposal module, architecture container, ADR, integration point, and NFR maps to specific work items. Returns a structured validation report with blocking and non-blocking findings. Use after producing or updating a WBS, before handing off to estimation, or when the user asks to check backlog completeness.
license: "AP AI-Native Delivery Toolkit v3.5 - AP Confidential - governed by LICENSE.md"
metadata:
  user-invocable: true
---

# Skill: Validate Backlog

## Purpose

Validate an existing work breakdown structure in `.project/backlog/` against the project's source artifacts. This skill reads the proposal, architecture document, ADRs, and project context, then cross-checks them against the WBS summary, EPIC files, and Feature files to confirm complete coverage.

The output is a structured validation report with findings classified as **Blocking** (must resolve before estimation) or **Non-blocking** (should be noted but do not halt progress). This skill is read-only — it never modifies backlog files.

---

## Invocation

Use this skill when:
- A WBS has been produced and needs a quality gate before estimation
- The proposal or architecture document has been updated and the existing WBS may have gaps
- The user asks to "validate the backlog", "check WBS coverage", "run traceability checks", or "is the backlog complete?"
- The Delivery Lead agent invokes it as the quality gate step in its WBS Workflow

**Prerequisites:** The backlog must exist in `.project/backlog/`. The skill reads:
- WBS summary: `.project/backlog/wbs-*.md`
- EPIC files: `.project/backlog/epics/E-NN-*.md`
- Feature files: `.project/backlog/features/F-NN.MM-*.md`
- The current proposal (from `.project/proposals/`)
- The architecture document (from `.project/architecture/`)
- All ADRs (from `.project/adr/` subfolders)
- Project context (`.project/context/project-context.md`)

---

## Traceability Checks

Verify that every source artifact maps to specific work items in the backlog.

| # | Check | Severity |
|---|-------|----------|
| T1 | Every in-scope item from the proposal's scope section maps to at least one Feature | **Blocking** |
| T2 | Every architecture container/service maps to Features covering its implementation | **Blocking** |
| T3 | Every integration point in the architecture has dedicated Feature(s) | **Blocking** |
| T4 | Every accepted ADR's implementation consequences are covered by Stories | **Blocking** |
| T5 | Every proposed ADR has a decision/evaluation Feature in the discovery EPIC | Non-blocking |
| T6 | Every open/gap ADR has an investigation Story | Non-blocking |
| T7 | Every NFR has explicit Stories for implementation and verification | **Blocking** |
| T8 | Every delivery phase milestone maps to EPIC completion criteria | Non-blocking |
| T9 | Every risk mitigation from the risk register is covered by a Story or Feature | Non-blocking |
| T10 | Post-launch support commitments are covered by an EPIC | **Blocking** |
| T11 | ADR Decision–Feature Scope Alignment: For each accepted ADR whose Change Log contains an entry dated after the WBS summary's version date, read the ADR's Decision section and the mapped Feature's Description/Scope Detail — verify the Feature reflects the ADR's *current* decision, not a prior version | **Blocking** |
| T12 | ADR Status Transition Impact: When an ADR has transitioned status (e.g., Conflict→Accepted, Proposed→Accepted, Open→Accepted) since the last WBS version date, verify that every Feature referencing that ADR has been updated to reflect the resolved decision. Check the Feature's ADR References, Description, Scope Detail, and Story acceptance criteria for stale content that contradicts the current ADR decision | **Blocking** |
| T13 | Deprecated/Superseded ADR References: Features must not list a deprecated or superseded ADR in their ADR References header without noting the replacement ADR. If a Feature's scope was built on an ADR that has since been deprecated or superseded, the Feature must either reference the replacement ADR or be flagged for scope review | Non-blocking |

### How to run T11–T13

These checks require comparing ADR content against Feature content — not just confirming a mapping exists.

1. **Identify candidate ADRs.** Read the WBS summary's version date from the header. Scan all ADR files for Change Log entries dated after that version date. These are the ADRs that may have drifted from the backlog.
2. **For each candidate ADR (T11):** Read the ADR's Decision section. Read the mapped Feature(s)' Description and Scope Detail. Look for contradictions — e.g., the ADR says "single-level approval" but the Feature says "multi-level hierarchical approval"; or the ADR says "platform-native, not delegated to third-party service" but the Feature says "third-party service handles publishing." Any semantic mismatch between the ADR decision and the Feature scope is a Blocking finding.
3. **For status transitions (T12):** Check the ADR's Status field against the status recorded in the WBS traceability matrix. If they differ (e.g., WBS says "Conflict" but ADR says "Accepted"), every Feature mapped to that ADR must be checked for stale scope.
4. **For deprecated/superseded ADRs (T13):** Grep Feature files for references to ADRs in `.project/adr/deprecated/` or `.project/adr/superseded/`. Flag any Feature that references a deprecated/superseded ADR without noting the replacement.

---

## Technology Stack Option Checks

When a Proposed ADR introduces a platform consolidation alternative (e.g., ADR-0063), run these additional checks:

| # | Check | Severity |
|---|-------|----------|
| O1 | The WBS summary header states which stack option (A or B) the backlog reflects | Non-blocking |
| O2 | Features covering components that differ between options are tagged with the option dependency (e.g., "Option A: [Vendor A]" or "Option B: [Vendor B]") | Non-blocking |
| O3 | For every gate-blocked story (stories that cannot be completed until a validation gate clears), an estimation blocker note is present in the parent Feature file | **Blocking** |
| O4 | The discovery EPIC contains a Feature for the pending ADR evaluation (running demos, getting quotes, running PoCs) | **Blocking** |
| O5 | No Feature asserts Option B as the confirmed implementation when the consolidation ADR is still Proposed | **Blocking** |

**How to run O5:** Search Feature files for Option B-specific implementation claims that name a specific vendor or product associated with the consolidation alternative. If found and the consolidation ADR is still Proposed, flag as Blocking — the Feature must either be written option-neutral or explicitly marked as contingent on that ADR.

---

## Coverage Checks

Verify that cross-cutting concerns are represented across the backlog.

| # | Check | Severity |
|---|-------|----------|
| C1 | All delivery surfaces identified in the proposal are represented where applicable | **Blocking** if a surface is entirely absent |
| C2 | Admin/operator capabilities are included (not just end-user-facing) | Non-blocking |
| C3 | Compliance requirements cascade through all EPICs that handle affected data | **Blocking** if a compliance requirement is entirely absent |
| C4 | Security requirements cascade through all EPICs | Non-blocking |
| C5 | Monitoring and observability is included for every service/container | Non-blocking |
| C6 | All subscription/access models identified in the proposal have dedicated Features | **Blocking** if a model is missing |

---

## Structural Checks

Verify that the backlog structure is well-formed.

| # | Check | Severity |
|---|-------|----------|
| S1 | No EPIC has fewer than 2 Features | **Blocking** |
| S2 | No Feature has fewer than 2 Stories | **Blocking** |
| S3 | No Story is so large it cannot be completed in 1-5 days by one developer | Non-blocking |
| S4 | Cross-EPIC dependencies are identified and noted | Non-blocking |
| S5 | No Feature file referenced in an EPIC's feature table is absent from `.project/backlog/features/` | **Blocking** |

---

## Output Format

The validation report is returned in the conversation as structured markdown. It is not written to a file — the Delivery Lead agent may copy blocking findings into the WBS summary's Coverage Notes section at its discretion.

```markdown
## Validation Report: [Project Name] WBS [version]

**Date:** YYYY-MM-DD
**WBS version validated:** [wbs filename]
**Source artifacts read:** [list of filenames]

### Summary

| Category | Blocking | Non-blocking |
|----------|----------|--------------|
| Traceability | N | N |
| Coverage | N | N |
| Structural | N | N |
| **Total** | **N** | **N** |

**Verdict:** Ready for estimation | Not ready — resolve [N] blocking findings first

---

### Blocking Findings

#### [Check ID]: [Finding title]

**What is missing:** [specific item not covered]
**Expected location:** [which EPIC or Feature should cover it]
**Source:** [which proposal module / ADR / architecture container requires this]
**Resolution:** [what needs to be added to the backlog]

---

[Repeat for each blocking finding]

---

### Non-Blocking Findings

- [Check ID]: [Finding description] — [suggested action]

---

### Passed Checks

- [Check ID]: [Brief confirmation of what was verified]
```

---

## When the Backlog Is Empty

If `.project/backlog/` does not exist or contains no WBS summary, report this as a single blocking finding:

> **No WBS summary found.** The backlog cannot be validated until a `wbs-*.md` summary exists in `.project/backlog/`. Run the Delivery Lead agent's WBS Workflow to produce a work breakdown structure before validation can proceed.

Do not attempt to validate against missing files.
