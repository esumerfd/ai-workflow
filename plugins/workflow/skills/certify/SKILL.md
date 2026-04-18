---
name: workflow:certify
description: "Phase 7: Pre-PR walkthrough. Engineer internalizes the implementation before putting their name on the PR."
argument-hint: <feature-name> [unit-name]
---
# Skill: /workflow:certify

**Phase 7 — Ownership Before PR**

The engineer internalizes the implemented code before putting their name on the PR. Knowledge transfer and quality catch in one pass. Collaborative, not adversarial.

---

## When to Use This Skill

Use `/workflow:certify` when:
- Review is complete for all units (`06-review/*/findings.md` exist with no unresolved blocking findings)
- You are preparing to open a PR

---

## Inputs

- Feature name
- Optionally: a specific unit name to certify (can certify units independently)

---

## Step 1 — Bootstrap Workspace

Follow `skills/_shared/workspace.md`.

For each unit being certified, create if not present:
```
<feature>/07-certify/<unit-name>/
<feature>/00-status/07-certify/<unit-name>/
```

Set `<feature>/00-status/07-certify/<unit-name>/status.md` to `status: started`.

Load before starting:
- `<feature>/05-refine/<unit-name>/stories.json` — what was intended
- `<feature>/06-review/<unit-name>/findings.md` — what was found and resolved
- `<feature>/03-design/design.md` — the architecture diagram
- The actual code changes (git diff or specific files)

Read the code. Do not rely on what was described in earlier phases — read what is there now.

---

## Step 2 — Walkthrough Strategy

Not all code needs the same depth of walkthrough. Focus time on what will matter most for ownership and PR review. Ask:

1. "Are there parts of this implementation you already understand well and want me to skip?"
2. "Are there specific areas — edge cases, coupling points, judgment calls — that you want to walk through in detail?"

Plan the walkthrough accordingly:
- **Skip:** boilerplate, obvious CRUD, framework scaffolding
- **Skim:** standard patterns already confirmed in context.d/
- **Deep walk:** coupling points, error paths, edge cases, anything that diverged from the design

---

## Step 3 — Collaborative Walkthrough

Walk through the implementation collaboratively. The tone is: "Here's what I noticed" — not "this doesn't meet standards."

For each section:

1. Describe what the code does in plain terms.
2. Note what is non-obvious: why a specific approach was taken, what edge case it handles, what it depends on.
3. Ask: "Does this match your mental model of how this should work?"
4. If yes: proceed.
5. If no: that is a finding — note it and decide whether it routes to a fix or to an explanation.

### Focus areas (in order):

**Coupling points:** Any place this code calls out to, or is called by, another component. These are the most important for PR reviewers to understand.

**Edge cases and error paths:** What happens when inputs are invalid, external services fail, or data is in an unexpected state?

**Judgment calls:** Places where the implementation made a non-obvious choice. Why was this approach taken over the obvious one?

**Test coverage:** Walk through the test cases. Does each acceptance condition from `stories.json` have a corresponding test?

---

## Step 4 — Test Coverage Mapping

This is mandatory, not optional.

For each story in `stories.json`, map each acceptance condition to a test:

> "Acceptance condition: [condition]. Which test verifies this?"

For each condition:
- **Covered:** note the test name/location.
- **Not covered:** this is a finding — a test gap.

Test gaps identified here are treated as findings. They can be resolved by:
- Writing the missing test now (preferred)
- Defining a new story for the test and routing to refine

Do not mark a unit as certified if there are unresolved mandatory test gaps.

---

## Step 5 — CLAUDE.md Promotion Candidates

During the walkthrough, surface any conventions or patterns that should be promoted to CLAUDE.md:

> "This implementation used [pattern] to handle [situation]. Is this a convention you want to standardize for future features?"

Record each candidate. The final promotion decision happens in the archive phase — here we flag, not promote.

---

## Step 6 — Write certify.md

Write `<feature>/07-certify/<unit-name>/certify.md`:

```markdown
# Certify: <unit-name>

**Session:** <ISO date>
**Unit:** <unit-name>
**Status:** Certified | Certified with notes | Pending test fixes

---

## Summary

[Two to three sentences: what this unit does, the implementation approach used, and overall quality assessment.]

## Test Coverage

| Acceptance Condition | Test | Status |
|---------------------|------|--------|
| [condition from stories.json] | [test name / location] | Covered / Gap |

## Non-Obvious Implementation Notes

### [Section Name]
[Plain-language explanation of what this code does and why — written for a PR reviewer who has not read the stories or design.]

[Repeat for each non-obvious section]

## Coupling Points

| This code | Calls / Is called by | Contract | What breaks if it changes |
|-----------|---------------------|----------|---------------------------|
| [component] | [other component] | [interface description] | [failure mode] |

## Flags for PR Reviewers

[The two to four things a human reviewer should focus their attention on. Not a list of everything — the high-stakes items.]

- **[Flag 1]:** [what to look at and why]
- **[Flag 2]:** [what to look at and why]

## CLAUDE.md Promotion Candidates

- [pattern description] — candidate for CLAUDE.md, confirmed in archive phase

## Findings from Walkthrough

### CW01 — <short title>
**Type:** Test gap | Implementation concern | Explanation needed
**Action:** [fix now / new story / document]
**Status:** Resolved | Pending

## Delta Mode (if re-certified after changes)

**Changed since last certification:** [description of what changed]
**Re-walked:** [sections re-walked and outcome]
```

---

## Step 7 — Certification Declaration

After the walkthrough:

> "You've walked through this unit. Do you feel you own this code — could you answer questions about it in a PR review, explain why specific choices were made, and defend its correctness?"

If yes: the unit is certified.
If no: identify the specific area the engineer is not comfortable with and walk through it again.

The engineer's ownership is the product of this phase. Do not accept a pro-forma "yes" — the question should prompt reflection.

---

## Step 8 — Status Update

Once certified:
1. Update `<feature>/00-status/07-certify/<unit-name>/status.md` to `status: complete`.
2. Update `<feature>/00-status/status.md`.
3. If all units are certified: "All units are certified. Ready to open the PR. Run `/workflow:archive <feature-name>` before or after the PR to extract patterns."

---

## Rules

- **Collaborative, not adversarial.** "Here's what I noticed" — not "this is wrong." The goal is understanding, not judgment.
- **Boilerplate is skipped.** Focus time on what is non-obvious. Framework scaffolding and standard CRUD do not need walkthrough.
- **Test coverage mapping is mandatory.** Every acceptance condition must map to a test. Gaps are findings, not footnotes.
- **Flags for PR reviewers are the deliverable.** The `certify.md` is used by both the engineer during the PR and by reviewers assessing the change.
- **Delta mode for re-certification.** If the implementation changes after certification, only the diff is re-walked — not the entire unit.
