---
name: workflow:decompose
description: "Phase 4: Find provable boundaries. Identify the smallest independently testable units before writing any stories."
argument-hint: <feature-name>
---
# Skill: /workflow:decompose

**Phase 4 — Find the Provable Boundaries**

Identify the natural seams in the work and define the smallest independently provable units before any stories are written. Answer "What is the right shape of the work?" before "What are the steps?"

---

## When to Use This Skill

Use `/workflow:decompose` when:
- Architecture is settled in `03-design/design.md`
- You need to identify the right work boundaries before generating stories
- The feature is large enough that doing everything in one unit would be risky

Skip to `/workflow:refine` if the work is small enough that the entire feature is obviously one unit — but err toward decomposing rather than assuming.

---

## Inputs

- Feature name
- Optionally: a proposed decomposition to evaluate rather than generate from scratch

---

## Step 1 — Bootstrap Workspace

Follow `skills/_shared/workspace.md`.

Create if not present:
```
<feature>/04-decompose/
<feature>/00-status/04-decompose/
```

Set `<feature>/00-status/04-decompose/status.md` to `status: started`.

Load:
- `<feature>/01-explore/explore.md` — coupling map (loose couplings = candidate seams; tight = single unit)
- `<feature>/03-design/design.md` — the architecture diagram delta (new components, modified interfaces)

If `<feature>/04-decompose/units.md` already exists, read it and ask: revise an existing boundary, or start over?

---

## Step 2 — Seam Analysis (Parallel)

Run three analysis threads in parallel. Each answers a different question about where work can be split:

**Thread 1 — Architectural Layer Analysis**
- Which architectural layers does this change touch? (API, domain logic, persistence, integration, UI)
- Can each layer be changed and tested in isolation?
- Where does a change in one layer require changes in another? (these are NOT seams)

**Thread 2 — End-to-End Workflow Analysis**
- What are the user-facing flows this feature enables or modifies?
- What is the thinnest slice that produces a demonstrable, testable result end-to-end?
- Can a thin slice be implemented before full layer coverage is complete?

**Thread 3 — Coupling Cross-Reference**
- From `explore.md`: which couplings are loose (candidate seams) and which are tight (must stay together)?
- From `design.md`: does the proposed architecture introduce new boundaries or remove existing ones?
- Which tight couplings require a wider unit scope?

Return structured findings from each thread. The orchestrating session merges into candidate unit boundaries.

---

## Step 3 — Boundary Validation

For each candidate unit boundary, apply the independence test:

> "Can this unit be built, run, and verified — start to finish — without the adjacent unit being complete?"

If yes: it is a real seam. If no: expand the unit or introduce a stub strategy.

Ask the engineer for each candidate boundary:
1. "Unit [A] and Unit [B] are separated at [seam]. Can [A] be verified independently, or does it require [B] to be partially complete?"
2. "Is there shared state between [A] and [B] that would make them order-dependent at runtime even if they can be built separately?"

Use Ralph Wiggum technique (`skills/_shared/ralph-wiggum.md`) for these confirmations — one boundary at a time, confirmed before moving on.

---

## Step 4 — Unit Definition

For each confirmed unit, define:

| Field | Description |
|-------|-------------|
| **Name** | Short identifier used in all downstream directories (`05-refine/<name>/`, etc.) |
| **Scope** | What this unit contains — no more, no less |
| **Entry criteria** | What must exist or be true before this unit can begin |
| **Exit criteria** | What must be true for this unit to be considered done |
| **Dependencies** | Which other units must be complete (or stubbed) before this can run |
| **Approach** | `thin-slice` (end-to-end, minimal) / `layer` (horizontal slice) / `spike-first` (unverified assumption) |
| **Rationale** | Why this boundary was chosen at this seam |

Present the full set to the engineer before writing `units.md`:
> "Here are the proposed units. Does each one represent work that can be proven independently? Are the names clear enough to use as directory names throughout the rest of the workflow?"

Do not write `units.md` until the full set is approved.

---

## Step 5 — Challenge Questions

Before writing, surface the two or three questions that could invalidate the decomposition.

Examples:
- "Unit U3 depends on U1 being complete — is that a hard dependency or can U3 be stubbed independently?"
- "The diagram shows U2 and U4 as parallel — do they share any state that could make them order-dependent at runtime?"
- "The decomposition assumes [seam] is stable — is that boundary likely to shift during implementation?"

Ask each. Record the answer. Revise units if needed.

---

## Step 6 — Write units.md

Write `<feature>/04-decompose/units.md`:

```markdown
# Decomposition: <Feature Name>

**Session:** <ISO date>
**Status:** Draft | Approved

---

## Summary

[Two to three sentences: how many units, what the primary seams are, and the recommended execution order.]

## Decomposition Diagram

[REQUIRED: Mermaid diagram showing the units, their dependencies, and execution order.]

\`\`\`mermaid
graph LR
    U1[unit-auth] --> U2[unit-api]
    U1 --> U3[unit-storage]
    U2 --> U4[unit-integration]
    U3 --> U4

    style U1 fill:#1976D2,color:#fff
    style U2 fill:#1976D2,color:#fff
    style U3 fill:#1976D2,color:#fff
    style U4 fill:#7B1FA2,color:#fff
\`\`\`

## Units

### <unit-name>
**Scope:** [what this unit contains]
**Approach:** thin-slice | layer | spike-first
**Entry criteria:** [what must exist before this starts]
**Exit criteria:** [what must be true when done]
**Dependencies:** [other unit names, or "none"]
**Rationale:** [why this boundary was chosen]

[Repeat for each unit]

## Execution Order

| Priority | Unit | Can Run In Parallel With |
|----------|------|--------------------------|
| 1 | [unit-name] | — |
| 2 | [unit-name] | [unit-name] |
| 3 | [unit-name] | — |

## Challenge Questions Asked

**Q:** [question]
**A:** [answer] → [impact: none / revised unit [name]]
```

**Unit names are locked once approved.** They flow through `05-refine/`, `06-review/`, `07-certify/`, and all `00-status/` subdirectories. Renaming a unit after this point requires updating every downstream reference.

---

## Step 7 — Certification and Status Update

> "These unit boundaries define the shape of all downstream work. Are you satisfied with them? Unit names will be used as directory names through the rest of the workflow."

Once approved:
1. Update `<feature>/00-status/04-decompose/status.md` to `status: complete`.
2. Update `<feature>/00-status/status.md`.
3. Create placeholder status directories for each unit under `00-status/05-refine/<unit-name>/`.
4. State next phase: `/workflow:refine <feature-name>`.

---

## Rules

- **One confirmation per boundary.** Use the Ralph Wiggum technique. Do not batch boundary confirmations.
- **The diagram is required.** A `units.md` without a decomposition diagram is incomplete.
- **Unit names are permanent.** Names defined here flow through every downstream phase. Choose them carefully.
- **Independence is the test.** A boundary that cannot be independently verified is not a seam.
- **Tight couplings are a single unit.** Do not split what cannot be separately proven. Mark it as a wider unit and note the coupling.
