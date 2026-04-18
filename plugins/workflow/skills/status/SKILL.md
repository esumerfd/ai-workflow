---
name: workflow:status
description: Summarise workflow progress in the current workspace. Add 'detail' arg for per-unit breakdown.
argument-hint: [detail]
---
# Skill: /workflow:status

**Workflow Status**

Summarise the current state of the workflow by reading all `00-status/` files in the current workspace. Shows which phases are complete, in progress, or blocked at a glance.

**Usage:**
- `/workflow:status` — one-line summary per phase
- `/workflow:status detail` — full breakdown including per-unit status and any blocker text

---

## Step 1 — Resolve the Workspace

The workspace is the current working directory. Look for a `00-status/` directory here.

If `00-status/` does not exist in the current directory, report:
> "No workflow workspace found in the current directory. Run `/workflow:explore <feature-name>` to start one, or `cd` into an existing workspace first."

Stop.

---

## Step 2 — Read Status Files

Read the following files (skip gracefully if a file does not exist — treat missing as `pending`):

**Root:**
- `00-status/status.md`

**Phase-level:**
- `00-status/01-explore/status.md`
- `00-status/02-plan/status.md`
- `00-status/03-design/status.md`
- `00-status/04-decompose/status.md`
- `00-status/05-refine/status.md`
- `00-status/06-review/status.md`
- `00-status/07-certify/status.md`
- `00-status/08-archive/status.md`

**Unit-level** (for phases that have units — 05 through 07):
- `00-status/05-refine/<unit>/status.md` for each subdirectory found
- `00-status/06-review/<unit>/status.md` for each subdirectory found
- `00-status/07-certify/<unit>/status.md` for each subdirectory found

Also read `04-decompose/units.md` if it exists to get the canonical list of unit names.

---

## Step 3 — Render Output

### Default (no `detail` arg)

Print a compact phase-by-phase table. Derive the feature name from the current directory name.

```
Feature: <current directory name>
Overall: <status from 00-status/status.md>

Phase           Status      Notes
─────────────────────────────────────────────────
01 explore      complete
02 plan         complete
03 design       complete
04 decompose    complete
05 refine       started     3 of 5 units complete
06 review       pending
07 certify      pending
08 archive      pending
```

**Status indicators:**
- `complete` — all work in this phase done
- `started` — phase is in progress
- `blocked` — phase cannot advance; show first blocker inline
- `pending` — not yet started (or status file absent)

For phases with units, the Notes column shows `<n> of <total> units complete`.

If the overall status is `blocked`, append a **Blockers** section after the table listing each blocked phase and the blocker text from its status file.

---

### Detail mode (`detail` arg present)

Print the same phase table, then for each phase that has unit-level status, expand into a per-unit breakdown:

```
Feature: <feature-name>
Overall: <status>

Phase           Status      Notes
─────────────────────────────────────────────────
01 explore      complete
02 plan         complete
03 design       complete
04 decompose    complete
05 refine       started     3 of 5 units complete
  ├─ unit-auth       complete
  ├─ unit-api        complete
  ├─ unit-storage    complete
  ├─ unit-events     started
  └─ unit-ui         pending
06 review       pending
  ├─ unit-auth       pending
  ├─ unit-api        pending
  ├─ unit-storage    pending
  ├─ unit-events     pending
  └─ unit-ui         pending
07 certify      pending
  ├─ unit-auth       pending
  ├─ unit-api        pending
  ├─ unit-storage    pending
  ├─ unit-events     pending
  └─ unit-ui         pending
08 archive      pending

─────────────────────────────────────────────────
Blockers
  05 refine / unit-events: waiting on external API schema confirmation
```

For each unit with status `started` or `blocked`, include the description text from the status file on the next line, indented under the unit:

```
  ├─ unit-events     started
  │    3 of 7 stories complete. Blocked on external API schema.
```

For units with status `complete` or `pending`, show the status only — no description text needed.

---

## Rules

- **Read only — never write.** This skill reads status files but makes no changes to the workspace.
- **Missing files are `pending`.** Do not error on absent status files — treat them as not yet started.
- **Unit order follows `units.md`.** When listing units in detail mode, use the order from `04-decompose/units.md` if it exists, not filesystem order.
- **Blockers are always shown.** Whether in default or detail mode, any `blocked` status with blocker text is surfaced at the bottom of the output — never silently hidden.
