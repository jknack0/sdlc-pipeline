# SDLC Pipeline — Claude Code Plugin

A full software development lifecycle pipeline for Claude Code. Type `/feature` and watch your idea go from concept to shipped code.

## The Pipeline

```
/feature "I want to build X"
        │
        ▼
   ┌──────────┐
   │ IDEATOR  │  Explore, challenge, refine the idea
   └────┬─────┘
        ▼
   ┌──────────┐
   │    PO    │  Requirements, acceptance criteria, scope
   └────┬─────┘
        ▼
   ┌────────────┐
   │ COMPLIANCE │  GDPR / HIPAA / SOC2 (parallel sub-agents)
   └────┬───────┘
        │ auto-check: PASS → continue, FAIL → back to PO
        ▼
   ┌───────────┐
   │ ARCHITECT │  System design (parallel sub-agents)
   └────┬──────┘
        ▼
   ┌──────────┐
   │    UX    │  User flows, states, interactions
   └────┬─────┘
        ▼
   ┌──────────┐
   │ QA/SDET  │  Test plan & test code (parallel sub-agents)
   └────┬─────┘
        ▼
   ┌─ GATE ───┐  User reviews design, UX & test plan
   └────┬─────┘
        ▼
   ┌───────────────┐
   │ ENGINEER TEAM │  Parallel TDD implementation
   └──────┬────────┘
        │ auto-verify: tests pass → done, fail → retry
        ▼
    ✅ DONE
```

## Installation

### Claude Code Plugin Marketplace

```
/plugin install sdlc-pipeline
```

### Manual Install

Clone this repo into your Claude Code plugins directory:

```bash
cd ~/.claude/plugins
git clone <repo-url> sdlc-pipeline
```

Or register as a local plugin:

```bash
/plugin add /path/to/sdlc-pipeline
```

### Verify

Start a new Claude Code session. You should see the pipeline is available. Type `/feature` to kick it off.

## Usage

```
/feature I want to add user authentication with OAuth2 and email/password
```

The orchestrator takes over and runs each phase autonomously. Each phase includes an adversarial agent that stress-tests the output before moving on. You only need to approve once:

- **After QA/SDET** — The pipeline pauses to show you the architecture, UX design, and test plan. This is where you see what the feature will look like and approve or request changes.

Everything else is automatic — compliance failures loop back to the PO, and test failures loop back to the Engineer without needing your input.

## How It Works

The pipeline uses **test-first development**, **parallel sub-agents**, and **adversarial review**:

- **Parallel sub-agents** — Compliance runs GDPR/SOC2/HIPAA assessors in parallel. Architect splits across data model, API, and infrastructure concerns. QA/SDET writes tests per domain in parallel. Engineer dispatches implementation agents per domain.
- **Adversarial agents at every step** — Each phase (except Engineer) includes an adversarial agent that attacks the output: a devil's advocate for ideas, a spec critic for requirements, a red-teamer for compliance, an architecture attacker, a UX critic, and a test plan critic.
- **Test-first** — QA/SDET writes executable tests before implementation. The Engineer's job is to make them pass.
- **One approval point** — You review everything (architecture + UX + tests) at a single gate after QA/SDET. Compliance and verification are automatic.

## What Gets Produced

Each feature creates a documentation trail in `docs/sdlc/[feature-name]/`:

| File | Agent | Contents |
|------|-------|----------|
| `01-concept.md` | Ideator | Refined concept, alternatives, recommendation |
| `02-spec.md` | PO | Requirements, acceptance criteria, scope |
| `03-compliance.md` | Compliance | Regulatory assessment, conditions |
| `04-architecture.md` | Architect | System design, data model, APIs |
| `05-ux-design.md` | UX | User flows, states, interactions |
| `06-test-plan.md` | QA/SDET | Test plan, acceptance tests, test code |
| `07-implementation-plan.md` | Engineer | Task breakdown, team assignment, results |

## Customization

### Compliance Frameworks

Edit `skills/compliance/SKILL.md` to add or remove regulatory frameworks. The default set covers GDPR, HIPAA, and SOC 2.

### Agent Behavior

Each agent is a self-contained skill in `skills/`. Edit any SKILL.md to adjust that agent's process, rules, or output format.

### Pipeline Order

The pipeline order is defined in `skills/orchestrator/SKILL.md`. You can reorder phases, add new agents, or remove phases by editing the orchestrator.

### Adding New Agents

1. Create a new directory in `skills/[agent-name]/`
2. Add a `SKILL.md` with frontmatter (`name` and `description`)
3. Add the phase to the orchestrator's pipeline definition
4. Define what deliverable it produces and what gate (if any) follows it

## License

MIT
