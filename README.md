# SDLC Pipeline — Claude Code Plugin

A full software development lifecycle pipeline for Claude Code. Type `/feature` and watch your idea go from concept to shipped code with customer docs.

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
   ┌─ GATE ───┐  User approves spec
   └────┬─────┘
        ▼
   ┌────────────┐
   │ COMPLIANCE │  GDPR / HIPAA / SOC2 assessment
   └────┬───────┘
        ▼
   ┌─ GATE ───┐  Must PASS to proceed
   └────┬─────┘
        ▼
   ┌───────────┐
   │ ARCHITECT │  System design, data flow, API boundaries
   └────┬──────┘
        ▼
   ┌──────────┐
   │    UX    │  User flows, states, interactions
   └────┬─────┘
        ▼
   ┌─ GATE ───┐  User approves design
   └────┬─────┘
        ▼
   ┌──────────┐
   │ ENGINEER │  TDD implementation
   └────┬─────┘
        ▼
   ┌──────────┐
   │    QA    │  Independent verification & sign-off
   └────┬─────┘
        ▼
   ┌─ GATE ───┐  No critical/high issues
   └────┬─────┘
        ▼
   ┌──────────┐
   │  SALES   │  Customer docs & talking points
   └────┬─────┘
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

The orchestrator takes over and runs each phase. You'll be asked for input at gate checkpoints:

1. **After Ideation** — Orchestrator presents refined concept
2. **After PO** — You approve or revise the feature spec
3. **After Compliance** — Assessment must pass (automatic gate)
4. **After UX** — You approve or revise the design
5. **After QA** — Sign-off must pass (automatic gate)

## What Gets Produced

Each feature creates a documentation trail in `docs/sdlc/[feature-name]/`:

| File | Agent | Contents |
|------|-------|----------|
| `01-concept.md` | Ideator | Refined concept, alternatives, recommendation |
| `02-spec.md` | PO | Requirements, acceptance criteria, scope |
| `03-compliance.md` | Compliance | Regulatory assessment, conditions |
| `04-architecture.md` | Architect | System design, data model, APIs |
| `05-ux-design.md` | UX | User flows, states, interactions |
| `06-implementation-plan.md` | Engineer | Task breakdown with TDD steps |
| `07-test-report.md` | QA | Verification results, sign-off |
| `08-release-materials.md` | Sales | Customer docs, talking points, FAQ |

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
