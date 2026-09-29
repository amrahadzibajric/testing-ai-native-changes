---
name: requirements-reviewer
description: Analyze a requirements document, RFP, or project brief and generate insightful, high-level client questions for a first meeting or RFP response phase. Use when the user provides a requirements document and wants to prepare discovery questions that demonstrate expertise while closing real gaps between scope and documentation. Triggers on requests like "review this requirements doc", "prepare client questions", "what should I ask the client", or "analyze this brief for gaps".
license: "AP AI-Native Delivery Toolkit v3.5 - AP Confidential - governed by LICENSE.md"
metadata:
  user-invocable: true
---

# Requirements Reviewer

Generate insightful, strategically chosen client questions from a requirements document. The goal is to close actual gaps between scope and the RFP/requirements docs — while demonstrating competence and domain understanding.

## When to Use

Use this skill when:
- the user provides a requirements document, RFP, or project brief and wants to prepare for a discovery meeting
- generating client questions to surface scope gaps, ambiguities, or missing constraints before a proposal
- the user runs `/requirements-reviewer` on a source document
- gap analysis and safe-assumption identification are needed before project intake

## Mindset

This skill operates in the pre-sales or early discovery phase. The stakeholder's trust is at stake. Two failure modes exist:

1. **Too many detailed questions** — You get the information but lose credibility. The client thinks "they don't get it" or "this will be exhausting to work with." You lose the RFP.
2. **Too few questions / too many assumptions** — You look confident but miss key requirements. You over-promise or under-estimate. The project fails later.

The sweet spot: dig deep into the document, make safe assumptions where possible, skip questions that wouldn't materially change the estimate, and surface only the questions that close real gaps. Every question should signal that you have read the document thoroughly and understand the domain.

## Workflow

1. **Receive the document** - The user provides a requirements document (file path, pasted text, or attached document). Read the full document carefully.

2. **Deep analysis** - Go through the document thoroughly and identify:
   - What is clearly defined — these become safe assumptions (no questions needed)
   - What can be reasonably assumed without risk — state the assumption, skip the question
   - What is vague but wouldn't significantly change the estimate either way — skip it
   - What is genuinely ambiguous and would materially affect scope, timeline, or cost — these become questions
   - Contradictions or inconsistencies that could derail the project

3. **Filter ruthlessly** - For each potential question, apply the gate: "If we get this wrong, does it significantly change the estimate, the architecture direction, or the project outcome?" If no, drop it.

4. **Generate questions** - Produce questions following the rules below.

5. **Present output** - Format using the output structure below.

## Question Generation Rules

- **Only ask what truly matters.** Every question must close a gap that could materially affect scope, estimate, or project direction. If being wrong about something only costs a few hours of rework, it's not worth asking.
- **Make safe assumptions instead of asking.** When the document implies something or when industry convention makes one answer far more likely, state the assumption rather than asking a question. Let the client correct you if needed — this shows confidence and experience.
- **Non-technical language only.** Questions must be understandable by a non-technical stakeholder. No jargon, no implementation details, no architecture terms.
- **Show you've done your homework.** Frame questions in a way that demonstrates you've read and understood the document. Reference specific parts of the document when relevant. Never ask something the document already answers.
- **First-meeting caliber.** These are the questions a senior consultant asks — strategic, business-oriented, showing pattern recognition from similar projects. Not a checklist of unknowns.
- **Aim for insight, not coverage.** A few sharp questions beat many generic ones. The client should think "good question, I hadn't considered that" — not "didn't they read the document?"

## Question Categories

Group questions into these categories (skip any category with no questions):

1. **Project Vision & Goals** - Objectives, success criteria, business drivers
2. **Users & Stakeholders** - Who uses it, who decides, who is affected
3. **Scope & Boundaries** - What is in/out, MVP vs. future, deal-breakers
4. **Existing Systems & Context** - Current landscape, integrations, migration (brownfield)
5. **Timelines & Priorities** - Deadlines, phasing, dependencies
6. **Content & Data** - Who provides what, data ownership, migration
7. **Processes & Workflows** - Business processes, approval chains, edge cases
8. **Constraints & Risks** - Budget, compliance, regulatory, known blockers

## Output Format

```
## Document Analysis

### Well-Defined Areas
- [Bullet list of topics clearly covered — no questions needed]

### Safe Assumptions
- [Assumption based on document + reasoning] — assumed because [brief rationale]
- [Assumption] — assumed because [rationale]
...

### Areas Intentionally Skipped
- [Topic] — ambiguous but low impact on estimate/scope either way

## Questions for the Client

### [Category Name]

**Q: [Question text]**
_Context: [1 sentence showing you understood the document and why this gap matters]_

**Q: [Question text]**
_Context: [1 sentence]_

...
```

## Guidelines

- Aim for **8–15 questions** total. Fewer is a sign of a thorough document, not a failure. More than 15 means you're probably going too granular.
- The safe assumptions section is as important as the questions — it demonstrates expertise and thoroughness without burdening the client.
- Order questions within each category from highest to lowest impact on estimate/scope.
- If the document is very complete, say so clearly. Having only 3–5 questions for a well-written RFP is a great outcome.
- If the project type (greenfield vs. brownfield) is unclear, include a question about it — this always materially affects scope.
- Tone: professional, confident, consultative. You are a trusted advisor, not an interviewer with a checklist.
