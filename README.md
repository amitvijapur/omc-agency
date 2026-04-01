# Orchestrator

> Workflow orchestration for Claude Code — the right agent, pattern, coordination topology, and safety rails for any task.

Describe a task. Orchestrator selects the right specialist, picks the execution pattern, wires up context paths, adds safety rails, and outputs a ready-to-paste prompt.

---

## Why this exists

Claude Code has 150+ specialist agents, multiple orchestration frameworks (OMC, ECC), and dozens of workflow patterns scattered across different repos. Using them effectively means knowing which agent to pick, which pattern to use, how to coordinate multi-agent runs, when to add safety rails, and how to structure the prompt so the agent actually delivers.

Orchestrator knows all of this so you don't have to. You describe what you want, it generates the optimal prompt.

---

## How it works

```
You: "I need to build a payment webhook handler — it has to be secure"

Orchestrator generates:

  — Step 1: Build —
  Use the Backend Architect agent. ralplan "Implement the payment webhook handler.
  Read 'CLAUDE.md' and 'docs/stripe-integration.md'.
  Save to 'src/webhooks/stripe.ts'"

  — Step 2: Multi-model review —
  /ccg "Review src/webhooks/stripe.ts for correctness, security, and edge cases"

  — Step 3: Sentinel gate —
  Use the Security Engineer agent. autopilot: "Audit src/webhooks/stripe.ts.
  Check for: signature verification, idempotency, error leakage, missing event types.
  Block on critical issues, warn on medium. Save to 'reviews/sentinel-audit.md'"
```

Three agents, three steps, zero ambiguity. Copy, paste, execute.

---

## What's inside

### 9 Orchestration Patterns

Every way you can run agents in Claude Code, with clear guidance on when to use each:

| Pattern | What it does | When to use it |
|---------|-------------|----------------|
| **ralplan** | Plan → execute → verify loop | **Default for serious work.** Architecture, features, anything that matters. |
| **autopilot** | One-shot autonomous execution | Well-defined tasks you trust. Quick wins. |
| **ralph** | Won't stop until verified complete | Must be bulletproof. Security fixes, production deploys. |
| **team** | N agents working in parallel | Large tasks benefiting from multiple perspectives. |
| **deep-interview** | Socratic requirements clarification | When you're not sure what you're building yet. |
| **scripted** | Headless `claude --bare -p` | CI/CD pipelines, cron jobs, automation scripts. |
| **loop** | Iterative execution with exit conditions | Strategy discovery, monitoring, migrations. |
| **loop + ralph** | Loop where each iteration is ralph-verified | Long-running AND must be bulletproof per step. |
| **sentinel** | Separate auditor agent reviews output | Quality gate before shipping. Different agent than the builder. |

**Why this matters:** Without a decision framework, you default to autopilot for everything or waste time figuring out which pattern fits. The skill's pattern selection logic (Rule 2) maps task characteristics directly to the right pattern — no thinking required.

---

### 6 Prompt Generation Rules

These rules shape every prompt the skill generates. Each one prevents a specific class of failure:

#### Rule 1: Context pointers + output destinations
Every prompt includes `Read '[file]'` and `Save to '[path]'`. Without these, agents hallucinate project context and dump output to random locations.

#### Rule 2: Pattern selection
A flat decision map: task type → pattern. No branching logic, no ambiguity. Includes `--channels` for remote approval on long-running tasks, and explicit computer use scoping (target URL, fallback, single-app rule).

#### Rule 3: Agentic engineering principles
Four sub-principles that prevent waste:

- **Eval-first:** For AI/ML tasks, define success criteria before building. Prevents "built the wrong thing" loops.
- **15-minute rule:** Decompose tasks that exceed ~15 min of focused agent work. Prevents mega-prompts that agents choke on.
- **Model routing:** Haiku for boilerplate, Sonnet for features, Opus for architecture. Prevents overspending on simple tasks.
- **Skill extraction:** After corrections, prompt the agent to run `/learner` and capture the pattern. Prevents repeating the same mistake.

#### Rule 4: Inter-agent memory
For multi-agent or multi-session tasks, agents share state via a namespaced file (`shared/agent-state.md`). Each agent reads prior findings at start, appends their own at end. Without this, parallel agents duplicate work or contradict each other.

#### Rule 5: Multi-model review
For high-stakes outputs, get external perspectives via `/ask codex` (correctness) and `/ask gemini` (scale/performance), or `/ccg` for full tri-model orchestration. **Critical safety mechanism:** Claude independently evaluates each external suggestion (AGREE/REJECT) before applying — never auto-applies. Prevents blindly accepting bad advice from a different model.

#### Rule 6: Sentinel gate
After a build completes, a DIFFERENT specialist agent audits the output before shipping. Three gate levels:
- **warn** — log the concern, continue
- **review** — flag for human decision
- **block** — reject and send back to the builder

The sentinel must always be a different agent than the builder to avoid same-model blind spots (the same model that wrote the bug won't catch it in review).

---

### 15-Domain Agent Selection

The skill maps 15 domains to the right specialist agent(s), so you don't need to know which of the 150+ agents to use:

| Domain | Agent(s) |
|--------|----------|
| Backend / API / architecture | Software Architect (design), Backend Architect (implementation) |
| Database / schema | Database Optimiser |
| Security / threats | Security Engineer |
| AI / ML pipeline | AI Engineer |
| DevOps / infra | DevOps Automator |
| Compliance / legal | Compliance Auditor |
| Frontend / UI | UI Designer |
| UX research | UX Researcher |
| Brand / copy | Brand Guardian |
| Growth / marketing | Growth Hacker |
| Finance / pricing | Finance Tracker |
| Content / social | Content Creator, Twitter Engager |
| SEO | SEO Specialist |
| Testing / QA | QA Engineer |
| Browser / GUI | Browser Automator |

For multi-domain tasks, the skill generates parallel prompts with a reconciliation step — one agent synthesises all outputs into a unified deliverable.

---

### Loop Patterns + Safety

Three loop-specific patterns for autonomous extended tasks:

| Pattern | Use case |
|---------|----------|
| **Infinite agentic loop** | Outer prompt spawns sub-agents per iteration. Strategy discovery, monitoring. |
| **Continuous PR loop** | Each iteration = one PR, CI-gated. Large refactors, migrations. |
| **De-sloppify** | Cleanup add-on after any pattern. Catches dead imports, unused vars, inconsistent naming. |

#### Circuit breaker safety (from [ralph-claude-code](https://github.com/frankbria/ralph-claude-code))

Every loop prompt includes safety rails to prevent runaway execution and token waste:

| Trigger | What happens |
|---------|-------------|
| **No progress** | Halt if no git changes or new output for 3 consecutive iterations |
| **Repeated errors** | Halt if the same error appears 3+ times in a row |
| **Output decline** | Halt if output length drops >70% (agent looping on nothing) |
| **Test saturation** | Exit if 3 consecutive iterations only run tests with no code changes |
| **Completion signals** | Exit when 2+ iterations report "done" or all checklist items checked |

**Why this matters:** Without circuit breakers, a loop that gets stuck burns tokens indefinitely. The original ECC loop patterns had iteration caps but no stuck-loop detection. These five triggers catch the failure modes that iteration caps miss.

---

### Swarm Coordination (from [Claude Flow](https://github.com/ruvnet/claude-flow))

Two multi-agent topologies beyond simple parallel + reconcile:

| Topology | How it works | When to use |
|----------|-------------|-------------|
| **Hierarchical** | One coordinator agent delegates to specialist workers, synthesises results | Clear authority needed, strong task dependencies, complex projects |
| **Mesh** | Peer-to-peer agents share findings via shared file, pick up unclaimed work | Fault tolerance, collaborative exploration, no clear hierarchy |

Hierarchical works in any Claude Code setup. Mesh works best with `/team` or parallel tmux sessions; for sequential runs, later agents pick up work earlier agents flagged but didn't complete.

---

### 6 Examples

Each example demonstrates a different capability and serves as a template the model copies:

1. **Single agent + ralplan** — Rich, detailed prompt with numbered requirements, specific deliverables, and data flow diagram request. Teaches the model what a high-quality prompt looks like.

2. **Multi-agent parallel** — Three specialists (Brand Guardian, SEO, UI Designer) running in parallel with an explicit reconciliation step. Shows the coordination pattern.

3. **Eval-first (AI/ML)** — Four-step eval workflow: define criteria → baseline → implement → re-evaluate. Shows how to prevent "built the wrong thing."

4. **Computer use (Browser Automator)** — Step-by-step QA walkthrough with screenshots, test data, and a fallback clause. Shows computer use scoping.

5. **Sentinel + multi-model review** — Three-step build → cross-model review → security audit. The most advanced example, combining three new patterns in one flow.

6. **Scripted pipeline (CI/cron)** — Three chained `claude --bare` commands with step dependencies. Shows headless automation.

---

## Installation

```bash
mkdir -p ~/.claude/skills/orchestrator
cp SKILL.md ~/.claude/skills/orchestrator/SKILL.md
```

### Prerequisites

Works best with:

- [Oh My Claude Code](https://github.com/Yeachan-Heo/oh-my-claudecode) — orchestration patterns (ralplan, ralph, autopilot, team, `/learner`, `/ccg`)
- [Agency Agents](https://github.com/msitarzewski/agency-agents) — 150+ specialist agents at `~/.claude/agents/`

Without these, the generated prompts still work as structured Claude Code prompts — you just won't get the orchestration loops or specialist personas.

---

## Sources

This skill synthesises patterns from:

| Source | What it contributed |
|--------|-------------------|
| [Oh My Claude Code](https://github.com/Yeachan-Heo/oh-my-claudecode) | Orchestration patterns (ralplan, ralph, autopilot, team), `/learner`, `/ccg` |
| [Agency Agents](https://github.com/msitarzewski/agency-agents) | 150+ specialist agent personas and domain routing |
| [Everything Claude Code](https://github.com/affaan-m/everything-claude-code) | Autonomous loops, eval-driven development, agentic engineering, sentinel/auditor pattern |
| [ralph-claude-code](https://github.com/frankbria/ralph-claude-code) | Circuit breaker safety (3-state model, exit detection, rate limiting) |
| [Claude Flow](https://github.com/ruvnet/claude-flow) | Swarm coordination topologies (hierarchical, mesh), inter-agent memory patterns |
| [levnikolaevich/claude-code-skills](https://github.com/levnikolaevich/claude-code-skills) | Multi-model review pipeline (parallel external agents, AGREE/REJECT verification) |

Designed to keep evolving — new patterns get merged as they emerge from the community.

---

## Version history

| Version | Date | Changes |
|---------|------|---------|
| v3.0 | 2026-03-31 | +6 patterns (loop safety, swarm coordination, inter-agent memory, skill extraction, multi-model review, sentinel gate). Compacted to 253 lines with zero redundancy. |
| v2.0 | 2026-03-31 | Renamed to `orchestrator`. Merged ECC patterns (autonomous loops, agentic engineering). Compressed from 290 → 204 lines. |
| v1.0 | 2026-03-26 | Initial release as `omc-agency`. 6 patterns, 5 examples, 205 lines. |

---

## Licence

MIT
