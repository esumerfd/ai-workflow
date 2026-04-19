# Workspace Bootstrap

A **workspace** is the root directory where all workflow artifacts and progress for a single feature are stored. It is separate from any code repository.

## Resolving the Workspace Root

Check in order — use the first match:

1. **Current directory is a workspace** — if the current directory contains `project.json` or `00-status/`, treat it as the workspace root. Do not create a subdirectory.
2. **`$WORKFLOW_HOME` is set** — use `$WORKFLOW_HOME/<feature-name>/`
3. **Fallback** — use `./<feature-name>/` relative to the current working directory

The workspace root is always an absolute path. Record it in `project.json` as `workspace_root`.

## Bootstrapping on First Use

When a skill is invoked for the first time on a feature, create the workspace scaffold if it does not already exist:

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

## Directory Layout

```
<feature>/
├── project.json
├── plan.md                         # Root: terrain map + requirements + ADR + phase list
├── patterns.md                     # Discovered conventions (updated throughout)
├── testing.md                      # Testing patterns
├── standards.md                    # Project standards
│
├── 00-status/
│   └── status.md
│
└── <phase>/                        # One directory per phase (omit level if no phases)
    ├── plan.md                     # Phase requirements + workstream list
    │
    └── <workstream>/               # One directory per workstream
        ├── definition.md           # Written by plan — build contract
        ├── stories.md              # Written by build — executable stories, source of truth
        ├── stories.json            # Derived by ralph-tui converter — only present if using ralph-tui
        └── proof.md                # Written by build — proof results and challenge record
```

For features with no meaningful phases, workstream directories sit directly under the feature root alongside `plan.md`:

```
<feature>/
├── plan.md
├── patterns.md
├── testing.md
├── standards.md
├── 00-status/
└── <workstream>/
    ├── definition.md
    ├── stories.md
    └── proof.md
```

## Resuming

If the primary artifact for a skill already exists (`plan.md`, `definition.md`, `stories.md`), read it before starting. Present a summary of the existing state and ask: resume from where it left off, or start a new session (appending, not overwriting).

## Rules

- Never write workflow artifacts into a code repository. The workspace is always separate.
- Workstream names from `plan/<phase>/plan.md` must be used consistently in all downstream work.
- Status files are updated at the start and end of every skill invocation.
- `stories.json` is a derived artifact — always regenerated from `stories.md`, never hand-edited.
