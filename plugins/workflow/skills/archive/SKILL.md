---
name: workflow:archive
description: "Phase 8: Institutional memory. Extract patterns and promote them to CLAUDE.md."
argument-hint: <feature-name>
---
# Skill: /workflow:archive

**Phase 8 — Institutional Memory**

Every completed feature leaves the codebase and team smarter. Extract the patterns that prevent repeated failures, promote the conventions that ensure consistency, and record the decisions that future engineers will need.

---

## When to Use This Skill

Use `/workflow:archive` when:
- All units are certified and the PR is ready (or already open)
- You want to ensure the knowledge from this feature compounds into future sessions

Can be run before or after the PR is opened. Running before means the patterns inform the PR description.

---

## Inputs

- Feature name

---

## Step 1 — Bootstrap Workspace

Follow `skills/_shared/workspace.md`.

Create if not present:
```
<feature>/08-archive/
<feature>/00-status/08-archive/
```

Set `<feature>/00-status/08-archive/status.md` to `status: started`.

Load everything from the completed workspace:
- `01-explore/explore.md` and `context.d/`
- `02-plan/requirements.md`
- `03-design/design.md`
- `04-decompose/units.md`
- `06-review/*/findings.md` — patterns flagged for promotion
- `07-certify/*/certify.md` — CLAUDE.md promotion candidates

---

## Step 2 — Exit Interview

The exit interview separates signal from noise. Ask the engineer three questions:

1. **Surprise:** "What did you learn during this feature that you did not expect? Anything that changed how you think about this area of the codebase?"

2. **Recurrence:** "If you did this feature again, what would you do differently? What would you tell someone starting a similar feature?"

3. **Standards:** "Did this feature surface any conventions that should be part of every future implementation in this area — naming, structure, error handling, test patterns?"

Listen for:
- One-off bugs: these are not patterns. Do not promote.
- Feature-specific decisions: these belong in the design ADR, not in standards. Do not promote.
- Generic knowledge: these are candidates for promotion. Ask a follow-up to confirm generalizability.

---

## Step 3 — Pattern Extraction

Collect promotion candidates from:
- `06-review/*/findings.md` — "Patterns Flagged for Promotion"
- `07-certify/*/certify.md` — "CLAUDE.md Promotion Candidates"
- Exit interview answers

For each candidate, apply the generalizability test:

> "Would this pattern prevent a failure or maintain consistency in a future, unrelated feature — not just in this one?"

If yes: promote. If no: discard.

Categories of promotable patterns:
- **Coding conventions:** naming, structure, error handling approaches specific to this codebase
- **Testing patterns:** fixtures, factories, assertion strategies that proved effective
- **Architectural conventions:** patterns in how components should be integrated, called, or decoupled
- **Anti-patterns:** things that looked right but caused problems — equally valuable as positive patterns

---

## Step 4 — CLAUDE.md Diff Preview

For each confirmed promotion candidate, draft the addition to CLAUDE.md.

Present the full diff to the engineer before writing:

> "Here is what I propose to add to CLAUDE.md. Review each addition — we're only promoting what you'd want every future session to follow."

Show the diff section by section. For each proposed addition:
1. State what convention it captures.
2. State which finding or observation generated it.
3. Ask: "Should this be added as-is, revised, or dropped?"

Do not write to CLAUDE.md until every addition is confirmed. This is an irreversible promotion — it will affect every future session in this project.

---

## Step 5 — context.d/ Updates

If the exploration or implementation revealed new or corrected knowledge about the codebase:

- Update `<feature>/01-explore/context.d/patterns.md`, `testing.md`, or `standards.md`
- These feed into future `/workflow:explore` and `/workflow:refine` sessions for the same feature or related features

Ask: "Did we discover anything during implementation that corrects or extends the context map in `explore.md`? If so, I'll update `context.d/` before archiving."

---

## Step 6 — Write summary.md

Write `<feature>/08-archive/summary.md`:

```markdown
# Archive Summary: <Feature Name>

**Completed:** <ISO date>
**Phases completed:** [list phases that ran]
**Units:** [list unit names]
**PR:** [link or description, if available]

---

## What Was Built

[Two to four sentences: the feature, its scope, and the primary implementation approach used.]

## Key Decisions

[The two or three most important decisions made during this feature, with brief rationale. Reference the design ADR for full context.]

| Decision | Rationale | Phase |
|----------|-----------|-------|
| [decision] | [why] | design |
| [decision] | [why] | decompose |

## Lessons Learned

[From the exit interview. Surprises, recurrence advice, and things to do differently. Written for a future engineer starting a similar feature.]

- [lesson]
- [lesson]

## Patterns Promoted to CLAUDE.md

[List the conventions promoted and where they landed in CLAUDE.md.]

| Pattern | Category | CLAUDE.md Section |
|---------|----------|-------------------|
| [pattern] | testing | [section name] |
| [pattern] | architectural | [section name] |

## Patterns Discarded

[Candidates that were evaluated but not promoted, with reason.]

| Candidate | Reason Not Promoted |
|-----------|---------------------|
| [candidate] | One-off / feature-specific / already documented |

## Routing History

[Phases that were re-entered due to review findings — useful for understanding where the most rework occurred.]

| Finding | Routed to | Resolved |
|---------|-----------|---------|
| [finding] | [phase] | Yes |

## Open Items

[Anything explicitly deferred — improvements noted in review or certify that were not fixed in this feature.]

- [ ] [deferred item] — [rationale for deferring]
```

---

## Step 7 — Workspace Archive

Move the completed workspace to the archive location:

```
./<workflow-name>-archive/<ISO-date>-<feature-name>/
```

This preserves the full hierarchy including all status history. The workspace is not deleted — it is moved.

Ask before moving: "Ready to archive the workspace to `./<workflow-name>-archive/<date>-<feature-name>/`?"

---

## Step 8 — Final Status Update

1. Update `<feature>/00-status/08-archive/status.md` to `status: complete`.
2. Update root `00-status/status.md` to `status: complete`.
3. Confirm: "Feature [name] is archived. Patterns from this session are now active in CLAUDE.md."

---

## Rules

- **Signal over noise.** One-off bugs and feature-specific decisions are not promoted. Only what prevents future failures or ensures future consistency.
- **Preview before writing CLAUDE.md.** The diff is shown and confirmed before any write. Promotions are irreversible in effect — they will shape all future sessions.
- **Exit interview is the source.** Engineer reflection surfaces what automated analysis cannot. Do not skip it.
- **Workspace is moved, not deleted.** The full artifact history is preserved in the archive location. Nothing is lost.
- **Deferred items are tracked.** Improvements noted in review or certify that were not fixed are recorded in `summary.md` so they are not silently dropped.
