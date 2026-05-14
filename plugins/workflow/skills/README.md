# AI Workflow Skills

Two skills. Everything else is supporting material.

---

## The Problem This Workflow Solves

AI-assisted development has two failure modes that are hard to see until they are expensive:

1. **Building the wrong thing.** The AI anchors on its first interpretation of a feature and generates plausible-sounding requirements, designs, and code — for something subtly different from what was intended. The engineer doesn't notice until they're reviewing a PR or hitting production.

2. **Code nobody owns.** The AI writes the code, tests pass, and the engineer merges it without being able to explain what it does or why it made the choices it made. The next incident requires a full re-read of code that feels foreign.

This workflow is structured to close both gaps. `/workflow:plan` forces shared understanding before any design is committed. `/workflow:build` forces ownership before any workstream is closed.

---

## How It Works: Two Skills

```
/workflow:plan <feature>              → plan.md (terrain + requirements + architecture + phases)
/workflow:plan <feature> <phase>      → <phase>/plan.md + <phase>/<workstream>/definition.md per workstream
/workflow:build <feature> <workstream> → stories, implementation, proof, archived patterns
```

Plan produces written, certified artifacts. Build consumes those artifacts and produces proven, owned code. Neither skill assumes anything from memory — they read files.

---

## /workflow:plan — Understand Before You Build

### Intent

Force shared understanding into a form both the engineer and future Claude sessions can read. The plan is not a formality — it is the mechanism by which wrong assumptions are surfaced before they become wrong code.

The skill runs twice at different depths:

- **Level 1** (no phase argument): maps the terrain, confirms requirements, selects an architecture, and names the phases.
- **Level 2** (with a phase argument): takes one phase from Level 1 and breaks it into independently buildable workstreams.

### Inputs / Outputs

| | Level 1 | Level 2 |
|---|---|---|
| **Requires** | Feature name | Feature name + phase name + certified `plan.md` |
| **Produces** | `plan.md`, `patterns.md`, `testing.md`, `standards.md` | `<phase>/plan.md`, `<phase>/<workstream>/definition.md` per workstream |

---

### Level 1: Project Plan

#### Step 1 — Bootstrap Workspace

Creates `project.json` and `status.md` if absent; reads them if present and summarises the current state.

**Why it exists:** Plans evolve across multiple sessions. Without persistent workspace state, every session re-discovers the same terrain and re-debates the same decisions. The bootstrap step makes prior work visible immediately and forces a conscious choice: continue or restart.

---

#### Step 2 — Familiarity Assessment

Asks the engineer three calibration questions about their depth of knowledge in this area before any exploration begins.

**Why it exists:** Exploration depth should match familiarity. For an engineer who has worked in this area for years, systematic traversal wastes time they don't have — they need targeted confirmation of what they already believe. For an engineer who is new, surface-level exploration misses the connections that only show up when you look carefully. Calibrating first means the terrain analysis produces the right level of detail rather than generic output.

---

#### Step 3 — Parallel Terrain Analysis

Dispatches multiple specialist agents simultaneously across the codebase, each covering a natural separation (service, architectural layer, major module).

**Why it exists:** A single-agent read is sequential and misses cross-cutting patterns. Parallel agents cover more ground in less time, and each agent can specialize in its area rather than context-switching. The parallelism also surfaces couplings that are invisible inside any single component — you only see the coupling when one agent describes what it calls and another describes what it receives.

---

#### Step 4 — Observation and Confirmation Loop

Every finding from terrain analysis is presented to the engineer for explicit confirmation before it is recorded. Uses the Ralph Wiggum technique: state what you believe, ask if it's right, only write what's confirmed.

**Why it exists:** An LLM produces confident observations about code regardless of whether those observations are correct. Recording unconfirmed observations produces a terrain map that feels authoritative but contains errors. Errors in the terrain map propagate into requirements and architecture — they are almost never caught until something breaks. The confirmation loop is slow but it is the only mechanism that filters AI confidence from engineer-verified fact.

---

#### Step 5 — Terrain Challenge Questions

Before writing the terrain map, surface two or three questions whose wrong answers would invalidate the entire terrain analysis.

**Why it exists:** Confirmation questions catch errors. Challenge questions catch structural assumptions. There is a difference between "is ComponentA responsible for X?" (a confirmation question) and "the diagram shows ComponentA only depends on ComponentB — are there runtime dependencies not visible in the code?" (a challenge question). Challenge questions are specifically designed to falsify the terrain map, not just verify it. This catches the kind of wrong belief that passes all confirmation checks but fails in production.

---

#### Step 6 — Engineer Interview

The engineer describes the feature in their own words before any requirements are written. The AI paraphrases back and waits for explicit confirmation before proceeding.

**Why it exists:** The most reliable way to generate requirements for the wrong feature is to let the AI generate them first. Once the AI has produced a plausible interpretation, the engineer reviews it through a confirmation bias lens — they see what they expected to see and miss what's wrong. Having the engineer articulate first anchors the entire requirements session on their understanding, not the AI's. The paraphrase-and-confirm step makes the AI's interpretation explicit so any gap is visible.

---

#### Step 7 — Scope Declaration

Before requirements are drafted, explicitly state what is in scope, what is out of scope, and what constraints (technical, team, policy) apply.

**Why it exists:** Requirements written without a declared boundary accumulate. An engineer who says "add user notifications" and an AI that hears "add a full notification system with preferences, delivery channels, and an admin panel" are having two different conversations. The scope declaration makes the boundary a first-class artifact that every requirement is checked against. Requirements outside the boundary are flagged at the time they appear, not after they've been implemented.

---

#### Step 8 — Disposition of Terrain Findings

Every coupling and fragility flagged in terrain analysis is explicitly assigned a disposition: address it with a requirement, accept the risk with documented rationale, or mark it out of scope.

**Why it exists:** Known risks that are not formally dispositioned become silent accepted risks. When something later fails because of a coupling that was flagged in the terrain map, "we knew about this" is a very different situation from "we didn't notice it." Formal disposition also prevents the terrain findings from being treated as optional reading — every flagged item must be resolved before requirements are considered complete.

---

#### Step 9 — Requirement Drafting

Each requirement is stated, tied to either the engineer's interview or a confirmed terrain observation, and confirmed before being recorded.

**Why it exists:** Ungrounded requirements are hypotheses masquerading as specifications. A requirement that doesn't trace to something the engineer said or something the terrain confirmed is something someone assumed was needed. Requiring every requirement to have a source prevents scope creep from building up through plausible-but-unverified additions. The format ("The system shall [behaviour] when [condition]") also makes requirements testable rather than aspirational.

---

#### Step 10 — Architectural Constraints First

Before any design alternatives are presented, the engineer is asked to declare what approaches are off the table: technical constraints, team conventions, policy requirements, latency and scale requirements.

**Why it exists:** Presenting a technically excellent design that violates an organizational constraint — a technology the team has banned, a service boundary that can't be crossed, a latency budget that can't be exceeded — wastes the engineer's time and erodes trust in the AI as a design partner. Collecting constraints first eliminates infeasible options before they are presented, so every alternative the engineer evaluates is one they can actually implement.

---

#### Step 11 — Design Analysis and Proposal

Parallel agents investigate the component map, enumerate at least two alternative architectural approaches, and identify unverified spike candidates. A recommendation is made with explicit reasoning; alternatives are presented with strengths, weaknesses, and rejection rationale.

**Why it exists:** Without explicit comparison, the first design that comes to mind becomes the design. Engineers and AIs both anchor on early ideas. Structured enumeration of alternatives forces the comparison to be conscious rather than implicit. The spike identification is equally important: unverified architectural assumptions are named before the design is committed, so the engineer knows what they are betting on.

---

#### Step 12 — Design Challenge Questions

After a design direction is confirmed but before the ADR is written, surface two or three questions whose wrong answers would invalidate the architecture.

**Why it exists:** Same principle as terrain challenge questions, but applied to the design. A design may look solid against the requirements but depend on an assumption that was never verified — "this design assumes the event bus has at-least-once delivery." Challenge questions are chosen specifically to falsify the design, not to poke at implementation details. Wrong answers require revising the architecture before writing it down, not after it's been built.

---

#### Step 13 — Phase Scoping

Break the feature into named phases, each a logical milestone with stated scope and explicit dependencies on prior phases.

**Why it exists:** Phases are not a reporting structure — they are the mechanism for isolating sequential dependencies. A phase boundary should coincide with a point where earlier work is fully proven before later work begins. Without explicit phase scoping, implementation starts in parallel where it shouldn't, or phases are completed in the wrong order, or the scope of each phase is ambiguous enough that two engineers start building conflicting things.

---

#### Step 14 — Write plan.md

Write the terrain map, confirmed requirements, architectural decision, and phase list into a single, structured artifact that will be the input to every downstream skill invocation.

**Why it exists:** Shared understanding that exists only in a conversation is not shared understanding — it evaporates when the session ends. The plan.md artifact makes confirmed knowledge persistent, transferable, and reviewable. It is also the anchor that future sessions use to avoid re-discovering the same terrain and re-debating the same decisions.

---

#### Step 15 — Level 1 Certification

Ask the engineer: "Would you defend this document to a peer?" Make any changes. Once confirmed, lock the status and state the next step.

**Why it exists:** "Does this look right?" and "would you defend this to a peer?" produce very different quality answers. Certification is not a formality — it is the gate that converts a draft into a committed understanding. An engineer who certifies a plan has internalized it. An engineer who just says "looks good" has skimmed it. The plan.md is only reliable as a future input if the engineer has genuinely committed to it.

---

### Level 2: Phase Plan (Workstream Decomposition)

Level 2 runs after Level 1 is certified, once per phase. Its job is to find the right work boundaries.

---

#### Step 2 — Load Context

Read the certified `plan.md` and the phase entry that corresponds to the argument.

**Why it exists:** Level 2 must not re-derive terrain or re-debate architecture. It takes the confirmed knowledge from Level 1 as given and applies it to the specific phase. Loading context first ensures workstream boundaries are designed against confirmed terrain, not the AI's memory of what it thinks was confirmed.

---

#### Step 3 — Phase Scope Confirmation

Present the phase scope from `plan.md` to the engineer and ask whether it's still accurate.

**Why it exists:** Plans are written at a point in time. By the time a phase is being decomposed into workstreams, the scope may have shifted — the architecture changed, a dependency was resolved earlier than expected, the team's priorities adjusted. Confirming scope at this gate catches drift cheaply, before workstream definitions are written around stale scope.

---

#### Step 4 — Workstream Breakdown

Break the phase into workstreams using explicit boundary rules: prefer end-to-end thin slices over isolated layers; each workstream must be independently buildable and provable; seams should come from loose couplings identified in the terrain map.

**Why it exists:** This is the highest-leverage intellectual act in the entire workflow. A workstream with a wrong boundary is either untestable in isolation, incomplete when proven, or dependent on another workstream in ways that weren't declared. End-to-end slices are preferred because they surface integration failures that layer-level tests cannot catch — a service that works perfectly in isolation may fail when wired to its actual dependencies. The challenge question for each workstream ("is this independently buildable?") is the filter that catches wrong boundaries before they become blocked builds.

---

#### Step 5 — Write definition.md Per Workstream

For each approved workstream, write a `definition.md` that specifies scope, entry criteria, exit criteria, end-to-end proof description, interfaces, dependencies, and constraints.

**Why it exists:** The workstream boundary is only real if it's written. Without `definition.md`, "what's in scope for this workstream" is answered differently by different people on different days, and the build skill has no anchor. The exit criteria in `definition.md` are specifically the acceptance conditions that stories.md must cover — they are the contract between plan and build.

---

#### Step 6 — Write Phase plan.md

Write the phase summary with a workstream table showing risk profiles, dependencies, and statuses.

**Why it exists:** The build skill reads this artifact to understand which workstreams exist, which can run in parallel, and which must sequence. Without it, the transition from plan to build requires re-reading the entire `plan.md` and re-deriving the workstream structure.

---

#### Step 7 — Level 2 Certification

Present the workstream list and definitions. Ask for explicit approval. State next step.

**Why it exists:** This is the last cheap gate. After Level 2 is certified, build begins and boundary changes become expensive — code is being written against the definition. Certification forces a final check: does every workstream have a definition the engineer would approve, and does the overall breakdown reflect a coherent implementation strategy?

---

## /workflow:build — Prove Before You Close

### Intent

Translate certified workstream definitions into proven, owned code. The key word is *owned*: the engineer must be able to explain what was built and why it made the choices it made before the workstream is considered complete.

### Inputs / Outputs

| | |
|---|---|
| **Requires** | Feature name + workstream path + `definition.md` (from plan) + `plan.md` |
| **Produces** | `stories.md`, implementation code, `proof.md`, updated `patterns.md` / `testing.md` / `standards.md` |

---

### Step 1 — Bootstrap Workspace

Load `definition.md`, `plan.md`, `patterns.md`, `testing.md`, and `standards.md`. If `stories.md` exists, report the count of locked stories and ask whether to add or replace.

**Why it exists:** Build must not re-discover anything that plan already confirmed. Loading all context files first means the entire session operates against verified facts. The "add or replace" question for existing stories prevents a resumed session from silently discarding prior work.

---

### Step 2 — Standards Check

Before generating stories, ask whether there are conventions for this workstream not yet captured in `standards.md`.

**Why it exists:** Standards applied after stories are generated are applied unevenly — some stories follow them, some don't, depending on when the convention was mentioned in the session. Capturing all relevant standards before story generation means every story is generated against a complete constraint set. This also prevents the engineer from discovering mid-implementation that a pattern they expected was never communicated.

---

### Step 3 — Story Generation

Read `definition.md` and generate stories that translate exit criteria and proof description into discrete, independently executable units of work, prioritizing end-to-end slices first.

**Why it exists:** Exit criteria in `definition.md` are written at the workstream level. Stories are the mechanism that makes them executable — they break "the system handles X" into "as a [role], when [condition], the system should [observable behaviour]." The end-to-end priority is critical: a story that exercises the full path from input to output, even if it does very little, forces the integration wiring to be proven first. Stories that only test internal logic can all pass while the integration is broken. Acceptance conditions are required to be testable at the system boundary ("returns HTTP 200 matching schema X") rather than at the implementation boundary ("service method returns correct value") — because system-boundary conditions are the ones that matter and the ones that catch integration failures.

---

### Step 4 — Story Approval

Present each story to the engineer individually. Ask whether it is clear, correctly scoped, and complete enough to implement autonomously. Revise any story that doesn't pass. Lock approved stories.

**Why it exists:** A story the engineer can't confidently approve is a story that will produce uncertain code. The approval step builds a mental model in the engineer's head before any implementation begins — they are describing what they expect the code to do in advance, which means they can evaluate the implementation against an internal model rather than encountering it fresh at review. Locking stories after approval prevents the scope from drifting during implementation, which creates a gap between what was approved and what was built.

---

### Step 5 — Choose Execution Path

Ask the engineer whether to implement directly in this Claude session or to hand off to ralph-tui. If ralph-tui: generate `stories.json` from `stories.md` and stop.

**Why it exists:** Not all engineers run implementation inside Claude. Some prefer ralph-tui for interactive story-by-story control. The choice is made explicit here so the stories are formatted correctly for the target executor before any implementation begins. The conversion to `stories.json` is deterministic and one-way — it is always derived from `stories.md`, never hand-edited — so there is a single source of truth regardless of which path is taken.

---

### Step 6 — Execution Loop (Red / Green / Refactor)

For each story in order: write the failing proof first (red), implement the minimum to pass it (green), refactor structural debt (refactor), lock the story as passing. If a story cannot be completed as written, stop and ask whether to revise the story, route back to Level 2 to revise the workstream definition, or route back to Level 1 to revise the architecture.

**Why it exists:** Red-before-green is not a preference — it changes what code gets written. When tests are written after implementation, they are written to cover what the code already does, including its incorrect behaviours. When tests are written first, they are written to specify what the code *should* do, from the outside. The system-boundary constraint (acceptance conditions must be observable at the system boundary) ensures the tests are the kind that catch integration failures, not the kind that pass while the system misbehaves. The exit route back to plan when a story can't be completed is equally important: it prevents the engineer from accumulating technical debt from wrong assumptions by forcing a conscious decision about where the plan was wrong.

---

### Step 7 — Proof of Completion

After all stories pass: run the full proof suite for this workstream, then run the proof suite for any workstream this one depends on.

**Why it exists:** All stories passing proves the workstream. But a workstream that proves its own stories while breaking an earlier-proven workstream is not complete — it has introduced a regression. The dependency regression check surfaces this before the PR is opened, when it is still easy to address. Without this step, regressions accumulate across workstreams and surface only when the entire feature is assembled.

---

### Step 8 — Challenge

Walk through the implementation with the engineer, focused on coupling points, edge cases, error paths, and judgment calls. Map every acceptance condition in `stories.md` to a specific test. Ask: "Do you feel you own this code — could you defend its correctness in a PR review?"

**Why it exists:** Tests passing is not the same as the engineer understanding the code. AI-assisted implementation produces a specific failure mode: the engineer reviews a PR, the tests pass, and the code ships — but the engineer never built a mental model of what the code does or why it made the choices it made. When the next incident requires understanding this code, they have to re-read it as if they've never seen it. The challenge step is the mechanism that closes this gap while the implementation is fresh. The acceptance-condition-to-test mapping makes "coverage" concrete: for each thing the workstream was supposed to do, there is a named test that proves it does that thing.

---

### Step 9 — Archive Ritual

Exit interview (what was unexpected, what would you tell the next person, what standards should be universal). For each candidate pattern: apply the generalizability test ("would this prevent a failure in an unrelated future workstream?"). Promote only what passes. Update `plan.md` terrain if anything was discovered that corrects it. Present any CLAUDE.md promotions as a diff for explicit approval.

**Why it exists:** Institutional memory has a very short half-life. Patterns learned during an implementation are vivid and accessible immediately after the work is done and completely inaccessible three weeks later. The archive ritual captures them at the moment of maximum freshness. The generalizability test is the filter that prevents one-off solutions from being promoted as universal patterns — a pattern that only applies to this workstream is noise in a future session's context. The CLAUDE.md promotion step is specifically high-stakes: patterns promoted there apply to every future session, so they receive the highest standard of review.

---

### Step 10 — Write proof.md

Write a structured artifact covering story-to-test mapping, non-obvious implementation notes, coupling points, flags for PR reviewers, promoted patterns, lessons learned, and deferred items.

**Why it exists:** PR reviewers who weren't in the session have no context for non-obvious choices. Without `proof.md`, they either request explanations that take time, or they approve code they don't understand. The proof is written from the reviewer's perspective — it explains the things a competent reviewer would want to know but couldn't derive from reading the code alone. The coupling points table is particularly valuable: it documents where this code affects or is affected by other parts of the system, which is exactly what a reviewer needs to assess the blast radius of a change.

---

### Step 11 — Status Update

Update `status.md` with the completed workstream. If all workstreams in the phase are complete, say so. If this was the last workstream across all phases, declare the feature complete.

**Why it exists:** Progress state needs to survive session boundaries. Without updating `status.md`, the next session re-reads all workstream directories to reconstruct what's complete. The status update also makes the completion boundary explicit: a workstream is not done when the code is merged — it is done when the proof is written and the status is updated.

---

## The Artifact Chain

```
/workflow:plan <feature>
  └─ produces: plan.md, patterns.md, testing.md, standards.md

/workflow:plan <feature> <phase>
  └─ reads:    plan.md
  └─ produces: <phase>/plan.md
               <phase>/<workstream>/definition.md  (one per workstream)

/workflow:build <feature> <phase>/<workstream>
  └─ reads:    definition.md, plan.md, patterns.md, testing.md, standards.md
  └─ produces: stories.md (→ stories.json if ralph-tui)
               implementation code
               proof.md
               updated patterns.md, testing.md, standards.md
```

Each artifact is a contract between phases. `definition.md` is the contract between plan and build: it specifies exactly what build must prove. `proof.md` is the contract between build and the PR reviewer: it specifies what was proven and where. The chain is only as strong as the certification gates — which is why both plan and build require explicit engineer sign-off before the downstream phase begins.
