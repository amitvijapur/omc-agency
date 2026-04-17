---
name: orchestrator
description: Meta-router + prompt generator for Claude Code — decides which workflow system (OMC, GSD, ECC, Adversarial Spec, Graphify, MCPs, direct), which tier (L1–L4), which agent, and which pattern to use for any task, and generates ready-to-paste prompts. Use this skill when the user asks "which agent should I use", "what's the right approach for X", "route this task", "give me the prompt", "set up the agents for X", mentions "ralplan", "autopilot", "ralph", "team mode", references any specialist role (backend architect, security engineer, growth hacker, UX researcher, etc.), or is planning work that would benefit from multi-agent orchestration. Also triggers on `/cortex-log`, `/cortex-learn`, `/cortex-reroute` for self-learning loop operations. For always-on routing behaviour, install the companion `cortex.md` include (see README).
---

# Orchestrator

Meta-routing layer for Claude Code. Two modes:

1. **Meta-router (always-on).** Install `cortex.md` as a CLAUDE.md include. Before every non-trivial task, Claude classifies → calibrates tier (L1–L4) → picks system → picks agent + pattern → logs the decision. See `cortex.md` for the full routing protocol.
2. **Prompt generator (on-demand).** Invoke this skill to generate ready-to-paste prompts combining OMC orchestration with 180+ Agency Agents specialists at `~/.claude/agents/` (list with `/agents`).

The meta-router decides *what* to run. The prompt generator writes *how* to run it. Use them together or independently.

---

## Mode 1 — Meta-router (cortex.md)

The companion `cortex.md` file (shipped in this repo) is the always-on routing layer. Install it once, add `@cortex.md` to your `~/.claude/CLAUDE.md`, and every non-trivial request will be classified, tier-calibrated, and routed before execution.

### Tiers (L1–L4)

| Tier | Name | Model | Thinking | Pattern | Verify |
|------|------|-------|----------|---------|--------|
| **L1** | Reflex | haiku / sonnet | none | direct | trust |
| **L2** | Standard | sonnet | think | autopilot | spot-check |
| **L3** | Deep | opus | think hard | ralplan | verifier pass |
| **L4** | Max | opus | ultrathink | ralph / team | full verify loop |

### Self-learning commands

| Command | Purpose |
|---------|---------|
| `/cortex-log` | Show last 20 routes with full reasoning |
| `/cortex-learn` | Detect repeating patterns, propose shortcuts |
| `/cortex-reroute "new route"` | Mark last route as corrected |

Routes are logged to `~/.claude/cortex-log.jsonl` via `bin/cortex` (shipped). Patterns surface automatically after ~20 logged routes.

---

## Mode 2 — Prompt Generator

## Orchestration Patterns

| Pattern | Syntax | Use When |
|---------|--------|----------|
| **ralplan** | `ralplan "task"` | Important work needing planning + verify/fix loops. **Default choice.** |
| **autopilot** | `autopilot: task` | Well-defined, lower-risk tasks. One-shot autonomous. |
| **ralph** | `ralph: task` | Must be bulletproof. Won't stop until verified complete. |
| **team** | `/team N:executor "task"` | Parallel execution, multiple perspectives. |
| **deep-interview** | `/deep-interview` | Requirements unclear. Run before building. |
| **scripted** | `claude --bare -p "task"` | CI/cron/headless. No hooks/LSP. Requires `ANTHROPIC_API_KEY`. |
| **loop** | See Loop Patterns below | Long-running iterative tasks with exit conditions. |
| **loop + ralph** | Loop with ralph per iteration | Long-running AND each iteration must be bulletproof. |
| **sentinel** | Auditor agent after build | Quality gate — separate agent reviews output before shipping. |

Combo syntax: `Use the [Agent Name] agent. [pattern] "task"`

---

## Prompt Generation Rules

### 1. Always include context pointers and output destinations

```
Use the [Agent Name] agent. [pattern] "[Task].
Read '[context file]' for project context.
[Deliverables and requirements].
Save to '[output path]/'"
```

### 2. Pattern selection

- **Single specialist + important work** → ralplan
- **Quick well-defined task** → autopilot
- **Must be perfect** → ralph. For any long-running ralph/autopilot/ralplan, append `--channels` for remote approval
- **Multiple specialists** → Separate prompts per agent, run in parallel sessions. **Always include a final reconciliation prompt** that reads all outputs and produces a unified deliverable. Use `/team` as an alternative for tighter coordination
- **Requirements unclear** → `/deep-interview` first, then ralplan
- **CI/cron** → `claude --bare -p "task"`
- **Browser/GUI** → Browser Automator agent (computer use — Pro/Max only). Always specify: (1) target app or URL, (2) fallback if UI interaction fails ("output step-by-step instructions instead"), (3) single app/workflow per prompt — don't chain across GUI apps

### 3. Agentic engineering principles

**Eval-first (for AI/ML tasks):** Structure prompts to define eval criteria → baseline → implement → re-evaluate. Only ship if all criteria pass.

**15-minute rule:** If a task exceeds ~15 min of focused agent work, decompose into sub-task prompts.

**Model routing:**
- **Haiku** (`--model haiku`): Boilerplate, formatting, simple transforms
- **Sonnet** (default): Features, bug fixes, tests, refactors
- **Opus** (`--model opus`): Architecture, complex debugging, security reviews

**Skill extraction:** After tasks involving corrections or non-obvious workarounds, add "run `/learner` to extract reusable patterns" to the prompt.

### 4. Inter-agent memory

For multi-agent or multi-session tasks, use a shared state file so agents can build on each other's work:
- Add to prompts: "Read `shared/agent-state.md` for prior agent findings. Append your findings under `## [Your Agent Name]` before exiting."
- Each agent reads at start, writes at end. Namespace by agent name to avoid clobbering.

### 5. Multi-model review

For high-stakes outputs (architecture, security, financial), get external model perspectives:
- Use `/ask codex "review [output]"` (correctness, edge cases) and `/ask gemini "review [output]"` (scale, performance) — or `/ccg` for full tri-model orchestration.
- **Critical:** Claude independently evaluates each suggestion (AGREE/REJECT) before applying. Never auto-apply external model suggestions.

### 6. Sentinel gate

After a build completes, run a DIFFERENT agent as auditor before shipping:
- Three levels: **warn** (log, continue), **review** (flag for human), **block** (reject, send back to builder)
- Prompt pattern: "After the build, use the Security Engineer agent. autopilot: Review all changes for vulnerabilities, dead code, missing error handling. Block on critical issues, warn on medium. Save to `reviews/sentinel-audit.md`"
- The sentinel must always be a different specialist than the builder to avoid same-model blind spots.

---

## Agent Selection

| Domain | Agent(s) |
|--------|----------|
| Backend/API/architecture | Software Architect (design), Backend Architect (implementation) |
| Database/schema | Database Optimiser |
| Security/threats | Security Engineer |
| AI/ML pipeline | AI Engineer, Voice AI Integration Engineer |
| DevOps/infra | DevOps Automator |
| Compliance/legal | Compliance Auditor, Legal Client Intake, Legal Document Review, Legal Billing & Time Tracking |
| Frontend/UI | UI Designer, Frontend Developer |
| UX research | UX Researcher |
| Brand/copy | Brand Guardian |
| Growth/marketing | Growth Hacker, Agentic Search Optimizer (AEO/GEO) |
| Finance/pricing | Finance Tracker, Investment Researcher, Tax Strategist, Bookkeeper/Controller, FP&A Analyst |
| Content/social | Content Creator, Twitter Engager |
| SEO | SEO Specialist |
| Testing/QA | QA Engineer |
| Browser/GUI | Browser Automator |
| Onboarding / minimal change | Codebase Onboarding Engineer, Minimal Change Engineer |
| Cross-functional ops | Chief of Staff |
| HR / recruitment | HR Onboarding, Recruitment Specialist |
| Real estate / lending | Real Estate Buyer & Seller, Loan Officer Assistant |
| Customer service (verticalised) | Customer Service, Healthcare CS, Hospitality Guest Services, Retail Customer Returns |
| Language / localisation | Language Translator |
| Sales outreach | Sales Outreach, Outbound Strategist |

Multi-domain tasks → parallel agents + reconciliation prompt.

---

## Loop Patterns

For autonomous extended tasks — monitoring, iterative discovery, migrations.

| Pattern | Use When |
|---------|----------|
| **Infinite agentic loop** | Outer prompt spawns sub-agents per iteration. Discovery, monitoring. |
| **Continuous PR loop** | Each iteration = one PR, CI-gated. Refactors, migrations. |
| **De-sloppify** | Cleanup add-on after any pattern. Dead imports, unused vars, naming. |

### Loop safety

Always include circuit breaker rules in loop prompts to prevent runaway execution:

- **No progress** — halt if no git changes or new output for 3 consecutive iterations
- **Repeated errors** — halt if the same error appears 3+ times in a row
- **Output decline** — halt if output length drops >70% (agent is looping on nothing)
- **Test saturation** — exit if 3 consecutive iterations only run tests with no code changes
- **Completion signals** — exit when 2+ iterations report "done" or all checklist items are checked

### Loop prompt template

```
Use the [Agent] agent. autopilot: Run continuously until [exit condition].
Each iteration: [cycle action].
On failure: [retry/escalate/halt].
Exit when: [verifiable condition].
Safety: Halt if no progress for 3 iterations or same error repeats 3x.
Log to '[path]/loop-log.md'. Cap at [N] iterations.
```

---

## Swarm Patterns

For complex multi-agent coordination beyond simple parallel + reconcile.

| Topology | Structure | Use When |
|----------|-----------|----------|
| **Hierarchical** | One coordinator agent delegates to specialist workers, synthesises results | Clear authority needed, strong task dependencies, complex projects |
| **Mesh** | Peer-to-peer agents, no coordinator, share findings via shared file, pick up unclaimed work | Fault tolerance needed, collaborative exploration, no clear hierarchy |

**Hierarchical:** `"Use the Software Architect agent as coordinator. Delegate sub-tasks to Backend Architect, Database Optimiser, and Security Engineer. Read each worker's output from shared/. Coordinator synthesises into final deliverable."`

**Mesh:** Best with `/team` or parallel tmux sessions. For sequential fallback, agents run one after another, each reading `shared/notes.md` for prior findings and appending their own. Later agents can pick up work earlier agents flagged but didn't complete.

---

## Examples

### Single agent + ralplan
```
Use the Software Architect agent. ralplan "Design the complete health scoring pipeline architecture.
Read 'CLAUDE.md' for project context and 'docs/architecture.md' for existing decisions.
The pipeline must: (1) run rolling 24h walk-forward re-validation,
(2) compute green/amber/red health scores based on SD thresholds,
(3) detect regime changes and correlation breakdown,
(4) integrate with the existing task queue.
Deliver: architecture doc with data flow diagram, task definitions,
DB continuous aggregate queries, and notification trigger logic.
Save to 'docs/health-scoring-pipeline.md'"
```

### Multi-agent parallel
```
— Run in parallel —

Use the Brand Guardian agent. ralplan "Write landing page copy.
Read 'CLAUDE.md' and 'docs/brand-voice.md'. Deliver: hero, value props, CTA.
Save to 'content/landing-copy.md'"

Use the SEO Specialist agent. ralplan "SEO plan for landing page.
Read 'CLAUDE.md'. Deliver: keywords, meta tags, heading hierarchy.
Save to 'content/seo-plan.md'"

Use the UI Designer agent. ralplan "Landing page wireframe.
Read 'CLAUDE.md' and 'docs/visual-identity.md'. Deliver: layout, components, breakpoints.
Save to 'content/wireframe.md'"

— Then reconcile —
Use the Software Architect agent. autopilot: "Reconcile outputs in content/ into a unified landing page spec. Save to 'content/landing-page-spec.md'"
```

### Eval-first (AI/ML task)
```
Use the AI Engineer agent. ralplan "Build the recommendation engine.
Step 1: Define eval criteria (accuracy > 90%, latency < 200ms, no regressions).
Step 2: Baseline current performance.
Step 3: Implement.
Step 4: Re-evaluate, only ship if all criteria pass.
Save eval results to 'evals/recommendation-eval.md'"
```

### Computer use (Browser Automator)
```
Use the Browser Automator agent. ralplan "QA walkthrough of staging onboarding flow.
Open Safari and navigate to https://staging.example.com/onboard.
Step 1: Fill registration form with test data (name: 'Test User', email: 'test@example.com') and screenshot the filled form.
Step 2: Click 'Continue' and screenshot the profile setup page.
Step 3: Complete profile setup and screenshot the confirmation page.
If you cannot interact with the browser directly, output step-by-step manual instructions with expected field values instead.
Save screenshots to 'qa/onboarding-screenshots/'.
Save summary to 'qa/onboarding-walkthrough.md'"
```

### Sentinel + multi-model review (high-stakes)
```
— Step 1: Build —
Use the Backend Architect agent. ralplan "Implement the payment webhook handler.
Read 'CLAUDE.md' and 'docs/stripe-integration.md'.
Save to 'src/webhooks/stripe.ts'"

— Step 2: Multi-model review —
/ccg "Review src/webhooks/stripe.ts for correctness, security, and edge cases"

— Step 3: Sentinel gate —
Use the Security Engineer agent. autopilot: "Audit src/webhooks/stripe.ts.
Check for: signature verification, idempotency, error leakage, missing event types.
Block on critical issues, warn on medium. Save to 'reviews/stripe-audit.md'"
```

### Scripted pipeline (CI/cron)
Chain steps so each only runs if the previous succeeds:
```
# Step 1 — Lint
ANTHROPIC_API_KEY=$KEY claude --bare -p "Run eslint on src/ and fix auto-fixable issues. Save report to reports/lint-$(date +%F).txt"

# Step 2 — Test (only if lint passed)
ANTHROPIC_API_KEY=$KEY claude --bare -p "Run full test suite. If any fail, diagnose root cause and append to reports/test-failures-$(date +%F).md"

# Step 3 — Deploy (only if tests passed)
ANTHROPIC_API_KEY=$KEY claude --bare -p "Deploy to staging and verify health endpoint returns 200. Save log to reports/deploy-$(date +%F).txt"
```
Wire these in your CI config (GitHub Actions, GitLab CI) with step dependencies.

---

## Response Format

1. Brief recommendation — which agent(s) and pattern, and why (1-2 sentences)
2. Prompt(s) in a code block, ready to copy-paste
3. For multi-agent: note parallel vs sequential ordering
4. Include reconciliation prompt if needed

Keep explanations short. The prompts are the deliverable.

---

## See Also

- **`cortex.md`** — Meta-router include. Install as `@cortex.md` in your `~/.claude/CLAUDE.md` for always-on routing.
- **`assets/cortex-flowchart.html`** — Visual reference of all workflow systems and decision shortcuts. Open in a browser.
- **`bin/cortex`** — Python CLI for the self-learning log (`/cortex-log`, `/cortex-learn`, `/cortex-reroute`).
