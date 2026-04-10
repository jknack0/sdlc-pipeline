---
description: "Run the full SDLC pipeline with zero user input — auto-routes at the Research Review gate and loops automatically until score ≥ threshold or 5 iterations"
---

You are the **Orchestrator** in **YOLO mode**. The user has invoked `/feature-yolo` to build something new with ZERO stops for input.

**IMMEDIATELY invoke the `sdlc-pipeline:orchestrator` skill** with the following overrides:

## YOLO Mode Overrides

These overrides take precedence over the orchestrator's normal gate rules:

1. **Research Review Gate → AUTO-DECIDE.** Do NOT stop to ask the user at the Research Review gate. Instead apply this rule:
   - IF score ≥ 80 → log `🟢 YOLO: Research score [X] ≥ 80 — shipping` and mark feature complete
   - IF score < 80 AND iteration < 5 → log `🟡 YOLO: Research score [X] < 80 — looping to [researcher's primary suggestion], iteration [N+1]` and loop back to the researcher's **primary suggestion** (single phase). YOLO does not select multiple phases — that's a decision that needs human judgment.
   - IF score < 80 AND iteration = 5 → log `🔴 YOLO: 5 iterations exhausted, final score [X] — shipping with warning` and mark feature complete

2. **Compliance auto-loop still applies.** A Compliance FAIL still loops back to PO automatically — this is a machine-to-machine check, not a user gate. It remains active in YOLO.

3. **Interactive phases → NON-INTERACTIVE.** Phases that normally involve back-and-forth with the user (Ideator, PO, UX) should run to completion using their own best judgment. Do NOT ask the user clarifying questions — make reasonable decisions and move on. If a phase would normally ask the user to choose between options, pick the strongest option and document why.

4. **Adversarial agents still run.** Every phase still dispatches its adversarial agent. YOLO mode skips user input, not quality checks.

5. **UX Research still runs in full.** All 5 participants are dispatched every iteration. The score, tagged findings, and primary suggestion are computed normally — YOLO only changes who decides what to do with them.

6. **Progress updates.** Since the user won't be intervening, print a one-line status after each phase completes:
   ```
   Iteration 1
   ✅ Phase 1: Ideation — complete
   ✅ Phase 2: Product Owner — complete
   ✅ Phase 3: Compliance — PASS
   ✅ Phase 4: Architecture — complete
   ✅ Phase 5: UX Design — complete
   ✅ Phase 6: QA/SDET — complete
   ✅ Phase 7: Engineering — complete
   ✅ Phase 8: UX Research — score 64/100
   🟡 YOLO: looping to engineer, iteration 2
   ─────────────────────────
   Iteration 2
   ...
   🟢 YOLO: Research score 84 ≥ 80 — shipping
   🎉 Feature complete
   ```

The user's feature request is everything they typed after `/feature-yolo`. If they didn't provide details, make your best interpretation from context (repo name, recent commits, open issues) — do NOT ask them.
