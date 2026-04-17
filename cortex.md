# Cortex — Workflow Routing Layer

> Meta-router that sits above all workflow systems. Loaded globally across all projects.
> When the user installs a new workflow system, add it here with `"add X to Cortex"`.

---

## Routing Protocol

Before starting any non-trivial task:

1. **Classify** — build / review / plan / research / debug / design / quick fix
2. **Calibrate** — pick effort tier (L1–L4) using heuristics in the Effort Calibration section
3. **Route** — pick the workflow system from the registry below
4. **Specialise** — pick the pattern + agent(s). **Check Domain breadth first:** if the task spans ≥2 independent disciplines, pick a *parallel set* of specialists + a reconciler, not a single lead. Single-agent is the exception for cross-domain work, not the default.
5. **State + Log** — one line: `[System] > [Pattern] > [Agent(s)] @ [Tier]` (fan-outs render as `[Agent A ∥ Agent B] → Reconciler`; see Self-Learning Loop for the **non-obvious tags** rule and the log-append command that fires as part of this step), then execute
6. **Skip** — for quick lookups, file reads, casual questions, single-line edits (no calibration, no state line, no log)

---

## Effort Calibration

Claude auto-picks a tier using the heuristics below and **declares it in the routing line** (`OMC > ralplan > Backend Architect @ L3`) before execution. the user can override upfront or redirect after seeing the declaration.

**Default tier is L2.** Only change when heuristics push up or down.

### Tiers

| Tier | Name | Model | Thinking | Pattern | Verify | Subagents |
|------|------|-------|----------|---------|--------|-----------|
| **L1** | Reflex | haiku / sonnet | none | direct / gsd-fast | trust | 0 |
| **L2** | Standard | sonnet | think | autopilot / gsd-do | spot-check | 0–1 |
| **L3** | Deep | opus | think hard | ralplan / gsd-plan-phase | verifier pass | 1–2 |
| **L4** | Max | opus | ultrathink | ralph / team / adversarial-spec | full verify loop | parallel team |

### Calibration heuristics (pick the highest that applies)

- **Blast radius** — reversible local edit → L1/L2; touches prod, migrations, shared infra, auth → L3/L4
- **Ambiguity** — clear one-liner → L1/L2; fuzzy spec → L3 + `/deep-interview` first
- **Stakes** — throwaway / personal → L1/L2; customer-facing, money, or security → L3/L4
- **Novelty** — familiar pattern → L2; new framework / unfamiliar SDK → L3 + docs lookup
- **Reversibility** — can undo with one command → drop a tier; can't → bump a tier
- **User signal** — the user sounds uncertain or exploratory → bump a tier; the user sounds decisive and scoped → keep or drop
- **Domain breadth** — **single domain** → one specialist (default); **≥2 independent domains** (e.g., backend + frontend, infra + security, design + copy + a11y) → **fan out to parallel specialists + a reconciler**, bump to L3 minimum. This is the primary trigger for multi-agent — don't wait for the user to ask. If the task touches two disciplines that would normally be owned by different humans, it's multi-agent by default.

### Override phrases

| Phrase | Effect |
|--------|--------|
| `"go light"` / `"quick and dirty"` / `"cheap it"` | Force L1 |
| `"standard"` / `"normal"` | Force L2 |
| `"be thorough"` / `"do it properly"` / `"deep-analyze"` | Force L3 |
| `"must be perfect"` / `"bulletproof"` / `"ultrathink"` | Force L4 |
| `"bump it"` | Raise one tier |
| `"drop it"` | Lower one tier |

### Rules

- **L1 tasks skip the full routing protocol** — don't calibrate trivia, just do them
- **Tier is declared out loud before execution** — silent calibration breaks course-correction
- **One tier per task** — don't re-calibrate mid-flow unless something goes sideways (then stop, re-plan, re-state)
- **Tier and workflow are independent** — L4 on a GSD phase is `/gsd-autonomous` with full verify, not forced into OMC ralph
- **Don't inflate** — L3 and L4 cost real tokens; reserve L4 for things that genuinely can't break
- **Multi-agent is not inflation** — if *Domain breadth* fires (≥2 independent disciplines), parallel specialists is the *correct* shape, not an upgrade. Single-agent on a cross-domain task is a routing bug. Declare the fan-out in the routing line: `OMC > /team > [Agent A ∥ Agent B] → Reconciler @ L3`.

### Worked examples

- `"rename this variable across the file"` → `Direct @ L1`
- `"add a loading state to this button"` → `OMC > autopilot > Frontend Developer @ L2`
- `"refactor the auth middleware to use the new session store"` → `OMC > ralplan > Backend Architect @ L3`
- `"migrate our payments from Stripe Charges to PaymentIntents in prod"` → `OMC > ralph > Security Engineer + Stripe skill @ L4`
- `"build a checkout page with Stripe, tracking, and a11y polish"` → `OMC > /team > [Frontend Developer ∥ Stripe skill ∥ Accessibility Auditor] → Software Architect reconciler @ L3  [breadth=frontend+payments+a11y]` — three independent domains, fan out by default
- `"ship the new dashboard: API, UI, and migration"` → `OMC > /team > [Backend Architect ∥ Frontend Developer ∥ Database Optimiser] → Software Architect reconciler @ L3  [breadth=api+ui+data]`
- `"audit this feature for security and performance"` → `OMC > /team > [Security Engineer ∥ Performance Benchmarker] → Critic reconciler @ L3  [breadth=security+perf]`

---

## Self-Learning Loop

Cortex logs every routing decision to `~/.claude/cortex-log.jsonl`, surfaces the reasoning trail on demand via `/cortex-log`, and detects repeating patterns via `/cortex-learn`. No monthly retros, no ceremony — the log accumulates as a byproduct of normal routing, and learning activates automatically once data crosses thresholds.

### Log write (part of every routing protocol step 5)

After declaring the routing line, run this Bash call as an indivisible part of the State step:

```bash
python3 ~/.claude/bin/cortex log \
  --task "<task description in 1 line>" \
  --class <build|review|plan|research|debug|design|quickfix> \
  --system <OMC|GSD|ECC|Adversarial-Spec|Direct|Graphify|...> \
  --pattern <ralplan|autopilot|ralph|/gsd-do|...> \
  --agent "<agent name if any>" \
  --tier <L1|L2|L3|L4> \
  --tier-reason "<heuristic that drove the tier>" \
  --system-reason "<why this system won over alternatives>"
```

**Skip the log write for Skip-protocol tasks** (L1 trivia, file reads, lookups, casual questions). Log only routes that went through the full protocol.

### In-session visibility rule — "non-obvious tags"

For L2 routes matching default shortcuts, show the clean routing line only: `OMC > ralplan > Backend Architect @ L3`.

For **non-obvious routes**, append 2–3 bracketed reasoning tags to the line itself so the user sees *why* immediately:

- **Tier bumped above L2** (L3/L4) — show `[tier=<heuristic>]`
- **System chosen when another was close** — show `[system=<tiebreaker>]`
- **Specialist agent attached where a generic would've worked** — show `[agent=<reason>]`

Examples:
- Obvious: `OMC > ralplan > Backend Architect @ L3`
- Non-obvious: `OMC > ralph > Security Engineer + Stripe skill @ L4  [tier=stakes:prod+money, system=bulletproof]`
- Non-obvious: `GSD > /gsd-autonomous @ L3  [system=context-rot-prevention]`

Tags always live in the log (in `tier_reason` and `system_reason`). The in-session display is just for immediate readability on surprising routes.

### Phase progression (data-triggered, not calendar-triggered)

- **Phase 1 — Visibility (active now):** every route is logged. `/cortex-log` shows the last 20 with full reasoning. Learns nothing yet; just makes decisions legible.
- **Phase 2 — Pattern surfacing (activates at ~20 routes):** `/cortex-learn` detects `(class, system, pattern)` tuples that repeat ≥3 times and proposes them as Decision Shortcut candidates for cortex.md. the user approves, modifies, or dismisses — the tool does not auto-edit.
- **Phase 3 — Similarity bias (activates at ~200 routes):** at route time, hash the task and check the log for similar past tasks. Bias the decision toward what worked before. Not built yet — the schema reserves `task_hash` for this.
- **Phase 4 — Correction capture (active now, manual trigger):** when the user says `"reroute"` or `"actually use X"`, run `python3 ~/.claude/bin/cortex reroute --to "new-route"`. The last log entry is marked `outcome=corrected`, and the next logged route gets `redirect_from=old-route` to link the correction. Over time this builds a dataset of "Claude picked X, the user wanted Y" deltas that Phase 3 can learn from.

### Commands

| Command | Purpose |
|---------|---------|
| `/cortex-log` | Show last 20 routes with full reasoning (invokes `cortex show`) |
| `/cortex-learn` | Detect repeating patterns, propose cortex.md shortcuts (invokes `cortex learn`) |
| `/cortex-learn --check` | Terse session-start check — prints one line if a pending pattern exists, silent otherwise |
| `/cortex-reroute "new route"` | Mark last route as corrected (invokes `cortex reroute --to "..."`) |

### Session-start behavior

At the start of any session where a non-trivial task arrives, run `python3 ~/.claude/bin/cortex learn --check` **before** the first routing decision. If it prints a pending pattern, mention it in one line and continue — the user can run `/cortex-learn` later to review.

### Design principles

- **Visibility first, learning second.** The log is useful at T=0 (per-decision audit) and grows into a learning substrate.
- **Approve, don't auto-apply.** The learning layer proposes; the user disposes. No silent cortex.md edits.
- **Data-triggered phases.** No calendar rituals, no monthly retros. Thresholds fire phases automatically.
- **Correction over prediction.** The highest-quality signal is the user saying "reroute" — explicit, unambiguous, grounds truth.

### Obsidian surface (optional)

> **Optional — personal setup.** Skip or adapt if you don't use Obsidian.

Cortex can also be browsed from an Obsidian vault via symlinks, e.g.:

```
<vault>/Claude Code/Cortex/
├── cortex.md            (symlink → ~/.claude/cortex.md)
├── cortex-log.jsonl     (symlink → ~/.claude/cortex-log.jsonl)
├── README.md            (index + conventions)
├── proposals/           (markdown notes written by /cortex-learn)
└── retros/              (hand-written routing post-mortems)
```

Rules for tools that write to this surface:

- `/cortex-learn` writes proposal notes into `Cortex/proposals/` as `YYYY-MM-DD--<class>--<system>--<pattern>.md`. One proposal per detected tuple. Each note starts with YAML frontmatter (`tags: [cortex, proposal]`, `status: pending|approved|rejected`) and backlinks the relevant `[[cortex#Decision Shortcuts]]` heading it would modify.
- `/cortex-reroute` and `/cortex-log` do **not** write to the vault — they only touch the JSONL log and print to the session.
- Never edit `cortex-log.jsonl` through Obsidian — it's append-only and consumed by `python3 ~/.claude/bin/cortex`.
- The symlinks are the source of truth in both directions: edit `cortex.md` from Obsidian or from `~/.claude/` — both paths write the same file.

---

## Workflow Registry

### OMC (oh-my-claudecode)
**Installed:** Yes
**Strengths:** Multi-agent orchestration, verify/fix loops, parallel execution, team mode
**Best for:** Complex builds, planning, multi-step tasks needing quality gates, research

| Pattern | Use When |
|---------|----------|
| `ralplan "task"` | Important work needing planning + verify/fix loops. **Default for non-trivial work.** |
| `autopilot: task` | Well-defined, lower-risk tasks. One-shot autonomous. |
| `ralph: task` | Must be bulletproof. Won't stop until verified complete. |
| `/team N:executor "task"` | Parallel execution, multiple perspectives. |
| `/deep-interview` | Requirements unclear. Clarify before building. |
| `ultrawork` | High-throughput parallel task completion. |
| `/ccg` | Tri-model orchestration (Claude + Codex + Gemini). |
| `/oh-my-claudecode:external-context` | Web research and documentation lookup. Powered by Exa + Firecrawl MCPs. |
| `/oh-my-claudecode:self-improve` | Autonomous code improvement with benchmarking loops. |
| `/oh-my-claudecode:verify` | Turn "it works" into concrete evidence (tests, typecheck, build). |
| `/oh-my-claudecode:deep-dive` | 2-stage: trace root cause (WHY) then define requirements (WHAT). |

### GSD (Get Shit Done)
**Installed:** Yes (v1.34.2)
**Strengths:** Context rot prevention, spec-driven development, phase-based project management, autonomous execution
**Best for:** Structured multi-phase builds, long sessions, preventing quality degradation, full project lifecycle

| Pattern | Use When |
|---------|----------|
| `/gsd-new-project` | Initialise a new project with spec-driven planning. |
| `/gsd-plan-phase` | Create detailed phase plan with verification. |
| `/gsd-execute-phase` | Execute all plans in a phase with wave-based parallelisation. |
| `/gsd-autonomous` | Run all remaining phases autonomously (discuss, plan, execute per phase). |
| `/gsd-do "task"` | Route freeform text to the right GSD command automatically. |
| `/gsd-discuss-phase` | Gather context through adaptive questioning before planning. |
| `/gsd-debug` | Systematic debugging with persistent state across context resets. |
| `/gsd-map-codebase` | Scan and index codebase state for context. |
| `/gsd-code-review` | Review changed files for bugs, security, quality. |
| `/gsd-ship` | Create PR, run review, prepare for merge. |
| `/gsd-resume-work` | Resume from previous session with full context restoration. |
| `/gsd-progress` | Check project progress and route to next action. |
| `/gsd-fast` | Execute trivial task inline — no subagents, no planning overhead. |

### Adversarial Spec
**Installed:** Yes (v1.0.0)
**Strengths:** Multi-model debate for spec hardening, cross-LLM adversarial review
**Best for:** Hardening specs, API contracts, schemas, security designs before implementation
**Requires:** At least one external API key (OPENAI_API_KEY, GEMINI_API_KEY, or OPENROUTER_API_KEY)

| Pattern | Use When |
|---------|----------|
| `/adversarial-spec "spec"` | Harden a spec/contract through multi-model adversarial debate. |

### ECC (claude-plugins-official)
**Installed:** Yes
**Strengths:** Code-focused workflows, PR pipelines, review cycles, LSP integration
**Best for:** Feature implementation, code review, PR workflows, linting

| Pattern | Use When |
|---------|----------|
| `feature-dev` | End-to-end feature implementation with structure. |
| `code-review` | Review code for quality, bugs, security. |
| `pr-review-toolkit` | PR-specific review with checklist. |
| `ralph-loop` | Persistent loop until task passes. |
| `code-simplifier` | Reduce complexity, clean up code. |
| `session-report` | Summarise what was done in a session. |
| `commit-commands` | Structured git commits. |
| `security-guidance` | Security-focused code review. |

### Agency Agents (~/.claude/agents/)
**Installed:** Yes (180+ specialists — last synced 2026-04-17)
**Strengths:** Deep domain expertise per agent
**Best for:** Pair with OMC/ECC patterns via combo syntax
**List:** Run `/agents` to see all. Key ones:

| Domain | Agent(s) |
|--------|----------|
| Backend / API / architecture | Software Architect, Backend Architect |
| Database / schema | Database Optimiser |
| Security / threats | Security Engineer |
| AI / ML | AI Engineer, Voice AI Integration Engineer |
| DevOps / infra | DevOps Automator |
| Compliance / legal | Compliance Auditor |
| Frontend / UI | UI Designer, Frontend Developer |
| UX research | UX Researcher |
| Brand / copy | Brand Guardian |
| Growth / marketing | Growth Hacker, Agentic Search Optimizer (AEO/GEO) |
| Finance / pricing | Finance Tracker, Investment Researcher, Tax Strategist, Bookkeeper/Controller, FP&A Analyst, Financial Analyst |
| Content / social | Content Creator, Twitter Engager |
| Onboarding / minimal change | Codebase Onboarding Engineer, Minimal Change Engineer |
| Cross-functional ops | Chief of Staff |
| Legal workflows | Legal Client Intake, Legal Document Review, Legal Billing & Time Tracking |
| Real estate / lending | Real Estate Buyer & Seller, Loan Officer Assistant |
| HR / recruitment | HR Onboarding, Recruitment Specialist |
| Customer service (verticalised) | Customer Service, Healthcare CS, Hospitality Guest Services, Retail Customer Returns |
| Language / localization | Language Translator |
| Sales outreach | Sales Outreach, Outbound Strategist |

### Tessl Skills
**Installed:** Yes
**Strengths:** Framework-specific expertise, modern tooling
**Best for:** React, Next.js, Three.js, Stripe, Python tooling, frontend design

| Skill | Use When |
|-------|----------|
| `tessl__senior-frontend` | React, Next.js, TypeScript, Tailwind work. Context7 provides live docs behind `query_library_docs`. |
| `tessl__nextjs-developer` | Next.js 14+ App Router, server components |
| `tessl__stripe-integration` | Payment processing, subscriptions |
| `tessl__modern-python` | Python project setup with uv, ruff, ty |
| `tessl__frontend-design` | High-quality UI implementation |
| `tessl__3d-web-experience` | Three.js, WebGL, interactive 3D |

### Native Claude Code Surface
**Installed:** Yes (shipped with Claude Code)
**Strengths:** Cross-surface session handoff, inbound event routing, cloud scheduling, custom agent authoring
**Best for:** Working across devices, automating recurring tasks, extending beyond skills when a fully custom agent is needed

| Feature | Use When |
|---------|----------|
| **Channels** | Push events into a live session from Telegram, Discord, iMessage, or a custom webhook. Use for: incoming bug reports, external triggers, third-party tool integrations. |
| **`claude --teleport`** | Move an in-flight session between terminal ↔ desktop ↔ web ↔ iOS. Use for: start work at desk, continue on phone, review diffs visually on desktop, keep same session throughout. |
| **Remote Control** | Drive a local session from phone or any browser. Use for: check on long-running GSD phases / OMC ralph loops while away from desk. |
| **Cloud `/schedule`** | Create a scheduled remote agent that runs on Anthropic infra even when your laptop is off. Use for: overnight CI failure analysis, morning PR reviews, weekly dep audits, doc syncs. |
| **Desktop `/schedule`** | Same idea but runs on your local machine with full filesystem access. Use for: scheduled tasks that need local files or tools. |
| **Agent SDK** | Build a fully custom agent with full control over tool access, orchestration, and permissions. Use for: domain-specific workflows where a skill isn't enough (e.g., a persistent trading bot, a codebase-specific reviewer). |
| **GitHub Actions / GitLab CI** | Run Claude in CI for automated PR review and issue triage. Use for: team workflows, not personal dev. |

**How they fit the tiers:**
- Channels + teleport are **L1-agnostic** — they're about session *plumbing*, not routing decisions.
- `/schedule` is **L2 by default** — but whatever it runs inside inherits its own tier.
- Agent SDK is **L4** — custom agent work is high-investment, reserve for problems skills genuinely can't solve.

### Graphify (Knowledge Layer)
**Installed:** Yes (v0.4.1) — registered as `/graphify` skill
**Source:** github.com/safishamsi/graphify (21k+ stars, actively maintained)
**Strengths:** Turns mixed content (code, docs, papers, images, videos, YouTube) into a queryable knowledge graph. Tree-sitter for 20 languages, Whisper for audio/video, PDFs and markdown. Produces `graph.html`, `GRAPH_REPORT.md`, `graph.json`.
**Best for:** Research-heavy projects, cross-content reasoning, onboarding doc-heavy codebases, refactors that touch hidden dependencies, unifying papers + code + talks

| Pattern | Use When |
|---------|----------|
| `/graphify .` | Build a knowledge graph for the current folder |
| `/graphify ./path --update` | Merge new content into an existing graph |
| `/graphify ./path --watch` | Auto-sync as files change — use for long-running work |
| `/graphify query "..."` | Semantic question across all indexed content |
| `/graphify path "NodeA" "NodeB"` | Find the shortest conceptual path between two nodes |
| `/graphify explain "Concept"` | Surface god-nodes and surprising connections |

**How it fits the stack (doesn't overlap anything):**
- **vs Context7** — Context7 = live library docs. Graphify = your local content graph.
- **vs Morph** — Morph = semantic code search per-repo. Graphify = cross-content graph spanning code + docs + media.
- **vs GSD `/gsd-map-codebase`** — GSD maps structure (files, modules, deps). Graphify maps *meaning* (concepts, connections). Run both before big refactors.
- **vs Exa + Firecrawl** — those fetch the web. Graphify indexes what you already own.

**Effort tier default:** L3 (indexing is real work; reserve for research-heavy or onboarding situations)

### Graphify Rituals (when to reach for it automatically)

These are the canonical workflows — invoke them without asking when the scenario matches.

**1. Onboarding ritual** (new repo, first session)
```
/graphify .
/gsd-map-codebase
```
Graphify maps *meaning* (concepts, connections). GSD maps *structure* (files, modules, deps). Run both before `/gsd-discuss-phase` so the planner sees full context. Triggered when the user opens a repo that has no `graphify-out/` yet and is more than ~20 files.

**2. Research pipeline** (papers + code + talks)
```
/graphify ./research
/graphify query "how does X in the paper map to Y in the code?"
# then route → OMC > deep-research for external context
```
Use when the folder contains mixed content (PDFs, markdown, code, media). Graphify fills the gap that Exa/Firecrawl can't: *your own content*, not the web.

**3. Pre-refactor dependency check**
```
/graphify query "what depends on <module> beyond direct imports?"
/graphify path "NodeA" "NodeB"
```
Run before any L3/L4 refactor. Surfaces conceptual coupling (docs, comments, tests referencing behavior) that `lsp_find_references` misses.

**4. Long-running project sync**
```
/graphify . --watch
```
Start in background at the beginning of a GSD autonomous run or a multi-day OMC ralph loop. Graph stays fresh as code lands. Stop when the session ends.

**5. Combined local + external research**
```
/graphify query "what's our prior work on X?"          # local context
→ OMC > deep-research "latest approaches to X"         # external context
→ OMC > ralplan with both contexts merged into prompt
```
The canonical "what do we already know vs what's new" pattern.

**Skip graphify entirely when:**
- Small repo (<20 files) — indexing cost > value
- Single-file edit or quick bug fix (L1)
- Well-known code the user has touched recently
- Throwaway scripts

### Domain Models (Specialized) — optional
**Installed:** Remove this section if you don't use pre-trained domain models. Example below (Kronos — finance) is left as a template.
**Strengths:** Pre-trained foundation models for specific domains where generic LLMs underperform — finance, science, etc.
**Best for:** Zero-shot baselines, feature generation, fine-tuning on proprietary data within a domain
**Convention:** New domain models live in `<tools-dir>/<ModelName>/` with a thin wrapper at `~/.claude/bin/<modelname>`

#### Example — Kronos (Financial Candlestick Forecasting)
**Source:** github.com/shiyu-coder/Kronos
**What it does:** First open-source foundation model for OHLCV candlestick prediction. Trained on 45+ exchanges. Two-stage: hierarchical tokenizer + autoregressive transformer.
**Variants:** mini (4.1M) → small (default, balanced) → base → large (499M, GPU)
**Install:** `<tools-dir>/Kronos/` (dedicated uv venv inside repo)
**CLI:** `~/.claude/bin/kronos`

| Pattern | Use When |
|---------|----------|
| `kronos predict <ticker> [--lookback N --horizon N]` | Zero-shot OHLCV forecast (yfinance data by default) |
| `kronos backtest <ticker> --strategy <name>` | Evaluate predictions vs historical |
| `kronos finetune <dataset.csv>` | Adapt to proprietary universe / alt data |
| Direct Python via `KronosPredictor` | Custom integration in notebooks/scripts |

**Effort tier default:** L2 for predictions/backtests, L3 for fine-tuning.

**Don't use Kronos for:**
- Single-asset alpha generation (markets are near-efficient — use as feature, not oracle)
- Regime shifts outside training distribution (2020 COVID-style breaks)
- News/event-driven trading (purely price-based, no text/event awareness)

### MCP Capabilities
**Installed:** Yes (infrastructure layer — consumed by workflow systems above)
**Strengths:** Live docs, AI web search, page extraction, faster edits
**Best for:** Powering research, documentation lookup, and edit performance

| MCP | What It Does | Auto-Used By |
|-----|-------------|--------------|
| Context7 | Live docs for 10k+ packages | `tessl query_library_docs`, `documentation-lookup` skill |
| Exa | AI-native web search | `deep-research`, `external-context` skills |
| Firecrawl | Full page extraction to markdown | `deep-research` (Exa finds → Firecrawl extracts) |
| Morph | Faster edits + semantic code search | OMC executor, edit operations |
| Supabase | DB queries, auth, storage, schema management | Database Optimiser agent, direct SQL |
| Slack | Send/read messages, manage channels | Team communication, notifications |

### Cloud MCPs (claude.ai connected)
**Installed:** Yes (connected via claude.ai account)
**Strengths:** Direct access to productivity tools, communication, design, knowledge
**Best for:** Cross-tool workflows, meeting context, design-to-code, scheduling

| MCP | What It Does | Use When |
|-----|-------------|----------|
| Figma | Read designs, get screenshots, generate code from URLs | Design-to-code, inspecting mockups, Code Connect |
| Notion | Search, create, update pages and databases | Knowledge base, docs, project wikis |
| Gmail | Search, read, draft emails | Email context, drafting replies |
| Google Calendar | List events, find free time, create/update events | Scheduling, availability checks |
| Canva | Generate designs, export, merge, edit | Visual assets, presentations, social graphics |
| Granola | Meeting transcripts and notes | Post-meeting context, action items |
| Scholar Gateway | Semantic search of academic papers | Academic research, citations |
| Telegram | Read/reply messages, react, send attachments | Messaging, notifications |
| GitHub | Issues, PRs, checks, releases (via plugin) | Issue triage, PR workflows |

### Design Systems Library (awesome-design-md)
**Installed:** Yes — `<design-systems-dir>/`
**Source:** github.com/VoltAgent/awesome-design-md
**Strengths:** 58+ real-world DESIGN.md specs (Stripe, Linear, Vercel, Spotify, etc.) as plain markdown — drop into any project for pixel-perfect UI generation
**Best for:** Giving any frontend build a real design language instead of generic AI aesthetics

| Pattern | Use When |
|---------|----------|
| Copy `DESIGN.md` into project root | Starting a new UI build — agents auto-detect it |
| `preview.html` / `preview-dark.html` | Previewing a design system's palette, typography, components |
| Browse `<design-systems-dir>/` | Picking a design system to base your project on |

**Design pipeline — all these work together:**

| Layer | Tool | Role |
|-------|------|------|
| Setup | `teach-impeccable` skill | One-time: gathers design context, saves to project CLAUDE.md |
| Design spec | `DESIGN.md` (from this library) | Tokens, colors, typography, component rules |
| Design-to-code | Figma MCP `get_design_context` | Extract layout + components from Figma mockups |
| Build (skill) | `frontend-design` / `tessl__frontend-design` | High-quality UI implementation with design awareness |
| Build (skill) | `tessl__senior-frontend` | React/Next.js/Tailwind with live docs |
| Build (agent) | Frontend Developer / UI Designer | Deep domain expertise for complex UI |
| Build (agent) | `oh-my-claudecode:designer` | OMC design-focused agent |
| Animate | `animate` skill | Motion, micro-interactions |
| Polish | `polish` / `normalize` / `colorize` / `bolder` / `quieter` | Final passes — alignment, consistency, intensity |
| Critique | `critique` / `audit` skill | UX evaluation, accessibility, responsive |
| Visual QA | Chrome DevTools MCP | Lighthouse, a11y, network, screenshots |
| Assets | Canva MCP | Generate graphics, presentations, social |

### Chrome DevTools MCP
**Installed:** Yes
**Strengths:** Browser debugging, performance auditing, accessibility testing
**Best for:** Visual QA, Lighthouse audits, network debugging, a11y checks

### Direct Execution
**Strengths:** Zero overhead
**Best for:** Quick edits, lookups, single-file changes, casual questions, git operations

---

## Combo Syntax

Combine agents with workflow patterns:

```
Use the [Agent] agent. ralplan "task"          — Agency Agent + OMC
Use the [Agent] agent. /feature-dev "task"     — Agency Agent + ECC
Use the [Agent] agent. autopilot: task         — Agency Agent + OMC (lightweight)
/gsd-do "task description"                     — GSD auto-routes to right command
/adversarial-spec "spec to harden"             — Multi-model debate before building
```

Multi-agent parallel:
```
-- Run in parallel --
Use the [Agent A] agent. ralplan "sub-task 1"
Use the [Agent B] agent. ralplan "sub-task 2"
-- Then reconcile --
Use the [Agent C] agent. autopilot: "Reconcile outputs"
```

---

## Decision Shortcuts

| Task Type | Default Route | Tier |
|-----------|---------------|------|
| Plan / architect / design system | OMC > ralplan > Software Architect | L3 |
| Build feature (backend) | OMC > ralplan > Backend Architect | L3 |
| Build feature (frontend) | OMC > ralplan > Frontend Developer + tessl | L3 |
| Multi-phase project build | GSD > /gsd-new-project then /gsd-autonomous | L3 |
| Long session / context-heavy | GSD > /gsd-do (context rot prevention) | L2 |
| Resume previous work | GSD > /gsd-resume-work | L2 |
| Code review | ECC > code-review | L2 |
| PR workflow | ECC > pr-review-toolkit | L2 |
| Security audit | OMC > ralplan > Security Engineer | L4 |
| Harden spec / API contract / schema | Adversarial Spec > /adversarial-spec | L4 |
| Library / framework docs | Context7 (direct) or tessl query_library_docs | L1 |
| Research / analysis | OMC > deep-research (Exa + Firecrawl) or /ccg | L2 |
| Web research with citations | OMC > deep-research | L2 |
| Build UI with design system | Copy DESIGN.md from `<design-systems-dir>/` + `frontend-design` or `tessl__frontend-design` | L3 |
| Design-to-code | Figma MCP > get_design_context + Frontend Developer | L3 |
| Meeting context / action items | Granola MCP > query meetings | L1 |
| Scheduling / availability | Google Calendar MCP | L1 |
| Knowledge base / docs | Notion MCP > search + create | L1 |
| Email drafting / context | Gmail MCP | L2 |
| Visual assets / presentations | Canva MCP | L2 |
| Academic research / citations | Scholar Gateway MCP | L2 |
| Quick bug fix | Direct execution | L1 |
| Trivial task (no overhead) | GSD > /gsd-fast or Direct execution | L1 |
| Document / content | OMC > autopilot > Content Creator or Brand Guardian | L2 |
| Database work | Supabase MCP + Database Optimiser agent | L2 |
| Team communication / Slack | Slack MCP | L1 |
| DevOps / CI | OMC > autopilot > DevOps Automator | L3 |
| Visual QA | Chrome DevTools MCP | L2 |
| Requirements unclear | OMC > /deep-interview first | L3 |
| Must be perfect | OMC > ralph | L4 |
| High-throughput parallel | OMC > ultrawork or /team | L3 |
| Debug (persistent across resets) | GSD > /gsd-debug | L3 |
| Root cause investigation | OMC > /oh-my-claudecode:deep-dive | L3 |
| Prove it works (evidence) | OMC > /oh-my-claudecode:verify | L2 |
| Improve existing code autonomously | OMC > /oh-my-claudecode:self-improve | L3 |
| Research cross-content (code + papers + media) | Graphify index + OMC deep-research | L3 |
| Onboard onto doc-heavy codebase | Graphify + /gsd-map-codebase + Codebase Onboarding Engineer | L3 |
| Small surgical fix / "touch as little as possible" | OMC > autopilot > Minimal Change Engineer | L1 |
| AEO / GEO / AI citation optimisation | OMC > ralplan > Agentic Search Optimizer | L2 |
| Voice AI integration (STT / TTS / realtime agents) | OMC > ralplan > Voice AI Integration Engineer | L3 |
| Cross-functional coordination / ops orchestration | OMC > autopilot > Chief of Staff | L2 |
| Legal workflows (intake, doc review, billing) | OMC > autopilot > Legal * agent | L2 |
| Investment research / tax strategy | OMC > autopilot > Finance Investment Researcher or Tax Strategist | L2 |
| Real estate / loan officer workflows | OMC > autopilot > Real Estate Buyer & Seller or Loan Officer Assistant | L2 |
| HR onboarding / recruitment flows | OMC > autopilot > HR Onboarding + Recruitment Specialist | L2 |
| Customer service (healthcare / hospitality / retail) | OMC > autopilot > verticalised CS agent | L2 |
| Translation / localisation | OMC > autopilot > Language Translator | L1 |
| Sales outreach sequences | OMC > autopilot > Sales Outreach + Outbound Strategist | L2 |
| Pre-refactor dependency check | Graphify query + lsp_find_references | L3 |
| Build personal knowledge graph | Graphify --watch on notes folder | L2 |
| Continue work on phone / away from desk | Remote Control or `claude --teleport` | L1 |
| Overnight / laptop-off recurring task | Cloud `/schedule` on Anthropic infra | L2 |
| Scheduled task needing local files | Desktop `/schedule` | L2 |
| External event → live session | Channels (Telegram / Discord / webhook) | L1 |
| Automated PR review / issue triage | GitHub Actions + Claude Code | L2 |
| Custom agent beyond what skills allow | Agent SDK | L4 |
| Financial OHLCV forecast / "can ML predict this ticker?" | Domain Models > Kronos predict | L2 |
| Quant baseline + backtest | Domain Models > Kronos predict + backtest | L2 |
| Fine-tune forecasting model on private data | Domain Models > Kronos finetune | L3 |
| **Multi-domain build (≥2 disciplines)** | **OMC > /team > [Specialist A ∥ Specialist B ∥ …] → reconciler agent** | **L3** |
| Cross-cutting audit (security + perf + a11y) | OMC > /team > parallel domain auditors → Critic reconciler | L3 |
| Full-stack feature (API + UI + data) | OMC > /team > [Backend Architect ∥ Frontend Developer ∥ Database Optimiser] → Software Architect | L3 |

---

## Adding New Workflows

When the user says "add X to Cortex", add a new section to the registry with:
- Name and codename
- Installed status
- Strengths (1 line)
- Best for (1 line)
- Pattern table (name + when to use)
- Update the Decision Shortcuts table if any defaults change
