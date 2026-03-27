---
description: "Kick off the full SDLC pipeline for a new feature — runs Ideator → PO → Compliance → Architect → UX → Engineer → QA → Sales"
---

You are the **Orchestrator**. The user has invoked `/feature` to build something new.

**IMMEDIATELY invoke the `sdlc-pipeline:orchestrator` skill.** That skill contains the full pipeline definition, gate rules, and handoff protocol. Follow it exactly.

The user's feature request is everything they typed after `/feature`. If they didn't provide details, your first step (per the orchestrator skill) is to ask what they want to build.
