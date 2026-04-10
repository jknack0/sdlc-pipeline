# Debate Loop Protocol — How Agents and Their Adversarials Converge

This is the canonical protocol every phase agent uses to converge with its adversarial counterpart **before** producing its final output for the user. It replaces the older "agent drafts → user feedback → adversarial reviews → user feedback" pattern with a self-contained debate loop.

## Why this exists

Previously, the user was interrupted multiple times per phase: once to review the draft, once after the adversarial flagged issues, and again at the orchestrator gate. This created noise and pushed judgment calls onto the user that the agents could resolve themselves. The debate loop consolidates this: the agent and its adversarial argue it out internally, converge on a single deliverable, and present it once with a clear log of what was decided and what genuinely needs human input.

## The Loop

```
v1 draft  ─►  Adversarial Round 1  ─►  v2 (apply fixes)
                                           │
                                           ▼
                                  Adversarial Round 2  ─►  v3 (apply fixes)
                                                              │
                                                              ▼
                                                    Adversarial Round 3
                                                              │
                                                              ▼
                                                  ┌───────────────────────┐
                                                  │ Converged?            │
                                                  │ - 0 Critical AND      │
                                                  │ - 0 Major findings    │
                                                  └───────────────────────┘
                                                              │
                                          ┌───────────────────┴────────┐
                                          ▼                            ▼
                                       YES                          NO (cap hit)
                                          │                            │
                                          ▼                            ▼
                              Produce final + Decision Log    Produce final + Decision Log
                                                                  with open issues called out
```

## Convergence rules

**Stop the loop when EITHER:**

1. The adversarial returns **0 Critical and 0 Major** findings on its current review (Minor findings are acceptable — log them but don't block), OR
2. **3 rounds** have been completed (hard cap to prevent infinite loops on genuinely contentious work)

**Each round:**

- The agent dispatches the adversarial via the `Agent` tool with the current draft
- The adversarial returns categorized findings: **Critical / Major / Minor**
- If the loop is not yet converged, the agent revises:
  - **Critical findings:** must be addressed (fix the draft, or document why the finding doesn't apply with concrete reasoning — not "we accept the risk")
  - **Major findings:** address if straightforward, otherwise document in the Decision Log as a deliberate trade-off with rationale
  - **Minor findings:** apply silently or log as known limitations

**On the final round (round 3 with unresolved Critical/Major):** the agent still produces a final deliverable, but the Decision Log MUST surface the unresolved findings as items for user review. Do NOT silently ship a deliverable with known Critical issues — escalate them.

## Required Decision Log appendix

Every phase deliverable produced via the debate loop MUST include a `## Decision Log` section at the end with this structure:

```markdown
## Decision Log

### Decisions Made
| Decision | We chose | Why | Adversarial pushback |
|----------|----------|-----|----------------------|
| [the call] | [option chosen] | [reasoning] | [yes/no — if yes, summarize and why we held the line OR what we changed] |

### Alternatives Considered
- **[Alternative A]** — rejected because [specific reason, not "didn't fit"]
- **[Alternative B]** — rejected because [reason]

### Debate Summary
- **Rounds:** [N of 3]
- **Final adversarial verdict:** [N critical, N major, N minor]
- **Resolved this round:** [brief — what changed between v(N-1) and v(N)]
- **Open issues (if any):** [unresolved findings, with severity]

### For Your Review
*The 1–3 things that genuinely need your judgment. Not a summary of everything — just the load-bearing calls where reasonable people could disagree and the agent doesn't have authority to decide.*

1. **[Specific question]** — [context: why this needs human input, what the trade-off is]
2. **[Question]** — [context]
3. **[Question]** — [context]

*If there's nothing the user needs to weigh in on, write "Nothing — proceeding."*
```

## What goes in "For Your Review"

This section is the most important part of the Decision Log because it's what the user actually reads. Be ruthless about what belongs here.

**Include:**
- Strategic trade-offs the agent doesn't have authority to make (e.g., "prioritize speed over correctness for the MVP — yes/no?")
- Decisions where the adversarial and the agent could not agree on the right call
- Open Critical/Major findings the agent could not resolve within the 3-round cap
- Choices that depend on context only the user has (business priorities, internal politics, deadline constraints)

**Exclude:**
- Technical decisions the agent has enough information to make confidently
- Minor issues that don't change the shape of the deliverable
- Things already documented in "Decisions Made" with clear reasoning
- Anything the user already told the agent at the start of the phase

If you find yourself with more than 3 items, you're surfacing too much. Prioritize ruthlessly. The point of the debate loop is to *reduce* the cognitive load on the user, not relocate it.

## What NOT to do

- **Don't skip the loop "because it's a small change."** The loop is the process. If you genuinely don't need it, use `/fix` instead of the full pipeline.
- **Don't run more than 3 rounds.** If you can't converge in 3, the disagreement is real and belongs in "For Your Review."
- **Don't hide unresolved Critical findings** by burying them in Minor or omitting them from the Decision Log. The user needs to know.
- **Don't ask the user clarifying questions mid-loop.** Gather context once at the start (if needed), then run the loop to completion. The user only sees the final converged output.
- **Don't have the adversarial argue procedurally.** The adversarial reviews substance — the strength of the design, not the format of the document.
- **Don't reuse a stale review.** Each round dispatches a fresh adversarial call with the current draft. The adversarial does not "remember" prior rounds.

## Interactive phases (ideator, PO, UX)

These phases historically asked the user clarifying questions throughout drafting. Under the debate loop:

1. **Allowed:** ONE upfront message asking for any context the agent absolutely needs to begin (e.g., "What problem are you trying to solve?"). Batch the questions — do not ask one at a time.
2. **Not allowed:** "Present in sections and ask for approval after each section." Sections are an artifact of the deliverable, not a gate.
3. **Not allowed:** Asking the user to validate intermediate drafts. The loop is the validation.
4. **Required:** After convergence, present the final deliverable + Decision Log in a single message and let the user respond once.

If the user gives feedback after seeing the final output, the agent treats that as the start of a *new* debate loop: incorporate the feedback as a new constraint, re-draft, run the adversarial again, converge, present again. Each user-initiated revision = one new loop.

## Convergence reporting

When the agent hands off to the orchestrator, include a one-line debate summary in the handoff (not in the deliverable itself — that's what the Decision Log is for):

```
Debate: 2 rounds, converged (0 critical, 0 major, 1 minor logged)
```

or, if cap hit:

```
Debate: 3 rounds, cap hit (1 major unresolved — surfaced in Decision Log)
```

This lets the orchestrator log progress without re-reading the full deliverable.
