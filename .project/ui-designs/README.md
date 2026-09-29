# UI Designs — Meeting Room Booking

**Canvas:** https://claude.ai/artifact/To59cLQHzaCeyPeWuDtcNa
**Last updated:** 2026-09-29
**Design direction:** [design-direction.md](design-direction.md), approved by the project lead on 2026-09-29
**Target record:** [.design-canvas.json](.design-canvas.json)

## Surface used, and why

The designs were built on a claude.ai **Design canvas**, published as an Artifact. The `claude-design` MCP server was not connected in this session. `DesignSync` is limited by its own contract to user-started `/design-sync` library synchronisation, so it was not used to make designs. The canvas is the documented fallback surface in the `generate-ui-designs` skill.

**Sharing:** the canvas is **private by default**. Others cannot open the link until the owner shares it from the canvas's Share menu. This skill never changes sharing.

**Working files:** `canvas/project/` holds one `.dc.html` per artboard plus `canvas.json`. Later runs re-seed from these files and republish to the same URL.

## Verification status

Every artboard file is confirmed present on the canvas by the publish file listing. **No screen has been visually checked yet.** The canvas surface gives no screenshot capability, so the direction self-check, the 60/30/10 check and the layout check still need a human look. The target record therefore keeps `phase: "1-system"` and marks each group `pushed`, not `verified`.

## Inventory

| Page | Artboards | Covers |
|---|---|---|
| Style Guide | Brand, Status, Controls, Navigation, Surfaces | Tokens, booking status vocabulary, controls, navigation, timeline board, table and drawer, and empty, loading and error states |
| 01 Entry | Pick a user (web, mobile), states | M-04: the pilot pick-a-user list shows active people only |
| 02 Booking | Find a room (web timeline board, mobile per-room day list), Book drawer (web), Edit sheet (mobile), My bookings (web, mobile with check-in open), states | M-01 and M-02: availability, capacity filter, booking, editing, check-in, the late-cancellation warning, the 3-booking limit, a booking conflict and no-show release |
| 03 Admin | All bookings, Review, Rooms, People and Import, each on web and mobile, plus an admin hub (mobile) and states | M-03, M-04, M-05 and M-06: room and person deactivation confirms, the last-admin block, the import result with rejected rows, and an unreadable CSV header |
| 04 Prototype | One interactive artboard with a Web/Mobile switch | The flow: pick Lena, book Board 10:45, see the confirmation, then My bookings |

All the data shown is synthetic: 8 active people and 1 inactive person. The rooms are exactly as in the specification: Focus fits 2, Team fits 6, Board fits 12, and Old Room fits 4 and is not in use. The screens are set on Tuesday 6 October 2026 at 10:40 office time (CET/CEST).

## Branding summary

No client branding was supplied, so this is a neutral internal-tool look.
- **Colour:** a warm neutral canvas (`#F7F6F3`), graphite structure (`#1F2328` and `#5B616B`), and one accent, signal orange `#C2410C`. The accent is used only for the primary action, the selected slot and the live "now" line.
- **Status colours:** strictly semantic.
- **Type:** Inter Tight for display text and Inter for UI text, with tabular figures.
- **Deliberately absent:** KPI stat cards, room-photo grids, a hero section and gradients.

## Open items

- Visual review of every group, then set `phase` to `"2-screens"` in the target record.
- If the client supplies branding, re-base the direction and the tokens first, then the screens.
