---
name: workflow:refine
description: "Phase 5: Generate executable stories. Convert work units into stories ready for the proof loop."
argument-hint: <feature-name> [unit-name]
---
# Skill: /workflow:refine

**Phase 5 — Generate Executable Stories**

Convert each work unit into executable stories. This is the last engineer decision point before the proof loop runs. Stories approved here are handed to an implementation agent or Ralph TUI to execute autonomously.

---

## When to Use This Skill

Use `/workflow:refine` when:
- Work units are approved in `04-decompose/units.md`, OR
- Scope is fully understood and you are ready to generate stories directly (flexible entry point)

If entering directly without prior phases, ask the engineer to describe the unit scope and exit criteria — these replace `units.md` for the current session.

---

## Inputs

- Feature name
- Optionally: a specific unit name to refine (defaults to all pending units)

---

## Step 1 — Bootstrap Workspace

Follow `skills/_shared/workspace.md`.

Load:
- `<feature>/04-decompose/units.md` — unit scope, entry/exit criteria, dependencies
- `<feature>/03-design/design.md` — architectural constraints
- `<feature>/01-explore/` — coding standards, patterns, testing conventions (`patterns.md`, `testing.md`, `standards.md`)

For each unit to be refined, create if not present:
```
<feature>/05-refine/<unit-name>/
<feature>/00-status/05-refine/<unit-name>/
```

Set `<feature>/00-status/05-refine/<unit-name>/status.md` to `status: started`.

If `05-refine/<unit-name>/stories.json` already exists, read it and report the count of locked stories. Ask: add new stories, or replace all?

---

## Step 2 — Unit Risk Assessment

For each unit, before generating stories, decide the implementation approach:

Ask:
1. "For [unit-name]: is the riskiest part the integration between layers, or the correctness of the logic within a single layer?"
   - If integration: **tracer-bullet** — implement the thinnest end-to-end slice first.
   - If logic: **layered** — implement the most complex internal layer first with full tests before wiring.

2. "How tightly do you want to constrain the implementation — should stories specify the exact approach, or leave implementation choices to the agent?"
   - **Prescriptive:** stories include specific method names, data structures, or patterns to follow.
   - **Outcome-focused:** stories define what must be true, not how to achieve it.

Record the approach for each unit. It shapes story granularity.

---

## Step 3 — Standards Check

Before generating stories, ask:
> "Are there any coding conventions, patterns, or standards for this unit that are NOT already in `01-explore/`? If so, I'll factor them in before locking stories."

If the engineer provides new standards:
- Add them to `<feature>/01-explore/standards.md` before generating stories.

---

## Step 4 — Parallel Story Generation

Units with no dependencies on each other can have their stories generated in parallel. Use the Agent tool to run one agent per unit. Each agent:

1. Reads the unit scope and exit criteria from `units.md`
2. Reads the architecture diagram from `design.md`
3. Reads `01-explore/` for applicable standards (`patterns.md`, `testing.md`, `standards.md`)
4. Generates stories in the format below
5. Returns the story set for engineer approval — does not lock them

The orchestrating session collects all story sets and presents them for approval.

---

## Step 5 — Story Approval Loop

For each unit's story set:

1. Present the stories grouped by unit.
2. For each story, ask: "Is this story clear, correctly scoped, and complete enough for an agent to implement autonomously?"
3. If yes: mark `"approved": true`. If no: revise before proceeding.
4. After all stories in a unit are approved, lock them (`"locked": true`).

**Engineer approval is required before locking.** A locked story is not re-opened — issues found later become new stories.

---

## Step 6 — Write stories.json and prompt.md

For each unit, write two files:

### stories.json

```json
{
  "unit": "<unit-name>",
  "feature": "<feature-name>",
  "approach": "tracer-bullet | layered",
  "generated": "<ISO date>",
  "stories": [
    {
      "id": "S01",
      "title": "<short title>",
      "as_a": "<role>",
      "i_want": "<goal>",
      "so_that": "<reason>",
      "acceptance": [
        "<condition 1 — observable, testable>",
        "<condition 2>"
      ],
      "notes": "<implementation hints, constraints, or patterns to follow>",
      "approved": true,
      "locked": true,
      "passes": false
    }
  ]
}
```

**Story rules:**
- Each story has a single, coherent goal.
- Acceptance conditions are observable and testable — not "works correctly" but "returns HTTP 200 with body matching schema X."
- `passes` starts as `false`. It is set to `true` by the implementation agent when the acceptance conditions are verified.
- Locked stories are never edited. New stories are added as new objects with incremented IDs.

### prompt.md

A lean, mission-focused execution prompt for this unit:

```markdown
# Execution Prompt: <unit-name>

## Mission

[One sentence: what this unit achieves.]

## Stories

Read `stories.json` in this directory. Implement each story in order. Mark `passes: true` when acceptance conditions are verified. Do not modify locked stories — add new ones if issues arise.

## Context

Load from the filesystem before starting:
- `../../01-explore/explore.md` — component map
- `../../03-design/design.md` — architectural constraints
- `../../01-explore/` — standards and patterns (`patterns.md`, `testing.md`, `standards.md`)

## Constraints

- [Any unit-specific constraints not in `01-explore/`]

## Definition of Done

All stories in this unit have `passes: true` and the unit exit criteria from `../../04-decompose/units.md` are satisfied.
```

---

## Step 7 — Status Update

After all units are locked:
1. Update each `00-status/05-refine/<unit-name>/status.md` to `status: complete`.
2. Update `00-status/05-refine/status.md` aggregate.
3. Update `<feature>/00-status/status.md`.
4. State next step: "Stories are locked. Run the implementation loop for each unit using the `prompt.md` in `05-refine/<unit-name>/`. Units with no dependencies can run in parallel."

---

## Rules

- **Engineer approval before locking.** No story is locked without explicit confirmation.
- **Acceptance conditions are testable.** Vague conditions ("works as expected") are not accepted — ask the engineer to make them concrete.
- **Locked stories are immutable.** Issues discovered during implementation become new stories, not amendments to locked ones. This preserves traceability.
- **prompt.md is lean.** It is not a planning document. Context loads from the filesystem. The prompt is the mission statement and the pointer to the stories.
- **Units without dependencies run in parallel.** Maximise parallel execution — it is one of the primary leverage points of this workflow.
