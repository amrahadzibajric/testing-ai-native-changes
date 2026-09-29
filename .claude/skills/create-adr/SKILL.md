---
name: create-adr
description: Create or update an Architecture Decision Record in /.project/adr/. Handles intake, numbering, folder placement by status (accepted/proposed/conflict/gap/deprecated/superseded), and lifecycle transitions.
license: "AP AI-Native Delivery Toolkit v3.5 - AP Confidential - governed by LICENSE.md"
metadata:
  user-invocable: true
---

# Skill: Create ADR

## Purpose

Create a new Architecture Decision Record (ADR) in `.project/adr/`, based on information gathered from the user or extracted from source materials.

---

## Invocation

Use this skill when:
- A significant decision has been made or needs to be recorded (technology choice, integration approach, scope boundary, architectural pattern, vendor selection, process agreement)
- A previously open question is resolved
- A decision is explicitly deferred or rejected with documented rationale
- The user says something like "log this decision", "create an ADR for...", or "record that we decided to..."

---

## Intake Questions

Ask only the questions not already answered by available context (uploaded documents, prior conversation, or the agent's Project Context section).

1. **What is the decision?** — One sentence: "We will [do X]."
2. **What triggered this decision?** — What problem, requirement, or event made this decision necessary?
3. **What is the source?** — Where did this decision originate or get confirmed?
   - Meeting (name, date, participants)
   - Email (sender, date, subject)
   - Document (name, section)
   - Workshop or stakeholder interview
   - Internal discussion (with whom, when)
4. **What options were on the table?** — List the alternatives considered, even if briefly.
5. **What criteria drove the choice?** — e.g., cost, speed, team familiarity, scalability, compliance.
6. **Who are the deciders?** — Names or roles of people who made or approved the decision.
7. **What are the known trade-offs or follow-up actions?**

> If the user says "just log it" with minimal detail, capture what is known and mark unclear fields as `[To be confirmed]` rather than blocking on completeness.

---

## Numbering

Before creating a new ADR:
1. Check `.project/adr/` and all its subfolders for existing files matching `ADR-NNNN-*.md`.
2. Use the next available number, zero-padded to 4 digits (e.g., `ADR-0003-...`).
3. If no ADRs exist yet, start at `ADR-0001`.

---

## File Creation

ADRs are organised into subfolders under `.project/adr/` by status. Save each new ADR to the folder matching its status at creation:

| Status | Folder | When to use |
|--------|--------|-------------|
| `Accepted` | `.project/adr/accepted/` | Decision confirmed in source documents or by a stakeholder |
| `Proposed` | `.project/adr/proposed/` | Decision identified but not yet made — options are being evaluated |
| `Conflict` | `.project/adr/conflict/` | Two or more source documents contradict each other on this point |
| `Open` | `.project/adr/gap/` | Information or a decision is missing and must be obtained before proceeding |
| `Deprecated` | `.project/adr/deprecated/` | Decision was accepted but is no longer relevant |
| `Superseded` | `.project/adr/superseded/` | Decision replaced by a later ADR |
| `Rejected` | `.project/adr/proposed/` | Decision was considered and explicitly not taken — keep for rationale |

- **Filename:** `ADR-NNNN-short-hyphenated-title.md`
- **Template:** `.project/adr/template.md`

After saving, confirm the filename, status, and folder to the user and note any fields left as `[To be confirmed]`.

---

## Updating an Existing ADR

To change status or add information to an existing ADR:
1. Read the existing file.
2. Update the `Status` field.
3. Move the file to the folder matching the new status.
4. If superseded, add `Superseded by ADR-NNNN` and link the new record.
5. Append a `## Change Log` section at the bottom if one does not exist:

```
## Change Log
- YYYY-MM-DD: [What changed and why]
```

---

## ADR Lifecycle

```
Open ──────────────────────┐
Conflict ──────────────────┤
Proposed ──────────────────┴──► Accepted ──► Deprecated
                                           ──► Superseded by ADR-NNNN
                            └──► Rejected
```

When a status changes, **move the file to the matching folder** as well as updating the `Status` field.

- `Open` / `Conflict` → `Accepted`: move from `gap/` or `conflict/` to `accepted/`
- `Proposed` → `Accepted`: move from `proposed/` to `accepted/`
- `Accepted` → `Deprecated`: move to `deprecated/`
- `Accepted` → `Superseded`: move to `superseded/`, add `Superseded by ADR-NNNN` link
- Any → `Rejected`: move to `proposed/` (kept for rationale)
