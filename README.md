# Orchestrator

> Generate ready-to-paste Claude Code prompts with the right agent, orchestration pattern, and context wired up.

Describe a task. Get a prompt you can paste straight into Claude Code — with the right specialist selected, the right execution pattern chosen, and context/output paths configured.

## What it does

Orchestrator bridges multiple Claude Code ecosystems into a single prompt generator:

- **Agency Agents** — 150+ specialist agents at `~/.claude/agents/` (Software Architect, Security Engineer, Finance Tracker, Growth Hacker, etc.)
- **OMC Patterns** — orchestration patterns that control *how* agents execute (planning loops, parallel runs, autonomous mode)
- **ECC Patterns** — autonomous loop patterns, eval-driven development, and agentic engineering principles
- **Evolving** — designed to absorb new workflow patterns as they emerge from the community

## Orchestration patterns

| Pattern | Syntax | Use when |
|---------|--------|----------|
| **ralplan** | `ralplan "task"` | Important work needing a plan → execute → verify loop |
| **autopilot** | `autopilot: task` | Well-defined tasks you trust to run without checkpoints |
| **ralph** | `ralph: task` | Must be bulletproof — won't stop until verified |
| **team** | `/team N:executor "task"` | Parallel execution across multiple agents |
| **deep-interview** | `/deep-interview` | Nail down requirements before building |
| **scripted** | `claude --bare -p "task"` | CI/cron/headless automation |
| **loop** | Continuous iteration | Long-running tasks with exit conditions |
| **loop + ralph** | Loop with ralph per iteration | Long-running AND bulletproof per step |

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
— Run in parallel —
Use the Brand Guardian agent. ralplan "Write landing page copy..."
Use the SEO Specialist agent. ralplan "Keyword research and on-page SEO..."
Use the UI Designer agent. ralplan "Design the landing page wireframe..."

— Then reconcile —
Use the Software Architect agent. autopilot: "Reconcile outputs into a unified spec..."
```

**Eval-first — AI/ML task:**
```
Use the AI Engineer agent. ralplan "Build the recommendation engine.
Step 1: Define eval criteria (accuracy > 90%, latency < 200ms).
Step 2: Baseline current performance.
Step 3: Implement.
Step 4: Re-evaluate, only ship if all criteria pass.
Save eval results to 'evals/recommendation-eval.md'"
```

**Loop — iterative discovery:**
```
Use the AI Engineer agent. autopilot: Run a strategy discovery loop.
Each iteration: generate one signal variant, backtest 90 days, score.
Exit when: Sharpe > 1.5 and max drawdown < 8%.
Log to 'research/loop-log.md'. Cap at 20 iterations.
```

## Agent selection guide

| Domain | Agent(s) |
|--------|----------|
| Backend / API / architecture | Software Architect (design), Backend Architect (implementation) |
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
| Browser / GUI | Browser Automator |

## Installation

### As a Claude Code skill

```bash
mkdir -p ~/.claude/skills/orchestrator
cp SKILL.md ~/.claude/skills/orchestrator/SKILL.md
```

### Prerequisites

Works best with:

- [Oh My Claude Code](https://github.com/Yeachan-Heo/oh-my-claudecode) for orchestration patterns
- [Agency Agents](https://github.com/msitarzewski/agency-agents) at `~/.claude/agents/`

Without these, the generated prompts still work as structured Claude Code prompts — you just won't get the orchestration loops or specialist personas.

## Sources

This skill synthesises patterns from:

- [Oh My Claude Code](https://github.com/Yeachan-Heo/oh-my-claudecode) — orchestration patterns (ralplan, ralph, autopilot, team)
- [Agency Agents](https://github.com/msitarzewski/agency-agents) — 150+ specialist agent personas
- [Everything Claude Code](https://github.com/affaan-m/everything-claude-code) — autonomous loops, eval-driven development, agentic engineering principles

## Licence

MIT
