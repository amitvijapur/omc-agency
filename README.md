# OMC Agency

> Generate ready-to-paste Claude Code prompts that combine [Oh My Claude Code](https://github.com/jasonm23/oh-my-claude-code) orchestration with [Agency](https://github.com/cline/agency) specialist agents.

Describe a task. Get a prompt you can paste straight into Claude Code — with the right agent selected, the right orchestration pattern chosen, and context/output paths wired up.

## What it does

OMC Agency bridges two powerful Claude Code ecosystems:

- **Agency Agents** — 150+ specialist agents installed at `~/.claude/agents/` (Software Architect, Security Engineer, Finance Tracker, Growth Hacker, etc.)
- **OMC Patterns** — orchestration patterns that control *how* agents execute (planning loops, parallel runs, autonomous mode)

Instead of manually figuring out which agent to use and how to structure your prompt, you describe what you want and OMC Agency generates the optimal prompt.

## Orchestration patterns

| Pattern | Syntax | Use when |
|---------|--------|----------|
| **ralplan** | `ralplan "task"` | Important work needing a plan → execute → verify loop |
| **autopilot** | `autopilot: task` | Well-defined tasks you trust to run without checkpoints |
| **team** | `/team N:executor "task"` | Parallel execution across multiple agents |
| **ralph** | `ralph: task` | Must be bulletproof — won't stop until verified |
| **deep-interview** | `/deep-interview` | Nail down requirements before building |

## Quick examples

**Single agent — architecture design:**
```
Use the Software Architect agent. ralplan "Design the auth pipeline.
Read 'CLAUDE.md' for project context.
Deliver: architecture doc, data flow diagram, API contracts.
Save to 'docs/auth_architecture.md'"
```

**Multi-agent parallel — landing page:**
```
— Agent 1: Brand Guardian —
Use the Brand Guardian agent. ralplan "Write landing page copy..."

— Agent 2: SEO Specialist —
Use the SEO Specialist agent. ralplan "Keyword research and on-page SEO..."

— Agent 3: UI Designer —
Use the UI Designer agent. ralplan "Design the landing page wireframe..."
```

**Quick task — autopilot:**
```
Use the Compliance Auditor agent. autopilot: Extract disclaimer texts
from 'docs/compliance.md' and create a reference file.
Save to 'docs/disclaimers.md'
```

## Agent selection guide

| Domain | Agent(s) |
|--------|----------|
| Backend / API / architecture | Software Architect, Backend Architect |
| Database / schema | Database Optimiser |
| Security / threat modelling | Security Engineer |
| AI / ML pipeline | AI Engineer |
| DevOps / infra / deploy | DevOps Automator |
| Compliance / legal | Compliance Auditor |
| Frontend / UI | UI Designer |
| UX research | UX Researcher |
| Brand / copy / voice | Brand Guardian |
| Growth / marketing | Growth Hacker |
| Finance / pricing / P&L | Finance Tracker |
| Content / social / blog | Content Creator, Twitter Engager |
| SEO | SEO Specialist |
| Testing / QA | QA Engineer |

## Installation

### As a Claude Code skill

Copy the `SKILL.md` file to your Claude Code skills directory:

```bash
# Create the skill directory
mkdir -p ~/.claude/skills/omc-agency

# Copy the skill file
cp SKILL.md ~/.claude/skills/omc-agency/SKILL.md
```

### Prerequisites

This skill works best with:

- [Oh My Claude Code](https://github.com/jasonm23/oh-my-claude-code) installed for orchestration patterns
- [Agency](https://github.com/cline/agency) agents installed at `~/.claude/agents/`

Without these, the generated prompts will still work as structured Claude Code prompts — you just won't get the orchestration loops or specialist personas.

## How it works

1. You describe a task (in Claude.ai or Claude Code)
2. The skill selects the right agent based on domain
3. It picks the best orchestration pattern based on complexity
4. It generates a copy-paste prompt with context paths and output destinations
5. You paste it into Claude Code and the agent executes

For multi-domain tasks, it generates parallel prompts with a reconciliation step.

## Roadmap

- [ ] Custom domain agents (financial research, deep research, etc.)
- [ ] MCP integration patterns
- [ ] Multi-agent workflow templates
- [ ] Community-contributed agent configurations

## Contributing

Contributions welcome — especially new agent configurations and workflow templates. Open a PR or issue.

## Licence

MIT
