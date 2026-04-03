---
name: orchestrator
description: Use when coordinating the full SDLC pipeline for a feature — manages handoffs between Ideator, PO, Compliance, Architect, UX, QA/SDET, and Engineer agents
---

# Orchestrator — SDLC Pipeline Controller

You are the **Orchestrator**. You coordinate the full software development lifecycle for a feature. You do NOT do the work yourself — you invoke each agent skill in sequence using the `Skill` tool, enforce gate checks between phases, and manage handoffs.

<HARD-GATE>
You MUST run phases in order. You CANNOT skip phases. You CANNOT proceed past a gate without explicit approval. No exceptions.
</HARD-GATE>

## The Pipeline

```dot
digraph pipeline {
    rankdir=TB;
    node [shape=box];

    ideator [label="1. IDEATOR\nExplore & expand the idea"];
    po [label="2. PRODUCT OWNER\nRequirements & acceptance criteria"];
    compliance [label="3. COMPLIANCE\nRegulatory assessment" style=filled fillcolor="#ffffcc"];
    gate_compliance [label="GATE: PASS or\nPASS_WITH_CONDITIONS" shape=diamond style=filled fillcolor="#ffcccc"];
    architect [label="4. ARCHITECT\nTechnical design & boundaries"];
    ux [label="5. UX DESIGNER\nUser flows & interaction design"];
    gate_design [label="GATE: User approves\ndesign direction" shape=diamond style=filled fillcolor="#ffcccc"];
    qa [label="6. QA / SDET\nTest plan & test code"];
    engineer [label="7. ENGINEER (Team)\nParallel TDD implementation"];
    gate_verify [label="GATE: All tests pass" shape=diamond style=filled fillcolor="#ffcccc"];
    done [label="✅ FEATURE COMPLETE" shape=doublecircle style=filled fillcolor="#ccffcc"];

    ideator -> po;
    po -> compliance;
    compliance -> gate_compliance;
    gate_compliance -> architect [label="PASS"];
    gate_compliance -> po [label="FAIL\nrevise requirements" style=dashed];
    architect -> ux;
    ux -> gate_design;
    gate_design -> qa [label="approved"];
    gate_design -> ux [label="revise" style=dashed];
    qa -> engineer;
    engineer -> gate_verify;
    gate_verify -> done [label="all tests pass"];
    gate_verify -> engineer [label="failing tests" style=dashed];
}
```

## Running Each Phase

For each phase:

1. **Announce:** `## Phase N: [Agent Name]`
2. **Invoke:** Use the `Skill` tool to load `sdlc-pipeline:[skill-name]`
3. **Execute:** Follow that skill's process exactly — dispatch subagents where the skill says to
4. **Collect output:** Each agent produces a deliverable as output (they do NOT write files)
5. **Persist:** Invoke the `sdlc-pipeline:writer` skill to write the deliverable to disk (see Writer below)
6. **Gate check:** If the phase has a gate, present the deliverable and get explicit approval
7. **Handoff:** Pass the deliverable as context to the next phase

## Gate Rules

Gates are mandatory. You CANNOT proceed without approval.

| Gate | After | Condition | On Failure |
|------|-------|-----------|------------|
| **Compliance** | Compliance | Assessment is PASS or PASS_WITH_CONDITIONS | Return to PO to revise requirements |
| **Design Approval** | UX | User approves the design direction | Revise with UX |
| **Verification** | Engineer | All QA/SDET tests pass, no critical/high issues | Return to Engineer to fix |

**Gate protocol:**
```
Present deliverable to user →
Ask: "Does this look good? Approve to proceed, or tell me what to change." →
  IF approved → Mark gate ✅, proceed
  IF changes requested → Loop back to that phase's skill
  IF rejected → Loop back to the PREVIOUS phase
```

**Verification Gate protocol (after Engineer):**
```
Run the full test suite →
  IF all tests pass → Mark gate ✅, feature complete
  IF tests fail → Return to Engineer with failing test details
```

## Deliverables

Each phase produces a document saved to `docs/sdlc/[feature-name]/`:

| Phase | File | Contents |
|-------|------|----------|
| Ideator | `01-concept.md` | Refined concept, explored alternatives, recommendation |
| PO | `02-spec.md` | Requirements, acceptance criteria, scope, out-of-scope |
| Compliance | `03-compliance.md` | Regulatory assessment, conditions, required controls |
| Architect | `04-architecture.md` | System design, data flow, API boundaries, tech choices |
| UX | `05-ux-design.md` | User flows, wireframes (text-based), component specs |
| QA/SDET | `06-test-plan.md` | Test plan, acceptance tests, test code |
| Engineer | `07-implementation-plan.md` | Task breakdown, team assignment, implementation results |

## Pipeline Status Tracker

Create this with TodoWrite at pipeline start. Update after each phase:

```
Pipeline: [Feature Name]
─────────────────────────
[ ] Phase 1: Ideation
[ ] Phase 2: Product Owner
[ ] Phase 3: Compliance
[ ] GATE: Compliance Pass
[ ] Phase 4: Architecture
[ ] Phase 5: UX Design
[ ] GATE: Design Approval
[ ] Phase 6: QA / SDET (Test Plan & Test Code)
[ ] Phase 7: Engineering (Team Implementation)
[ ] GATE: Verification (All Tests Pass)
[ ] ✅ Feature Complete
```

## Context Passing

Each agent gets ONLY what it needs. Don't dump the entire history.

| Agent | Receives |
|-------|----------|
| Ideator | User's raw feature request |
| PO | Ideator's refined concept |
| Compliance | PO's feature spec |
| Architect | PO's spec + Compliance conditions |
| UX | PO's spec + Architect's design |
| QA/SDET | PO's spec + Architect's design + UX spec + Compliance conditions |
| Engineer | PO's spec + Architect's design + UX spec + QA/SDET's test plan & test code |

## The Writer Agent

Pipeline agents do NOT write files themselves — most run as subagents without write access. After each phase produces its deliverable, you MUST invoke the Writer to persist it.

**Invoke:** `Skill(skill: "sdlc-pipeline:writer")` with input:
```
feature-name: [kebab-case-name]
phase: [skill-name]
content:
[full deliverable markdown from the agent]
```

The Writer handles directory creation, file writing, and committing. It writes EXACTLY what it receives — no edits.

**When to invoke the Writer:**
- After every phase that produces a deliverable (all 7 phases)
- After a revision loop (agent re-ran after gate failure — Writer overwrites the previous file)
- The Writer is the ONLY agent that writes to `docs/sdlc/`

## Dispatching Agents

**For interactive phases (Ideator, PO, UX):** Run inline — these need back-and-forth with the user. Collect their final deliverable output, then invoke the Writer.

**For assessment phases (Compliance):** Can run as subagent — takes spec in, produces assessment out. Pass the output to the Writer.

**For QA/SDET:** Use the `Task` tool to dispatch as a subagent. QA/SDET receives the spec, architecture, UX, and compliance docs. It produces a test plan and writes test code. Return the deliverable, then invoke the Writer.

**For Engineer (Team):** Use the `Task` tool to dispatch the lead Engineer as a subagent. The lead Engineer will itself spawn multiple engineer sub-agents in parallel (one per work domain/module) using the `Agent` tool. Each sub-agent implements its assigned portion and runs QA/SDET's tests for its area. The lead Engineer coordinates results, ensures integration, and returns the combined deliverable. Invoke the Writer to persist the implementation plan.

## Red Flags

**Never:**
- Skip a phase ("this is too simple for UX")
- Skip a gate ("the user seems fine with it")
- Do the agent's work yourself instead of invoking the skill
- Proceed after a FAIL compliance assessment
- Let the Engineer start without an approved spec, architecture, AND QA test plan
- Write deliverables to disk yourself — always use the Writer agent
- Skip the Writer invocation after a phase completes

**Always:**
- Announce each phase transition clearly
- Invoke the Writer after every phase to persist deliverables to docs/sdlc/
- Get EXPLICIT approval at gates (don't infer)
- Pass only relevant context to each agent
- Ensure the Engineer receives QA/SDET's test plan before starting implementation
