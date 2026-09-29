# Dual-Option Estimates

Read this only when a Proposed ADR introduces a platform consolidation alternative that materially changes effort (≥10 PD delta). Most estimates never need this — it's a situational structure, not part of the default output.

## When it applies

The estimate must include a dual-option section alongside the standard output whenever such an ADR is pending.

## Option A / Option B structure

**Section: Technology Stack Option Comparison**

| EPIC | Option A (AI+H PD) | Option B Delta | Option B (AI+H PD) | Key change |
|------|-------------------|----------------|-------------------|------------|
| E-NN | N | −N / +N | N | What changes |
| **Total** | **N** | **−N** | **N** | |

- **Option A** = the confirmed point-solution baseline (unchanged from prior estimate)
- **Option B Delta** = the per-EPIC saving or addition; show as signed number (e.g., −20, +5)
- Source the delta from the consolidation ADR's estimated saving claim — note the confidence level

**Section: Hybrid Fallback Scenario Matrix** (when the ADR defines gates)

| Scenario | Gate outcome | Affected EPICs | Delta vs Option A | Net AI+H Total |
|----------|-------------|----------------|-------------------|----------------|
| Full Option B | All gates pass | E-NN, E-NN | −N PD | N PD |
| Hybrid (Gate 1 fail) | Gate 1 fail, Gate 2 pass | E-NN | −N PD | N PD |
| Option A retained | All gates fail | None | 0 | N PD |

## Rules

- **Do not re-estimate Option A** — it is the unchanged baseline. Only compute the delta.
- **Flag gate-blocked stories** — stories that depend on an uncleared gate are estimation blockers; list them explicitly in the Estimation Blockers section.
- **Both Human Only and AI+Human figures** are required for Option A, Option B, and each hybrid scenario.
- **When the ADR resolves**, retire the dual-option section and produce a clean estimate based on the confirmed stack only.
