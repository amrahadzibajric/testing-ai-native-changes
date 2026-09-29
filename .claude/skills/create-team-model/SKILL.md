---
name: create-team-model
description: Design the delivery team structure and allocation — role seniority levels, weekly hour constraints, SE track patterns, buffer weeks, and capacity. Produces the team model, developer assignments document, and team-allocation.yaml for Excel export. Use when staffing a programme, deciding headcount and seniority, or building the allocation Gantt.
license: "AP AI-Native Delivery Toolkit v3.5 - AP Confidential - governed by LICENSE.md"
metadata:
  user-invocable: true
---

# Skill: Create Team Model

## Purpose

Design **who delivers the work**: role composition, seniority levels, weekly allocation, track structure, and the calendar shape of the engagement.

This is separate from effort estimation. `create-estimate` answers *how much work is there* in person-days; this skill answers *who is on the team, at what seniority, for which weeks, at how many hours*. The estimate is an input here — never re-derive effort figures in a team model.

| Concern | Skill | Output |
|---------|-------|--------|
| How much work | `create-estimate` | `estimate-*.md` (person-days) |
| **Who delivers it** | **`create-team-model`** | **`team-model-*.md`, `developer-assignments-*.md`, `team-allocation.yaml`** |
| What it consumes | `create-consumption-plan` | `consumption-*.md` |

**Never produce allocation values that fall outside the ranges defined below.**

---

## Invocation

Runs at **Phase 10 (Plan delivery)**, after estimates are finalised. Use this skill when:
- A programme needs a team model — roles, headcount, seniority, ramp-up
- Allocation hours or seniority levels need deciding or reviewing
- SE capacity needs structuring into parallel tracks
- `team-allocation.yaml` needs producing or updating for the Excel export
- A delivery timeline needs buffer weeks placed

**Prerequisite:** estimates exist and story hours are written into the feature files. Allocation built on unfinished estimates will not reconcile with the export.

---

## Inputs to Read

| Input | What it provides |
|-------|------------------|
| `.project/estimates/estimate-*.md` | Total effort, confirmed `ai_factor` and adjustment factors |
| `.project/backlog/features/` | Story hours per EPIC (to verify capacity reconciles) |
| `.project/backlog/epics/` | EPIC IDs, labels, sequencing |
| `.project/estimates/delivery-plan-*.md` | Phase boundaries, critical path, milestones |
| `.project/context/project-context.md` | Project name, client constraints |

---

## Role Seniority Levels

AP uses the following seniority ladder. Always show the abbreviation next to the role name in team allocation tables and estimates (e.g. `Software Engineer (S2)`).

| Abbreviation | Level | Description |
|--------------|-------|-------------|
| J1 | Junior I | Entry-level; needs close guidance on most tasks |
| J2 | Junior II | Can work independently on well-defined stories; needs review |
| J3 | Junior III | Solid execution on complex stories; beginning to mentor |
| S1 | Senior I | Owns a bounded domain end-to-end; minimal supervision |
| S2 | Senior II | Leads a technical track; makes design decisions within the track |
| L | Lead | Owns delivery decisions across the whole workstream or EPIC |
| SL | Senior Lead | Programme-level authority; client-facing accountability |

**Role responsibilities:**

| Role | Abbreviation | Primary responsibilities |
|------|--------------|--------------------------|
| Engagement Lead (L) | EL | Project budget, commercial governance, resource allocation, executive client relationship |
| Delivery Solution Lead (L) | DSL | Technical leadership, milestone ownership, overall delivery on time and on schedule |
| Program Architect (S2) | PA | Leads architecture decisions, requirements shaping, and backlog quality — not just documentation of decisions others made. Supervises SE execution architecturally (including S1 tracks that are otherwise day-to-day independent), programme quality, internal demos (last gate before client), solutioning |
| Program Manager (S2) | PM | Project management + Scrum Master — sprint planning, ceremonies, invoicing, cross-team coordination |

**Minimum seniority by role:**

| Role | Abbreviation | Default level | Increase to | When to increase |
|------|--------------|---------------|-------------|------------------|
| Engagement Lead (L) | EL | L | SL | Project is very large, multi-client, or high-risk commercially |
| Delivery Solution Lead (L) | DSL | L | SL | Project has very high technical complexity or multiple concurrent workstreams |
| Program Architect (S2) | PA | S2 | L | Architecture is highly complex, multi-system, or involves novel technology decisions |
| Program Manager (S2) | PM | S2 | L | Programme has large team, multiple EPICs running concurrently, or heavy governance overhead |
| QA Engineer | S1 | — | Must own test strategy for a domain independently |
| SE – track lead (Track A / main backend) | S2 | — | End-to-end ownership of a complex technical track |
| SE – domain / burst track | S1 | — | Bounded scope but still senior-level complexity |

**When to use J-levels:** Junior engineers can be added to support tracks but must be paired with an S1 or higher. Never put a junior on Track A or as the only SE in a phase.

---

## Role Allocation Constraints

Hours are complexity-dependent. Use the ranges below as a starting point and justify deviations in the team model.

| Role | Typical weekly hours | Complexity scaling |
|------|---------------------|--------------------|
| Engagement Lead (L) | 2–4 h/wk (5–10%) routine; up to 8h escalated | 2–4h for routine commercial oversight (the normal case); 8h only for active budget negotiations, escalations, or multi-client programmes |
| Delivery Solution Lead (L) | 2–4 h/wk (5–10%) routine; 8–16h+ escalated | 2–4h is the default when DSL is providing governance/review only (the normal case for a lean team); 8–16h+ only when DSL is ALSO doing hands-on execution — actively unblocking technical decisions, managing multiple concurrent EPICs, or (only where a dedicated PA is genuinely not justified — see Allocation Principles) personally covering PA-persona work under a documented, disclosed exception. Justify the escalated tier explicitly; don't default to it. The escalated tier tops out around 8-16h/wk — if a leadership role would need to go to 40h/wk (full-time) to cover reassigned work, that work belongs on an execution role instead, not a bigger DSL spike (see the Reconciliation reference's tool-limitation note for why). |
| Program Architect (S2) | 20–40 h/wk | 20h (50%) when architecture is defined and the team is executing; 40h (100%) during discovery, high-complexity design phases, or when acting as primary technical reviewer before client demos |
| Program Manager (S2) | 20 or 40 h/wk (50% or 100% only, when separately staffed) | 20h (50%) for standard teams; 40h (100%) for large teams, multiple concurrent EPICs, or ceremony-heavy programmes. PM doubles as Scrum Master — size for both responsibilities. Never allocate a standalone PM row at a lower fraction (e.g. 16h/40%) — if ceremony load is genuinely lighter than 50%, consolidate PM into EL instead (see Allocation Principles) rather than under-allocating a separate row. |
| Software Engineer (L) | 40 h/wk; 20 h/wk only if phase wind-down warrants it | Track lead across a full EPIC. Maximise allocation. |
| Software Engineer (S2) | 40 h/wk; 20 h/wk only if phase wind-down warrants it | Domain or main track lead — earns S2 either by leading a track *relative to other tracks/engineers on the programme*, OR by carrying the Program Architect hat for the whole engagement (architecture leadership, supervising any other SEs) even as the sole SE. Either justification is sufficient on its own. On a lean, single-continuous-SE engagement, defaulting to S2 because that SE carries the PA hat is the norm (see Allocation Principles: Program Architect), not an exception requiring extra justification. |
| Software Engineer (S1) | 40 h/wk; 20 h/wk only if phase wind-down warrants it | Bounded domain or burst track. Right level only when *neither* S2 justification applies — a domain/burst track running alongside a different S2 who already owns the PA hat, or an engagement whose architecture decisions are genuinely trivial enough for DSL to supervise directly without a dedicated architect hat. "Minimal supervision" describes day-to-day execution, not architecture: an S1 still works under someone's architectural direction (the team's PA-hat holder, or DSL on a trivial-architecture engagement) and does not self-direct architecture decisions or own PA-persona requirements/backlog work. |
| Software Engineer (J3) | 40 h/wk; 20 h/wk only if phase wind-down warrants it | Must be paired with S1+. Can own well-defined stories independently. |
| Software Engineer (J2) | 40 h/wk; 20 h/wk only if phase wind-down warrants it | Must be paired with S1+. Works on clearly scoped stories with regular review. |
| Software Engineer (J1) | 40 h/wk; 20 h/wk only if phase wind-down warrants it | Must be paired with S1+. Close guidance required; never sole engineer in a phase. |
| QA Engineer (S1+) | 40 h/wk; 20 h/wk only if phase wind-down warrants it | QAs own test strategy, test plans, test design, and test automation once staffed — staffing does not have to start day 1 of Discovery. Default to onboarding once a real testable surface exists (the first track's stories are far enough along to test) and go straight to a clean full rate (20h or 40h) from that point — prefer one continuous block over a multi-step ramp (see "Concentrate capacity" in Allocation Principles below); a ramp that only exists to track rising SE headcount reads as reverse-engineered, not staffed. The 25% dev-capacity floor applies from QA's onboarding point onward, not before it. They are the gate for all QA test artifacts. Test case execution is a minor part of this role. |
| QA Engineer (J2–J3) | 40 h/wk; 20 h/wk only if phase wind-down warrants it | Must be paired with S1+ QA. Executes test cases under supervision; test strategy, test plan, and automation ownership always stays with S1+. |

## Allocation Principles

- **Prefer fewer people at high allocation over more people at low allocation.** Staff to cover the work, not to spread names across a RACI.
- **Concentrate capacity — do not spread it.** Prefer a shorter engagement at 40 h/wk over a longer engagement at 20 h/wk. Two weeks at full capacity delivers faster and maintains sharper context than four weeks at half capacity.
- **Do not staff to a blocked dependency.** If a person cannot start meaningful work until another EPIC, track, or story completes, either start them later (after the dependency clears) or assign them to parallel work elsewhere — never model them as allocated but idle. Idle allocation is not a valid cost; it is a planning mistake.
- **Avoid onboard → offboard → re-onboard cycling for the same role.** A schedule with a gap in the middle (active, then nothing, then active again) reads as poor staffing practice even when each gap is individually justified by "no new story to test yet" — a real QA/SE engineer usually has legitimate prep, planning, or regression work to fill a short lull. Prefer picking ONE continuous start week (even if slightly later than the earliest theoretically-possible start) over a fragmented on/off schedule, and check whether the resulting single block still reconciles within the 10–25% overhead band — search a few candidate start weeks if needed, the same clean block often exists. Reserve a true mid-schedule gap for a lull too long to plausibly fill with real work (multiple weeks with nothing to prep, plan, or review).
- **Prefer 50% or 100% (20h/wk or 40h/wk) over smaller fractional rates where possible.** 4h/8h/16h weeks are legitimate for genuine light-touch governance (EL/DSL) or a short ramp, but for an execution role's steady-state engagement, a clean 50%/100% commitment reads as a real staffing decision rather than an oddly-specific fraction. When reconciling hours against committed story totals, search for a continuous window at 20h or 40h/wk before reaching for 8h/16h/wk steady-state rates.
- **All delivery roles are programme-scoped, not EPIC-scoped or feature-scoped.** Software Engineers and QA Engineers move between EPICs/features as work completes rather than being hired per EPIC or per feature — for these two roles that typically means spanning most or all of the programme timeline. DevOps is programme-scoped in the same "not per-EPIC" sense, but its *duration* is usually bounded to a front-loaded setup block rather than the full timeline (see DevOps Staffing Pattern) — one person covering the whole programme's infrastructure need is still correct even when that need doesn't last the whole programme. The sole exception is a programme with truly independent, concurrent delivery streams across distinct technical domains (e.g., a separate mobile track running in parallel), where dedicated per-stream staffing may be warranted.
- **QA Engineers are programme-scoped.** One QA Engineer covers all EPICs sequentially as features complete. Never assign one QA Engineer per EPIC — this is the most common over-staffing mistake in AP team allocations. A single web application with multiple backend EPICs does not justify multiple QA Engineers.
- **Role names must not embed EPIC IDs, feature IDs, or per-EPIC suffixes.** `"QA Engineer (S1)"` and `"SE Backend (S1)"` are correct. `"QA Engineer — E02 (PA)"` and `"Software Engineer Backend — E03 (PA)"` are wrong. Timing is communicated via `start`/`end` week numbers — encoding scope into the role label creates false parallelism and multiplies headcount on paper.
- **The parenthetical after a role name is the seniority title (J1–J3 / S1–S2 / L / SL), not the role abbreviation.** Write `"Engagement Lead (L)"` not `"Engagement Lead (EL)"`. EL, DSL, PA, PM are role abbreviations — they belong in the role name, not the parenthetical. The parenthetical is reserved for the AP seniority ladder position.
- **Engagement Lead is mandatory on every engagement, regardless of size.** Every team model includes a dedicated `Engagement Lead (L)` row — commercial/budget ownership is never folded into the Delivery Solution Lead or any other role, even on the leanest engagement. On a small programme, shrink EL's hours toward the 4h/wk floor (routine commercial oversight); do not remove the role or merge its responsibilities elsewhere.
- **On a lean team, consolidate Program Manager into Engagement Lead — not into Delivery Solution Lead.** EL and PM are both administrative/commercial-facing (budget & client relationship; ceremonies & coordination) and pair naturally into one row, e.g. `"Engagement Lead / Program Manager (L)"`. Delivery Solution Lead stays its own dedicated row even on the leanest engagement — it's the technical-leadership voice, a different skill set from PM's process work, and shouldn't be diluted by merging PM's hours into it. DSL's routine hours normally sit in the same 2–4h/wk (5–10%) band as EL's own — similar order of magnitude, though not necessarily identical — and a consolidation elsewhere is never on its own a reason to push DSL to the 8–16h+ escalated tier; escalate DSL only for the reasons already listed in the Role Allocation Constraints table above.
  - **On the leanest engagement shape (single continuous SE + one shared QA, no multi-person sprint ceremonies to run), the combined EL/PM row itself stays near the routine 2–4h/wk floor** — do not default to PM's standalone "16h for small stable teams" figure just because PM folded in. That 16h figure assumes real ceremony volume (multiple engineers, regular sprint events); a team this small doesn't generate it. Bump the combined row toward PM's standard range only when the delivery team is actually big enough to need it (2+ SE tracks, real sprint cadence).
- **Program Architect scales with engagement size — it is not always a separate headcount.** PA leads architecture decisions, requirements shaping, and backlog quality, and provides architectural supervision even over an S1 SE working with day-to-day independence — "minimal supervision" describes execution, not architecture. How PA gets staffed depends on scale:
  - **Lean engagement (the common small-project case):** PA is *combined* with the team's S2-level SE — one person, labelled e.g. `"Software Engineer / Program Architect (S2)"`, carrying both their track's execution and architecture leadership for the whole engagement. If the engagement only needs a solo SE, that SE defaults to S2 specifically to carry the PA hat (see Software Engineer (S2) row above) — this is the default pattern, not an exception to justify. Prefer this over folding PA-persona work into DSL: the person doing the technical work is naturally positioned to own the architecture decisions about that work, whereas DSL stays governance-only and shouldn't take on solutioning.
  - **Standard-size engagement:** PA is its own separate dedicated person (S2) — see Standard team shape below.
  - **Very complex / huge engagement (exceptional):** PA escalates to Lead title (fully separate from any SE), and DSL correspondingly escalates to Senior Lead — reserved for genuinely huge estimated complexity, not the default even on a mid-size engagement (see Minimum seniority by role).
- **Standard team shape for a typical AP engagement:** 1 EL (L) + 1 DSL (L) + 1 PA (S2) + 1 PM (S2) (programme leadership), 2–3 SE tracks (backend lead S2 + domain backend S1 + frontend S1), 1 QA Engineer (S1, shared), DevOps Engineer (S1, one front-loaded setup block, not full-time 40h throughout and not a second cutover window by default — see DevOps Staffing Pattern below). Justify any deviation explicitly.
- **Round to clean values only:** 4, 8, 16, 20, 40 hrs/week. Never use 32, 6, 12, 14, 18, 22, 24, 28 or similar.
- **EL and DSL are leadership, not execution.** Justify any EL allocation above 8h/wk or any DSL allocation above 16h/wk with a specific reason (active escalation, hands-on technical crisis, etc.).
- **A separately-staffed PA below 50% is a governance risk.** Below that threshold the role becomes a rubber-stamp reviewer — either justify why 50% is sufficient or increase to 100% where the work demands it. Doesn't apply to the combined SE/PA pattern above, where there's no separate PA allocation to measure.
- **When EL and PM are not the same person, PM sits at exactly 50% or 100% (20h or 40h) — never a lower fraction.** A standalone PM at 16h/wk (40%) or similar doesn't read as a real staffing decision the way EL/DSL's routine governance floor does — PM is sized for actual scrum-master and coordination throughput, not light-touch oversight. If the ceremony load genuinely doesn't justify 50%, that's a signal to consolidate PM into EL (see the lean-team consolidation rule above), not to leave a separate PM row under-allocated.

## SE Track Patterns for Parallel Delivery

When an EPIC has independently deliverable workstreams, structure SE capacity into named tracks rather than listing N identical "Software Engineer" rows. Name each track after the work it owns, not a letter.

| Track type | Duration | Purpose |
|------------|----------|---------|
| **Main** | Full EPIC duration | Core application layer. The anchor SE — starts in discovery, present through Hypercare at reduced hours. |
| **Domain** | Ends when its domain work is complete | A self-contained technical concern that another track builds on top of. Exits cleanly once its stories are done. |
| **Burst** | Short peak window only | Absorbs parallel capacity at the highest-intensity implementation point. Prefer rotating a person from a finishing EPIC or track over adding new headcount. |

**Default is one continuous SE track.** On a small engagement, a second track that only runs for part of the programme (e.g., a Domain track active weeks 1–3 while Main runs weeks 1–8) reads as arbitrary fragmentation — "one person for three weeks, a different person for the rest" — even when the underlying work genuinely is separable. Prefer one SE working through all features sequentially unless splitting demonstrably shortens the critical path by more than the coordination overhead it adds.

**When to add a track:** only when removing it would materially extend the EPIC's calendar duration and the work is genuinely independent (no shared state with other tracks during the same weeks). State the calendar-time savings explicitly when justifying a second track — "the domain is self-contained" is necessary but not sufficient; it must also be worth the fragmentation. When two tracks genuinely map to two fully independent EPICs (not a split of one EPIC's work), name each after the EPIC or capability it owns — not a letter — and let each run its own full EPIC start-to-finish, exiting cleanly when its own scope is done.

**Peak concurrent SE limit:** 6 concurrent SEs is a reasonable upper bound for a single programme. Above 6, favour sequencing EPICs over growing the team.

**Cross-EPIC rotation:** model a rotating SE as one continuous row in the programme summary — not separate rows per EPIC. One person, one row, spanning the full period they are active.

---

## Timeline and Capacity Planning Rules

### Buffer Weeks

Insert one **empty buffer week** for every 10 active working weeks in the delivery timeline (per EPIC) — week 11, 22, 33, 44, … Label these weeks "Buffer / Vacation / Sick Leave"; do not schedule stories, milestones, or demos into them. **This is a threshold rule, not a reflex** — a programme whose total active length (Discovery + Build + Hypercare, summed) doesn't reach 10 weeks gets no formula-driven buffer at all. Don't insert one right after Discovery "just in case"; if a specific named risk (e.g. access delays) needs covering on a short programme, prefer sizing organic slack into the phase that carries the risk (allocate somewhat more capacity than the bare minimum) over adding a dedicated unstaffed week.

For programmes with 3 or more concurrent EPICs, add at least **4 dedicated programme-level buffer weeks** on top of the per-EPIC buffers, placed at key transition points (after major EPICs close, before final deployment) and shown separately in the programme Gantt.

Buffer weeks are the only unscheduled capacity in the timeline. Protect them — do not collapse them when milestones slip.

### Hypercare Staffing

**Whenever Hypercare is in scope, staff at least one SE and one QA into it, at reduced/part-time hours — never zero engineering or QA presence.** A freshly-launched feature needs someone able to triage and fix a live defect and someone able to verify it, even if every committed story is otherwise done. This applies even when a track's committed work finishes cleanly before Hypercare starts (e.g. a Domain track, or the anchor/Main track once its build work is complete) — extend that person's row into Hypercare at a reduced clean rate (e.g. 20h/wk) rather than letting them exit with zero presence. On a multi-track engagement, the anchor/Main SE is the natural person to carry this, not a new hire.

### DevOps Staffing Pattern

**Default: DevOps is a front-loaded, bounded-scope role — one setup block (starting week 1 if there's already real work to act on, later if Discovery is still resolving what that work is), ending cleanly once environments/IaC/CI-CD are built and hardened against the first real deploys. It does not return later in the programme.** This is the same shape as a Domain SE track that "ends when its domain work is complete" — not the onboard→offboard→re-onboard anti-pattern, because there's no *return* to explain. Once the pipeline is self-service, the existing team (the SE who owns the track, or PA) runs it themselves for the rest of the programme, including Deployment cutover — that is what a good pipeline is for. Don't stand up a separate DevOps re-entry at Deployment "just in case"; if the team can self-serve the deploy, they should.

- **Anchor the setup block to when there's real, decided work to act on — not reflexively to week 1.** If the tech stack and vendors are already fixed going in, that's week 1: environments and CI/CD should be ready before the team starts building against them. But if Discovery itself is resolving the decisions the setup work depends on (which payroll provider, which push/SMS/email vendor, which cloud region), DevOps has nothing concrete to build against until those land — starting week 1 anyway just means idle allocation waiting on a dependency, which the Allocation Principles section already forbids for any role. In that case, the setup block legitimately starts near the *end* of Discovery, once the decisions it depends on are made.
- **Concentrate the setup block — don't spread it.** Prefer a short block that ramps to a brief full-rate push over a longer block held flat at half rate throughout, same as the general "concentrate capacity" principle. A common shape: one lighter week while the last Discovery decisions land (e.g. 20h), then one or two weeks at full 40h to actually build once there's something concrete to build against — not five weeks at a flat 20h just because the calendar has five weeks in it.
- **Let the setup block run a bit into early Implementation** if hardening against the first real builds/deploys genuinely takes that long — that's still one continuous block, not a second window. Size it to the real work, then let it end.
- **Escalate to a dedicated cutover-window re-entry only for a specific, named cutover risk the existing team genuinely can't self-serve** — e.g. a regulated/compliance-sensitive go-live, a first-time production credential/secrets cutover for a new external integration, or a complex multi-environment migration. State the specific risk; "cutover is always risky" isn't specific enough on its own to justify a second DevOps window.
- **A genuine ongoing infrastructure need during Implementation** (new environments as integrations are added, recurring secrets rotation) is a reason to extend the *same* continuous setup block's end date further into Implementation — not a reason to invent a separate light "middle" rate bridging to a cutover window that may not need to exist at all.

A DevOps block that starts or ends disconnected from an actual phase boundary or real milestone is a sign of arbitrary placement — anchor the start to when there's real decided work to act on (week 1, or the point in Discovery those decisions land) and the end to the point the pipeline is genuinely self-service, and justify anything beyond that single block explicitly.

**DevOps's hours are not free-floating overhead — they should be backed by real committed work in the estimate, the same as SE/QA.** `create-work-breakdown` deliberately has no "DevOps Engineer" story persona: infrastructure/CI/CD/environment work is meant to be written as Software Engineer stories (or Program Architect stories for the design/planning side — "deployment planning" is explicitly a PA-persona activity), and `create-estimate` explicitly lists "Environment setup (CI/CD, staging, infrastructure)" as a Dev Activity every estimate should already be budgeting, with a dedicated `infra` category for when it's the dominant shape of the work. So when a WBS is properly estimated, the environment/pipeline/deployment work DevOps will execute already exists as committed Software-Engineer-persona (or Program-Architect-persona) hours in the Stories sheet — it is not automatically unmatched.

- **Check the estimate for infra-shaped stories before assuming DevOps is pure overhead.** If the WBS has `infra`-category features or environment-setup-flavoured stories, size DevOps's allocation against that specific slice of the already-committed SE/PA hours, not as an unrelated number layered on top.
- **Watch for double-counting.** If those infra-shaped hours are already inside what the SE capacity is being sized to cover, and you then *also* add a full separate DevOps row for the same work, you've counted it twice. Either let the SE(s) execute those specific stories directly (no separate DevOps row — the default for modest infra scope, the same "combined" pattern PA uses on lean engagements), or size DevOps's row to the infra slice specifically and reduce what you expect SE capacity to cover by that same amount.
- **A DevOps allocation genuinely is unmatched overhead only when the estimate has a real gap** — no `infra`-category features, no environment-setup language anywhere in the Dev activities of the affected stories. When that's the case, the better fix is usually to go back and size that work properly in the estimate (it's real work; it should be committed work), not to wave it through as accepted overhead.

### Team Capacity Calculation

1. Establish contracted hours per person per week (typically 40h).
2. Apply the role allocation constraints above.
3. Do not double-count the ceremonies adjustment — it is already embedded in the AI+Human estimate. Use effective delivery capacity only for sprint planning and story assignment.

### Reconcile Allocated Hours Against Committed Story Hours

Before finalising any allocation, sum each role's committed story-persona hours from the Stories sheet / estimate (SE-tagged stories, QA-tagged stories, PA-tagged stories), then compare against the hours you're actually allocating that role across the whole programme.

**Target: allocated hours should track committed persona hours within roughly 10–25% overhead.** That margin covers real, nameable overhead — code review, ramp-up, a short defect-fix or standby window — not "whatever a clean-round-number weekly schedule happens to add up to." A role allocated 50%+ beyond its committed persona hours is a sign the schedule was built by rounding to convenient weekly blocks per phase rather than by checking the total against the actual work.

**DevOps has no exclusion flag in the export tool, and — done correctly — usually doesn't need one.** There is no `devops_capacity_excluded` config, so any DevOps row's hours always land in the delivery-capacity total (column C of the validation row). That's fine as long as the estimate actually committed the infra/environment/pipeline work DevOps will execute (see DevOps Staffing Pattern above on where that work lives and the double-counting risk) — in that normal case, DevOps's hours reconcile against real committed hours the same way SE/QA's do, just without a tidy separate line for it. Treat a DevOps allocation as genuinely unmatched overhead only when you've checked the estimate and confirmed the gap is real, not as a default assumption. Either way, don't inflate DevOps with an unneeded cutover-window re-entry to look thorough, and don't squeeze a genuinely-needed setup block down just to protect the overall percentage — staff by real, stated need and let the resulting number land where it lands.

Full worked examples of the failure modes (padded tails, double-booked validation work, double-counted reassignments, and a real tool limitation around leadership-role capacity exclusion) are in [`references/rationale.md`](references/rationale.md) — read it before reconciling an allocation, and especially before concluding a hand-calculated margin is what the generated workbook will actually show.

---

## Outputs

Three outputs, produced together at Phase 10 once estimates are finalised. Full templates for each are in [`references/examples.md`](references/examples.md) — read that file when you're ready to write the actual document.

| Output | Folder | Naming convention | When to produce |
|--------|--------|-------------------|------------------|
| Team model | `.project/estimates/` | `team-model-[project-slug]-v[N.N].md` | Always — includes the required Team Allocation Timeline Gantt diagram |
| Developer assignments | `.project/estimates/` | `developer-assignments-[project-slug].md` | Multi-EPIC programmes, or any engagement with more than 2 Software Engineers |
| Team allocation config | `.project/backlog/` | `team-allocation.yaml` | Always — the machine-readable config `_export_wbs.py` consumes to generate the Excel workbook |

---

## Quality Checklist

Before finalising a team model or allocation, verify:

- [ ] Every role name carries its seniority title in parentheses (`Engagement Lead (L)`), never the role abbreviation
- [ ] No role name embeds an EPIC ID, feature ID, or per-EPIC suffix
- [ ] Engagement Lead (L) is staffed as its own row in `programme_allocation`, even at 4h/wk — never merged into DSL or omitted, regardless of engagement size
- [ ] If leadership is consolidated on a lean team, Program Manager folded into Engagement Lead — never into Delivery Solution Lead, which stays its own row at the routine 2–4h/wk band regardless
- [ ] Program Architect responsibility is staffed somewhere at S2+ — either its own dedicated row (standard/complex engagements), or combined with the team's S2 SE (the lean-engagement default) — never quietly absorbed into DSL or an S1 SE with no seniority bump
- [ ] SE, QA, and DevOps roles are programme-scoped — one continuous row per person, never one row per EPIC (DevOps's row is normally shorter than the full programme — see DevOps Staffing Pattern — but is still one row, not split per EPIC)
- [ ] At least one SE and one QA are present (part-time is fine) during Hypercare if it's in scope — no track exits to zero engineering/QA presence
- [ ] DevOps defaults to one front-loaded setup block — anchored to week 1 if the stack/vendors are already decided, or to when Discovery's relevant decisions land if it's resolving them — concentrated (short and full-rate, not spread thin) and ending cleanly once the pipeline is self-service; no separate cutover-window re-entry unless a specific, named cutover risk justifies it (the existing SE/PA team runs Deployment via the pipeline otherwise)
- [ ] QA's start date does not have to be week 1 — but wherever it starts, QA goes straight to a clean full rate (20h/40h) with no multi-step ramp, and the 25% dev-capacity floor is checked from that onboarding point forward
- [ ] Exactly one shared QA Engineer unless genuinely concurrent independent streams justify more
- [ ] QA capacity is at least 25% of dev capacity on every active feature stream
- [ ] Every hour value is a clean figure: 4, 8, 16, 20, or 40
- [ ] Execution roles (SE, QA, DevOps) sit at 20h or 40h/wk only — a 4/8/16h steady-state rate on one of these is only valid as genuine light governance or a short ramp, never as a way to spread low committed hours across more weeks (shorten the window instead — see Reconciliation reference)
- [ ] EL ≤ 8 h/wk and DSL ≤ 16 h/wk unless a specific reason is stated; a lean-team combined EL/PM row stays near the same low floor unless real multi-person ceremony volume justifies more
- [ ] PA is at 50% or above, or the lower figure is justified
- [ ] If PM is staffed as its own row (not consolidated into EL), it sits at exactly 20h (50%) or 40h (100%) — never 16h or another fraction
- [ ] No junior is the sole engineer in a phase, and every junior is paired with S1+
- [ ] Nobody is modelled as allocated but idle waiting on a dependency
- [ ] Peak concurrent SEs ≤ 6, or sequencing was considered and rejected with a reason
- [ ] A buffer week only exists if the programme's total active length reaches the 10-active-week threshold — not inserted reflexively after Discovery
- [ ] `team-allocation.yaml` `ai_factor` matches the factor confirmed in the estimate
- [ ] `programme.buffer_weeks` is the union of per-EPIC buffers (programme-absolute) plus programme-level buffers
- [ ] No effort figures are re-derived here — person-days come from the estimate
- [ ] Allocated hours reconcile against committed story-persona hours within ~10–25% overhead (see Reconciliation reference for how to check this properly)
