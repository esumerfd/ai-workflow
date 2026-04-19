---
name: workflow:plan
description: "Recursive planning skill. Level 1: map terrain, define requirements, decide architecture, scope phases. Level 2: scope a phase and break it into independently buildable workstreams."
argument-hint: <feature-name> [phase-name]
---
# Skill: /workflow:plan

**Plan — Understand, Decide, Scope**

Planning is a recursive drill-down. The same skill runs at two levels, applying different rules at each depth. Use it once at project level to produce a full plan, then once per phase to break the phase into buildable workstreams.

- `/workflow:plan <feature-name>` — Level 1: project plan (terrain + requirements + architecture + phase list)
- `/workflow:plan <feature-name> <phase-name>` — Level 2: phase plan (scope + workstream breakdown)

---

## Inputs

- Feature name (always required)
- Phase name (Level 2 only — the phase from the parent `plan.md` phase list)

If the feature name is missing, ask before starting.

---

## Step 1 — Bootstrap Workspace

Follow `skills/_shared/workspace.md`.

**Level 1:** Create if not present:
```
project.json
status.md
```

**Level 2:** The feature workspace must already exist with a certified `plan.md`. If `plan.md` does not exist, stop and say: "Run `/workflow:plan <feature-name>` first to create the project plan."

Create the phase directory if not present:
```
<NN>-<phase-name>/
```

Set `status.md` to `status: started` if this is the first phase invoked.

If the target artifact already exists (`plan.md` for Level 1, `<phase>/plan.md` for Level 2):
- Read it before proceeding.
- Summarise the current state and ask: resume (appending) or start a new session?

---

## LEVEL 1 — Project Plan

Run when no `plan.md` exists, or when revisiting the project-level plan.

---

### Step 2 — Familiarity Assessment

Before exploring, calibrate depth and vocabulary. Ask:

1. "How familiar are you with this area — worked in it, read the code, or completely new territory?"
2. "Is there a specific concern, coupling, or behaviour you most want to understand?"
3. "Are there files, modules, or components you know are central?"

Use the answers to set mode:
- **High familiarity:** confirmatory — validate existing beliefs, surface surprises.
- **Low familiarity:** discovery — systematic traversal, wider coverage, more confirmations.

---

### Step 3 — Parallel Terrain Analysis

Dispatch analysis across the target area. Use the Agent tool to run specialist agents in parallel where the codebase has natural separation (distinct services, architectural layers, major modules). Each agent:

- Identifies primary responsibilities of its area
- Lists the public interface (exported functions, API endpoints, event types)
- Identifies direct dependencies (imports, service calls, shared state)
- Notes patterns or standards in use

For tightly coupled areas, use a single agent. Do not split an area that cannot be understood without the other side of the coupling.

Each agent returns structured observations — not prose. The orchestrating session merges and presents for confirmation.

---

### Step 4 — Observation and Confirmation Loop

Use the **Ralph Wiggum technique** (`skills/_shared/ralph-wiggum.md`) for every observation. Never record before the engineer confirms.

Work through the terrain in order:

**4a. Component Responsibilities**
For each major component:
1. State its primary responsibility in one sentence.
2. Confirm: "Is that an accurate description of what [Component] does?"
3. Record only what is confirmed.

**4b. Interface Inventory**
For each component:
1. List its public interface.
2. Confirm: "Does this look complete, or are there other entry points?"

**4c. Coupling Assessment**
For each coupling discovered:
1. Name both components and describe the coupling plainly.
2. State the coupling type: data shared, call dependency, temporal, or external.
3. Ask the stakes-weighted question:
   - Low: "Is this coupling intentional, or an accident of the current structure?"
   - High: "If [A] changes its interface, what breaks in [B]?"
4. Record coupling type and whether it is a candidate seam.

**4d. Testing Patterns**
1. What testing approach is in use?
2. Are there shared fixtures, factories, or test helpers?
3. What is covered and what is notably absent?

Confirm each finding before recording.

---

### Step 5 — Terrain Challenge Questions

Before writing, surface the questions whose wrong answers would invalidate the context map. Pick the two or three that matter most.

Examples:
- "This diagram shows [A] depends only on [B] — are there runtime dependencies not visible in the code?"
- "The coupling between [X] and [Y] is marked Low — does that hold under load or failure?"
- "I marked [Component] as owning [responsibility] — is there another service that shares that ownership?"

Ask each. Record the answer. Update any confirmed observation changed by the answer.

---

### Step 6 — Engineer Interview

The engineer articulates the feature in their own words first. Ask in sequence:

1. "Describe the feature in your own words — what should it do and why?"
2. "Who benefits, and what problem does it solve for them?"
3. "What does success look like — how would you know it's working correctly?"
4. "What is explicitly out of scope?"

Paraphrase back: "So if I understand correctly, you want to [paraphrase]. Is that right?"

Only proceed once confirmed.

---

### Step 7 — Scope Declaration

Before writing requirements:

1. "Are we changing only [area from terrain], or does this touch other areas?"
2. "Are there constraints — technical, team, timeline, or policy?"
3. "Are there existing commitments (API contracts, data formats) that must not change?"

Record the scope boundary. Requirements written outside it are flagged.

---

### Step 8 — Disposition of Terrain Findings

Every coupling and fragility noted in the terrain needs disposition before requirements are written. For each:

> "The terrain flagged [coupling/fragility]. For this feature, should we: (a) address it with a requirement, (b) accept the risk and note rationale, or (c) treat it as out of scope?"

Record each disposition. A requirement that ignores a known coupling without explanation is a hidden risk.

---

### Step 9 — Requirement Drafting

For each requirement:
1. State it: "The system shall [behaviour] when [condition]."
2. Tie it to the engineer's stated goal or a confirmed terrain observation.
3. Present for confirmation before recording.

Flag contradictions with terrain facts. Flag ambiguous requirements.

Challenge questions before writing: surface two or three whose wrong answers would invalidate the requirements.

---

### Step 10 — Architectural Constraints First

Before proposing any design:

1. "Are there architectural approaches that are off the table — technical, team, or policy reasons?"
2. "Are there existing patterns a new design should follow or deliberately diverge from?"
3. "Are there latency, scale, or operational requirements that constrain the approach?"
4. "How much change to existing interfaces are you willing to accept?"

Record constraints. Any approach violating them is eliminated before presenting.

---

### Step 11 — Design Analysis and Proposal

Run specialist agents in parallel:
- **Agent A:** Analyse the component map. Identify which components the change must integrate with, modify, or replace.
- **Agent B:** Research alternative architectural approaches. Enumerate at least two alternatives.
- **Agent C (if applicable):** Identify spike candidates — assumptions the design depends on that are not yet verified.

For each alternative:
1. State the approach in one sentence.
2. List strengths relative to requirements.
3. List weaknesses or risks.
4. State why it was rejected or recommended.

Present to the engineer:
> "My recommendation is [X] because [reason]. Do you agree, or do you want to explore [alternative] further?"

Do not write the ADR until the direction is confirmed.

If any architectural assumption is unverified, define a spike before proceeding:
- One specific assumption being tested
- Explicit pass/fail criteria
- Minimal scope

---

### Step 12 — Design Challenge Questions

Before writing the ADR, surface two or three assumptions that could invalidate it.

Examples:
- "This design assumes the event bus has at-least-once delivery — what is the failure mode if it does not?"
- "The proposed cache layer depends on key stability — what happens when the upstream data model changes?"

Ask each. Record. Revise approach if needed.

---

### Step 13 — Phase Scoping

Break the feature into named phases. Phases are logical milestones with sequential dependencies — each phase can be planned and built independently.

For each phase:
1. Name it (descriptive, not a number).
2. State its scope: what it builds and what it depends on from prior phases.
3. Confirm: "Does this phase boundary make sense as an independent milestone?"

Record the phase list. Phase names are used as directory names — keep them short and lowercase-hyphenated.

---

### Step 14 — Write plan.md

Write `plan.md`:

```markdown
# Plan: <Feature Name>

**Session:** <ISO date>
**Status:** Draft | Certified

---

## Terrain Map

### Summary

[Two to four sentences: what was explored, what is now understood, what remains uncertain.]

### Component Map

[REQUIRED: Mermaid diagram. Apply Mermaid styling from CLAUDE.md.]

### Components

#### <ComponentName>
**Confidence:** High | Medium | Low | Unverified
**Responsibility:** [one sentence]
**Interface:** [key public methods, endpoints, events]
**Dependencies:** [what it depends on]

[Repeat for each component]

### Couplings

| Component A | Component B | Type | Strength | Candidate Seam? |
|-------------|-------------|------|----------|-----------------|

### Testing

**Approach:** [unit / integration / E2E / none]
**Notable gaps:** [what is not tested]

---

## Requirements

### Scope

**In scope:** [items]
**Out of scope:** [items]
**Constraints:** [technical / team / policy]

### R1 — <short title>
**The system shall** [behaviour] **when** [condition].
**Grounded in:** [terrain observation or engineer statement]
**Acceptance criterion:** [testable condition]
**Status:** Confirmed

[Repeat for each requirement]

### Accepted Risks

| Risk | Source | Rationale |
|------|--------|-----------|

---

## Architecture Decision

### Context

[Situation requiring a decision. Reference terrain and requirements.]

### Decision

[The chosen approach: "We will [do X] by [mechanism Y]."]

### Architecture Diagram

[REQUIRED: Mermaid diagram showing proposed change against terrain map. Gray = existing unchanged, green = new, orange = modified.]

### Consequences

**Positive:** [what this enables]
**Negative / Trade-offs:** [what this costs]
**Risks:** [unverified assumptions]

### Alternatives Considered

#### <Alternative Name>
[One sentence] — **Rejected because:** [reason]

### Accepted Unverified Assumptions

[Assumptions not spiked and rationale for proceeding.]

---

## Phases

| Phase | Scope | Depends On | Status |
|-------|-------|------------|--------|
| <phase-name> | [brief scope] | — | pending |

---

## Open Questions

- [ ] [question]

## Challenge Questions Asked

**Q:** [question]
**A:** [answer] → [impact]
```

Write supporting files if not present (append if they are):
- `patterns.md` — confirmed code conventions and idioms
- `testing.md` — confirmed testing patterns and fixtures
- `standards.md` — confirmed project standards

---

### Step 15 — Level 1 Certification

> "Here is the project plan — terrain map, requirements, architectural decision, and phase list. Would you sign off on this as a document you'd defend to a peer? If anything is wrong or missing, let's correct it now."

Make any requested changes, then re-present. Once certified:
1. Update `status.md` to `status: started` with a note of which phases are pending.
2. State next step: "Run `/workflow:plan <feature-name> <first-phase-name>` to plan the first phase."

---

## LEVEL 2 — Phase Plan

Run when `plan.md` is certified and you are planning a specific phase.

---

### Step 2 — Load Context

Load:
- `plan.md` — terrain map, requirements, architectural constraints, phase list
- `patterns.md`, `testing.md`, `standards.md`

Locate the phase in the plan.md phase list. If the phase name is not found, stop and list the available phases.

---

### Step 3 — Phase Scope Confirmation

Present the phase scope from `plan.md` to the engineer:

> "This phase is scoped as: [scope from plan.md]. Is this still accurate, or has the scope changed since the project plan was written?"

If the scope has changed, update `plan.md` before proceeding.

---

### Step 4 — Workstream Breakdown

Break the phase into workstreams using proof-boundary rules:

**Rules for workstream identification:**
- **Prefer end-to-end workstreams.** A workstream that exercises a thin vertical slice (input → processing → output/storage) is preferred over one that isolates a single horizontal layer. End-to-end proofs surface integration failures that layer tests cannot catch.
- **Smallest independently provable unit.** Each workstream must be buildable and provable without the others being complete (except for declared dependencies).
- **Find natural seams.** Look for loose couplings in the terrain map — these are the right boundaries.
- **Layer isolation is a last resort.** A workstream that only exercises one layer (e.g., a service class in isolation with mocks) is acceptable only when no end-to-end slice is feasible.
- **Assign risk profile:** tracer-bullet (thinnest E2E slice, integration risk dominant) or layered (logic complexity dominant, E2E not yet feasible).

For each proposed workstream:
1. Name it (descriptive, lowercase-hyphenated).
2. State its scope and what it excludes.
3. Describe the end-to-end proof: "The proof will demonstrate [observable behaviour] from [input] to [output/state change]."
4. State its risk profile.
5. Identify dependencies on other workstreams.
6. Confirm: "Does this workstream boundary make sense — is it independently buildable?"

---

### Step 5 — Write definition.md Per Workstream

For each approved workstream, write `<phase>/<workstream>/definition.md` (relative to the workspace root):

```markdown
# Workstream: <name>

**Feature:** <feature>
**Phase:** <phase>
**Risk profile:** tracer-bullet | layered
**Session:** <ISO date>

---

## Scope

[What this workstream builds and what it explicitly excludes.]

## Entry Criteria

[What must exist or be true before this workstream can start.]

## Exit Criteria

[Observable conditions that prove the workstream is complete. Written from the outside — what a user or test can observe, not what the code does internally.]

## End-to-End Proof Description

[The behaviour the proof will demonstrate: input → expected output or state change. Written at the system boundary, not the implementation boundary. No layer references.]

## Interfaces

[Contracts this workstream must satisfy or consume from other workstreams.]

## Workstream Dependencies

[Other workstreams that must complete before this one can start, and why.]

## Constraints

[Architectural decisions from plan.md that directly constrain this workstream.]
```

---

### Step 6 — Write Phase plan.md

Write `<phase>/plan.md` (relative to the workspace root):

```markdown
# Plan: <Phase Name>

**Feature:** <feature>
**Session:** <ISO date>
**Status:** Draft | Certified

---

## Phase Scope

[What this phase builds. Reference the parent plan.md phase entry.]

## Requirements Addressed

[Requirements from plan.md that this phase satisfies, fully or partially.]

## Workstreams

| Workstream | Risk Profile | Depends On | Status |
|------------|--------------|------------|--------|
| <name> | tracer-bullet | — | pending |

## Open Questions

- [ ] [question]
```

---

### Step 7 — Level 2 Certification

Present the workstream list and definitions to the engineer:

> "Here are the workstreams for the [phase] phase. Each has a definition.md with scope, exit criteria, and end-to-end proof description. This is the last gate before building begins — do you approve this workstream breakdown?"

If changes are requested: update the relevant `definition.md` files and the phase `plan.md`, then re-present.

Once approved:
1. Update `status.md`.
2. State next step: "Run `/workflow:build <feature-name> <phase>/<first-workstream>` to build the first workstream. Independent workstreams can be built in parallel."

---

## Rules

- **Terrain first, requirements second, design third.** Never write requirements before the terrain is confirmed. Never propose a design before requirements are confirmed.
- **Engineer articulates first.** Do not propose requirements before the engineer describes the feature.
- **Every requirement is grounded.** No requirement without a source in the interview, terrain map, or a confirmed open question answer.
- **The diagram is required at Level 1.** Both the terrain component map and the architecture diagram are required. A plan without either is incomplete.
- **Workstreams prefer end-to-end.** A workstream that only tests one layer is a last resort, not the default.
- **Append, never overwrite.** Prior requirements and decisions are never deleted — superseded with rationale.
- **Accepted risks are recorded.** Silence is not acceptance. Every flagged coupling gets a written disposition.
- **Do not write the ADR until direction is confirmed.** Present and confirm before writing.
