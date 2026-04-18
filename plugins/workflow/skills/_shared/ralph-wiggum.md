# Ralph Wiggum Technique

The Ralph Wiggum technique is a method for building mutually certified, factual observations before acting on them. It is named after the character's habit of stating things plainly and literally.

## The Core Rule

State each observation in plain language. Confirm it with the engineer before recording it. Do not interpret, infer, or embellish.

**Wrong:** "The UserService appears to be the central orchestrator of the authentication subsystem, likely managing session state."

**Right:** "UserService has methods `login`, `logout`, and `refreshToken`. I see it is imported by `AuthController` and `SessionMiddleware`. Is that correct?"

## How It Works

1. Make one observation at a time.
2. State it plainly — what you see in the code, not what you think it means.
3. Ask: "Is that correct?" or "Does that match your understanding?"
4. If confirmed: record it as a fact.
5. If corrected: record the correction, not the original observation.
6. Never batch multiple observations into a single confirmation request.

## Stakes-Weighted Treatment

Not all observations are equally risky. Weight your confirmation effort by consequence:

- **Low stakes** (function names, file counts, method signatures): light confirmation — "I see X, Y, Z in this file — does that look right?"
- **Medium stakes** (module responsibilities, ownership boundaries): confirm each separately.
- **High stakes** (coupling assessments, data flows, failure modes): confirm carefully and ask a challenge question before recording.

## Coupling Observations

Couplings deserve special treatment. A coupling is any observation where changing one thing requires changing another. These are candidate decomposition seams and potential sources of risk.

For each coupling, confirm:
1. The nature of the coupling (data shared, call dependency, temporal, external)
2. Whether it is intentional or incidental
3. Whether it is a candidate seam or must be treated as a single unit

## Recording Confirmed Observations

Once confirmed, record observations in this format:

```
## [Area Name]

**Confidence:** High | Medium | Low | Unverified

- [Observation 1]
- [Observation 2]
- **Coupling:** [description] — [type: data / call / temporal / external]
```

Only write to the artifact after the engineer confirms the observation. A disagreement is a finding — record it, do not discard it.
