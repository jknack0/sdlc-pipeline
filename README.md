# SDLC Pipeline Guide

The SDLC Pipeline is a Claude Code plugin that runs a full software development lifecycle for features — from idea exploration through implementation and synthetic user testing — using specialized AI agents at each phase with adversarial quality checks.

## Quick Start

### Run the full pipeline (with user approval gates)

```
/feature Add secure messaging between clinicians and clients
```

You'll be asked to review and approve at the Research Review gate after UX Research scores the implementation.

### Run the full pipeline hands-free

```
/feature-yolo Add secure messaging between clinicians and clients
```

Zero stops. Auto-ships when score >= 80, auto-loops when < 80, hard-stops at 5 iterations.

### Fix a small bug or make a minor change

```
/fix The co-sign banner doesn't show the supervisor's name
```

Lightweight pipeline — skips Ideation, PO, Compliance, and UX Research.

## What It Produces

Every pipeline run creates a `docs/sdlc/<feature-name>/` directory with these deliverables:

| File | Phase | What's In It |
|------|-------|-------------|
| `01-concept.md` | Ideator | Problem statement, approach options, recommendation, alternatives explored |
| `02-spec.md` | Product Owner | Functional requirements, Given/When/Then acceptance criteria, NFRs, scope |
| `03-compliance.md` | Compliance | HIPAA + SOC 2 + GDPR assessment, PASS/FAIL verdict, binding conditions |
| `04-architecture.md` | Architect | Data model, API design, system diagram, compliance controls map, file structure |
| `05-ux-design.md` | UX Designer | User flows, component states, interactions, copy, accessibility notes |
| `06-test-plan.md` | QA/SDET | Test plan + executable test code organized by domain |
| `07-implementation-plan.md` | Engineer | Task breakdown, dispatch waves, coverage map, self-audit |
| `08-research-report-iter-N.md` | UX Researcher | Score, persona reports, tagged findings (one per iteration) |

The `/fix` pipeline produces a single `fixes/fix-YYYY-MM-DD-<slug>.md` file.

## The Pipeline Phases

### Phase 1: Ideator

Takes a raw feature idea and stress-tests it before anyone writes requirements. Explores 2+ alternative approaches, challenges assumptions, and identifies risks.

**Adversarial:** Devil's Advocate tries to kill the idea. Asks "why will this fail?" and "what's the simpler alternative?"

**Output:** A refined concept with clear direction and documented trade-offs.

### Phase 2: Product Owner

Translates the concept into precise, testable requirements. Every requirement gets Given/When/Then acceptance criteria specific enough that an engineer can implement without guessing and QA can verify without interpretation.

**Adversarial:** Spec Critic finds ambiguity, missing edge cases, untestable criteria, and scope gaps.

**Output:** A complete feature spec with functional requirements, NFRs, authorization matrix, scope boundaries.

### Phase 3: Compliance

Assesses the spec against regulatory frameworks before any design work begins. Always runs GDPR and SOC 2; adds HIPAA when healthcare data is involved (which it always is for Steady).

**Verdict is one of three:**
- **PASS** — proceed freely
- **PASS_WITH_CONDITIONS** — proceed but conditions (COND-1, COND-2, etc.) are binding on downstream phases
- **FAIL** — auto-loops back to PO to revise the spec (no user involvement)

**Adversarial:** Compliance Red-Teamer thinks like a regulator, a plaintiff's attorney, and a breach investigator.

**Output:** Per-framework assessment tables, numbered conditions that Architecture and Engineering must satisfy.

### Phase 4: Architect

Designs the technical solution. Dispatches three sub-agents in parallel (Data Model, API Design, Infrastructure), then consolidates into a unified architecture. Must incorporate every compliance condition.

**Adversarial:** Architecture Attacker finds security holes, scalability bottlenecks, race conditions, and compliance gaps.

**Output:** System diagram, Prisma schema, API endpoints with request/response shapes, data flow diagrams, compliance controls map, complete file structure.

### Phase 5: UX Designer

Designs how humans interact with the feature. Covers every flow (happy path, error, edge case), every component state (default, loading, error, empty, disabled), and all copy/labels.

**Adversarial:** UX Critic thinks like an impatient user, a first-time user, and a user with accessibility needs.

**Output:** User flows, component state tables, interaction specs, content/copy table, accessibility notes.

### Phase 6: QA/SDET

Writes the test plan and executable test code BEFORE implementation. Tests define "done" — the Engineer's job is to make them pass.

**Adversarial:** Test Plan Critic finds missing acceptance criteria coverage, missing compliance verification, missing edge cases.

**Output:** Coverage tables (AC, compliance, UX flows) + runnable test files organized by domain.

### Phase 7: Engineer (Team)

The heaviest phase. A Lead Engineer dispatches a Senior Engineer for task breakdown, then coordinates parallel waves of engineer sub-agents — one per task. After all waves complete, runs an Architecture Coverage Check to verify every component from the architecture doc exists in code.

**Dispatch flow:**
1. Senior Engineer reads full architecture + test plan, produces granular task list
2. Lead Engineer dispatches tasks in parallel waves (independent tasks in same wave)
3. Each sub-agent implements one task with its architecture excerpt embedded in the prompt
4. Lead Engineer integrates, runs full test suite, verifies architecture coverage

**Output:** Task list with coverage map, dispatch order, per-task results.

### Phase 8: UX Researcher

Runs a synthetic user study with 5 personas against the implemented feature. Personas are weighted toward older/less tech-savvy users (3 of 5 must be low-comfort) because they surface integration gaps fastest.

**Scoring (0-100, threshold 80):**
- Reachability (25pts) — can users find the feature?
- Comprehension (25pts) — do they understand what they're looking at?
- Task Completion (25pts) — can they finish the task?
- Satisfaction (25pts) — how do they feel about it?

**Hard caps prevent soft grading:**
- Any participant can't reach the feature → Reachability capped at 10/25, overall capped at 40
- 3+ can't complete → Task Completion capped at 10/25
- 2+ older personas give up in frustration → Satisfaction capped at 10/25

**Output:** Score, tagged findings (each tagged with the phase that could fix it), per-participant reports.

## The Research Review Gate

This is the ONE user-facing decision point in the pipeline. After each UX Research iteration, you see:

```
Score: 71 / 100   (threshold: 80)
Status: NEEDS_REVISION

Findings:
  1. "Notes" not in sidebar nav    → engineer
  2. DueBadge always null           → engineer
  3. No co-sign banners             → engineer

Researcher's primary suggestion: engineer
```

**Your options:**
- **Ship it** — accept current state, mark feature complete
- **Abort** — stop the pipeline, deliverables stay in place
- **Loop** — pick one or more phases to re-run (e.g., just engineer, or ux-designer + engineer)

In YOLO mode, this gate is auto-decided: ship if >= 80, loop to the researcher's primary suggestion if < 80, hard-stop at 5 iterations.

## The Adversarial Debate Protocol

Every phase (except Engineer) runs an internal quality check:

1. Agent produces a draft
2. Adversarial agent tears it apart (up to 3 rounds)
3. Agent addresses Critical/Major findings, logs Minor ones
4. Stop when 0 Critical + 0 Major, or 3 rounds hit

Every deliverable ends with a **Decision Log** appendix:
- **Decisions Made** — what was chosen, why, what the adversarial pushed back on
- **Alternatives Considered** — what was rejected and why
- **Debate Summary** — rounds, final verdict, what changed
- **For Your Review** — max 3 items needing human judgment (or "Nothing — proceeding")

## How to Use Each Command

### `/feature` — Full pipeline with gates

Best for: New features where you want to review the design before implementation.

```
/feature Add client self-scheduling with online booking widget
```

The pipeline runs straight through Phases 1-8. You only stop at the Research Review gate. Compliance FAIL auto-loops to PO without asking you.

**Providing context helps:** The more detail you give, the fewer clarifying questions the Ideator and PO ask.

```
/feature Add client self-scheduling. Clients should see available slots
on a public booking page, select a time, and confirm. Clinician sets
availability windows in practice settings. No payment at booking.
Reminders via existing email/push system.
```

### `/feature-yolo` — Full pipeline, zero stops

Best for: When you trust the pipeline to iterate and want to come back to finished work.

```
/feature-yolo Add waitlist management with smart matching
```

All gates auto-decided. Interactive phases (Ideator, PO, UX) make their own best judgment. Ships automatically when research score hits 80+. Max 5 iterations.

### `/fix` — Lightweight pipeline for small changes

Best for: Bug fixes, small UI tweaks, minor behavior changes to already-implemented features.

```
/fix The overdue badge should show relative time like "3h ago" not just "Overdue"
```

Runs: Architect + UX (parallel) → SDET → Engineer. No Ideation, no PO, no Compliance, no UX Research. Produces a fix report in `docs/sdlc/<feature>/fixes/`.

The fix orchestrator will refuse and redirect to `/feature` if the change is too large.

## Invoking Individual Agents

You can invoke any agent directly via the Skill tool if you want to run just one phase:

```
/sdlc-pipeline:ideator Explore the idea of adding ePrescribing
/sdlc-pipeline:architect Design the architecture for secure messaging (spec at docs/sdlc/secure-messaging/02-spec.md)
/sdlc-pipeline:compliance Assess docs/sdlc/client-scheduling/02-spec.md
```

This is useful for:
- Re-running a single phase after making manual edits to an upstream doc
- Getting a compliance assessment on a spec you wrote yourself
- Running just the architect on an external design

## Tips

### Start with YOLO for exploratory work
If you're not sure about scope or approach, `/feature-yolo` will explore alternatives, challenge assumptions, and iterate without needing your input. Review the deliverables after.

### Read the Decision Logs
Every deliverable's Decision Log tells you what was decided, what was considered, and why. This is where the adversarial debate surfaces — it's the most valuable part of each doc for understanding trade-offs.

### The compliance conditions flow downstream
When Compliance produces COND-1 through COND-N, those conditions are binding. The Architect must address each one, QA must test each one, and the Engineer must implement each one. Check the compliance controls map in `04-architecture.md` to trace how each condition is satisfied.

### UX Research is a code audit, not just a review
The UX Researcher dispatches synthetic users who actually try to use the running app. They find integration gaps that no amount of spec review catches — entry points not wired, navigation missing, components not connected.

### The test plan is the implementation contract
QA writes tests BEFORE the Engineer implements. The Engineer's job is to make those tests pass. This is test-driven development at the feature level.

### Iteration history is preserved
Each UX Research iteration produces a separate file (`08-research-report-iter-1.md`, `iter-2.md`, etc.). The score trajectory tells you whether iterations are improving or regressing.

## File Structure

```
docs/sdlc/
  <feature-name>/
    01-concept.md              # Ideator output
    02-spec.md                 # PO output
    03-compliance.md           # Compliance output
    04-architecture.md         # Architect output
    05-ux-design.md            # UX Designer output
    06-test-plan.md            # QA/SDET output
    07-implementation-plan.md  # Engineer output
    08-research-report-iter-1.md  # UX Research iteration 1
    08-research-report-iter-2.md  # UX Research iteration 2 (if looped)
    fixes/
      fix-2026-04-12-cosign-banner.md  # Fix reports
```

## Example: What a Full Run Looks Like

```
/feature-yolo Add clinical progress notes with AI-assisted drafting

Iteration 1
  Phase 1: Ideation — complete (3 adversarial rounds)
  Phase 2: Product Owner — complete (11 FRs, 94 ACs)
  Phase 3: Compliance — PASS_WITH_CONDITIONS (15 binding conditions)
  Phase 4: Architecture — complete (13 models, 6 routes)
  Phase 5: UX Design — complete (11 flows, 6 component specs)
  Phase 6: QA/SDET — complete (94 tests)
  Phase 7: Engineering — complete (19 tasks, 6 waves)
  Phase 8: UX Research — score 71/100
  YOLO: score 71 < 80 — looping to engineer

Iteration 2
  Engineering fixes — 3 critical + 6 major bugs fixed
  UX Research — score 68/100 (regression found)
  YOLO: score 68 < 80 — looping to engineer

Iteration 3
  Engineering fixes — 1 critical regression + 3 major fixed
  UX Research — score 82/100 PASS
  YOLO: score 82 >= 80 — shipping

Feature complete. 10 deliverable docs produced.
```

## License

MIT
