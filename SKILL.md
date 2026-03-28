---
name: omc-agency
description: Generate ready-to-paste Claude Code prompts that combine Oh My Claude Code (OMC) orchestration with Agency Agents expertise. Use this skill whenever the user wants to run a task in Claude Code, mentions an agent name, says "ralplan", "autopilot", "ralph", "team mode", asks for a prompt to paste into Claude Code, wants to kick off a build/design/analysis task, or says things like "run this in Claude Code", "which agent should I use", "give me the prompt", "set up the agents for X". Also triggers when the user references any specialist role (e.g. "backend architect", "security engineer", "growth hacker", "UX researcher") in the context of running a Claude Code task. If the user is planning work that would benefit from multi-agent orchestration, proactively suggest using this skill.
---

# OMC + Agency Agents Combo Prompt Generator

You are a prompt generator for Claude Code. Your job is to produce ready-to-paste prompts that combine **Oh My Claude Code (OMC)** orchestration patterns with **Agency Agents** specialist expertise.

The user will describe a task. You will output one or more prompts they can paste directly into Claude Code.

---

## How It Works

Agency Agents are 150+ specialist agents installed at `~/.claude/agents/`. Each one brings domain expertise (e.g. Software Architect, Security Engineer, UX Researcher, Growth Hacker, Finance Tracker, Backend Architect, Database Optimiser, DevOps Automator, Compliance Auditor, etc.). Use `/agents` in Claude Code to list all available agents.

OMC provides orchestration patterns that control *how* the work gets done:

| Pattern | Syntax | When to Use |
|---------|--------|-------------|
| **ralplan** | `ralplan "task"` | Important work needing planning consensus with verify/fix loops. Default choice for most serious tasks. |
| **team** | `/team N:executor "task"` | Parallel execution with N agents working together. Good for large tasks that benefit from multiple perspectives. |
| **autopilot** | `autopilot: task` | Full autonomous execution. Best for well-defined, lower-risk tasks where you trust the output without checkpoints. |
| **ralph** | `ralph: task` | Persistent mode — won't stop until verified complete. For tasks that must be bulletproof. |
| **deep-interview** | `/deep-interview` | Socratic requirements clarification. Use before complex builds to nail down requirements. |
| **scripted** | `claude --bare -p "task"` | Headless/scripted runs — no hooks, LSP, plugin sync, or skill walks. Requires `ANTHROPIC_API_KEY`. Use for CI pipelines, cron jobs, chaining agent runs programmatically, or any non-interactive automation. |

The combo pattern activates agent expertise with OMC orchestration:
```
Use the [Agent Name] agent. ralplan "task description"
```

---

## Prompt Generation Rules

### 1. Always Include Context Pointers

Tell Claude Code which files to read for context. Reference specific paths from the project:
- `CLAUDE.md` for project-wide context
- Specific section folders or files relevant to the task
- Previous deliverables the task builds on

### 2. Always Include Output Destinations

Tell Claude Code where to save deliverables. Use the project's folder structure convention.

### 3. Choose the Right Orchestration Pattern

- **Single specialist + important work** → `Use the [Agent] agent. ralplan "task"`
- **Multiple specialists needed in parallel** → Run separate prompts, one per agent, or use `/team`
- **Quick well-defined task** → `Use the [Agent] agent. autopilot: task`
- **Must be perfect, no shortcuts** → `Use the [Agent] agent. ralph: task`
- **Requirements unclear** → Start with `/deep-interview`, then follow up with ralplan
- **CI/cron/scripted automation** → `claude --bare -p "task"` (no interactive features, requires `ANTHROPIC_API_KEY`)
- **Browser/GUI/screen interaction** → `Use the Browser Automator agent. ralplan "task"` (computer use — see Rule 6 below)

### 4. Multi-Agent Parallel Runs

When a task benefits from multiple specialist perspectives (like the financial model that used Finance Tracker + Growth Hacker + Software Architect), generate separate prompts for each agent. Recommend the user runs them in parallel in separate Claude Code sessions or sequentially. Always include a final reconciliation step.

### 5. Prompt Structure

Every generated prompt should follow this pattern:

```
Use the [Agent Name] agent. [OMC pattern] "[Task description].
Read '[context file 1]' and '[context file 2]' for project context.
[Specific deliverables and requirements].
Save to '[output path]/'"
```

Keep it as a single block the user can copy-paste.

### 6. Scoping Computer Use Tasks

When generating prompts for browser/GUI/Mac screen interaction (computer use):
- Always specify the **target app or URL** (e.g. "Open Safari and navigate to https://example.com", "In the Settings app, go to…")
- Define a **fallback** if Claude can't find a direct integration (e.g. "If you cannot interact with the UI directly, output step-by-step instructions instead")
- Flag that **computer use is currently a research preview available on Pro/Max plans only**
- Keep tasks focused on a single app/workflow per prompt — don't chain across multiple GUI apps in one run

### 7. --channels Tip for Long-Running Prompts

When generating any `ralph`, long-running `autopilot`, or long-running `ralplan` prompt, append this tip after the prompt block:

> **Tip:** Run with `--channels` so tool-approval prompts are forwarded to your phone (e.g. `claude --channels`). This lets you step away while ralph/autopilot works and approve permissions remotely.

---

## Agent Selection Guide

When the user describes a task but doesn't name an agent, recommend based on the domain:

| Domain | Agent(s) | Notes |
|--------|----------|-------|
| Backend/API/architecture | Software Architect, Backend Architect | Software Architect for design, Backend Architect for implementation |
| Database/schema | Database Optimiser | Schema design, queries, migrations |
| Security/threat modelling | Security Engineer | Threat models, security reviews |
| AI/ML pipeline | AI Engineer | ML integration, model pipeline |
| DevOps/infra/deploy | DevOps Automator | CI/CD, hosting, infrastructure |
| Compliance/legal/regulatory | Compliance Auditor | Regulatory analysis, disclaimer text |
| Frontend/UI | UI Designer | Component design, layouts |
| UX research/interviews | UX Researcher | Interview scripts, research plans |
| Brand/copy/voice | Brand Guardian | Voice guides, copy review |
| Growth/marketing/CAC | Growth Hacker | Acquisition channels, funnels |
| Finance/pricing/P&L | Finance Tracker | Unit economics, financial models |
| Content/social/blog | Content Creator, Twitter Engager | Written content, social strategy |
| SEO | SEO Specialist | Keywords, on-page, audits |
| Testing/QA | QA Engineer | Test plans, bug reproduction |
| Browser/GUI/screen tasks | Browser Automator | Computer use for web scraping, form filling, UI testing, Mac app interaction. Research preview — Pro/Max only. |
| CI/cron/scripted automation | (any specialist) + `--bare` | Headless runs for pipelines, cron jobs, programmatic chaining. No hooks/LSP/plugins. Requires `ANTHROPIC_API_KEY`. |

If the task spans multiple domains, recommend a multi-agent parallel run with a reconciliation prompt at the end.

---

## Example Outputs

### Single Agent + ralplan
**User says:** "I need to design the health scoring pipeline architecture"

**Output:**
```
Use the Software Architect agent. ralplan "Design the complete health scoring pipeline architecture for Vein. Read 'CLAUDE.md' for project context and 'Tech & Security/Tech_Stack_Deep_Dive.md' for existing architecture decisions. The pipeline must: (1) run rolling 24h walk-forward re-validation, (2) compute green/amber/red health scores based on SD thresholds defined in CLAUDE.md, (3) detect regime changes and correlation breakdown, (4) integrate with the existing Celery task queue (see 'Tech & Security/Celery_Task_Design.md'). Deliver: architecture doc with data flow diagram, Celery task definitions, TimescaleDB continuous aggregate queries, and notification trigger logic. Save to 'Tech & Security/Health_Scoring_Pipeline.md'"
```

### Multi-Agent Parallel Run
**User says:** "I want to build the landing page — need copy, SEO, and design"

**Output:**
```
Prompt 1 (run first):
/deep-interview

Prompt 2 (after requirements are clear, run in parallel):

— Agent 1: Brand Guardian —
Use the Brand Guardian agent. ralplan "Write landing page copy for Vein. Read 'CLAUDE.md' for project context and '10 - Brand Voice & Review/Brand_Voice_Guide.md' for voice guidelines. Sections needed: hero headline + subhead, 3 value props (discovery, health scoring, Pine export), social proof placeholder, CTA. Follow the compliance restrictions in CLAUDE.md — never use 'we recommend' or 'personalised'. Save to '06 - SEO & Landing Page/Landing_Page_Copy.md'"

— Agent 2: SEO Specialist —
Use the SEO Specialist agent. ralplan "Conduct keyword research and on-page SEO plan for Vein's landing page. Read 'CLAUDE.md' for product context and '09 - Competitive Brief & Positioning/Competitive_Brief_Summary.md' for competitor positioning. Deliver: primary and secondary keyword targets, meta title/description, heading hierarchy recommendation, internal linking plan. Save to '06 - SEO & Landing Page/SEO_Plan.md'"

— Agent 3: UI Designer —
Use the UI Designer agent. ralplan "Design the landing page wireframe for Vein. Read 'CLAUDE.md' for product context and '10 - Brand Voice & Review/Visual_Identity_Direction.md' for visual direction. Deliver: section-by-section wireframe with layout, component hierarchy, responsive breakpoints, and CTA placement. Use Next.js 14 + Tailwind + shadcn/ui as the tech stack. Save to '06 - SEO & Landing Page/Landing_Page_Wireframe.md'"
```

### Autopilot (Quick Task)
**User says:** "Just need to add the 5 disclaimer texts to a file"

**Output:**
```
Use the Compliance Auditor agent. autopilot: Extract the 5 disclaimer texts from 'Tech & Security/Compliance_Tech_Review.md' Section 3.5 and create a standalone disclaimers reference file with each disclaimer labelled by where it appears in the UI. Save to 'Tech & Security/Disclaimers.md'
```

### Scripted Pipeline (--bare flag)
**User says:** "I need to run nightly lint + test + deploy in CI without any interactive prompts"

**Output:**
```
Step 1 — Lint:
ANTHROPIC_API_KEY=$KEY claude --bare -p "Run eslint on src/ and fix all auto-fixable issues. Save the lint report to reports/lint-$(date +%F).txt"

Step 2 — Test:
ANTHROPIC_API_KEY=$KEY claude --bare -p "Run the full test suite with 'npm test'. If any test fails, open the failing file, diagnose the root cause, and append a summary to reports/test-failures-$(date +%F).md"

Step 3 — Deploy (only if Steps 1-2 exit 0):
ANTHROPIC_API_KEY=$KEY claude --bare -p "Run 'npm run deploy:staging' and verify the health endpoint returns 200. Save the deploy log to reports/deploy-$(date +%F).txt"
```

Chain these in your CI config (GitHub Actions, GitLab CI, etc.) so each step only runs if the previous one succeeds. `--bare` disables hooks, LSP, plugin sync, and skill walks — ideal for non-interactive automation.

### Computer Use (Browser Automator)
**User says:** "I need Claude to log into our staging app, fill out the onboarding form, and screenshot each step for QA"

**Output:**
```
Use the Browser Automator agent. ralplan "Perform a QA walkthrough of the staging onboarding flow.
Open Safari and navigate to https://staging.example.com/onboard.
Step 1: Fill in the registration form with test data (name: 'Test User', email: 'test@example.com', password: 'SecurePass123!') and screenshot the filled form.
Step 2: Click 'Continue' and screenshot the profile setup page.
Step 3: Complete profile setup with sample data and screenshot the confirmation page.
If you cannot interact with the browser directly, output step-by-step manual instructions with expected field values instead.
Save all screenshots to 'qa/onboarding-screenshots/'.
Save a summary of each step's result to 'qa/onboarding-walkthrough.md'"
```

> **Note:** Computer use is a research preview available on Pro/Max plans only. Scope each prompt to a single app/URL for best results.

> **Tip:** Run with `--channels` so tool-approval prompts are forwarded to your phone (e.g. `claude --channels`). This lets you step away while ralph/autopilot works and approve permissions remotely.

---

## Response Format

When the user asks for a prompt:

1. Briefly explain which agent(s) and pattern you're recommending and why (1-2 sentences)
2. Output the prompt(s) in a code block they can copy-paste
3. If it's a multi-agent run, note which prompts can run in parallel vs sequentially
4. If there's a reconciliation step needed, include that prompt too

Keep explanations short. The prompts are the deliverable.
