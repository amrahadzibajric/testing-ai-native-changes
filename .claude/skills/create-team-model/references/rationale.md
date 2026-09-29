# Hour Reconciliation — Worked Examples and Known Pitfalls

Supporting detail for `../SKILL.md`'s "Reconcile Allocated Hours Against Committed Story Hours" rule. Read this when actually reconciling an allocation, or when a reconciliation looks wrong and you need to know which known failure mode it is.

## Worked check: the padded-tail pattern

If an SE's committed hours are 300h and you've allocated 400h (33% over) with a multi-week post-build "standby" tail contributing most of that gap, that tail is the problem — cut it to what real defect-fix/monitoring risk justifies (often folds into a single short Hypercare week already staffed by DSL/QA) rather than carrying a redundant SE presence alongside them.

## Do not double-book already-counted validation work

If QA's committed persona hours for a Feature already include its E2E/integration test execution (check the story descriptions), don't also staff a separate multi-week "Sign-off" or "hardening" phase at similar hours after Build completes — that duplicates work already priced into the Build/V&V window. A short, explicitly-scoped Sign-off window is fine when it covers genuinely separate activity (UAT support, final regression pass) — say what that activity is, don't default to it.

## Keep post-build tails short and light

A "standby" or "wind-down" allocation for a role whose committed story hours are already fully delivered should be sized to real remaining risk (defect triage, Hypercare monitoring), typically days not weeks, and should shrink the overall programme rather than being tacked on as extra weeks by default.

## Reassigning execution across roles: count each hour exactly once

When you reassign a story's execution to a different role than its Stories-sheet persona tag (e.g., DSL absorbs a PA-tagged story, or an SE absorbs a QA-tagged Discovery-prep story), count that story's hours toward exactly ONE role's committed target — the role that actually executes it — never both.

A recurring mistake: reassigning a story's execution to Role B but leaving its hours in Role A's "committed target" reference *as well as* adding them to Role B's target double-counts the same hours across two roles' reconciliation checks, making both look better-margined than they really are and masking a real gap if one role is later re-examined alone.

Before finalising, add up every role's stated committed target and confirm the sum equals the Stories-sheet total exactly once — no story's hours should appear in two roles' target math, and no story should be missing from all of them.

## Don't dilute an execution role's rate to force reconciliation

If a role's committed persona hours are too low to sustain 20h/wk or 40h/wk across the span you first sketched, the fix is a **shorter, later, or otherwise more concentrated active window** — not an oddly-specific weekly rate (8h/wk sustained for a multi-week steady state, 14h/wk, etc.) invented to make the total divide out evenly. A QA row with 204h committed spread thin across 15 weeks at a diluted rate "reconciles" on paper but violates the clean-value rule and reads as a rate picked backward from a target percentage, not a real staffing decision. Solve it from the other direction: pick 20h or 40h/wk first, then work out how many weeks of that rate the committed hours actually justify, and start the row later (or end it earlier) to fit — the same "concentrate capacity, don't spread it" principle from the main allocation rules, applied specifically to reconciliation.

## Tool limitation: leadership-role capacity is excluded per-role, not per-week

The Team Allocation sheet's capacity validation excludes a leadership role (EL/DSL/PM) by name, for every week that role appears — there is no way to count only *some* of its weeks toward capacity.

If a leadership role takes a one-time escalation to cover real committed hours for a single week (e.g., DSL absorbing a week of Discovery-phase execution), that week's hours will NOT appear in the validation's capacity total even though they should — the tool has no per-week granularity for this exclusion, only per-role.

Don't hand-calculate a "combined capacity" figure that credits those hours and present it as what the sheet shows; it isn't. After building any allocation, **regenerate the actual workbook and read the real validation row** rather than trusting a hand cross-check — this has produced a wrong number more than once. If the true margin (what the tool actually computes) is uncomfortably thin because of this exclusion, size the escalation with extra headroom rather than relying on a credit the validation can't see — or better, avoid the exclusion entirely by routing the reassigned hours to a role the tool does count (an SE or QA row) instead of a leadership row.

## Tool limitation: `pa_capacity_excluded` only trims the story-hours side, never the capacity side

The flag filters PA-tagged rows out of the *committed story hours* formula (`Stories!$E:$E,"<>PA"` in the SUMIF) — it does **not** remove a dedicated "Program Architect" row from the *delivery capacity* sum. `_export_wbs.py`'s capacity-row exclusion list only covers Engagement Lead / Delivery Solution Lead / Program Manager by name; Program Architect is never in it. So a team model with its own dedicated PA row will have that row's full hours counted toward delivery capacity in the generated workbook **regardless of how this flag is set**. The flag only matters for the (different) case where PA has *no* dedicated row at all — its work is folded into an excluded leadership role's oversight time — where it keeps the numerator from crediting PA-tagged story hours that no capacity row is actually backing.

A team-model doc that hand-writes "Delivery capacity = SE + QA + DevOps (PA excluded)" while a dedicated PA row exists in `programme_allocation` is describing a number the tool will not actually produce — this has happened. Same fix as the leadership-exclusion pitfall above: regenerate the workbook and read the real total rather than asserting what the flag "should" do.

A related but separate point: whether PA even *has* committed persona hours to reconcile against is a modeling choice, not a fact about the role. `create-work-breakdown`'s persona table supports Program-Architect-persona stories (architecture design, integration design, deployment planning) — a WBS that writes some is reconciling PA the normal way. A WBS that models PA as pure oversight with zero PA-tagged stories will show PA's hours as unmatched, but that's a consequence of that WBS, not a property of the PA role in general — don't generalize "PA is usually unmatched" from one engagement's choice.

## DevOps is not "pure overhead by construction" — check the estimate before assuming it is

There's no DevOps story persona — `create-work-breakdown` explicitly bars it, and routes infrastructure/CI-CD/environment work into Software-Engineer-persona stories (or Program-Architect-persona for the design/planning side) instead. `create-estimate` lists "Environment setup (CI/CD, staging, infrastructure)" as a Dev Activity every estimate should already budget, with a dedicated `infra` category. So the work DevOps executes usually already exists as committed hours in the Stories sheet — just filed under SE or PA, not under a DevOps tag.

Before writing off a DevOps allocation as unmatched overhead, check the WBS for `infra`-category or environment-setup-flavoured stories. If they exist, size DevOps's hours against that specific slice of the already-committed SE/PA total (see the main SKILL.md's DevOps Staffing Pattern section) — and make sure the SE capacity figure used elsewhere doesn't also claim credit for the same slice; that's double-counting the same committed hours under two different rows. Only treat DevOps hours as genuinely unmatched when you've checked and the estimate really has no infra-shaped work behind them — and in that case, the better fix is usually to go back and size that work into the estimate properly, not to accept the gap.
