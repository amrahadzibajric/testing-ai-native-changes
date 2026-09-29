# Per-screen prompt format, style contract, and mobile adaptation

Applies to every screen brief, on either surface (a `@dsCard group="…"` file for
`DesignSync`, or a `.dc.html` artboard for the canvas fallback).

## Design direction — earn the aesthetic

Author this **before any token lands**, and treat it as the Phase 1 gate the Style Contract
below then implements.

### The failure this prevents

Left to its defaults, a model reproduces the statistical mean of every interface it was trained
on: hero section, stat-card grid, sidebar with icon labels, one blue accent, a grotesk at
default weight, a gradient somewhere. It is competent and it is anonymous — and it applies the
same "modern clean UI" to a trading terminal, a clinical workflow and a children's reading app.
A card grid is the default, never a decision. Use one only where the content is genuinely a set
of peer objects, and say why in the brief.

### The direction statement

Three to five sentences, written to `./.project/ui-designs/design-direction.md`, **agreed with
the user before a single token is authored**, and cited by every screen brief thereafter. A
resumed run reads that file back rather than re-deriving it.

1. **Sector archetype and register** — what this product is, and the feeling it should leave
   (calm, precise, celebratory, authoritative, effortless).
2. **The structural idea** — the one layout decision that makes this product not a card grid:
   table-first with a detail pane, editorial asymmetry, canvas/timeline, step-wise task flow,
   full-bleed imagery, split master-detail.
3. **Type rationale** — why this display face and this UI face, for this sector.
4. **Colour rationale** — the dominant surface, the secondary, and the single accent, plus what
   the accent is reserved for.
5. **What is deliberately absent** — name at least two defaults being refused (e.g. "no stat-card
   row", "no gradient hero", "no decorative illustration").

### Sector archetypes

Starting points, not a mandate — the client's own branding always outranks the row.

| Sector | Density | Type | Colour | Structure | Avoid |
|---|---|---|---|---|---|
| Fintech, trading, treasury | High, data-first | Structured grotesk, tabular figures | Near-monochrome, one signal accent | Table-first, dense panels, persistent context | Oversized stat cards, playful gradients |
| Healthcare, clinical | Medium, calm | Humanist sans, generous line-height | Soft neutrals; status colour strictly semantic | Task and step flows, clear zoning | Alarm-red as decoration, dark-first themes |
| Luxury, fashion, hospitality | Low, editorial | Display serif paired with a quiet UI sans | Deep neutrals, one restrained accent, imagery carries colour | Editorial asymmetry, full-bleed imagery, generous margin | Dashboard stat rows, busy chrome |
| Enterprise, operations, logistics | High | Compact UI sans — size for importance, weight for type, colour for state | Neutral base, colour reserved for state | Table plus detail pane, persistent nav, bulk actions | Marketing gradients, decorative illustration |
| Consumer, learning, wellness | Medium-low, warm | Rounded or geometric sans | One brighter accent, still one | Progressive disclosure, illustration with a job | Enterprise density, grey-on-grey |
| Public sector, civic | Medium | Highly legible, large targets | High contrast, minimal palette | Linear forms, plain language, obvious next step | Dense dashboards, low-contrast subtlety |

### Colour discipline

**60 / 30 / 10 allocates roles, not brand slots.** The rule comes from interior design and is
routinely misread as "the brand colour covers 60% of the screen". It does not. The bands are:

| Band | What it is | Typically |
|---|---|---|
| ~60% | The dominant **surface** — canvas, cards, panels | A neutral: white/off-white in light mode, near-black in dark mode |
| ~30% | Supporting structure — secondary surfaces, borders, nav chrome, body and muted text | Greys, plus the brand colour where it carries structure |
| ~10% | The **accent** — primary action, active and selected state, the one number that matters | Usually the brand colour |

A dark or saturated brand primary therefore lands in the 10% band, and often the 30%. Filling
60% of the screen with it produces the airless, oversaturated result the rule exists to prevent.

- **Branded neutral.** To make the brand present across the whole surface without darkening it,
  tint the neutral with the brand hue at very low saturation — same hue, roughly 4-8% saturation
  at 96-98% lightness (or its dark-mode inverse). The canvas reads as the brand's world while
  still behaving as a neutral.
- **A dark 60% is a theme decision, not a brand decision.** A navy or near-black shell is a
  legitimate archetype for dark-first fintech, trading and ops consoles — but then the accent
  must be bright enough to survive on it. Brand-dark on brand-dark collapses the hierarchy.
- An accent that appears everywhere stops meaning anything — restraint reads as confidence.
- **Cap the palette:** two hues plus neutrals, unless the client's brand mandates more.
- **Status colours are semantic**, never decorative. Success/warning/danger/info appear when
  they carry that meaning and nowhere else.
- **Gradients, glass and glow** only where the client's brand already uses them.
- Contrast meets WCAG 2.1 AA, including the accent on its own surface.

### Typographic craft

One display face and one UI face. A real scale (1.2-1.25 modular ratio), not ad-hoc sizes.
Weight contrast carries more hierarchy than size does — a 600 against a 400 at the same size
often beats two sizes at one weight. Tabular figures wherever numbers align in a column. Line
length 60-75 characters for prose; tighter tracking on display sizes, looser on small caps.

### The craft pass that separates award work

Purposeful whitespace rhythm on one spacing unit; optical rather than mechanical alignment;
real content at real lengths, including the longest plausible string; empty states with a voice
and a next action; motion with intent (120-200ms, easing that matches the register);
`focus-visible` on every interactive element; custom scrollbars and `::selection`; deliberate
dark mode rather than an inverted afterthought.

### Self-check — at the direction pause, and again before every push

- With the logo removed, would this read as **this sector's** product rather than generic SaaS?
- Name three decisions a template would not have made. If you cannot, the direction is not done.
- Which default did the direction statement refuse, and is that refusal visible in the output?

Award juries weight this the same way: Awwwards scores Design 40%, Usability 30%, Creativity
20%, Content 10% — creativity never at usability's expense — and the UX Design Awards judge
emotion, function and impact together.

## Style Contract

A brief leaving any axis below blank isn't done. Palette stays per-project (see Intake
Questions in `SKILL.md`); this table covers everything else the current output shipped as
zero — no shadows, radii, motion, or state. Values shown are neutral starting points, not a
mandate.

| Axis | Brief must state |
|---|---|
| Elevation | ≥4 levels with roles: resting card, raised card/menu, drawer (directional — shadow falls away from the sliding edge), modal/overlay; plus a focus ring (e.g. `0 0 0 3px`). Starting point: `0 4px 16px rgba(16,24,40,.08)` resting, `0 6px 20px rgba(16,24,40,.12)` raised, `-16px 0 48px rgba(16,24,40,.22)` drawer, `0 30px 70px rgba(16,24,40,.3)` modal. |
| Radius | A small scale with roles — control, card, container, pill (999px) — not ad-hoc per element. Starting point: 8px controls, 10-12px cards/containers, 999px pills. |
| Spacing & density | A base spacing unit (4px or 8px grid) and a density switch (compact/comfortable/spacious) the prototype can toggle. |
| Motion | Named entrances for toast, drawer, sheet, backdrop (e.g. `toastIn`, `drawerIn`, `sheetIn`, `backdropIn`); a control transition standard (e.g. `border-color .14s, box-shadow .14s`), 120-200ms. |
| Type | One display + one UI face **derived from client branding**, a weight range (e.g. 400-800), an explicit type scale. No face is mandated by the toolkit. |
| Detail pass | Custom scrollbars, `::selection`, `-webkit-font-smoothing: antialiased`, focus-visible rings on every interactive element. |

## Per-screen prompt template

```
Design a [device] screen for [app name].

Purpose: [what the user accomplishes on this screen]
User role: [who sees this — e.g. "Authenticated admin user"]

Layout:
[Top-to-bottom, left-to-right description of the screen]
- [Navigation bar description]
- [Main content area with specific components]
- [Sidebar or secondary content if applicable]
- [Footer or bottom navigation]

Content:
[Realistic mock data derived from the data model — not lorem ipsum]
[Exact quantities: "Show 6 cards in a 2x3 grid"]

States: Default view with [N] items loaded. Plus: empty (no items yet — what's shown
instead), loading (skeleton/spinner while data is fetched), and error (what failed and the
recovery action offered).

Style notes: [Reference the design system and the direction statement — "Following the
palette, type, and density defined in the Brand and Controls groups; carries the <archetype>
register and the <structural idea> layout, not a card grid"]
```

Same structure for the interactive prototype's per-screen sections, plus the router, state
and web/mobile view-toggle wiring.

## Prompt writing rules

- Plain language, not code — natural layout descriptions.
- Top-to-bottom, left-to-right — spatial reading order.
- Consistent component names and navigation descriptions, reused across every screen brief.
- Realistic mock data derived from the data model, in the correct language — never lorem
  ipsum, never generic placeholders like "Field 1".
- Exact quantities — "6 ArticleCards in 2x3 grid", not "some articles".
- One screen per file, per device.
- Every visible field maps to a real data-model attribute.

## Mobile adaptation

Single column layout, bottom sheets instead of modals, swipeable elements noted explicitly,
a bottom tab bar in place of top navigation, touch targets at least 44x44px, increased
spacing and reduced density, tables rendered as card lists or horizontal scrollers, and a
full standalone brief per screen — never "mobile version of the desktop screen." Do not
paint a fake status bar or virtual keyboard — the real device chrome renders on top when
this ships.

## Screen inventory grouping

Build the inventory from product-definition modules, architecture UI containers, data-model
entities (each implies list + detail views), and ADR-driven constraint screens (consent, RBAC,
and the like). Present it for confirmation before generating. Group by functional area:

| Group | Typical screens |
|-------|----------------|
| `01-public-and-onboarding` | Landing, registration, login, pricing |
| `02-core-experience` | Home/dashboard, main feature screens |
| `03-account-and-settings` | Profile, settings, notifications |
| `04-admin-and-ops` | Admin dashboard, user management |
| `05-content-management` | CMS, editor, media library |

Pull entity and field names from `.project/data-model/*.md` and `data-model.json`. Each screen
composes from the Phase 1 vocabulary and cites what it uses.

## Component group vocabulary

Every file's first line is `<!-- @dsCard group="…" -->` — each surface reads this to place it.
`register_assets` is legacy; leave it alone for these files.

| Group | Holds |
|---|---|
| `Brand` | palette, typography, logo |
| `Status` | success/warning/danger/info states |
| `Controls` | buttons, inputs, chips |
| `Navigation` | navbar, sidebar, tabs |
| `Surfaces` | cards, tables, modals, empty/loading/error states |
| `Screens <NN> <Group name>` | one group per functional area |
