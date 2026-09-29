# Design Direction — Meeting Room Booking

**Status:** Approved by project lead (Amra Hadzibajric), 2026-09-29
**Date:** 2026-09-29
**Surface:** claude.ai Design canvas (Artifact fallback — `claude-design` MCP not connected this session; `DesignSync` is reserved for user-started library sync)
**Branding source:** None supplied. No client style guide, logo or palette in `.project/input/`; direction below is a neutral internal-tool proposal — replace with client branding if provided.

## Direction statement

1. **Archetype and register.** An internal operations tool for up to 50 office employees and a handful of admins (ADR-0001, ADR-0024). It should feel calm, precise and trustworthy — like a well-run office front desk, not a SaaS marketing product. People should answer "which room is free, when?" in one glance and book it in under a minute.

2. **Structural idea — the day timeline board.** The core screen is a timeline, not a card grid: rooms as rows, the office day 08:00–20:00 as columns snapped to the 15-minute grid, bookings as blocks, and a single live "now" line. Free time is the empty track you click or drag to book. Capacity filter dims rooms that are too small rather than hiding them. Admin views are table-first with a detail pane. On mobile, the board becomes a per-room vertical time list with a "next free slot" summary per room.

3. **Type.** *Inter Tight* (600–700) for the few display moments — page titles and the date header — and *Inter* (400–600) for all UI text, with tabular figures everywhere a time or count appears so 09:15 and 10:45 align on the grid. Scale ratio 1.2. Hierarchy comes from weight, not size.

4. **Colour — 60/30/10.** 60%: a warm branded neutral (off-white `#F7F6F3`, surfaces white). 30%: graphite structure (`#1F2328` text, `#5B616B` muted, `#E3E1DC` borders and grid lines). 10%: one accent, **signal orange `#C2410C`** (AA on white), reserved for the primary action (Book, Save), the selected slot, and the live "now" line. Booking status colours are strictly semantic: Confirmed = graphite outline, Checked in = green, No-show = amber, Cancelled = grey strikethrough, Late cancellation = amber flag. Light mode only for the POC.

5. **Deliberately refused.**
   - No stat-card KPI row on any screen, including the admin home.
   - No grid of room cards with photos — rooms are rows on a timeline.
   - No hero, landing page or gradient; the entry point is a plain "pick a user" list (ADR-0005).
   - No decorative illustration; empty states use one line of plain language and a next action.
   - No second accent colour.

## Navigation

- **Web:** a slim top navbar. Employee: *Find a room*, *My bookings*. Admin adds: *All bookings*, *Review* (late cancellations and no-shows), *Rooms*, *People*, *Import*. The current user and a *Switch user* control sit at the right.
- **Mobile:** bottom tab bar — *Find*, *My bookings*, and for admins *Admin* (a list leading to the admin screens). Touch targets ≥ 44 px.

## Style Contract starting values

| Axis | Value |
|---|---|
| Elevation | resting `0 1px 2px rgba(31,35,40,.06)`; raised/menu `0 6px 20px rgba(31,35,40,.12)`; drawer `-16px 0 48px rgba(31,35,40,.20)`; modal `0 30px 70px rgba(31,35,40,.28)`; focus ring `0 0 0 3px rgba(194,65,12,.35)` |
| Radius | 6 px controls, 8 px booking blocks and cards, 12 px containers/drawers, 999 px pills |
| Spacing | 4 px base; comfortable default, compact option for admin tables |
| Motion | `drawerIn`, `sheetIn`, `toastIn`, `backdropIn`, 140–180 ms ease-out; controls `border-color .14s, box-shadow .14s` |
| Type | Inter Tight 600–700 display, Inter 400–600 UI, tabular numerals, 1.2 scale |
| Detail | Custom thin scrollbars on the timeline, orange-tint `::selection`, antialiased text, focus-visible on every control |

## Self-check

- **Sector-legible without a logo:** yes. The timeline board and the live "now" line read as scheduling or operations at once.
- **Three decisions a template would not make:**
  - A timeline board instead of room cards.
  - Rooms that are too small get dimmed, not hidden.
  - The only accent colour is reserved for "now" and the primary action.
- **Refusals visible:** there is no KPI row, no room-photo grid and no hero.
