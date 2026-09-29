---
name: create-presentation
description: Create on-brand PowerPoint (.pptx) decks and presentations for Authority Partners. Use this skill whenever the user asks for a PowerPoint deck, a .pptx file, an editable presentation, a PPT deck, a slide deck someone will edit in PowerPoint or Keynote or Google Slides, or any kind of editable slide-based content for Authority Partners or its consultants and teams, even if they do not explicitly say "Authority Partners". For HTML output, use the sibling skill `authority-partners-slides` instead.
license: "AP AI-Native Delivery Toolkit v3.5 - AP Confidential - governed by LICENSE.md"
metadata:
  user-invocable: true
---

# Authority Partners PPTX Slide Skill

> **Provenance note:** This skill was copied from the organization's shared `authority-partners-slides`-family skill with slight modifications for this toolkit. It may fall out of sync with the source over time — if something here looks outdated or inconsistent with current AP brand guidance, check for a newer version upstream before assuming this file is authoritative.

Create branded PowerPoint presentations for Authority Partners. Decks are content-driven (the user describes what each slide should communicate, and the skill produces a slide that reflects it) and visually consistent (every deck uses the same brand system so anything an Authority Partners person presents reads as Authority Partners). Output is an editable .pptx file optimized for internal collaborative work before the deck goes external.

## Reference files

The code that makes up a generated script is kept out of this file and loaded on demand:

| File | Contents | Read it when |
|------|----------|--------------|
| [`references/code-library.md`](references/code-library.md) | Everything you copy into a script, in assembly order: Part 1 brand dimensions/constants/logo/helpers (always needed), Part 2 all 11 slide pattern functions (cover, agenda, section divider, standard content, numbered grid, comparison, pillars, flow, KPI, quote, closing, timeline), Part 3 the library-based automatic appendix system (chrome-stripping, per-slide tweaks, `create_ap_presentation()`, `add_appendix_divider()` + `reorder_deck()`, opting out) | Step 3 of the workflow — read Part 1 always, Part 2 for the patterns the outline needs, Part 3 unless the user opts out of the appendix |
| [`references/examples.md`](references/examples.md) | A complete worked script for an 8-slide deck | When you want to see the pieces assembled end-to-end |

## Workflow

Follow this sequence every time. Do not skip the length question.

1. **Ask how long the deck should be.** Before drafting anything, ask the user how many slides they want. A single short question is enough, for example: "How many slides should the deck be?" If the user gives a range, pick the middle and confirm. If the user says "you decide", suggest 8 to 12 for typical decks. If the user comes with a long content outline (for example a 50-slide brief), match the outline length.

2. **Discuss content.** Once length is set, work with the user on what each slide should say. If they have already described the deck in detail, propose a slide-by-slide outline and confirm before generating. If they have given only a topic, ask one or two clarifying questions about audience and goal, then propose an outline.

3. **Generate the .pptx file.** Write a Python script that uses python-pptx. Unless the user has explicitly opted out of the appendix, start the presentation with `create_ap_presentation()` (from `references/code-library.md` Part 3) instead of a bare `Presentation()` — it returns a presentation pre-loaded with the 12 rebranded library slides, so it must be the very first call, before any main-deck slides are added. Build the deck using the brand constants and helper functions from `references/code-library.md` (Part 1) and the slide pattern functions from Part 2 that fit the outline (see "Quick reference: which slide pattern when" below). After the closing slide, call `add_appendix_divider(prs)` then `reorder_deck(prs)` to insert the appendix divider and reorder the deck so the main slides come first, then the divider, then the 12 library slides — together these complete the standard Authority Partners 13-slide appendix. Save the output to `/mnt/user-data/outputs/` with a descriptive filename like `ap-deck-<topic>-<YYYYMMDD>.pptx`.

4. **Visually QA the output.** Convert the .pptx to PDF using LibreOffice, then to images, then check that the first few slides render correctly (no overflowing text, no logo missing, no obvious layout breaks). See the Visual QA section below for the commands. Fix any defects in one cycle and stop.

5. **Present the file.** Use `present_files` to share the .pptx with the user. Keep the chat reply short: confirm what was built, mention the slide count, let the file speak for itself.

6. **Iterate.** If the user wants changes, edit the script and re-generate. Avoid rebuilding from scratch.

## Reskinning an existing deck

Sometimes the input is not a content brief but an existing deck (the user hands over a source .pptx, a set of slides or an exported outline) and asks for it in Authority Partners style. When that happens, the job is a **reskin, not a rewrite**. These rules are non-negotiable and take priority over the skill's normal generative freedom. Follow every one of them.

1. **Preserve the content and slide order. This is the most important rule.** Every slide in the source maps to a slide in the output, carrying the same message in the same sequence. Do not merge slides, drop slides, reorder slides, add slides or invent new content. One source slide in, one Authority Partners slide out, in the same position. If the source has 23 slides, the output has 23 slides in the same order. The appendix is the only addition, and it goes after the source content (and is still omitted if the user opts out).

2. **Fix the text to house style as you go, with light copyedits only.** Apply the canonical rules in the Voice and writing rules section below: no Oxford comma, American spelling, "Authority Partners" written in full (never "AP", the only exception being "AP Lab"), Title Case for slide titles, ALL CAPS for eyebrows and short labels. Em-dashes are avoided and used only when truly necessary, and never for ranges (a range always takes a short hyphen, for example 1-10). These are light copyedits to bring the text into house style. Do not change the substance or meaning of any slide.

3. **Re-create the visual assets in Authority Partners style.** Rebuild charts, tables and diagrams natively using the brand visual system (colors, type, layout from `references/code-library.md` Part 1) rather than pasting the originals. A chart from the source becomes a chart drawn with Authority Partners colors and type, not a screenshot of the old chart. For screenshots or photos that genuinely cannot be restyled, carry them over but frame them on-brand (place the image inside Authority Partners chrome rather than leaving the source framing).

4. **Map each source slide to the most fitting Authority Partners slide pattern.** Use the patterns in `references/code-library.md` (Part 2). See "Quick reference: which slide pattern when" for the mapping. Where a source slide does not fit any fixed pattern, use freeform composition with the brand constants and helper functions directly, staying inside the design system (lime as accent only, no gradients, mixed-case titles, ALL CAPS eyebrows).

5. **Flag judgment calls before you finalize.** After building the deck and before presenting it, list any slides where the content did not map cleanly to a pattern, where you made a layout judgment call or where you re-created a chart, table or diagram and had to interpret the original. Keep this list short and specific (slide number plus one line on what you decided) so the user can check exactly those slides.

## Copy and casing

Every Authority Partners deck uses the same two-tier text pattern:

- **Eyebrow**: ALL CAPS, short category label (1-3 words). Examples: `OUR FIRM`, `WHAT WE DO`, `AGILE DELIVERY`, `CAPABILITIES OVERVIEW`. The eyebrow names the section.
- **Title**: Title Case headline that makes the slide's claim. The title should not repeat the eyebrow verbatim; it should add information. Examples: `Driving Success for 28+ Years`, `100+ Client-Partners Over 28 Years`, `Better Outcomes & Lower Cost of Engagement`.

**Title Case rules** (applied throughout: cover, agenda, section dividers, content, comparison, pillars, flow, KPIs, quote, closing, appendix titles, agenda item titles):

- Capitalize: nouns, verbs (including is, am, be, are), adjectives, adverbs, pronouns
- Lowercase: articles (a, an, the), coordinating conjunctions (and, or, but, nor, for, yet, so), short prepositions (of, in, on, to, for, with, from, by, at, as, into, onto, over, that)
- Always capitalize: the first word, the last word, and the first word after a colon
- Hyphenated compounds: capitalize both parts (Client-Partners, Product-Centric, Outcome-Driven)
- Acronyms stay uppercase (KPIs, AI, SDLC, NPS)
- Ampersands (&) are allowed and encouraged in titles where they shorten the copy without losing clarity. Treat them like "and": lowercase concept, visible as the symbol.
- The word "that" stays lowercase even though it's a subordinating conjunction. Example: `KPIs that Prove the Work`.

**Examples of correct casing**

| Eyebrow | Title |
|---|---|
| OUR FIRM | Driving Success for 28+ Years |
| WHAT WE DO | Software Engineering & Technology Strategy |
| AGILE DELIVERY | Product-Centric & Outcome-Driven |
| WHY PAYERS CHOOSE US | Built for Regulated, Member-Facing Software |
| CAPABILITIES OVERVIEW | A Software Engineering Partner for Health Plans |

**Tagline and body copy** (the lede under a title, bullet text, paragraph text, KPI labels, meta-row values) use **sentence case** — only the first word and proper nouns are capitalized. The Title Case rule applies to titles, eyebrows are always ALL CAPS, and everything else is sentence case.

## Visual QA

After generating, convert the .pptx to images and inspect the first slide and any slide with dense content (KPI grids, comparison columns, pillar cards). The goal is to catch user-visible defects like overflow, missing logos or misalignment, not to chase pixel-perfect positioning.

```bash
cd /mnt/user-data/outputs
soffice --headless --convert-to pdf ap-deck-*.pptx
pdftoppm -jpeg -r 100 ap-deck-*.pdf slide
```

Then view `slide-1.jpg`, `slide-N.jpg` for any dense slides. Fix obvious defects in one pass and stop. Do not iterate on minor cosmetic issues.

## Voice and writing rules

Same as the HTML skill. These rules govern every piece of generated text including slide titles, bullets, meta labels, the closing call to action and any speaker notes.

- **Always write "Authority Partners" in full** on first reference and throughout. Never "Authority", never "AP" (the only exception is "AP Lab" as a proper noun).
- **American spelling** throughout: organize, color, optimize, behavior, center, analyze.
- **No oxford comma anywhere**, including in lists of clauses.
- **Avoid em-dashes (—)**. Reach first for commas, parentheses, semicolons, colons or a sentence break, which are almost always better. An em-dash is acceptable only when it is absolutely needed and nothing else reads as cleanly. **Never use an em-dash for a range**: ranges always take a short hyphen (for example 1-10, pages 3-7, 8-12 slides, May-June 2026). For deck filenames and titles, use a colon: "Authority Partners: Capabilities Overview" rather than "Authority Partners — Capabilities Overview".
- **Preferred terms**: consultant (not contractor or employee), engagements (not projects), people (not resources), Talent Management (not HR).
- **Tone**: professional, confident, people-centric. Clear, concise, authoritative but approachable. Avoid jargon. Spell out acronyms on first use.
- **Avoid**: heavy adjective stacks, vague verbs ("leverage", "synergize", "unlock" without an object), filler ("at the end of the day", "in today's world"), passive voice when active is cleaner.
- **Slide titles** are mixed case. **Eyebrows** are ALL CAPS. Do not invert this.

## Output rules

- **This document type is internally called an Approach Document** at Authority Partners. When referring to it in conversation or file descriptions, use "Approach Document" — the filename uses the standard versioned convention below.
- **One file per deck**, saved as `/mnt/user-data/outputs/ap-deck-<short-topic>-<YYYYMMDD>.pptx`. When produced as part of the elaboration workflow (Phase 11), save to `.project/presentations/` instead.
- **Use `present_files`** to share the output. Keep the chat reply short.
- **Do not embed fonts** unless the user specifically asks. Segoe UI is the brand primary and is bundled with Office on Windows. On macOS or Linux machines that lack it, PowerPoint substitutes (typically to Calibri or Open Sans). This is acceptable for the internal collaboration use case the skill is optimized for.
- **Speaker notes** are encouraged for client-facing decks. If the user provides them or asks for them, add to `slide.notes_slide.notes_text_frame.text`.

## Quick reference: which slide pattern when

- Opening the deck → Cover (1)
- Right after the cover, agenda for any deck of 5+ slides → Agenda (2)
- Major section break in a longer deck → Section divider (3)
- Most slides in most decks → Standard content (4)
- 4-6 short items that would leave empty space as plain bullets → Numbered card grid (4a)
- "X vs Y" or "before vs after" → Two-column comparison (5)
- 3 or 4 equal items → Pillar cards (6, light or dark)
- Sequential steps → Process flow (7)
- Numerical results → KPI metrics (8)
- Client voice → Quote (9, sparingly)
- Closing the deck → Closing (10)
- Delivery programme schedule or Gantt → Programme timeline (11)

When in doubt, use Standard content (4). It is the workhorse and looks correct in any deck. For unusual content the patterns do not cover, write a custom slide using the brand constants and helper functions directly (`references/code-library.md` Part 1). Stay within the design system: lime as accent only, no gradients, mixed-case titles, ALL CAPS eyebrows.
