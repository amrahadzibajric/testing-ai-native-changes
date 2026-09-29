# E-03: Frontend Foundation and Design System

**EPIC ID:** E-03
**Delivery Phase:** Phase 1 — POC (build)
**Dependencies:** E-02 (solution scaffold). Blocks the UI stories of E-04 to E-09.
**ADR References:** ADR-0003, ADR-0016, ADR-0024
**Architecture References:** arch v1.2 (Presentation: Razor Pages UI, responsive); `.project/ui-designs/design-direction.md`; `.project/ui-designs/README.md` (Style Guide artboards: Brand, Status, Controls, Navigation, Surfaces); nfrs.md NFR-016, NFR-017, NFR-018

## Description

This EPIC implements the approved design direction as reusable front-end foundations: design tokens and typography, the component library (status vocabulary, controls, surfaces), the application shell with web navigation and the phone tab bar, and the shared empty, loading, error, responsive and accessibility patterns. Before this EPIC the application has no shared look; afterwards every functional EPIC composes its screens from one vocabulary, so the POC reads as a calm, precise internal operations tool rather than a default card grid. It applies the design direction as approved on 2026-09-29: timeline board as the structural idea, one orange accent used only for the primary action, the selected slot and the live now line, and no KPI cards, hero or gradients.

## Scope Boundaries

- **Includes:** tokens, fonts, base detail; status vocabulary, controls, surfaces, feedback patterns; navbar, mobile tab bar, admin hub; empty/loading/error patterns, responsive behaviour, accessibility baseline, error pages.
- **Excludes:** feature-specific screens (built inside E-04 to E-09), including the timeline board itself (F-06.03); any client branding (none supplied); dark mode (light only for the POC).

## Key Technical Components

Razor Pages layout and partials, CSS tokens and components, small JavaScript for drawers, sheets and focus management.

## Key User Roles Affected

Employees and admins (all screens), presenter (phone-size demonstrations).

## Done Criteria

- All design tokens and the status vocabulary match the approved direction, with one accent colour only.
- The web navbar and phone tab bar render role-appropriate items and always show the acting person, role and Switch user.
- Empty, loading and error states, the confirm modal, drawer and phone sheet are reusable components.
- Main screens are keyboard operable and meet the WCAG 2.1 AA aim for contrast and semantics.
- Fonts are self-hosted so the demonstration works offline.

## Features

| Feature | Description | Stories | Surfaces |
|---------|-------------|---------|----------|
| [F-03.01: Design Tokens and Base Styles](../features/F-03.01-design-tokens-and-base-styles.md) | Colour, type, spacing, elevation, motion and base detail | 3 | Web, Phone browser |
| [F-03.02: Component Library](../features/F-03.02-component-library.md) | Status vocabulary, controls, surfaces, feedback | 4 | Web, Phone browser |
| [F-03.03: Application Shell and Navigation](../features/F-03.03-application-shell-and-navigation.md) | Navbar, tab bar, admin hub, role-aware navigation | 4 | Web, Phone browser |
| [F-03.04: States, Responsive Behaviour and Accessibility Baseline](../features/F-03.04-states-responsive-behaviour-and-accessibility-baseline.md) | Empty, loading, error patterns, breakpoints, accessibility, error pages | 4 | Web, Phone browser |
