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

---

## Response Format

When the user asks for a prompt:

1. Briefly explain which agent(s) and pattern you're recommending and why (1-2 sentences)
2. Output the prompt(s) in a code block they can copy-paste
3. If it's a multi-agent run, note which prompts can run in parallel vs sequentially
4. If there's a reconciliation step needed, include that prompt too

Keep explanations short. The prompts are the deliverable.
