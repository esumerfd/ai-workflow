---
name: workflow:plan
description: "Phase 2: Define the desired outcome. Write requirements grounded in explore.md facts."
argument-hint: <feature-name>
---
# Skill: /workflow:plan

**Phase 2 — Define the Desired Outcome**

Write requirements grounded in confirmed facts from `explore.md`. Describe what the change should achieve — in abstract terms, without implementation details.

---

## When to Use This Skill

Use `/workflow:plan` when:
- `01-explore/explore.md` exists and is certified, OR terrain is already well understood
- You are ready to define what the change should achieve before deciding how

Skip to `/workflow:design` if requirements are already clear. Skip to `/workflow:refine` if both requirements and architecture are clear.

---

## Inputs

- Feature name (to locate the workspace)
- Optionally: a feature description, brief, or ticket reference to seed the interview

If the feature name is missing, ask before starting.

---

## Step 1 — Bootstrap Workspace

Follow `skills/_shared/workspace.md`.

Create if not present:
```
<feature>/02-plan/
<feature>/00-status/02-plan/
```

Set `<feature>/00-status/02-plan/status.md` to `status: started`.

If `<feature>/02-plan/requirements.md` already exists:
- Read it before starting.
- Summarize existing requirements and ask: resume (appending) or treat as a new planning session?

Load `<feature>/01-explore/explore.md` if it exists. Note any open questions and flagged couplings — these need disposition before requirements are written.

---

## Step 2 — Engineer Interview

The engineer articulates the feature in their own words first. The AI's paraphrase comes after — not before.

Ask in sequence:

1. "Describe the feature in your own words — what should it do and why?"
2. "Who benefits, and what problem does it solve for them?"
3. "What does success look like — how would you know the feature is working correctly?"
4. "What is explicitly out of scope for this change?"

After the engineer answers, paraphrase: "So if I understand correctly, you want to [paraphrase]. Is that right?"

Only proceed once the paraphrase is confirmed.

---

## Step 3 — Scope Declaration

Before writing requirements, make scope explicit and confirmed.

Ask:
1. "Are we changing only [area from explore.md], or does this touch other areas?"
2. "Are there any constraints on the approach — technical, team, timeline, or policy?"
3. "Are there any existing commitments (API contracts, data formats, interfaces) that must not change?"

Record the scope boundary. Requirements written outside this boundary are flagged.

---

## Step 4 — Disposition of explore.md Findings

If `explore.md` exists, every coupling and fragility flagged there needs explicit disposition before requirements are written. Read the couplings section and for each item ask:

> "The exploration flagged [coupling/fragility]. For this feature, should we: (a) address it with a requirement, (b) accept the risk and note the rationale, or (c) treat it as out of scope?"

Record each disposition. A requirement that ignores a known coupling without explanation is a hidden risk.

Open questions from `explore.md` must also be resolved or explicitly accepted:

> "The exploration left this question open: [question]. Do you want to resolve it now, or accept the uncertainty and proceed?"

---

## Step 5 — Requirement Drafting

With the interview and dispositions complete, draft requirements. For each requirement:

1. State it in plain language: "The system shall [behaviour] when [condition]."
2. Tie it to the engineer's stated goal or a confirmed explore.md observation.
3. Present it for confirmation before recording.

**Requirement flags:**
- If a requirement contradicts a confirmed observation in `explore.md`, flag it: "This requirement assumes [X], but the context map shows [Y]. Do you want to resolve this first?"
- If a requirement is ambiguous, surface it: "This could mean [A] or [B] — which do you intend?"
- If a requirement has a directly testable form, note it: "This can be expressed as an acceptance criterion — do you want to add one?"

---

## Step 6 — Challenge Questions

Before writing `requirements.md`, surface the two or three questions whose wrong answers would invalidate the requirements.

Examples:
- "Requirement R3 assumes the current API contract is stable — is that guaranteed for this release?"
- "R4 addresses the coupling flagged in explore.md — is the proposed decoupling feasible without a larger refactor?"
- "The scope excludes [area] — if that area changes, does it break any of these requirements?"

Ask each. Record the answer. Revise requirements if an answer changes the picture.

---

## Step 7 — Write requirements.md

Write `<feature>/02-plan/requirements.md`:

```markdown
# Requirements: <Feature Name>

**Session:** <ISO date>
**Based on:** 01-explore/explore.md (if used) | Engineer interview
**Status:** Draft | Confirmed

---

## Feature Summary

[Two to three sentences from the confirmed engineer paraphrase.]

## Scope

**In scope:**
- [item]

**Out of scope:**
- [item]

**Constraints:**
- [technical / team / policy constraints]

## Requirements

### R1 — <short title>
**The system shall** [behaviour] **when** [condition].
**Grounded in:** [explore.md observation or engineer statement]
**Acceptance criterion:** [testable condition, if defined]
**Status:** Confirmed

### R2 — <short title>
...

## Accepted Risks

| Risk | Source | Rationale for Acceptance |
|------|--------|--------------------------|
| [coupling or fragility from explore.md] | explore.md | [reason not addressed] |

## Superseded Requirements

[Requirements from prior sessions that no longer apply. Never deleted — marked here with reason.]

## Open Questions

- [ ] [question that was not resolved]

## Challenge Questions Asked

**Q:** [question]
**A:** [answer] → [impact: none / updated R[n]]
```

**Append-only rule:** requirements from prior sessions are never deleted. Superseded requirements are moved to the Superseded section with a note.

---

## Step 8 — Certification and Status Update

Present the completed `requirements.md`:

> "Here are the requirements. Before I mark this phase complete: are these the right requirements — ones you'd stand behind in a design discussion?"

Once confirmed:
1. Update `<feature>/00-status/02-plan/status.md` to `status: complete`.
2. Update `<feature>/00-status/status.md`.
3. State next phase: `/workflow:design <feature-name>` or `/workflow:decompose <feature-name>` if architecture is already clear.

---

## Rules

- **Engineer articulates first.** Do not propose requirements before the engineer has described the feature.
- **Every requirement is grounded.** No requirement without a source in the interview, explore.md, or a confirmed open question answer.
- **Contradictions are surfaced, not silently resolved.** A requirement that conflicts with explore.md is a flag, not a silent rewrite.
- **Append only.** Prior requirements are never deleted — only superseded with rationale.
- **Accepted risks are recorded.** Silence is not acceptance. Every flagged coupling gets a written disposition.
