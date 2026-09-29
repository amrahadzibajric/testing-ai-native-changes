---
name: generate-ui-designs
description: "Build this project's design system, then its UI screens, in a claude.ai/design project. Phase 1 agrees a sector-specific design direction, then lands tokens and component vocabulary, pausing for review at each. Phase 2 composes screens from that vocabulary, verified group by group. Uses claude-design MCP, or a DesignSync or canvas fallback."
license: "AP AI-Native Delivery Toolkit v3.5 - AP Confidential - governed by LICENSE.md"
metadata:
  user-invocable: true
---

# Skill: Generate UI Designs (Claude Design)

## Purpose

Build one design project per elaboration project in two ordered phases: brand and component
vocabulary first (Phase 1), reviewable before anything else runs, then screens composed from
that vocabulary (Phase 2). A screen may only use a colour, type size, radius or control the
system already defines; a genuinely needed new value goes into the system first.

Phase 6 — after data model (Phase 5), before WBS (Phase 7). Briefs and the published target are
recorded in `./.project/ui-designs/`.

## When to Use

Use this skill when:
- the user asks to generate UI designs or screens for the project
- creating a new claude.ai/design project or updating existing screens
- UI wireframes or mockups are needed for web or mobile surfaces
- the user runs `/generate-ui-designs` to trigger the design workflow

## Reference modules

| Module | Read it before |
|---|---|
| [claude-design-mcp.md](references/claude-design-mcp.md) | the first `claude-design` call — tool contracts, call ordering, `.design-canvas.json` shapes |
| [fallback-surfaces.md](references/fallback-surfaces.md) | the first `DesignSync` or canvas call |
| [screen-brief-format.md](references/screen-brief-format.md) | Phase 1, and authoring any brief — **design direction** (sector archetypes, colour discipline, craft pass), Style Contract, brief template, mobile rules, group vocabulary, screen grouping |

## Step 1 — the availability gate

Claude Design depends on a first-party claude.ai login and an org policy permitting access.
Before anything else runs, probe three surfaces in order by calling `list_projects` on each until
one answers. Probe every run rather than assuming a prior run's result:

1. **`claude-design` MCP server** (primary) — the only surface exposing
   `get_claude_design_prompt` and `read_design_skill`, which load Claude Design's own system
   prompt and `hifi-design` skill. Skipping those two calls forfeits that guidance.
2. **`DesignSync`** — the same surface without the prompt and skill tools.
3. **A design canvas** — a multi-artboard `.dc.html` Artifact via the `design` skill.

Within whichever answers: writable projects exist → offer them plus "create new"; none →
`create_project`. If all three are unavailable, stop and say what is missing (no login, no org
policy, no `node`/`bun` on PATH); hand-authored mockups and partial runs are out of scope. Name
the surface used in the Phase 1 pause and in the final README. **The gate leaves no trace:** if
it stops, `.project/ui-designs/` stays exactly as found.

## The project shape

**One project per elaboration project** — one client's design stays out of every other client's
project, and out of any Authority Partners internal design project. One project normally holds
both the system and the screens; only Branch A in
[claude-design-mcp.md](references/claude-design-mcp.md) splits them across two servers. Files are
grouped as Brand, Status, Controls, Navigation, Surfaces, and `Screens <NN> <Group name>` — see
[screen-brief-format.md](references/screen-brief-format.md).

`design-direction.md` holds the agreed direction — written and approved before the system is
built, read back on every later run. `.design-canvas.json` records which surface built the
target and how far the push has been
**verified** to reach: `phase` `"1-system"` or `"2-screens"`, plus per-group status in Phase 2 so
a resumed run starts at the first group not `"verified"`. Read it first every run and reuse the
recorded target; one elaboration project gets one project or canvas for its lifetime.

## Inputs

Architecture (`.project/architecture/arch-*`), data model (`.project/data-model/`), ADRs
(`.project/adr/`), project context (`.project/context/project-context.md`), product definition
(`.project/product-definition/`), and the target record (`.project/ui-designs/.design-canvas.json`,
if updating).

Read branding sources before asking anything: a token table or style guide in `.project/input/`,
prior architecture or ADR brand mentions, or the client's public site.

**Required intake, for whatever that leaves open:** sector and product archetype (plus 1-3
reference *products*, not templates); palette with semantic roles (primary/accent, secondary,
surface, background, text, status); typography (headline/body, or "match the reference site");
light/dark intent; brand tone; nav convention (web: navbar/sidebar/both; mobile:
tabs/hamburger). **Optional:** logo, radius/density, locale, accessibility requirements.

Every Style Contract axis in [screen-brief-format.md](references/screen-brief-format.md) needs a
value before Phase 1 is done.

## Phase 1 — the design system

On `claude-design`, call `list_design_systems` first: an existing match → offer it plus "create
new"; none → resolve per "The design-system-creation question" in
[claude-design-mcp.md](references/claude-design-mcp.md). Always let the user pick.

### Phase 1a — the design direction, approved before anything is built

Write the direction statement to `./.project/ui-designs/design-direction.md`: sector archetype
and register, the one structural idea that keeps this product off a default card grid, type and
colour rationale, and at least two defaults deliberately refused. A model left to its defaults
reproduces the same hero-plus-card-grid, single-blue-accent interface for every sector; this
statement is what prevents that. Hold the palette to 60/30/10 with one accent reserved for the
primary action and live state. Full guidance, including the sector archetypes: "Design
direction" in [screen-brief-format.md](references/screen-brief-format.md).

**Pause here** and get the user's agreement on the direction before authoring a single token —
a wrong direction caught now costs one round-trip; caught after the system is live it costs the
whole system. On a resumed run, read the existing `design-direction.md` back instead of
re-deriving it, and pause again only if it is missing or the user asks to revisit it.

### Phase 1b — the system itself

Once the direction is agreed, land tokens (palette, type ramp, spacing, radii, elevation,
motion), then controls, inputs, tables, chips, navigation, surfaces, and empty/loading/error
states — one `@dsCard` file per component family, each one an expression of the direction
rather than a generic default.

1. **Author** the files on disk under `./.project/ui-designs/`.
2. **Push** — distinct from authoring: `finalize_plan`, then `write_files`.
3. **Verify** — `list_files`; confirm every step-2 file is present remotely.
4. **Persist** — once step 3 passes: write `.design-canvas.json` with the active `surface`,
   `"phase": "1-system"`, and the pushed file count.

A step 2/3 failure **is** the run's outcome: name what is on disk versus confirmed live, stop, and
leave the unverified phase unpersisted.

**Pause again**, once step 4 succeeds — tell the user the system is live, where, and which
surface built it; wait for their look before Phase 2.

## Phase 2 — the screens

Read `./.project/ui-designs/design-direction.md` back first — every screen brief cites it, and
it is what a drifting screen gets measured against. Then build and confirm the screen inventory
per "Screen inventory grouping" in [screen-brief-format.md](references/screen-brief-format.md).

On `claude-design`: `create_project` with `design_system_id` bound (or reuse the recorded
project), `get_project` to verify, then — **before any `write_files`**, mandatory —
`get_claude_design_prompt` + `read_design_skill("hifi-design")`, then `finalize_plan` with
`scope: "project"` once per run.

**Interleave per group.** Read `.design-canvas.json` and skip groups already `"verified"`. For
every remaining group, in order:

1. **Author** that group's `@dsCard` files — one per screen per device.
2. **Push** — `write_files`, one call for this group, carrying `if_match` etags.
3. **Verify** — `list_files`; confirm every step-1 path is present remotely.
4. **Self-check (`claude-design` only)** — `render_preview` + screenshot each page, then judge
   it: does it carry the direction statement's register and structural idea, hold 60/30/10, and
   use only Phase 1 vocabulary? A screen that has drifted back to a default card grid gets
   re-authored, not shipped. Without a screenshot capability, record the group as unverified.
5. **Persist the group** in `.design-canvas.json`, once steps 3 and 4 pass, before the next
   group's authoring.

Authoring the whole inventory before the first push is the failure mode this ordering exists to
prevent — see rule 1 in [claude-design-mcp.md](references/claude-design-mcp.md). A step 2/3/4
failure is that group's outcome: record it unverified, name what is confirmed live versus checked,
and move to the next group. Partial verified groups are the point.

Once every group is verified, push the **interactive prototype** (one file stitching the confirmed
flow behind a screen router) as its own final step, so the biggest inline payload cannot starve a
screen group.

**Finish**, once every group and the prototype verify: set `phase` to `"2-screens"`, then write
`./.project/ui-designs/README.md` — surface used and why, the link (`open_url` or `write_files`'
`url`, never `serve_url`), the inventory by group with verified status, a branding summary, and
(canvas only) the private-by-default sharing note. If README.md cannot be written, say so.

## Examples

**First run.** The gate answers on `claude-design` → intake fills the two axes the style guide in
`.project/input/` left open → `design-direction.md` commits to a table-first treasury console
with one signal accent and no stat-card row → **pause for approval** → Phase 1 pushes 7
`@dsCard` files expressing it, `list_files` confirms all 7,
`.design-canvas.json` written → **pause for review**. After sign-off, Phase 2 confirms a 5-group
inventory, then pushes and verifies `01-public-and-onboarding` before authoring
`02-core-experience`, and so on, prototype last.

**Resumed run.** `design-direction.md` is read back rather than re-derived, and
`.design-canvas.json` shows `01-*` `"verified"`, `02-*` `"pushed"`, `03-*` `"pending"`: reuse the
recorded project, re-verify `02-*`, continue at `03-*`.

## Troubleshooting

| Symptom | What to do |
|---|---|
| `claude-design` auth failure (e.g. HTTP 403) | The login carries no Claude Design access. Run `/design-login`, then re-probe; falling through to a later surface is valid if named. |
| `write_files` rejects `local_path` | On `claude-design` every write is inline `data`; `localPath` belongs to `DesignSync` only. |
| `finalize_plan` fails with "requires: deletes" | `DesignSync` needs both keys — pass `deletes: []`. |
| `write_files` returns cleanly but files are absent | A clean return is not confirmation. `list_files` is the gate; missing paths mean the group is unverified. |
| The run stops mid-inventory with files on disk | Expected under per-group ordering. Verified groups stay live; re-run to resume at the first group not `"verified"`. |
| Screens drift back to a generic card grid | The direction was not carried into the brief. Re-read `design-direction.md` and re-author the group against it rather than shipping the drift. |

## Validation Checklist — a completion gate

- [ ] Phase 1 completed and paused for review before any Phase 2 file was written
- [ ] The gate probed all three surfaces in order and the run says which one it used
- [ ] `.design-canvas.json` names a valid target and states which surface built it
- [ ] Branding came from a source or the user; every Style Contract axis has a value
- [ ] `design-direction.md` exists, was agreed with the user **before** any token was authored,
      and names the structural idea plus the refused defaults
- [ ] The output passes the direction self-check: sector-legible without the logo, three
      decisions a template would not have made, palette within 60/30/10
- [ ] Every module maps to ≥1 screen; entities have real fields; desktop and mobile artboards plus
      empty/loading/error states exist for each
- [ ] Every screen file declares a `box-shadow`/`border-radius` and cites system tokens
- [ ] Phase 2 was authored, pushed and verified one group at a time, each recorded individually,
      prototype last; `phase` reads `"2-screens"` only once all of them verify
- [ ] On an update run, the recorded project or canvas was reused
- [ ] Every push was confirmed with `list_files`; `README.md` names the surface and links
      `open_url` or `write_files`' `url`; a missing README or unrecorded target is reported
