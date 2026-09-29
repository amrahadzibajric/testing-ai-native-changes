# Estimate Output Template

Full scaffold for `../SKILL.md`'s estimate output. Read this when you're actually writing the estimate document — fill in every bracketed placeholder, don't leave any in a saved file.

```
## Estimate: [Work Item Name]

**Date:** YYYY-MM-DD
**Confidence:** Low | Medium | High
**T-Shirt Size:** XS | S | M | L | XL
**Category:** integration | complex_be | backend | frontend | infra | discovery | docs | ops
**Revision Multiplier:** ×N.NN (from category table)

### Scope
[What is included in this estimate. Link to architecture document or module spec if available.]

### Effort

**Adjustment factors applied:** AI −N% | Ceremonies +N% | Bug fixing +N%

| Activity | Human Only (PD) |
|----------|----------------|
| Development | ... |
| QA + Test Automation | ... |
| Integration / API work | ... |
| Environment / CI/CD | ... |
| Ramp-up | ... |
| **Human Only Subtotal** | **...** |

| Adjustment | Impact (PD) | Running Total (PD) |
|-----------|------------|-------------------|
| AI productivity reduction (−N%) | −... | ... |
| Ceremonies overhead (+N%) | +... | ... |
| Bug fixing buffer (+N%) | +... | **...** |

| | Human Only (PD) | AI+Human (PD) | Savings |
|-|----------------|----------------|---------|
| **Total** | **...** | **...** | **N% (N PD)** |

*Unit: person-days. All figures include Dev + QA.*

### Three-Point Range

| Mode | Optimistic | Expected | Pessimistic |
|------|------------|----------|-------------|
| Human Only | ... | ... | ... |
| AI+Human | ... | ... | ... |

### Per-Story Breakdown (if applicable)

| Story | ACs | Weight | Human (PD) | AI+Human (PD) | Hours (AI+H) |
|-------|-----|--------|-----------|----------------|--------------|
| S-XX.YY.01: [name] | N | N.NN | ... | ... | ... |
| S-XX.YY.02: [name] | N | N.NN | ... | ... | ... |
| **Feature Total** | | | **...** | **...** | **...** |

*Stories weighted by AC count × keyword multipliers. Min floor: N.N PD. Hours = AI+Human PD × 8.*

**After finalising estimates:** update each story in its Feature file with `**Estimated Hours:** N` so the WBS Excel export captures the hours per story.

### Assumptions
- [Assumption] — if false, impact: [description]

### Complexity Drivers
- [Driver] — why it adds effort or risk

### Dependencies
- [Dependency] — status: [resolved / pending / blocked]

### Risks

| Risk | Likelihood | Impact on Estimate |
|------|------------|-------------------|
| ...  | ...        | ...               |

### Estimation Blockers (if any)
- [What is missing and what is needed to unblock]
```
