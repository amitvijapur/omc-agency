---
name: orchestrator
description: Generate ready-to-paste Claude Code prompts that combine Oh My Claude Code (OMC) orchestration with Agency Agents expertise. Use this skill whenever the user wants to run a task in Claude Code, mentions an agent name, says "ralplan", "autopilot", "ralph", "team mode", "self-improve", asks for a prompt to paste into Claude Code, wants to kick off a build/design/analysis task, or says things like "run this in Claude Code", "which agent should I use", "give me the prompt", "set up the agents for X". Also triggers when the user references any specialist role (e.g. "backend architect", "security engineer", "growth hacker", "UX researcher") in the context of running a Claude Code task. If the user is planning work that would benefit from multi-agent orchestration, proactively suggest using this skill.
---

# Orchestrator

Workflow orchestration for Claude Code — selects the right agent, pattern, coordination topology, and safety rails for any task. Outputs ready-to-paste prompts. Uses 150+ specialists at `~/.claude/agents/` (list with `/agents`).

**OMC Version:** 4.10.1 | **OMC Agents:** 19 built-in (explore, analyst, planner, architect, debugger, executor, verifier, tracer, security-reviewer, code-reviewer, test-engineer, designer, writer, qa-tester, scientist, document-specialist, git-master, code-simplifier, critic)

---

## Orchestration Patterns

| Pattern | Syntax | Use When |
|---------|--------|----------|
| **ralplan** | `ralplan "task"` | Important work needing planning + verify/fix loops. **Default choice.** |
| **autopilot** | `autopilot: task` | Well-defined, lower-risk tasks. Full lifecycle: spec → plan → execute → QA → validate. |
| **ralph** | `ralph: task` | Must be bulletproof. PRD-driven, story-by-story, deslop pass, won't stop until verified. |
| **team** | `/team N:executor "task"` | Parallel execution, multiple perspectives. Uses team pipeline: plan → prd → exec → verify → fix. |
| **ultrawork** | `ulw: task` | Parallel burst execution. Model-tiered agents (haiku/sonnet/opus). No persistence — use ralph for that. |
| **ultraqa** | `/ultraqa` | QA cycling — test, verify, fix, repeat until goal met. |
| **self-improve** | `/self-improve` | Autonomous evolutionary code improvement with tournament selection and benchmarks. |
| **deep-interview** | `/deep-interview` | Requirements unclear. Socratic clarification with ambiguity gating. Run before building. |
| **sciomc** | `/sciomc` | Parallel scientist agents for comprehensive data analysis. |
| **ccg** | `/ccg` | Tri-model orchestration: Claude + Codex + Gemini synthesis. |
| **scripted** | `claude --bare -p "task"` | CI/cron/headless. No hooks/LSP. Requires `ANTHROPIC_API_KEY`. |
| **loop** | See Loop Patterns below | Long-running iterative tasks with exit conditions. |
| **loop + ralph** | Loop with ralph per iteration | Long-running AND each iteration must be bulletproof. |
| **sentinel** | Auditor agent after build | Quality gate — separate agent reviews output before shipping. |

**Pipeline chains** (most thorough → least):
- `deep-interview → ralplan → autopilot` — Socratic spec → consensus plan → autonomous execution (skips Phase 0+1)
- `ralplan → ralph` — consensus plan → bulletproof implementation
- `autopilot` — standalone full lifecycle

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

**Model routing (OMC agent catalog tiers):**
- **Haiku**: Boilerplate, formatting, simple transforms, quick lookups. OMC agents: explore, writer
- **Sonnet** (default): Features, bug fixes, tests, refactors. OMC agents: debugger, executor, verifier, tracer, security-reviewer, test-engineer, designer, qa-tester, scientist, document-specialist, git-master
- **Opus**: Architecture, complex debugging, security reviews, deep analysis. OMC agents: analyst, planner, architect, code-reviewer, code-simplifier, critic

**Skill extraction:** After tasks involving corrections or non-obvious workarounds, add "run `/learner` to extract reusable patterns" to the prompt.

**Commit protocol (OMC v4.10.1):** For non-trivial commits, use git trailers to preserve decision context:
- `Constraint:` active constraint that shaped this decision
- `Rejected:` alternative considered | reason for rejection
- `Directive:` warning or instruction for future modifiers
- `Confidence:` high | medium | low
- `Scope-risk:` narrow | moderate | broad
- `Not-tested:` edge case or scenario not covered by tests

**Deslop pass:** Ralph now includes a mandatory `ai-slop-cleaner` pass after reviewer approval. Cleans AI-generated patterns (dead imports, unused vars, generic naming). Opt out with `--no-deslop`.

### 4. Inter-agent memory (OMC state tools)

OMC provides built-in persistence tools — prefer these over manual shared files:

**State management:** `state_read`, `state_write`, `state_clear`, `state_list_active`, `state_get_status` — session-scoped mode tracking (autopilot, ralph, ultrawork, self-improve). Only one mode active at a time.

**Notepad:** `notepad_write_priority` (critical findings), `notepad_write_working` (in-progress notes), `notepad_write_manual` (manual entries), `notepad_read` — cross-agent knowledge sharing within a session.

**Project Memory:** `project_memory_read`, `project_memory_write`, `project_memory_add_note`, `project_memory_add_directive` — persistent project context that survives across sessions. Use for architectural decisions, patterns discovered, known issues.

**Code Intelligence:** `lsp_diagnostics` (check for errors), `lsp_goto_definition`, `lsp_find_references`, `ast_grep_search/replace` (structural code search), `python_repl` (quick prototyping/validation).

For multi-agent or multi-session tasks without OMC state tools, fall back to shared files:
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
- **OMC built-in sentinels:** `oh-my-claudecode:security-reviewer` (vuln check), `oh-my-claudecode:code-reviewer` (quality, Opus), `oh-my-claudecode:verifier` (evidence-based completion check). Autopilot Phase 4 runs all three in parallel automatically.
- **Post-sentinel cleanup:** Run `oh-my-claudecode:ai-slop-cleaner` (or `deslop`/`anti-slop`) to remove AI-generated patterns. Ralph does this automatically unless `--no-deslop`.

---

## Agent Selection

| Domain | Agent(s) |
|--------|----------|
| Backend/API/architecture | Software Architect (design), Backend Architect (implementation) |
| Database/schema | Database Optimiser |
| Security/threats | Security Engineer |
| AI/ML pipeline | AI Engineer |
| DevOps/infra | DevOps Automator |
| Compliance/legal | Compliance Auditor |
| Frontend/UI | UI Designer |
| UX research | UX Researcher |
| Brand/copy | Brand Guardian |
| Growth/marketing | Growth Hacker |
| Finance/pricing | Finance Tracker |
| Content/social | Content Creator, Twitter Engager |
| SEO | SEO Specialist |
| Testing/QA | QA Engineer |
| Browser/GUI | Browser Automator |

Multi-domain tasks → parallel agents + reconciliation prompt.

---

## Loop Patterns

For autonomous extended tasks — monitoring, iterative discovery, migrations.

| Pattern | Use When |
|---------|----------|
| **Infinite agentic loop** | Outer prompt spawns sub-agents per iteration. Discovery, monitoring. |
| **Continuous PR loop** | Each iteration = one PR, CI-gated. Refactors, migrations. |
| **De-sloppify** | Cleanup add-on after any pattern. Use `/deslop` or `anti-slop`. Built into ralph automatically. |

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

### Self-improve (autonomous optimisation)
```
/self-improve
# Interactive setup: point at target repo, define goal (e.g. "reduce backtest latency"),
# confirm benchmark command (e.g. "python benchmarks/backtester_benchmark.py").
# Runs autonomously: research → plan → execute → tournament selection → merge winners.
# Stops on: target reached, plateau, max iterations, or circuit breaker.
```

### Team pipeline (coordinated multi-agent)
```
/team 3:executor "Implement the OHLCV ingestion pipeline.
Read 'CLAUDE.md' for project context and '15 - Tech & Security/architecture/Tech_Stack_Deep_Dive.md' for architecture.
Agent 1: Celery task for CCXT → TimescaleDB ingestion.
Agent 2: FastAPI endpoint for triggering manual ingestion.
Agent 3: Unit tests for both.
Save to 'src/ingestion/'"
```

### UltraQA (test-fix cycling)
```
/ultraqa "Run full test suite, fix all failures, repeat until green.
Focus on src/backtester/ and src/discovery/.
Cap at 5 cycles. Report any fundamental blockers."
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
