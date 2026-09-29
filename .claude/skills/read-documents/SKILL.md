---
name: read-documents
description: Inventory, read, and synthesize source documents from /.project/input/ before producing any output. Defines extraction rules per document type, cross-referencing logic for conflicts and gaps, and the citation format all agents must use.
license: "AP AI-Native Delivery Toolkit v3.5 - AP Confidential - governed by LICENSE.md"
metadata:
  user-invocable: false
---

# Skill: Read Documents

## Purpose

Locate, read, and synthesize source documents from the `.project/input/` folder before producing any output. This skill defines the folder structure, per-type extraction rules, and citation format used across all agents in this workspace.

---

## Folder Structure

```
.project/input/   — all source materials (transcripts, RFPs, emails, specs, notes, etc.)
```

Drop any source document directly into `.project/input/`. No subfolders are required. Type is inferred from content at read time, not from file location or filename.

---

## How to Use This Skill

### Step 1 — Inventory available documents

Before starting any synthesis task, list the contents of `.project/input/` and identify which documents are relevant to the current task. Do not assume documents exist — check first.

Ignore `.project/input/README.md`. That file describes the folder and is not client source material.

If `.project/input/` contains no client source files after excluding `README.md`, report that no source materials are available and stop. Do not synthesize from the README.

```
Scan: /.project/input/
```

### Step 2 — Read relevant documents

Read all documents that could contain information relevant to the current task. When in doubt, read it — it is better to over-read than to miss a relevant detail.

### Step 3 — Detect document type and extract

For each document, infer its type from content cues and apply the matching extraction rules.

**Type detection cues:**

| Detected type | Cues |
|---------------|------|
| **Meeting** | Transcript markers, dialogue format, attendee lists, "meeting notes", "minutes", action items attributed to named individuals |
| **Formal document** | RFP / spec / contract / brief / report structure, numbered sections, formal headings, scope or requirements sections |
| **Email** | From/To/Subject headers, reply chains, email signature blocks, forwarded message markers |
| **Other** | Treat as a generic source document and extract all relevant facts |

The filename may serve as a hint (e.g. a date-prefixed name suggests a meeting) but is not authoritative. Content takes precedence.

#### Meeting documents
Extract:
- **Decisions made** — explicit agreements or choices reached during the meeting
- **Action items** — tasks assigned to named individuals, with due dates if mentioned
- **Open questions** — unresolved issues or topics deferred for follow-up
- **Key context** — background, constraints, or requirements surfaced in discussion
- **Participants** — who was present (relevant for ADR source attribution)

#### Formal documents
Extract:
- **Requirements** — stated functional and non-functional requirements
- **Scope statements** — what is included and excluded
- **Constraints** — technology, budget, timeline, regulatory
- **Evaluation criteria** — if the document is an RFP or brief
- **Assumptions** — stated by the document author
- **Open questions directed at the vendor/team** — questions the document poses that need a response

#### Emails
Extract:
- **Directives and decisions** — instructions or choices communicated in writing
- **Commitments** — things a party has agreed to do
- **Clarifications** — answers to previously open questions
- **New requirements or scope changes** — anything that adds to or modifies formal documents
- **Tone and priority signals** — urgency, concerns, or preferences expressed by stakeholders

#### Other / generic
Extract all facts, decisions, constraints, and open questions relevant to the current task.

### Step 4 — Cross-reference

After reading all relevant documents, check for:
- **Conflicts** — where documents contradict each other; flag explicitly, do not silently resolve
- **Gaps** — topics that are mentioned but not elaborated; flag as open questions
- **Supersessions** — where a later email or meeting overrides a formal document; note which takes precedence and confirm with the user if unclear

---

## Citation Format

All extracted information used in output must be cited. Use this format inline:

> *Source: [filename], [section or date if applicable]*

Examples:
- *Source: rfp-platform-modernisation-v1.md, Section 3.2*
- *Source: kickoff-notes-2026-02-24.md*
- *Source: jsmith-integration-requirements.md*

When a point is corroborated by multiple documents, cite all of them:
> *Sources: rfp-platform-modernisation-v1.md, kickoff-notes-2026-02-24.md*

---

## When Documents Conflict with User Input

If the user states something verbally that conflicts with a document, surface the conflict:
> "The kickoff notes say X, but you've mentioned Y — which should we treat as current?"

Do not silently prefer either source.

---

## Flagging Missing Documents

If the user refers to a document that does not exist in `.project/input/`, flag it:
> "I don't see [document name] in `.project/input/`. Can you upload it, or should I proceed based on what's available?"

---

## No Source Material Handling

If `.project/input/` contains only `README.md` or is empty, state explicitly that the repository has no client source materials available for synthesis yet.

In that case:
- Do not treat the README as evidence
- Do not produce sourced conclusions
- Tell the user which upstream step is missing, such as document upload or intake
