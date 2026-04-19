<div align="center">

![AI Workflow Logo](assets/images/ai-workflow-logo.gif)

</div>

# AI Workflow

A Claude Code plugin providing AI-assisted software development skills. Two commands cover the full lifecycle: plan (understand, decide, scope) and build (stories, implement, prove, challenge, archive).

> **Work in progress.** This repository evolves as the skills are used on real projects. Expect rough edges, missing pieces, and approaches that change as understanding deepens.

---

## Installation

### Step 1 — Add the marketplace

```bash
claude plugin marketplace add esumerfd/ai-workflow
```

### Step 2 — Install the plugin

```bash
claude plugin install ai-workflow@ai-workflow
```

Once installed, skills are available as `/workflow:plan`, `/workflow:build`, and `/workflow:status`.

---

## Overview

Software development with AI assistance works best when there is a shared, verified understanding of what exists, what is wanted, and why each decision was made. Without that structure, AI agents produce confident output on shaky foundations — and the mistakes are expensive to find late.

This workflow addresses that with two recursive skills:

- **Plan** maps the terrain, defines requirements, makes architectural decisions, and scopes work down to independently buildable workstreams — with the engineer certifying each level before proceeding.
- **Build** generates executable stories per workstream, runs the red/green/refactor loop, proves completion, challenges the engineer's understanding, and closes with an archive ritual.

Each skill produces written artifacts grounded in confirmed facts. Feedback loops route back to the appropriate level when something discovered late invalidates an earlier decision.

---

## Skills

### `/workflow:plan <feature> [phase]`

Planning is a recursive drill-down. The same skill runs at two levels:

| Level | Command | Output |
|---|---|---|
| Project | `/workflow:plan <feature>` | `plan.md` — terrain map, requirements, ADR, phase list |
| Phase | `/workflow:plan <feature> <phase>` | `<phase>/plan.md` + `definition.md` per workstream |

Level 1 explores the codebase, interviews the engineer, decides the architecture, and breaks the feature into named phases. Level 2 scopes a phase into workstreams — each independently buildable, with end-to-end proofs preferred over layer isolation.

The engineer certifies each level before proceeding. Level 2 certification is the last gate before autonomous execution begins.

### `/workflow:build <feature> [phase/]<workstream>`

Build runs per workstream:

1. **Stories** — generate executable stories from `definition.md`; engineer approves before any code is written
2. **Execution** — red/green/refactor loop per story (Claude-native or via ralph-tui)
3. **Proof** — run full proof suite; confirm no regressions
4. **Challenge** — walk through the implementation; engineer certifies they own the code
5. **Archive** — extract patterns, update `plan.md`, promote to `CLAUDE.md`

### `/workflow:status [detail]`

Summarise workspace progress. Derives workstream status from artifact presence — no separate status files needed per workstream.

---

## Workspace Layout

```
<feature>/
├── project.json
├── status.md
├── plan.md
├── patterns.md
├── testing.md
├── standards.md
└── <NN>-<phase>/               # NN = sequence; parallel phases share same number
    ├── plan.md
    └── <workstream>/
        ├── definition.md       # written by plan
        ├── stories.md          # written by build — source of truth
        ├── stories.json        # derived — only present if using ralph-tui
        └── proof.md            # written by build
```

---

## Complexity Range

The two-skill structure works at all problem sizes:

| Problem size | Plan | Build |
|---|---|---|
| Tiny (single workstream) | Level 1 only — produces one workstream directly | One story set, light challenge |
| Small (2–4 workstreams) | Level 1 scopes workstreams in one pass | One workstream at a time |
| Medium (5+ workstreams) | Level 1 then Level 2 per phase | Per workstream, full challenge |
| Large (multiple phases) | Level 1 produces phase list; Level 2 per phase | Per workstream per phase |

---

## Claude Code Focus

These skills are built for **Claude Code** — the CLI AI coding tool. They are implemented as Claude Code slash commands that run interactively in your terminal alongside the code you are working on.

---

## Status

Active work in progress. Core skills in use on real projects. Skill behavior and artifact formats continue to evolve.

Contributions, issues, and feedback welcome.
