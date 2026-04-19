# Workspace Bootstrap

A **workspace** is the root directory where all workflow artifacts and progress for a single feature are stored. It is separate from any code repository.

## Resolving the Workspace Root

Check in order — use the first match:

1. **Current directory is a workspace** — if the current directory contains `project.json` or `00-status/`, treat it as the workspace root. Do not create a subdirectory.
2. **`$WORKFLOW_HOME` is set** — use `$WORKFLOW_HOME/<feature-name>/`
3. **Fallback** — use `./<feature-name>/` relative to the current working directory

The workspace root is always an absolute path. Record it in `project.json` as `workspace_root`.

## Bootstrapping on First Use

When a phase is invoked for the first time, create the workspace scaffold if it does not already exist:

```
<workspace-root>/
├── project.json
└── 00-status/
    └── status.md
```

**project.json** — created interactively if it does not exist:

```json
{
  "feature": "<feature-name>",
  "repos": [],
  "branch_strategy": "feature-branch",
  "created": "<ISO date>",
  "workspace_root": "<absolute path to workspace root>"
}
```

Ask the engineer:
- What is the feature name?
- What repository paths should this feature touch? (can be added later)

## Status File Format

Every `status.md` uses this format:

```
status: pending | started | complete | blocked

---

[Optional: description of progress, decisions made, or context for the current state]

### Blockers

[Optional: list any blockers preventing advancement, with enough detail to act on them]
```

Phase-level status files summarise across their work units. The root `00-status/status.md` summarises across all phases.

## Phase Directory Layout

```
<feature>/
├── project.json
├── 00-status/
├── 01-explore/
│   ├── explore.md
│   ├── patterns.md
│   ├── testing.md
│   └── standards.md
├── 02-plan/
│   └── requirements.md
├── 03-design/
│   └── design.md
├── 04-decompose/
│   └── units.md
├── 05-refine/
│   └── <unit>/
│       ├── stories.json
│       └── prompt.md
├── 06-review/
│   └── <unit>/
│       └── findings.md
├── 07-certify/
│   └── <unit>/
│       └── certify.md
└── 08-archive/
    └── summary.md
```

## Resuming a Phase

If the phase artifact already exists, read it before starting. Present a summary of the existing state and ask: resume from where it left off, or start a new session (appending, not overwriting).

## Rules

- Never write workflow artifacts into a code repository. The workspace is always separate.
- Unit names from `04-decompose/units.md` must be used consistently in all downstream phases.
- Status files are updated at the start and end of every phase.
