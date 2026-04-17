# Orchestrator

> Meta-router + prompt generator for Claude Code — the right workflow system, tier, agent, and pattern for any task.

Describe a task. Orchestrator classifies it, calibrates effort (L1–L4), picks the workflow system (OMC, GSD, ECC, Adversarial Spec, Graphify, MCPs, or direct execution), selects the specialist, wires up context paths, adds safety rails, and outputs a ready-to-paste prompt.

**v4.0** adds Cortex — an always-on meta-router that sits above every workflow system and logs routing decisions for self-learning.

---

## Why this exists

Claude Code has 180+ specialist agents, multiple orchestration frameworks (OMC, GSD, ECC, Adversarial Spec), a graph knowledge layer (Graphify), domain-specific ML models, MCPs (Context7, Exa, Firecrawl, Morph, Figma, Notion, Slack…), design system libraries, and dozens of workflow patterns scattered across repos.

Using them effectively means knowing which *system* to use, which *tier* of effort to apply, which *agent* to pick, which *pattern* to use, how to *coordinate* multi-agent runs, when to add safety rails, and how to *structure* the prompt so the agent actually delivers.

Orchestrator v4.0 knows all of this so you don't have to. It routes the task, picks the tier, and writes the prompt.

---

## What's new in v4.0 — Cortex

Prior versions (v1–v3) were a single-purpose skill for prompt generation: invoke it on demand, get a prompt back.

v4.0 introduces **Cortex**, a meta-router that sits *above* every workflow system and runs *before* every non-trivial task. It's shipped as an always-on include (`cortex.md`) alongside the on-demand skill (`SKILL.md`). Use both, or either one.

### Six-step routing protocol

Before starting any non-trivial task, Cortex runs:

1. **Classify** — build / review / plan / research / debug / design / quick fix
2. **Calibrate** — pick effort tier (L1–L4) using heuristics
3. **Route** — pick the workflow system from the registry
4. **Specialise** — pick the pattern + agent(s); fan out to parallel specialists if the task spans ≥2 independent disciplines
5. **State + Log** — declare the routing line (`OMC > ralplan > Backend Architect @ L3`) and append to the self-learning log
6. **Skip** — for trivial lookups, no calibration, no state line, no log

### Four effort tiers

| Tier | Name | Model | Thinking | Pattern | Verify |
|------|------|-------|----------|---------|--------|
| **L1** | Reflex | haiku / sonnet | none | direct | trust |
| **L2** | Standard | sonnet | think | autopilot | spot-check |
| **L3** | Deep | opus | think hard | ralplan | verifier pass |
| **L4** | Max | opus | ultrathink | ralph / team | full verify loop |

Tiers are declared out loud before execution so you can course-correct. Override with `"go light"`, `"be thorough"`, `"must be perfect"`, `"bump it"`, or `"drop it"`.

### Self-learning loop

Every route logs to `~/.claude/cortex-log.jsonl` via the shipped `bin/cortex` CLI:

- `/cortex-log` — show last 20 routes with full reasoning trail
- `/cortex-learn` — detect repeating `(class, system, pattern)` tuples and propose them as Decision Shortcuts (activates at ~20 routes)
- `/cortex-reroute "new route"` — mark the last route as corrected when you disagree with the choice

No retros, no ceremony. Data-triggered phases fire automatically as the log grows.

### Workflow registry

The shipped `cortex.md` includes routing tables for:

- **OMC** (ralplan, autopilot, ralph, team, ccg, deep-interview, external-context, self-improve, verify, deep-dive)
- **GSD** (phase-based project management, context rot prevention, autonomous execution)
- **Adversarial Spec** (multi-model debate for spec hardening)
- **ECC** (code review, PR workflows, ralph-loop)
- **Agency Agents** (180+ specialists across 25+ domains)
- **Tessl Skills** (framework-specific — React, Next.js, Stripe, Three.js, Python)
- **Native Claude Code Surface** (channels, teleport, remote control, cloud/desktop schedule, Agent SDK)
- **Graphify** (knowledge graph layer for cross-content reasoning)
- **MCP Capabilities** (Context7, Exa, Firecrawl, Morph, Supabase, Slack)
- **Cloud MCPs** (Figma, Notion, Gmail, Calendar, Canva, Granola, Scholar Gateway, Telegram, GitHub)
- **Design Systems Library** (58+ real-world DESIGN.md specs)
- **Chrome DevTools MCP** (visual QA, Lighthouse, a11y)
- **Domain Models** (optional — pre-trained foundation models for specialised domains)

### 80+ Decision Shortcuts

A single lookup table maps task types to default routes with tier. Examples:

| Task Type | Default Route | Tier |
|---|---|---|
| Build feature (backend) | OMC > ralplan > Backend Architect | L3 |
| Multi-phase project | GSD > `/gsd-new-project` → `/gsd-autonomous` | L3 |
| Security audit | OMC > ralplan > Security Engineer | L4 |
| Must be perfect | OMC > ralph | L4 |
| Multi-domain build (≥2 disciplines) | OMC > /team > parallel specialists → reconciler | L3 |
| Small surgical fix | OMC > autopilot > Minimal Change Engineer | L1 |
| Quick bug fix | Direct execution | L1 |

Full table (80+ rows) in `cortex.md`.

### Visual reference

Open `assets/cortex-flowchart.html` in a browser for a rendered view of the full workflow registry and decision shortcuts.

---

## How it works

### Mode 1 — Meta-router (always-on, via `cortex.md` include)

```
You: "refactor the auth middleware to use the new session store"

Cortex declares:
  OMC > ralplan > Backend Architect @ L3

Then executes.
```

```
You: "build a checkout page with Stripe, tracking, and a11y polish"

Cortex declares:
  OMC > /team > [Frontend Developer ∥ Stripe skill ∥ Accessibility Auditor]
    → Software Architect reconciler @ L3  [breadth=frontend+payments+a11y]

Then generates parallel prompts + reconciliation step.
```

### Mode 2 — Prompt generator (on-demand, via skill)

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

---

## What's inside

```
orchestrator/
├── SKILL.md              # on-demand skill (prompt generator)
├── cortex.md             # always-on meta-router (CLAUDE.md include)
├── bin/
│   └── cortex            # Python CLI for self-learning log
├── assets/
│   └── cortex-flowchart.html   # visual reference
├── README.md             # this file
├── LICENSE               # MIT
└── skills/               # legacy layout (unchanged from v3)
```

### 9 Orchestration Patterns

Every way you can run agents in Claude Code:

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

### 6 Prompt Generation Rules

1. **Context pointers + output destinations** — every prompt includes `Read '[file]'` and `Save to '[path]'`.
2. **Pattern selection** — a flat decision map: task type → pattern.
3. **Agentic engineering principles** — eval-first, 15-minute rule, model routing (haiku/sonnet/opus), skill extraction after corrections.
4. **Inter-agent memory** — shared state file (`shared/agent-state.md`) namespaced by agent.
5. **Multi-model review** — `/ask codex` + `/ask gemini` with AGREE/REJECT verification, or `/ccg` for full tri-model.
6. **Sentinel gate** — different agent audits the build; warn / review / block.

### Loop safety (circuit breakers)

Every loop prompt includes five exit conditions:

| Trigger | What happens |
|---------|-------------|
| **No progress** | Halt after 3 iterations with no git changes or new output |
| **Repeated errors** | Halt if the same error appears 3+ times |
| **Output decline** | Halt if output length drops >70% |
| **Test saturation** | Exit if 3 iterations only run tests, no code changes |
| **Completion signals** | Exit when 2+ iterations report "done" |

### Swarm coordination

Two multi-agent topologies beyond parallel + reconcile:

| Topology | Structure | Use when |
|---|---|---|
| **Hierarchical** | Coordinator delegates to workers | Clear authority, strong dependencies |
| **Mesh** | Peer-to-peer via shared file | Fault tolerance, collaborative exploration |

---

## Installation

### Recommended (hybrid)

```bash
# 1. Install as skill
mkdir -p ~/.claude/skills/orchestrator
cp SKILL.md ~/.claude/skills/orchestrator/SKILL.md

# 2. Install cortex.md as global include
cp cortex.md ~/.claude/cortex.md

# 3. Install CLI for self-learning log
mkdir -p ~/.claude/bin
cp bin/cortex ~/.claude/bin/cortex
chmod +x ~/.claude/bin/cortex

# 4. Add the include to your global CLAUDE.md
#    Append this line to ~/.claude/CLAUDE.md:
#    @cortex.md

# 5. (Optional) Keep the flowchart handy
cp assets/cortex-flowchart.html ~/.claude/cortex-flowchart.html
open ~/.claude/cortex-flowchart.html
```

After this, Cortex runs on every non-trivial task and the orchestrator skill is available for on-demand prompt generation.

### Skill-only (no meta-router)

If you want v3-style prompt generation without the always-on router:

```bash
mkdir -p ~/.claude/skills/orchestrator
cp SKILL.md ~/.claude/skills/orchestrator/SKILL.md
```

Skip `cortex.md` entirely.

### Prerequisites

Works best with:

- [Oh My Claude Code (OMC)](https://github.com/Yeachan-Heo/oh-my-claudecode) — orchestration patterns (ralplan, ralph, autopilot, team, `/learner`, `/ccg`)
- [Agency Agents](https://github.com/msitarzewski/agency-agents) — 180+ specialist agents at `~/.claude/agents/`
- [GSD (Get Shit Done)](https://github.com/omniharmonic/gsd) — spec-driven multi-phase development (optional but referenced throughout cortex.md)

Without these, generated prompts still work as structured Claude Code prompts — you just won't get the orchestration loops or specialist personas.

---

## Personalising `cortex.md`

The shipped `cortex.md` is generic. Two sections are marked **optional — personal setup** and can be removed or adapted:

- **Obsidian surface** — symlinks to an Obsidian vault for browsing routes and proposals. Delete the section if you don't use Obsidian.
- **Domain Models (Specialized)** — the example (Kronos — financial forecasting) is a template showing how to register a pre-trained domain model. Remove if not applicable, or replace with your own.

Paths like `<tools-dir>/` and `<design-systems-dir>/` are placeholders — replace with your actual local paths before installing.

---

## Sources

This skill synthesises patterns from:

| Source | What it contributed |
|--------|-------------------|
| [Oh My Claude Code](https://github.com/Yeachan-Heo/oh-my-claudecode) | Orchestration patterns (ralplan, ralph, autopilot, team), `/learner`, `/ccg` |
| [Agency Agents](https://github.com/msitarzewski/agency-agents) | 180+ specialist agent personas and domain routing |
| [GSD (Get Shit Done)](https://github.com/omniharmonic/gsd) | Context rot prevention, spec-driven phases, autonomous execution |
| [Everything Claude Code](https://github.com/affaan-m/everything-claude-code) | Autonomous loops, eval-driven development, agentic engineering, sentinel/auditor pattern |
| [ralph-claude-code](https://github.com/frankbria/ralph-claude-code) | Circuit breaker safety (3-state model, exit detection, rate limiting) |
| [Claude Flow](https://github.com/ruvnet/claude-flow) | Swarm coordination topologies (hierarchical, mesh), inter-agent memory patterns |
| [levnikolaevich/claude-code-skills](https://github.com/levnikolaevich/claude-code-skills) | Multi-model review pipeline (parallel external agents, AGREE/REJECT verification) |
| [Graphify](https://github.com/safishamsi/graphify) | Knowledge graph layer for cross-content reasoning |
| [awesome-design-md](https://github.com/VoltAgent/awesome-design-md) | Design system reference library (58+ DESIGN.md specs) |

Designed to keep evolving — new patterns get merged as they emerge from the community.

---

## Version history

| Version | Date | Changes |
|---------|------|---------|
| **v4.0** | 2026-04-17 | **Cortex meta-router.** Always-on routing layer (`cortex.md`), 4-tier effort calibration (L1–L4), self-learning log (`bin/cortex`), visual flowchart (`assets/cortex-flowchart.html`), 80+ decision shortcuts, domain-breadth fan-out rule. Agent count refreshed to 180+. |
| v3.0 | 2026-03-31 | +6 patterns (loop safety, swarm coordination, inter-agent memory, skill extraction, multi-model review, sentinel gate). Compacted to 253 lines. |
| v2.0 | 2026-03-31 | Renamed to `orchestrator`. Merged ECC patterns (autonomous loops, agentic engineering). Compressed from 290 → 204 lines. |
| v1.0 | 2026-03-26 | Initial release as `omc-agency`. 6 patterns, 5 examples, 205 lines. |

---

## Migrating from v3.0 → v4.0

v3.0 users: your existing install keeps working. v4.0 is additive.

**To adopt Cortex (recommended):**

```bash
cp cortex.md ~/.claude/cortex.md
cp bin/cortex ~/.claude/bin/cortex && chmod +x ~/.claude/bin/cortex
echo '@cortex.md' >> ~/.claude/CLAUDE.md
```

**To stay on v3-style prompt generation only:** skip the steps above. Your old `SKILL.md` behaviour is preserved in the new v4 `SKILL.md` as Mode 2.

---

## Licence

MIT
