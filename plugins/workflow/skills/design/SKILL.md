---
name: workflow:design
description: "Phase 3: Architectural decisions. Produce an ADR with rationale, alternatives, and a design diagram."
argument-hint: <feature-name>
---
# Skill: /workflow:design

**Phase 3 — Architectural Decisions**

Decide the architectural approach and record the rationale as an Architecture Decision Record (ADR). The design is the WHY — why this approach was chosen and what alternatives were considered. No implementation steps. No test cases.

---

## When to Use This Skill

Use `/workflow:design` when:
- Requirements are confirmed in `02-plan/requirements.md`
- The architectural approach is not yet decided
- There is genuine uncertainty about feasibility, integration, or structure

Skip to `/workflow:decompose` if the architectural approach is already clear and uncontested.

---

## Inputs

- Feature name
- Optionally: a specific architectural question or constraint to anchor the session

---

## Step 1 — Bootstrap Workspace

Follow `skills/_shared/workspace.md`.

Create if not present:
```
<feature>/03-design/
<feature>/00-status/03-design/
```

Set `<feature>/00-status/03-design/status.md` to `status: started`.

Load:
- `<feature>/01-explore/explore.md` — component map and couplings
- `<feature>/02-plan/requirements.md` — confirmed requirements and accepted risks

If `<feature>/03-design/design.md` already exists, read it and ask: revisit (appending a new decision) or amend an existing decision?

---

## Step 2 — Engineer Constraints First

Before any codebase analysis or design proposal, ask the engineer to state their constraints and non-negotiables.

Ask:
1. "Are there any architectural approaches that are off the table — for technical, team, or policy reasons?"
2. "Are there existing patterns in this codebase that a new design should follow or deliberately diverge from?"
3. "Is there a latency, scale, or operational requirement that constrains the approach?"
4. "How much change to existing interfaces are you willing to accept?"

Record the constraints. Any proposed approach that violates them is eliminated before presenting it.

---

## Step 3 — Parallel Codebase Analysis

Before proposing a design, analyse the codebase for context. Run specialist agents in parallel where the work is naturally independent:

- **Agent A:** Analyse the component map from `explore.md`. Identify which components the proposed change must integrate with, modify, or replace.
- **Agent B:** Research alternative architectural approaches that have been used in this codebase or that fit the detected patterns. Enumerate at least two alternatives.
- **Agent C (if applicable):** Identify spike candidates — assumptions the proposed design depends on that are not yet verified by code or evidence.

Each agent returns structured findings. The orchestrating session merges and presents before any design is written.

---

## Step 4 — Thinking Mode Selection

Before proposing a design, ask:

> "This design involves [brief summary of complexity]. Would you like me to use extended thinking to explore the tradeoffs more deeply, or is standard analysis sufficient?"

Use extended thinking when:
- There are multiple viable approaches with non-obvious tradeoffs
- The design touches a high-stakes coupling flagged in explore.md
- Spike candidates exist that could invalidate the approach

---

## Step 5 — Design Proposal and Alternative Comparison

Present the proposed approach alongside the alternatives considered. For each alternative:

1. State the approach in one sentence.
2. List its strengths relative to the requirements.
3. List its weaknesses or risks.
4. State why it was rejected (or why it is the recommendation).

Present to the engineer:
> "Here are the approaches I considered. My recommendation is [X] because [reason]. Before I write the ADR, do you agree with this choice, or do you want to explore [alternative] further?"

Do not write `design.md` until the engineer confirms the direction.

---

## Step 6 — Spike Definition (when uncertain)

If any architectural assumption is unverified, define a spike before proceeding.

A spike is not "let's try it and see." It is:
- **One specific assumption** the design depends on
- **Explicit evaluation criteria** — the spike passes or fails; it does not produce ambiguous results
- **A minimal scope** — only what is needed to prove or disprove the assumption

For each spike:
1. Name it: what assumption is being tested?
2. Define pass/fail criteria: what outcome confirms the assumption?
3. Create `<feature>/03-design/<spike-name>/` with:
   - `stories.json` — the spike proof stories
   - `findings.md` — placeholder for the result

Ask: "Before I write the ADR, should we run the spike to confirm the approach is feasible?"

If yes: pause the design phase. Return after the spike findings are recorded in `findings.md`.
If no: note the unverified assumption in the ADR and record the accepted risk.

---

## Step 7 — Challenge Questions

Before writing `design.md`, surface the two or three assumptions that could invalidate the design.

Examples:
- "This design assumes the event bus has at-least-once delivery — what is the failure mode if it does not?"
- "The proposed cache layer depends on key stability — what happens when the upstream data model changes?"
- "This design adds a new dependency on [service] — what is the on-call and failure-mode story for that dependency?"

Ask each. Record the answer. Revise the approach if needed.

---

## Step 8 — Write design.md

Write `<feature>/03-design/design.md` as an ADR:

```markdown
# Design: <Feature Name>

**Session:** <ISO date>
**Status:** Proposed | Confirmed | Superseded

---

## Context

[The situation that makes a decision necessary. Reference specific findings from explore.md and requirements from plan.md. Two to four sentences.]

## Requirements Addressed

- R1: [title]
- R2: [title]
- [list only the requirements this design is responding to]

## Decision

[The chosen approach. State it plainly: "We will [do X] by [mechanism Y]." No implementation steps.]

## Architecture Diagram

[REQUIRED: Mermaid diagram showing the proposed change against the existing component map from explore.md. Make the delta explicit — what is new, what changes, what is unchanged.]

\`\`\`mermaid
graph LR
    A[Existing Component] -->|unchanged| B[Existing Component]
    A -->|new call| C[New Component]

    style A fill:#616161,color:#fff
    style B fill:#616161,color:#fff
    style C fill:#4CAF50,color:#fff
\`\`\`

Legend: gray = existing unchanged, green = new, orange = modified

## Consequences

**Positive:**
- [what this approach enables or improves]

**Negative / Trade-offs:**
- [what this approach costs or constrains]

**Risks:**
- [unverified assumptions, dependencies on external factors]

## Alternatives Considered

### Alternative 1 — <name>
[One sentence description]
**Rejected because:** [reason]

### Alternative 2 — <name>
[One sentence description]
**Rejected because:** [reason]

## Spikes

[List any spikes defined and their status: pending / passed / failed]

| Spike | Assumption Tested | Status | Outcome |
|-------|-------------------|--------|---------|
| [name] | [assumption] | pending | — |

## Accepted Unverified Assumptions

[Assumptions that were not spiked and the rationale for proceeding without proof.]

## Challenge Questions Asked

**Q:** [question]
**A:** [answer] → [impact: none / revised decision / new spike]
```

**Strictly WHY.** No implementation steps, no test cases, no story references.

---

## Step 9 — Validation Check

After writing `design.md`, run a validation pass:

1. Does the design address every requirement in `02-plan/requirements.md`? If not, flag the gap.
2. Does the design introduce any new couplings not present in `explore.md`? If so, confirm they are intentional.
3. Are there any requirements the design cannot satisfy? Surface as a routing decision: back to plan or accept as a constraint.

Present the validation result to the engineer before marking the phase complete.

---

## Step 10 — Certification and Status Update

> "Here is the ADR. Does this accurately capture the architectural decision — both the choice and the rationale for alternatives not chosen?"

Once confirmed:
1. Update `<feature>/00-status/03-design/status.md` to `status: complete`.
2. Update `<feature>/00-status/status.md`.
3. State next phase: `/workflow:decompose <feature-name>`.

---

## Rules

- **Engineer constraints first, analysis second.** Never propose a design before hearing what is off the table.
- **The diagram is required.** An ADR without an architecture diagram is incomplete. The delta from explore.md must be made explicit.
- **Strictly WHY.** If a sentence describes how to implement something, move it to a spike story or a refine story — not the ADR.
- **Unverified assumptions become spikes or accepted risks.** Silent assumptions do not exist in the ADR.
- **Do not write the ADR until the direction is confirmed.** Present and confirm before writing.
