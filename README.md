# AI Workflow

A collection of Claude Code skills for AI-assisted software development. Covers the full lifecycle from initial exploration through implementation, review, and archiving.

> **Work in progress.** This repository evolves as the skills are used on real projects. Expect rough edges, missing pieces, and approaches that change as understanding deepens.

---

## Overview

Software development with AI assistance works best when there is a shared, verified understanding of what exists, what is wanted, and why each decision was made. Without that structure, AI agents produce confident output on shaky foundations — and the mistakes are expensive to find late.

This workflow addresses that by moving through phases where each phase:

- produces a **written artifact** grounded in confirmed facts
- is **verified by the engineer** before the next phase begins
- builds on prior artifacts rather than starting from assumptions

The result is a progression from exploration through to implemented, certified code — with explicit feedback loops when something discovered late invalidates an earlier decision.

---

## Complexity Range

Not every feature needs eight phases. The workflow is designed to accommodate varying complexity:

| Complexity | Typical Entry | Phases Used |
|---|---|---|
| Small / well-understood | `/refine` | refine → review → certify |
| Medium / known terrain | `/plan` or `/design` | plan → design → refine → review → certify |
| Large / unfamiliar area | `/explore` | all eight phases |

Enter where your confidence runs out. The phases before your entry point are assumed done.

---

## Phases

```
/explore → /plan → /design → /decompose → /refine → /review → /certify → /archive
```

| Phase | Purpose | Output |
|---|---|---|
| `/explore` | Map what exists — facts only, no assumptions | `explore.md` — component map, couplings |
| `/plan` | Define what the change achieves | `requirements.md` — scoped, sourced requirements |
| `/design` | Decide the approach and record the rationale | `design.md` — Architecture Decision Record |
| `/decompose` | Find the smallest independently provable units | `units.md` — unit list with entry/exit criteria |
| `/refine` | Generate executable stories per unit | `stories.json`, `prompt.md` per unit |
| `/review` | Engineer acceptance — is it ready to ship? | `findings.md` per unit |
| `/certify` | Walk through the code before putting your name on the PR | `certify.md` per unit |
| `/archive` | Extract durable patterns into institutional memory | `summary.md` + diff to `CLAUDE.md` |

Review findings route back: a bug goes to `/refine`, a wrong boundary to `/decompose`, a violated architecture to `/design`, a wrong context map to `/explore`.

---

## Claude Code Focus

These skills are built for **Claude Code** — the CLI AI coding tool. They are implemented as Claude Code slash commands that run interactively in your terminal alongside the code you are working on.

Reuse in other AI coding assistants (Cursor, Copilot, etc.) is conceptually plausible but untested. The prompt structures and artifact conventions are not tool-specific, but the slash command mechanics are Claude Code native.

---

## Status

This is an active work in progress. Current state:

- Core phase skills drafted and in use
- Skill behavior and artifact formats continue to evolve
- Some phases are more mature than others
- Documentation lags implementation

Contributions, issues, and feedback welcome.
