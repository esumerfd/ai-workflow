---
name: workflow:build
description: "Build a workstream: generate stories, execute red/green/refactor loop, prove completion, challenge the engineer, archive patterns."
argument-hint: <feature-name> [<phase>/]<workstream-name>
---
# Skill: /workflow:build

**Build — Stories, Proof, Challenge, Archive**

Build one workstream at a time. Reads `definition.md` produced by `plan`, generates executable stories, runs the red/green/refactor loop, proves completion, challenges the engineer on what was built, and closes with an archive ritual.

- `/workflow:build <feature-name> <workstream-name>` — for features with no phases
- `/workflow:build <feature-name> <phase>/<workstream-name>` — for phased features

---

## Inputs

- Feature name
- Workstream path relative to the feature root: `<workstream>` or `<phase>/<workstream>`

If either is missing, ask before starting. If the workstream directory or `definition.md` does not exist, stop and say: "Run `/workflow:plan <feature-name> <phase>` first to create workstream definitions."

---

## Step 1 — Bootstrap Workspace

Follow `skills/_shared/workspace.md`.

Resolve the workstream directory:
- With phase: `<phase>/<workstream>/`
- Without phase: `<workstream>/`

Load before proceeding:
- `<workstream>/definition.md` — scope, exit criteria, end-to-end proof description, constraints
- `plan.md` — terrain map and architectural constraints
- `patterns.md`, `testing.md`, `standards.md` — project conventions

If `<workstream>/stories.md` already exists:
- Read it and report the count of locked stories.
- Ask: "Add new stories, or replace all?"

---

## Step 2 — Standards Check

Before generating stories, ask:
> "Are there any coding conventions, patterns, or standards for this workstream that are NOT already captured in `patterns.md` or `standards.md`? If so, I'll add them before generating stories."

If the engineer provides new standards:
- Add them to `standards.md` before proceeding.

---

## Step 3 — Story Generation

Generate stories from `definition.md`. Stories translate the exit criteria and end-to-end proof description into discrete, provable units of work.

**Story approach — default to end-to-end:**

Read the workstream's risk profile from `definition.md`:
- **tracer-bullet:** integration risk is dominant. Generate the thinnest end-to-end slice first — a story that exercises the full path from input to observable output, even if it does little. Subsequent stories add behaviour to this skeleton.
- **layered:** logic complexity is dominant, and a full E2E slice is not yet feasible. Layer stories build the most complex internal logic first, with a final story that wires the layers.

For any workstream, the first question is always: "Can we write a story that proves end-to-end behaviour?" If yes, that story comes first regardless of risk profile.

**Story format (stories.md):**

Write `<workstream>/stories.md`:

```markdown
# Stories: <workstream-name>

**Feature:** <feature>
**Phase:** <phase> (omit if no phase)
**Workstream:** <workstream>
**Approach:** tracer-bullet | layered
**Generated:** <ISO date>

---

## S01 — <short title>

**As a** <role>
**I want** <goal>
**So that** <reason>

**Acceptance:**
- [ ] <condition 1 — observable and testable, not "works correctly">
- [ ] <condition 2>

**Notes:** <implementation hints, patterns to follow, constraints>
**Approved:** false
**Locked:** false
**Passes:** false

---

## S02 — <short title>
...
```

**Story rules:**
- Each story has a single, coherent goal.
- Acceptance conditions are observable and testable at the system boundary — not at the internal layer boundary. "Returns HTTP 200 with body matching schema X" is acceptable; "service method returns correct value" is not.
- Acceptance conditions map directly to the exit criteria in `definition.md`.
- `Passes` starts as `false`. It is set to `true` when acceptance conditions are verified.
- Locked stories are never edited. New stories are added with incremented IDs.

---

## Step 4 — Story Approval

Present the story set to the engineer:

For each story:
1. "Is this story clear, correctly scoped, and complete enough to implement autonomously?"
2. If yes: set `Approved: true`. If no: revise before proceeding.

After all stories are approved, lock them (`Locked: true`).

**Engineer approval is required before any story is locked.** A locked story is not re-opened — issues found later become new stories.

---

## Step 5 — Choose Execution Path

Ask:
> "How would you like to execute these stories — should I implement them directly here in Claude, or will you run them through ralph-tui?"

**Claude-native:** proceed to Step 6. Claude reads `stories.md` and drives the loop.

**ralph-tui:** convert `stories.md` to `stories.json` first, then hand off.

To convert, generate `<workstream>/stories.json` from `stories.md`:

```json
{
  "unit": "<workstream-name>",
  "feature": "<feature-name>",
  "phase": "<phase-name>",
  "approach": "tracer-bullet | layered",
  "generated": "<ISO date>",
  "stories": [
    {
      "id": "S01",
      "title": "<short title>",
      "as_a": "<role>",
      "i_want": "<goal>",
      "so_that": "<reason>",
      "acceptance": ["<condition 1>", "<condition 2>"],
      "notes": "<hints>",
      "approved": true,
      "locked": true,
      "passes": false
    }
  ]
}
```

`stories.json` is always derived from `stories.md` — never hand-edited. If `stories.md` changes, regenerate `stories.json`.

If using ralph-tui: stop here and say "stories.json is ready. Run ralph-tui against `<workstream>/stories.json`. Return here for the proof and challenge steps when all stories pass."

---

## Step 6 — Execution Loop (Claude-native)

Work through each story in order. For each:

**Red:** Write the failing proof first. The proof must exercise the acceptance conditions at the system boundary — not internal units in isolation.

**Green:** Implement the minimum code to make the proof pass. Do not implement beyond what the story requires.

**Refactor:** If the passing implementation has obvious duplication or structural debt, address it now. The proof must still pass after refactoring.

**Lock:** Set `Passes: true` in `stories.md` for this story.

Do not start the next story until the current one passes.

If a story cannot be implemented as written (the definition was wrong, a dependency is missing, or an assumption failed):
- Stop.
- Record the issue in `stories.md` as a note on the relevant story.
- Ask the engineer: "This story cannot be completed as written because [reason]. Should we: (a) revise this story, (b) route back to plan at Level 2 to revise the workstream definition, or (c) route back to Level 1 to revise the architecture?"

---

## Step 7 — Proof of Completion

After all stories pass:

1. Run the full proof suite for this workstream.
2. Run the proof suite for any workstream this one depends on (regression check).
3. Confirm: "All [n] stories pass and no regressions detected." If regressions are found, add new stories to address them before proceeding.

---

## Step 8 — Challenge

Before closing the workstream, challenge the engineer's understanding of what was built. This is collaborative — not adversarial.

Ask to orient the walkthrough:
1. "Are there parts of this implementation you already understand well and want to skip?"
2. "Are there specific areas — coupling points, edge cases, judgment calls — you want to walk through in detail?"

Walk through the implementation with focus on:
- **Coupling points:** where this code calls out to, or is called by, another component
- **Edge cases and error paths:** what happens when inputs are invalid or external services fail
- **Judgment calls:** where the implementation made a non-obvious choice

For each section, describe what the code does in plain terms. Ask: "Does this match your mental model?"

**Test coverage mapping (mandatory):**

For each acceptance condition in `stories.md`, map it to a test:
> "Acceptance condition: [condition]. Which test verifies this?"

- **Covered:** note the test name/location.
- **Not covered:** this is a finding — record it and resolve before closing.

**Certification question:**

> "You've walked through this workstream. Do you feel you own this code — could you answer questions about it in a PR review and defend its correctness?"

If no: identify the specific area and walk through it again. Do not accept a pro-forma yes.

---

## Step 9 — Archive Ritual

After the challenge is complete, close the workstream by extracting what was learned.

**Exit interview (three questions):**

1. **Surprise:** "What did you learn during this workstream that you did not expect?"
2. **Recurrence:** "If you did this workstream again, what would you tell someone starting a similar one?"
3. **Standards:** "Did this workstream surface any conventions that should constrain every future implementation in this area?"

**Pattern extraction:**

For each candidate raised in the exit interview or walkthrough, apply the generalizability test:
> "Would this pattern prevent a failure or maintain consistency in a future, unrelated workstream — not just this one?"

If yes: promote. If no: discard.

**Explore updates:**

Ask: "Did we discover anything that corrects or extends the terrain map in `plan.md`?"

If yes:
- Update `plan.md` terrain section with the correction.
- Update `patterns.md`, `testing.md`, or `standards.md` as appropriate.

**CLAUDE.md promotions:**

For patterns confirmed as promotable to CLAUDE.md:
1. Draft the addition.
2. Present the full diff to the engineer: "Here is what I propose to add to CLAUDE.md. Review each addition — we're only promoting what you'd want every future session to follow."
3. Confirm each addition before writing.

---

## Step 10 — Write proof.md

Write `<workstream>/proof.md`:

```markdown
# Proof: <workstream-name>

**Completed:** <ISO date>
**Stories:** <n> stories, all passing
**Status:** Complete | Complete with notes

---

## Story Coverage

| Story | Acceptance Condition | Test | Status |
|-------|---------------------|------|--------|
| S01   | [condition] | [test name / location] | Covered |

## Non-Obvious Implementation Notes

### <Section>
[Plain-language explanation for a PR reviewer who has not read the stories or plan.]

## Coupling Points

| This code | Calls / Is called by | What breaks if it changes |
|-----------|---------------------|---------------------------|

## Flags for PR Reviewers

- **[Flag]:** [what to look at and why]

## CLAUDE.md Promotions

| Pattern | Category | Promoted |
|---------|----------|---------|
| [pattern] | testing | Yes / No |

## Patterns Promoted to Explore

| File | What was added |
|------|---------------|
| patterns.md | [description] |

## Lessons Learned

- [lesson from exit interview]

## Deferred Items

- [ ] [item deferred and rationale]
```

---

## Step 11 — Status Update

1. Update `status.md` to reflect the completed workstream.
2. If all workstreams in the phase are complete: "All workstreams in [phase] are complete. Run `/workflow:build <feature> <phase>/<next-workstream>` for the next workstream, or open your PR."
3. If this was the last workstream across all phases: "Feature [name] is complete. All workstreams proven. Ready to open the PR."

---

## Rules

- **End-to-end first.** The first story should always attempt to prove end-to-end behaviour. Layer isolation is the fallback, not the default.
- **Red before green.** Write the failing proof before implementing. Never implement first.
- **Stories approved before locking.** No story is locked without explicit engineer confirmation.
- **Acceptance conditions are testable at the system boundary.** Vague conditions are not accepted — ask the engineer to make them concrete.
- **Locked stories are immutable.** Issues during implementation become new stories, not amendments.
- **Challenge is mandatory.** Every workstream is challenged before it is closed. A workstream the engineer cannot explain is not done.
- **Archive is not optional.** The exit interview and pattern check happen every time. Institutional memory is never skipped because it feels like overhead.
- **stories.json is derived.** Never hand-edit it. Regenerate from `stories.md` if needed.
