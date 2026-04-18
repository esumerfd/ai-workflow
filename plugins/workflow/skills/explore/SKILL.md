---
name: workflow:explore
description: "Phase 1: Map the terrain. Build a certified, factual component map of the codebase area you are about to change."
argument-hint: <feature-name>
---
# Skill: /workflow:explore

**Phase 1 — Map the Terrain**

Build a mutually certified, factual map of the existing codebase. What IS — not what will be or should be. The output is an `explore.md` that the engineer would defend to a peer.

---

## When to Use This Skill

Use `/workflow:explore` when:
- You are unfamiliar with the area you are about to change
- You need confirmed facts before making any planning or design decisions
- You want a shareable, structured context map that reduces onboarding time for yourself or others

Skip to a later phase if terrain is already well understood.

---

## Inputs

- The feature name (used to locate or create the workspace)
- The target area to explore (a repository path, service name, or system description)
- Optionally: a specific question or concern driving the exploration

If any of these are missing, ask before starting.

---

## Step 1 — Bootstrap Workspace

Follow the workspace bootstrap process in `skills/_shared/workspace.md`.

Create these directories if they do not exist:
```
<feature>/01-explore/
<feature>/01-explore/context.d/
<feature>/00-status/01-explore/
```

Create `<feature>/00-status/01-explore/status.md` with `status: started`.

If `<feature>/01-explore/explore.md` already exists:
- Read it before doing anything else.
- Summarize what was already mapped and at what confidence.
- Ask: "Resume from here, or start a new exploration session for a different area?"
- If resuming: append, do not overwrite.
- If new session: add a session separator (`---`) before the new content.

---

## Step 2 — Familiarity Assessment

Before exploring, calibrate the depth and vocabulary of the session. Ask:

1. "How familiar are you with this area — have you worked in it before, read the code, or is this completely new territory?"
2. "Is there a specific concern, coupling, or behaviour you most want to understand from this exploration?"
3. "Are there any files, modules, or components you know are central and want me to start with?"

Use the answers to decide:
- **High familiarity:** confirmatory mode — validate what the engineer already believes, surface surprises.
- **Low familiarity:** discovery mode — systematic traversal, more confirmations, wider coverage.

Do not skip this step. The wrong calibration wastes both rounds.

---

## Step 3 — Parallel Terrain Analysis

Dispatch analysis across the target area. Where the codebase has natural separation (distinct services, architectural layers, major modules), use the Agent tool to run specialist agents in parallel. Each agent should:

- Identify the primary responsibilities of its area
- List the public interface (exported functions, API endpoints, event types, etc.)
- Identify direct dependencies (imports, service calls, shared state)
- Note any patterns or standards in use (framework idioms, naming conventions, test strategies)

For each agent: return a structured observation set, not prose. The orchestrating session (this skill) merges these and presents them for confirmation.

For tightly coupled areas, use a single agent to preserve coherence. Do not split an area that cannot be understood without the other side of the coupling.

---

## Step 4 — Observation and Confirmation Loop

Use the **Ralph Wiggum technique** (`skills/_shared/ralph-wiggum.md`) for every observation. Never record an observation before the engineer confirms it.

Work through the terrain in this order:

### 4a. Component Responsibilities
For each major component identified:
1. State its primary responsibility in one sentence.
2. Confirm: "Is that an accurate description of what [Component] does?"
3. If yes: record. If corrected: record the correction.

### 4b. Interface Inventory
For each component:
1. List its public interface (methods, endpoints, events, exported types).
2. Confirm: "Does this look like a complete list, or are there other entry points I should know about?"

### 4c. Coupling Assessment
For each coupling discovered:
1. Name both components and describe the coupling plainly.
2. State the coupling type: data shared, call dependency, temporal, or external.
3. Ask the stakes-weighted question:
   - Low coupling: "Is this coupling intentional, or an accident of the current structure?"
   - High coupling: "If [A] changes its [interface/data model/timing], what breaks in [B]?"
4. Record the coupling type and whether it is a candidate seam.

### 4d. Testing Patterns
1. What testing approach is in use? (unit, integration, E2E, none?)
2. Are there shared fixtures, factories, or test helpers?
3. What is covered and what is notably absent?

Confirm each finding before recording.

---

## Step 5 — Challenge Questions

Before writing `explore.md`, surface the questions whose wrong answers would invalidate the entire context map. These are not a checklist — pick the two or three that matter most for this terrain.

Examples:
- "This diagram shows [A] depends only on [B] — are there any runtime dependencies not visible in the code (config injection, service discovery, env vars)?"
- "The coupling between [X] and [Y] is marked Low — does that hold under load or failure conditions?"
- "I've marked [Component] as owning [responsibility] — is there another team or service that shares or disputes that ownership?"

Ask each question. Record the answer. If the answer changes a confirmed observation, update it before proceeding.

---

## Step 6 — Write explore.md

Write `<feature>/01-explore/explore.md` with this structure:

```markdown
# Explore: <Feature Name>

**Session:** <ISO date> | **Area:** <target area>
**Status:** Certified | Partial | In Progress

---

## Summary

[Two to four sentences: what was explored, what is now understood, and what remains uncertain.]

## Component Map

[REQUIRED: a Mermaid diagram showing the components, their relationships, and couplings.
Use the Mermaid styling from the user's CLAUDE.md preferences.
A component map without a diagram is incomplete.]

\`\`\`mermaid
graph LR
    A[ComponentA] -->|calls| B[ComponentB]
    B -->|reads| C[(DataStore)]

    style A fill:#1976D2,color:#fff
    style B fill:#1976D2,color:#fff
    style C fill:#F57C00,color:#fff
\`\`\`

## Components

### <ComponentName>
**Confidence:** High | Medium | Low | Unverified
**Responsibility:** [one sentence]
**Interface:** [key public methods, endpoints, events]
**Dependencies:** [what it depends on]
**Notes:** [anything unusual or important]

[Repeat for each component]

## Couplings

| Component A | Component B | Type | Strength | Candidate Seam? |
|-------------|-------------|------|----------|-----------------|
| [A]         | [B]         | call | tight    | No — shared state |
| [C]         | [D]         | data | loose    | Yes              |

## Testing

**Confidence:** High | Medium | Low | Unverified

- Approach: [unit / integration / E2E / none]
- Coverage areas: [what is tested]
- Notable gaps: [what is not tested]
- Shared fixtures: [yes/no, location if yes]

## Open Questions

[Questions raised during exploration that were not resolved.
These carry forward to the plan phase.]

- [ ] [Question 1]
- [ ] [Question 2]

## Challenge Questions Asked

[The challenge questions from Step 5 and the engineer's answers.]

**Q:** [question]
**A:** [answer] → [impact on map: none / updated [section]]
```

### Supporting Files in context.d/

Write only what was confirmed. Do not speculate:

- `context.d/patterns.md` — code conventions, idioms, framework patterns in use
- `context.d/testing.md` — testing patterns, fixtures, coverage approach
- `context.d/standards.md` — discovered project standards (naming, structure, error handling)

---

## Step 7 — Certification and Status Update

Present the completed `explore.md` to the engineer with this prompt:

> "Here is the terrain map. Before I mark this phase complete: would you sign off on this as a document you'd defend to a peer? If anything is wrong or missing, let's correct it now."

If the engineer requests changes: make them, then re-present.

Once certified:
1. Update `<feature>/00-status/01-explore/status.md` to `status: complete`.
2. Update `<feature>/00-status/status.md` to reflect current overall state.
3. State what the next phase is and how to invoke it: `/workflow:plan <feature-name>`.

---

## Rules

- **Never record an unconfirmed observation.** If the engineer has not confirmed it, it is not a fact yet.
- **The diagram is required.** An `explore.md` without a component diagram is incomplete. Do not close the phase without one.
- **Append, never overwrite.** If `explore.md` already exists, add a session separator and append. Prior confirmed observations are not erased by a new session.
- **Couplings are seam candidates.** Mark every coupling. Downstream phases depend on this.
- **Challenge questions are not optional.** Surface the two or three most dangerous assumptions before writing the artifact.
- **Keep context.d/ lean.** Only confirmed, reusable knowledge. Not a scratchpad.
