---
name: workflow:status
description: Summarise workflow progress in the current workspace. Add 'detail' arg for per-workstream breakdown.
argument-hint: [detail]
---
# Skill: /workflow:status

**Workflow Status**

Summarise the current state of the workflow by reading all status files and artifact directories in the current workspace. Shows what is planned, what is built, and what is blocked.

**Usage:**
- `/workflow:status` — one-line summary per phase and workstream
- `/workflow:status detail` — full breakdown including per-workstream status and blocker text

---

## Step 1 — Resolve the Workspace

The workspace is the current working directory. Look for a `00-status/` directory or `plan.md` here.

If neither exists, report:
> "No workflow workspace found in the current directory. Run `/workflow:plan <feature-name>` to start one, or `cd` into an existing workspace first."

Stop.

---

## Step 2 — Read Workspace State

Read `00-status/status.md` for overall status.

Read `plan.md` to determine:
- Whether phases exist (look for the Phases table)
- Phase names

For each phase found in `plan.md`:
- Read `<phase>/plan.md` to get the workstream list
- For each workstream: check whether `definition.md`, `stories.md`, and `proof.md` exist

Derive workstream status from artifact presence:
- No `definition.md` → `pending`
- `definition.md` exists, no `stories.md` → `defined`
- `stories.md` exists, no `proof.md` → `building`
- All stories in `stories.md` have `Passes: true`, no `proof.md` → `proving`
- `proof.md` exists → `complete`

For features with no phases, scan for workstream directories directly under the feature root (directories containing `definition.md`).

---

## Step 3 — Render Output

### Default (no `detail` arg)

```
Feature: <current directory name>
Overall: <status>

Plan     <status>    [Certified | Draft | pending]

Phase: <phase-name>
  <workstream>    <status>
  <workstream>    <status>

Phase: <phase-name>
  <workstream>    <status>
```

**Status values:**
- `pending` — not started
- `defined` — definition.md written, stories not yet generated
- `building` — stories in progress (not all passing)
- `complete` — proof.md written, all stories passing
- `blocked` — show first blocker inline

For features with no phases:
```
Feature: <name>
Overall: <status>

Plan     <status>

  <workstream>    <status>
  <workstream>    <status>
```

---

### Detail mode (`detail` arg present)

Same structure, but for each workstream expand to show story-level progress:

```
Feature: <name>
Overall: <status>

Plan     Certified

Phase: auth
  token-issue    building    3 of 5 stories passing
    S01  passes
    S02  passes
    S03  passes
    S04  building
    S05  pending
  token-validate  defined
```

For each workstream with status `building`, read `stories.md` and count passing stories.

If the overall status is `blocked`, append a **Blockers** section listing each blocked workstream and the blocker text.

---

## Rules

- **Read only — never write.** This skill reads artifacts but makes no changes to the workspace.
- **Missing files mean pending.** Do not error on absent files — treat them as not yet started.
- **Blockers are always shown.** Any blocked status with blocker text is surfaced at the bottom — never silently hidden.
