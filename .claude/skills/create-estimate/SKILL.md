---
name: create-estimate
description: Produce a structured effort estimate for a module, feature, or project phase using T-shirt sizing or three-point estimation with confidence levels. Estimates include Human Only and AI+Human modes, explicit Dev+QA splits, category-based multipliers, and QA buffer floors to prevent tight estimates.
license: "AP AI-Native Delivery Toolkit v3.5 - AP Confidential - governed by LICENSE.md"
metadata:
  user-invocable: true
---

# Skill: Create Estimate

## Purpose

Produce a structured effort estimate for a defined scope item — a module, feature, integration, or project phase. This skill defines the intake process, estimation method, and output template used by agents to size work consistently.

**Every estimate must include both Human Only and AI+Human person-day figures** so stakeholders can compare traditional and AI-assisted delivery modes.

---

## Invocation

Use this skill when:
- A module, feature, or work item needs to be sized for effort
- The user asks for an estimate, sizing, or level-of-effort assessment
- A delivery plan requires effort data for phasing or resource planning
- An existing estimate needs to be updated with new information

---

## Intake Questions

Gather as much as possible from source documents, architecture documents, and project context before asking the user.

1. **What is being estimated?** — Name the module, feature, or work item precisely.
2. **What is the scope?** — What does "done" look like? List the key deliverables or acceptance criteria.
3. **What is the technical approach?** — Link to an architecture document or describe the implementation approach. If no architecture exists, flag this as an estimation blocker.
4. **What are the complexity drivers?** — Integration points, data migration, new technology, regulatory requirements, performance targets, unknowns.
5. **What is the team shape?** — How many people, what roles, what experience level?
6. **What are the dependencies?** — What must be in place before this work can start? What is this work blocking?
7. **What is already known or decided?** — Link to relevant ADRs, tech evaluations, or prior estimates.

---

## Adjustment Factors Intake

**Before starting any estimate, always prompt the user to confirm the three project-level adjustment factors.** Present the defaults and ask whether any should be changed for this project. If the user provides no input or says "use defaults", proceed with the defaults below.

| Factor | Default | When to change |
|--------|---------|----------------|
| **AI productivity reduction** | −40% | Increase if the team is not yet proficient with AI tools; decrease if the project is heavily non-automatable |
| **Ceremonies overhead** | +10% | Increase for large teams or compliance-heavy processes; decrease for very small or async teams |
| **Bug fixing buffer** | +10% | Increase for high-uncertainty or legacy-adjacent work; decrease for well-understood greenfield work |

Present this prompt to the user before estimating:

> Before I begin the estimate, please confirm the adjustment factors for this project — or say "use defaults" to proceed with the values below:
>
> - **AI reduction:** −40% — how much does AI assistance cut effort on this project?
> - **Ceremonies:** +10% — planning sessions, demos, retrospectives, standups
> - **Bug fixing:** +10% — unplanned rework and defect resolution
>
> These percentages may vary per project. If you do not specify, I will use the defaults above.

Record the confirmed values and apply them consistently across every estimate in this session.

---

## Estimation Method

### T-Shirt Sizing (for early-stage or high-level estimates)

| Size | Person-Days (range) | Typical characteristics |
|------|---------------------|------------------------|
| **XS** | 1–3 | Single concern, no integration, well-understood pattern |
| **S** | 3–8 | Small feature, 1 integration point, low uncertainty |
| **M** | 8–20 | Multi-component, 1–2 integrations, moderate uncertainty |
| **L** | 20–50 | Cross-cutting, multiple integrations, significant unknowns |
| **XL** | 50–100 | Major subsystem, complex integrations, high uncertainty |
| **XXL** | 100+ | Should be broken down further before estimating |

### Three-Point Estimation (for detailed estimates)

For each line item, provide:
- **Optimistic (O):** Everything goes smoothly, no surprises, team is experienced
- **Expected (E):** Normal pace, typical issues encountered and resolved
- **Pessimistic (P):** Significant complications, unfamiliar territory, integration surprises

**Weighted estimate:** (O + 4E + P) / 6

---

## Dual Estimation: Human Only vs AI+Human

Every estimate must provide **two figures**:

| Mode | Description |
|------|-------------|
| **Human Only** | Traditional team effort without AI coding assistants |
| **AI+Human** | With AI coding assistants (Copilot, Claude, etc.) integrated into the workflow |

### AI+Human Calculation

Apply the three confirmed adjustment factors sequentially to derive the AI+Human total from the Human Only baseline:

**AI+Human = Human Only × (1 − AI%) × (1 + Ceremonies%) × (1 + BugFix%)**

With defaults: Human Only × 0.60 × 1.10 × 1.10 = Human Only × **0.726** (~27% total savings over Human Only)

Show each adjustment as a separate line item in the estimate output — do not collapse them into a single multiplier. Stakeholders must be able to see what was added and why.

| Adjustment | Default Factor | Applied to |
|-----------|----------------|-----------|
| AI productivity reduction | −40% | Human Only baseline |
| Ceremonies overhead | +10% | Post-AI subtotal |
| Bug fixing buffer | +10% | Post-ceremonies subtotal |

If the user confirmed custom values, substitute those values in the formula and label them clearly in the output.

---

## Category-Based Revision Multipliers

Base estimates are often systematically optimistic. Apply category multipliers to account for real-world overhead that initial sizing underweights:

| Category | Multiplier | Rationale |
|----------|-----------|-----------|
| **integration** | ×1.55 | New external API: learn docs, auth flows, sandbox setup, error handling, rate limits, retry/DLQ, contract tests |
| **complex_be** | ×1.40 | Multi-component data flows, state machines, business rules with edge cases, cross-service coordination |
| **backend** | ×1.30 | Standard services: CRUD, validation, moderate business logic |
| **frontend** | ×1.30 | Components, responsive design, a11y, cross-browser testing |
| **infra** | ×1.20 | Well-understood IaC patterns, but environment parity matters |
| **discovery** | ×1.10 | People-time; AI helps prep but not the meetings themselves |
| **docs** | ×1.20 | Training materials, runbooks, architecture documentation |
| **ops** | ×1.20 | Process-heavy support and maintenance |

### How to Classify Work

When estimating a feature, assign it to the **most demanding** category that applies:

- If it involves a **new external API** (CRM, payment, CMP, video platform, market data) → `integration`
- If it has **state machines, multi-component data flows, or complex business rules** but no new external API → `complex_be`
- If it's a **standard CRUD service or backend module** → `backend`
- If it's **primarily UI components and rendering** → `frontend`
- If it involves **infrastructure-as-code, CI/CD, or test frameworks** → `infra`
- If it's **workshops, meetings, or evaluations** → `discovery`
- If it's **documentation, training, or knowledge transfer** → `docs`
- If it's **ongoing support or maintenance** → `ops`

**Common misclassifications to avoid:**
- Video playback with SDK integration + token auth → `complex_be` (not `frontend`)
- AI-powered features requiring external AI API → `complex_be` or `integration` (not `frontend`)
- Mobile features with native SDK integration → `complex_be` (not `frontend`)
- **A confirmation, write-up, or execution-of-existing-tooling task → `docs` (not `frontend`/`backend`/`infra`), even when it sits inside a technical Feature.** "Confirm the existing test tooling," "produce a findings note on the identified system," "document the rendering guideline," "execute the pre-built Finding #N test case" are all `docs`-category — no new dev or QA work is being built, only something is being checked, written down, or run once. Forcing these into a code-bearing category drags in that category's PD floor (2.5–4.0 PD) for work that actually costs a fraction of that. Ask directly: *is this task producing new working code or a new test artifact, or is it confirming/writing down/running something that already exists?* — the latter is `docs`.
- **Watch for "formal [artifact]" language inflating a small feature's cost.** "Produce a formal integration assessment," "formal architecture review," etc. import the effort expectations of a large programme's template even when the actual scope is one system on a small feature. Reserve "formal" framing and its effort for engagements where the artifact will actually carry that weight (multi-system integration, client-facing deliverable); size a small feature's version as a short findings note instead.

---

## Confidence Levels

State the confidence level for every estimate:

| Level | Meaning | When to use |
|-------|---------|-------------|
| **Low** | < 50% of scope understood; estimate could swing ±50% or more | No architecture, vague requirements, unproven technology |
| **Medium** | 50–80% of scope understood; estimate could swing ±25–50% | Architecture exists but has open questions; some unknowns remain |
| **High** | > 80% of scope understood; estimate could swing ±10–25% | Clear requirements, proven patterns, experienced team, architecture approved |

---

## What to Include in Effort

**Every estimate includes Dev + QA time.** Do not assume QA is "free" or will be handled separately. Every implementation story must budget for:

### Dev Activities
- Feature implementation
- Code review and peer review
- Integration with other modules or external systems
- Environment setup (CI/CD, staging, infrastructure)
- Ramp-up for new technology, domain, or codebase

### QA Activities
- Unit and integration test writing
- Test automation (framework setup + test scripts)
- Manual exploratory testing
- Cross-browser / cross-device testing (for frontend)
- Sandbox / mock environment testing (for integrations)
- Performance and load testing (where applicable)

### Overhead
- Coordination (standups, planning, cross-team alignment) — typically 10–20% of development effort
- Documentation — API docs, runbooks, handover material (only if in scope)

---

## QA Buffer and Minimum Story Floors

### The No-Tight-Estimate Rule

**Never produce a story estimate that doesn't leave room for proper QA.** When estimates are used for client-facing RFQs, every story must be deliverable including development, testing, and test automation.

### Minimum Person-Days per Story

Enforce these floors based on the feature's work category:

| Category | Min PD per Story | Typical Split |
|----------|-----------------|---------------|
| **integration** | 4.0 PD | 2.5 dev + 1.5 QA (external API sandbox, mock services, contract tests) |
| **complex_be** | 3.5 PD | 2.0 dev + 1.5 QA (state machines, edge case testing) |
| **backend** | 3.0 PD | 1.5 dev + 1.5 QA (standard CRUD + validation testing) |
| **frontend** | 3.0 PD | 1.5 dev + 1.5 QA (component + cross-browser + a11y testing) |
| **infra** | 2.5 PD | 1.5 dev + 1.0 QA (IaC + environment parity verification) |
| **discovery** | No floor | No QA needed — pure workshops/documentation |
| **docs** | No floor | No QA needed — documentation deliverables |
| **ops** | No floor | No QA needed — process/support activities |

**Before applying any floor, confirm the category is right for what the story actually does** — a confirmation/write-up/execution-of-existing-tooling story reclassified to `docs` (see "How to Classify Work" above) has no floor at all. Applying a `frontend`/`backend`/`infra` floor to a task that's really a quick confirmation or a documentation update is the most common way a small feature's estimate balloons.

### Compression Detection

When distributing a feature's effort across stories, if **more than 50% of stories end up within 1 PD of the floor**, first check *why* before bumping anything: if those stories are near the floor because they were misclassified into a floor-bearing category when they're actually `docs`-type confirmation/write-up tasks, **reclassify them instead of bumping the total** — that's a category error, not a sign the feature needs more room. Only apply the 20% bump when the stories are genuinely floor-scale dev/QA work and heavy stories are getting squeezed to fund them.

### QA/Test-Automation Feature Proportionality

When a Feature exists solely to test-automate another Feature (e.g., "X QA & Test Automation" paired with "X"), sanity-check its size against the Feature it tests: a dedicated QA/automation Feature costing close to a third of its paired dev Feature's size, for a small/bounded scope, is a signal to look closer, not a default to accept. Two specific traps to check for:
- **Test *execution* costed like test *authoring*.** "Run the pre-built Finding #N test case" is a `docs`-scale execution task; "design and author a new test case" is real QA-floor work. Don't cost the former at the latter's rate.
- **Per-item testing cost assumed linear with content volume.** Five email templates sharing one test harness don't cost 5× a single template's test effort — the harness is built once; each additional item mostly adds assertion/fixture time, not a repeated full setup. Content-*authoring* volume (writing 5 translations) is a real linear-ish driver; content-*testing* volume behind a shared harness usually isn't.

---

## Weighted Story Distribution

When breaking a Feature's PD into per-story estimates, **never use equal splits**. Equal splits produce indefensible numbers for client-facing documents.

### Weight Calculation

Each story gets a weight based on:
1. **Base weight** = max(acceptance_criteria_count, 2)
2. **Keyword multipliers** applied to story name + description:

**Heavy indicators** (increase weight — these stories take more effort):
| Indicator | Keywords | Multiplier |
|-----------|----------|------------|
| Integration work | integrate, webhook, sync, ingest, provider, API | ×1.40 |
| Workflow logic | workflow, orchestrate, state machine, pipeline, approval chain | ×1.30 |
| Compliance work | GDPR, compliance, audit, consent, regulatory, RTBF, embargo | ×1.25 |
| Real-time work | real-time, streaming, cache, latency, WebSocket | ×1.20 |
| Data modelling | schema, data model, migration, transform | ×1.15 |
| Security work | encrypt, token, authentication, authorization, RBAC, MFA, JWT | ×1.20 |

**Light indicators** (decrease weight — only if no heavy indicator matched):
| Indicator | Keywords | Multiplier |
|-----------|----------|------------|
| Monitoring | monitor, dashboard, metric, alert, health check | ×0.90 |
| Display | display, render, layout, responsive, style | ×0.90 |
| Configuration | config, setting, toggle, feature flag | ×0.85 |
| Documentation | document, runbook, training, knowledge transfer | ×0.90 |

3. **Final weight** = base × adjustment (heavy product, capped at 2.0; or light minimum)
4. **PD allocation** = feature_total × (story_weight / sum_of_all_weights), rounded to 0.5 PD

### Distribution Rules
- Round to 0.5 PD increments
- Enforce minimum floor per story (see QA Buffer section)
- If floor enforcement causes redistribution from heavy stories, detect compression and bump feature total
- Feature total must be preserved (or increased) — never decreased by distribution

---

## Estimation Blockers

If any of the following are true, flag the item as **not estimable** rather than guessing:

- No architecture or technical approach defined
- Requirements are ambiguous or contradictory
- Critical dependency is unresolved (e.g., third-party API access not confirmed)
- Technology choice is still open

For blocked items, state what is needed to unblock the estimate.

**Writing the result into the Feature file's `**Estimated Hours:**` field** — `_export_wbs.py` parses this field literally; only these forms are recognised:
- A numeric value: `**Estimated Hours:** N (AI+Human; Human Only: M) — ...` — summed into EPIC/programme totals.
- Not yet estimated or genuinely not estimable: `**Estimated Hours:** TBD — ...` — the value must start with the literal token `TBD` (not prose like "Not estimable"); excluded from totals, explanation goes after the em dash.
- Contingent/excluded-from-baseline work (e.g., a fallback only needed if a spike fails): `**Estimated Hours:** N (AI+Human; Human Only: M) — CONTINGENT: ...` — the hour figure stays, but the word `CONTINGENT` anywhere on the line tells the export to display it without summing it into committed totals.

Any other leading text in this field will crash the Excel export.

---

## Output Template

Full scaffold in [`references/examples.md`](references/examples.md) — read it when writing the actual estimate document. Every estimate covers: Scope, Effort (activity table + adjustment factors + three-point range), Per-Story Breakdown, Assumptions, Complexity Drivers, Dependencies, Risks, and Estimation Blockers.

---

## Dual-Option Estimates

Only relevant when a Proposed ADR introduces a platform consolidation alternative that materially changes effort (≥10 PD delta) — most estimates never need this. Full structure (Option A/B comparison table, hybrid fallback scenario matrix, and the rules for retiring the section once the ADR resolves) is in [`references/dual-option-estimates.md`](references/dual-option-estimates.md).

---

## Rolling Up Estimates

When producing a full project estimate, use individual module estimates as building blocks. In the roll-up:
- Sum the ranges (do not sum the expected values and call it a range)
- State the overall confidence as the lowest confidence of any component
- List cross-module risks that individual estimates cannot capture (integration between modules, shared infrastructure, coordination at scale)
- **Always show both Human Only and AI+Human totals** with savings percentage and PD/Story averages
- **Verify no story falls below the QA floor** for its category before finalising

---

## Sizing Platform-Driven Work

Estimates are person-days. Where a platform or tooling decision creates *work*, that work is sized here even though the platform itself is not:

- **Effort caused by a cheaper tier.** A lower model tier or lighter tooling that needs extra implementation, retry handling, or QA is person-days in the affected stories. Under-provisioning shows up as effort, not savings — size it, and name the decision that drives it.
- **Consumption-driven stories.** Usage metering, cost dashboards, rate limiting, and token-budget enforcement are ordinary implementation work.

---

## Estimation Review Checklist

Before finalising any estimate, verify:

- [ ] Adjustment factors were confirmed with the user (or defaults explicitly adopted)
- [ ] AI reduction, ceremonies overhead, and bug fixing buffer are shown as separate line items
- [ ] Every figure includes both Human Only and AI+Human modes
- [ ] AI+Human total uses the formula: Human Only × (1 − AI%) × (1 + Ceremonies%) × (1 + BugFix%)
- [ ] Per-story breakdown includes Hours (AI+Human PD × 8) column
- [ ] Feature files have been updated with `**Estimated Hours:** N` per story after estimates are finalised
- [ ] Category multiplier has been applied (not just base sizing)
- [ ] Every implementation story has at least the minimum PD for its category (QA floor) — and the category itself was checked against the "confirmation/write-up/execution task → docs" rule before the floor was applied
- [ ] No feature has >50% of stories compressed against the floor (compression check) — and if it does, that it's genuine floor-scale work, not a category misclassification
- [ ] Story PD distribution uses weighted allocation, not equal splits
- [ ] Integration stories account for: API learning, sandbox setup, error handling, retry logic, contract tests
- [ ] A dedicated QA/test-automation Feature's size was sanity-checked against its paired dev Feature (see QA/Test-Automation Feature Proportionality)
- [ ] Confidence level is stated and justified
- [ ] Estimation blockers are flagged, not papered over with guesses
- [ ] Numbers are defensible for client-facing RFQ — "can we deliver this in N days including QA?"
- [ ] Every figure is person-days or hours — no currency, token volumes, seat counts, license items, or allocation hours
- [ ] Effort caused by a cheaper-tier or lighter-tooling decision is sized as person-days, with the driving decision named
- [ ] Consumption-driven stories (metering, cost dashboards, rate limiting, token-budget enforcement) are sized as implementation work
