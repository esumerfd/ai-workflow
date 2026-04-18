---
name: workflow:review
description: "Phase 6: Engineer acceptance. Evaluate what was built against spec and route findings back to the right phase."
argument-hint: <feature-name> [unit-name]
---
# Skill: /workflow:review

**Phase 6 — Engineer Acceptance**

Human judgment on what was built. "Did the story pass?" is a mechanical check. "Is this truly done and ready to ship?" is the question this phase answers.

---

## When to Use This Skill

Use `/workflow:review` when:
- Implementation is complete for one or more units (all stories have `passes: true`)
- You want a structured acceptance review before certifying and opening a PR

---

## Inputs

- Feature name
- Optionally: a specific unit name (defaults to all units with `passes: true`)

---

## Step 1 — Bootstrap Workspace

Follow `skills/_shared/workspace.md`.

For each unit being reviewed, create if not present:
```
<feature>/06-review/<unit-name>/
<feature>/00-status/06-review/<unit-name>/
```

Set `<feature>/00-status/06-review/<unit-name>/status.md` to `status: started`.

Load before doing anything else:
- `<feature>/05-refine/<unit-name>/stories.json` — what was supposed to be built
- `<feature>/04-decompose/units.md` — entry/exit criteria for the unit
- `<feature>/03-design/design.md` — the architecture that was intended
- The actual git diff for the unit's changes

**Read the diff first.** Do not ask the engineer questions until you have read the actual code changes. Observations must be grounded in the implementation, not in generic expectations.

---

## Step 2 — Parallel Unit Analysis

For each unit being reviewed, run a specialist agent to analyse independently. Each agent:

1. Reads the unit's `stories.json` and exit criteria
2. Reads the actual diff/changed files
3. Compares the implementation against the architecture diagram in `design.md`
4. Produces a structured observation set (not findings yet — observations first)

Use the Ralph Wiggum technique (`skills/_shared/ralph-wiggum.md`): plain observations, confirmed before becoming findings.

The orchestrating session collects all observation sets.

---

## Step 3 — Observation Confirmation Loop

Present each observation to the engineer using the Ralph Wiggum technique:

1. State the observation plainly: "I see that [component X] now calls [component Y] directly, bypassing the interface defined in the design diagram. Is that intentional?"
2. Wait for confirmation.
3. If confirmed as intentional: record as a design deviation (note, not a bug).
4. If confirmed as unintentional: record as a finding.
5. If the engineer says "that's how it should have been designed": note it and route to design, not back to implementation.

Do not batch observations. One at a time, confirmed before moving on.

---

## Step 4 — Exit Criteria Check

For each unit, evaluate each exit criterion from `04-decompose/units.md` explicitly:

> "The exit criterion for [unit] is: [criterion]. Based on the implementation, is this satisfied?"

For each criterion:
- **Satisfied:** record as confirmed.
- **Partially satisfied:** record as a finding with specific missing condition.
- **Not satisfied:** record as a finding with category and routing.

---

## Step 5 — Story Coverage Check

For each story in `stories.json`:

1. Is the story's primary acceptance condition demonstrably met by the implementation?
2. Are there acceptance conditions that were passed mechanically but are not actually correct (false passes)?

Flag any story where the `passes: true` state appears to have been set without genuine verification. These become findings.

---

## Step 6 — Diagram Comparison

Compare the implemented structure against the architecture diagram in `design.md`:

1. Are all new components present as designed?
2. Are all modified interfaces changed as designed?
3. Are there any new dependencies not shown in the design diagram?

State deviations as observations before classifying them. Deviations are not automatically bugs — they may be intentional improvements or corrections to the design. Engineer decides.

---

## Step 7 — Finding Classification and Routing

For each confirmed finding, classify and propose a route:

| Category | Definition | Route |
|----------|------------|-------|
| Bug | Implementation behaves contrary to a confirmed acceptance condition | `/workflow:refine` — new story in the affected unit |
| Missing functionality | A story's acceptance condition was not implemented | `/workflow:refine` — new story |
| Wrong work boundary | The implementation reveals the unit boundary was in the wrong place | `/workflow:decompose` — revise units |
| Context map was wrong | The implementation reveals `explore.md` contained an incorrect observation | `/workflow:explore` — correct the map |
| Architecture violated | The implementation diverges from `design.md` in a way the engineer did not intend | `/workflow:design` — revise ADR or `/workflow:refine` — fix implementation |
| Improvement opportunity | Something that works but could be better — not a blocker for shipping | Record for archive phase |

Present the proposed routing for each finding to the engineer. Engineer decides the final routing — not the AI.

---

## Step 8 — Pattern Identification

Look for patterns across findings within and across units:

- Are multiple findings related to the same coupling or fragility flagged in `explore.md`?
- Are there recurring violations of standards in `context.d/`?
- Are there new conventions emerging that should be promoted to `context.d/`?

Flag recurring patterns for promotion. These are the most valuable output of the review phase — they prevent future recurrence.

---

## Step 9 — Write findings.md

Write `<feature>/06-review/<unit-name>/findings.md` for each unit:

```markdown
# Review Findings: <unit-name>

**Session:** <ISO date>
**Unit status:** Accepted | Accepted with findings | Rejected — re-enter at [phase]

---

## Summary

[One to three sentences: what was built, overall quality assessment, blocking issues count.]

## Exit Criteria

| Criterion | Status | Notes |
|-----------|--------|-------|
| [criterion from units.md] | Satisfied / Partial / Not satisfied | [detail] |

## Findings

### F01 — <short title>
**Category:** Bug | Missing functionality | Wrong boundary | Wrong context | Architecture violated | Improvement
**Story:** S[n] (if applicable)
**Observation:** [plain description of what was seen]
**Route:** [phase to route to] — [specific action]
**Engineer decision:** [confirmed / deferred / accepted as-is]

[Repeat for each finding]

## Diagram Deviations

| Deviation | Intentional? | Action |
|-----------|-------------|--------|
| [description] | Yes / No | [note / fix / update design] |

## Patterns Flagged for Promotion

- [pattern description] → promote to `context.d/[file]`

## Accepted Improvements (non-blocking)

- [improvement description] — deferred to archive
```

---

## Step 10 — Certification and Status Update

> "Here are the findings. Do you accept this unit as complete — or do any findings need to be resolved before we can move to certify?"

If accepted:
1. Update `<feature>/00-status/06-review/<unit-name>/status.md` to `status: complete`.
2. Update `<feature>/00-status/status.md`.
3. State next phase: `/workflow:certify <feature-name>`.

If rejected:
1. Update status to `status: blocked`.
2. Record the blocking finding and its routing target.
3. Confirm: "I'll route [finding F01] back to [phase]. When that's resolved, re-run `/workflow:review` for this unit."

---

## Rules

- **Read the diff first.** Observations must be grounded in what was actually built, not in generic expectations.
- **Observations before findings.** Use the Ralph Wiggum technique. Confirm before classifying.
- **Engineer routes, not the AI.** The AI proposes routing. The engineer decides.
- **Route to the earliest wrong phase.** A wrong context map routes to explore, not design. A wrong design routes to design, not refine. The earliest phase whose artifact was incorrect gets corrected first.
- **Patterns are as valuable as findings.** A recurring pattern is more important than any single finding — surface and flag for promotion.
