# Workspace Bootstrap

A **workspace** is the root directory where all workflow artifacts for a single feature are stored. It is always separate from any code repository.

## Resolving the Workspace Root

Check in order — use the first match:

1. **Current directory is a workspace** — if it contains `project.json` or `status.md`, treat it as the workspace root.
2. **`$WORKFLOW_HOME` is set** — use `$WORKFLOW_HOME/<feature-name>/`
3. **Fallback** — use `./<feature-name>/` relative to the current working directory

The workspace root is always an absolute path. Record it in `project.json` as `workspace_root`.

## Bootstrapping on First Use

When a skill is invoked for the first time, create the workspace scaffold if it does not already exist:

```
<workspace-root>/
├── project.json
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

## Directory Layout

```
<feature>/
├── project.json
├── status.md                       # Overall workspace status — written by skills, human-readable
├── plan.md                         # Root: terrain map + requirements + ADR + phase list
├── patterns.md                     # Discovered conventions (updated throughout)
├── testing.md                      # Testing patterns
├── standards.md                    # Project standards
│
└── <NN>-<phase>/                   # One directory per phase
    ├── plan.md                     # Phase requirements + workstream list
    │
    └── <workstream>/               # One directory per workstream
        ├── definition.md           # Written by plan — build contract
        ├── stories.md              # Written by build — executable stories, source of truth
        ├── stories.json            # Derived — only present if using ralph-tui
        └── proof.md                # Written by build — proof results and challenge record
```

For features with no meaningful phases, workstream directories sit directly under the feature root:

```
<feature>/
├── project.json
├── status.md
├── plan.md
├── patterns.md
├── testing.md
├── standards.md
└── <workstream>/
    ├── definition.md
    ├── stories.md
    └── proof.md
```

## Phase Naming

Phase directories use `<NN>-<name>` — a two-digit sequence number followed by a descriptive name. This makes phase order visible in the filesystem.

- Sequential phases: `01-setup`, `02-api`, `03-ui`
- Parallel phases share the same number: `03-api`, `03-ui` (both depend on `02-setup`, neither on the other)

Workstream directories within a phase are not numbered — they are often run in parallel and their relative order is not fixed.

## Workstream Status

Workstream status is derived from artifact presence — no separate status files are needed per workstream:

| Artifacts present | Derived status |
|-------------------|---------------|
| No `definition.md` | `pending` |
| `definition.md`, no `stories.md` | `defined` |
| `stories.md`, not all passing, no `proof.md` | `building` |
| All stories passing, no `proof.md` | `proving` |
| `proof.md` exists | `complete` |

## status.md Format

```
status: pending | started | complete | blocked

---

[Optional: notes on current state, decisions made, next steps]

### Blockers

[Optional: blockers preventing advancement, with enough detail to act on them]
```

## Resuming

If the primary artifact for a skill already exists (`plan.md`, `definition.md`, `stories.md`), read it before starting. Summarise the existing state and ask: resume from where it left off, or start a new session (appending, not overwriting)?

## Rules

- Never write workflow artifacts into a code repository. The workspace is always separate.
- Workstream names must be used consistently across `plan.md`, `definition.md`, and `stories.md`.
- `stories.json` is a derived artifact — always regenerated from `stories.md`, never hand-edited.
