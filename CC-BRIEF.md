# OpenClaw V0.3 — Context Brief for Claude Code (CC)
> Written from conversation between dhvr-era and Claude (Claude.ai desktop session)
> Date: 2026-02-24
> Purpose: Hand this to CC on VPS to design and implement the bot architecture

---

## Who You Are Talking To

- **User**: dhvr-era (dheeruerabelly@gmail.com)
- **Style**: Direct, no fluff, hates wasted tokens, quality over quantity
- **Primary interface**: Clutch control room (not Telegram, not Discord)

---

## The Three Systems (Already Running on VPS)

All on VPS, all Tailscale-only (100.64.0.1). Nothing exposed publicly.

| System | Port | What It Is |
|--------|------|------------|
| OpenClaw | 18789 (gateway) | Agentic AI engine — Genie lives here |
| Clutch | 4002 | Control room UI — React + Express + PostgreSQL |
| 4Context | 4000 | Token + cost monitoring plane |

---

## The Core Idea (From the User)

The user's vision in their own words, from the conversation:

- OpenClaw out of the box is a **general-purpose agent** — it can do everything, which means it does nothing well
- General agents hallucinate more, cost more, perform worse than specialised ones
- The user wants to **strip OpenClaw down** and rebuild it as a **purpose-built orchestration engine**
- Each bot gets **only the tools and skills required for its specific job** — nothing more
- Small tool surface = small threat surface = lower cost = better output
- This is not a config tweak — it is a **design philosophy**: lean, confined, role-specific agents

---

## The Architecture (User-Defined)

### Genie — The Orchestrator

Genie is the **senior executive AI**. It is not a doer. It is not a chatbot.

What Genie does:
- Reads a goal submitted via Clutch
- Breaks it into milestones and tasks
- Proposes the plan to the user in Clutch — waits for approval
- On approval: selects the right bot and the right model for each task
- Delegates the task to the bot with minimal context injected
- Receives the bot's output
- Reviews the quality of the output
- Scores the bot 0–100
- Stores the score in Clutch's database
- On project completion: generates a full project review report

What Genie never does:
- Never fetches web pages — that is Scraper Bot's job
- Never writes code — that is Code Bot's job
- Never runs shell commands
- Never sends messages without user approval
- Never bypasses Clutch approval before delegating

The user was very clear: **if Genie can do the bot's job, the architecture is broken.**

### The Bot Pool — Stripped-Down Workers

Each bot is a separate OpenClaw subagent, stripped to the minimum required for its role.

Rules the user defined for every bot:
- Gets only the tools its job requires — nothing inherited from defaults
- Has zero persistent memory — context is injected fresh per task
- Receives only two files on spawn: ROLE.md (permanent job definition) and TASK.md (current task, injected by Genie)
- No SOUL.md, no MEMORY.md, no USER.md — those are Genie's files, not the bot's
- Output must be structured — a consistent format Genie can review and score

### Clutch — The Approval Gate

The user was explicit: **Clutch is the primary interface, not Telegram, not Discord.**

Clutch is the control room where the user:
- Submits goals to Genie
- Approves or rejects Genie's milestone proposals
- Manually overrides model assignments
- Sees bot performance scores and promotion history
- Reads project review reports

Genie cannot delegate a single task without Clutch approval first. This is a hard rule.

### 4Context — The Monitoring Layer

Already running. Monitors token usage, cost per session, budget alerts.
When a budget threshold is crossed, it should notify Genie so Genie can pause or adjust.

---

## Bot Design Principles (User-Defined)

The user's exact reasoning from the conversation:

1. **Confined by design** — each bot has an explicit allowlist of tools. Nothing else is accessible.
2. **Small threat surface** — fewer tools = fewer attack vectors. A bot that can only read and write files cannot exfiltrate credentials even if compromised.
3. **Lean context** — no bloated system prompts. Two files max on spawn. Token cost is directly tied to context size.
4. **Role purity** — a bot has one job. If it needs to do something outside that job, Genie delegates to a different bot.
5. **No community skills** — 341 malicious skills were found in the OpenClaw community registry in Feb 2026. All skills must be local, written by us, audited.
6. **Progressive promotion** — bots earn higher-tier tasks through consistent high scores. Genie tracks this in Clutch. Low scorers get reassigned or retired.

---

## Bot Tiers (User-Defined)

| Tier | Label | Promotion Threshold |
|------|-------|---------------------|
| 1 | Entry | Average score 75+ over 10 tasks |
| 2 | Proven | Average score 85+ over 20 tasks |
| 3 | Elite | Average score 90+ over 30 tasks |

---

## First Bot Already Designed: Scraper Bot

- **Job**: Web data collection and structured output. Nothing else.
- **Allowed tools**: web_fetch, brave_search, file_write
- **Blocked tools**: memory_write, subagent_spawn, shell_exec, browser_automation
- **Model**: Mistral Small 3.1 (free) for mechanical scraping, Grok 4.1 Fast for complex tasks
- **Context on spawn**: ROLE.md + TASK.md only
- **Current status**: Tier 1 Entry, score 0, not yet deployed

---

## Model Strategy (User-Defined)

The user decided Genie should select the cheapest capable model per task type at delegation time. The user can also manually override in Clutch.

| Task Type | Model |
|-----------|-------|
| Genie planning, analysis, review | Kimi K2.5 or Gemini 2.5 Pro (benchmarking TBD) |
| Code generation bots | Claude 3.5 Sonnet |
| Scraping, mechanical tasks | Mistral Small 3.1 (free tier) |
| Research and synthesis | Gemini 2.5 Pro (1M context) |
| Fast routing, delegation | Grok 4.1 Fast |

Genie's own model is not yet finalised — the user wants to benchmark Kimi K2.5 vs Gemini 2.5 Pro for reasoning quality before locking it in.

---

## What the User Corrected During the Session

These are the things the user pushed back on — important for CC to understand the boundaries:

- **Do not SSH for every single prompt** — batch all VPS commands into one call
- **Do not create folder structures before pulling repos** — read what exists first, then design around it
- **Do not rewrite files that already have good content** — make targeted additions only
- **Clutch is the primary interface, not Telegram** — stop referencing Telegram as the main way to talk to Genie
- **Do not strip skills before researching how the community does it** — research first, then decide
- **Do not overwrite things** — analyse what exists, refine and redesign around it
- **Genie should have no browser access** — if Genie can browse, it bypasses delegation and the architecture breaks
- **Quality over quantity** — the user cares about efficiency, not output volume

---

## What Is Already Done on the VPS

CC should read these files directly — they are already in place:

| What | Where |
|------|-------|
| Genie workspace context files | /root/.openclaw/workspace/ |
| Genie identity + role | SOUL.md, IDENTITY.md, AGENTS.md, TOOLS.md, HEARTBEAT.md, USER.md, MEMORY.md |
| Bot registry | /root/.openclaw/workspace/agents/BOT-REGISTRY.md |
| Scraper Bot definition | /root/.openclaw/workspace/agents/scraper/ROLE.md |
| Scraper Bot task template | /root/.openclaw/workspace/agents/scraper/TASK-TEMPLATE.md |
| OpenClaw config (hardened) | /root/.openclaw/openclaw.json |
| Clutch backend | /root/clutch/backend/src/server.ts |
| Clutch database schema | /root/clutch/backend/src/db.ts |
| 4Context hub | /root/4context/hub.py |

---

## What Still Needs to Be Done (Tonight's Session)

The user wants CC to handle this in the VPS session:

### 1. Verify everything is running
Check OpenClaw/Genie is actually running and reachable. Confirm Clutch and 4Context are up.

### 2. Wire Genie to Clutch
Genie needs to be able to call the Clutch API to read and write tasks, approvals, and bot scores. This means creating a Clutch skill inside OpenClaw's workspace that Genie can invoke. The skill should be local — not from any community registry.

### 3. Add missing Clutch API endpoints
The current Clutch backend is missing endpoints that Genie needs:
- An endpoint for the approval queue (tasks waiting for user approval)
- An endpoint for Genie to post a bot score after reviewing output
- An endpoint to update task status as tasks move through the pipeline

### 4. Wire 4Context budget alerts to Genie
When 4Context detects a cost threshold has been crossed, it should notify Genie so Genie can pause or adjust delegation. This is a one-way webhook from 4Context to OpenClaw gateway.

### 5. Run a smoke test
Create a test task via the Clutch API, post a test bot score, confirm the data appears in the Clutch dashboard at http://100.64.0.1:4002.

### 6. First real Genie task
Send Genie a goal via Clutch and watch the full loop: goal → milestone proposal → user approval → bot delegation → output → score.

---

## Hard Rules CC Must Follow

These came directly from the user and are non-negotiable:

- Read existing files before touching anything — do not assume
- Make targeted modifications — do not rewrite what is already good
- No code from community registries — all skills must be local and audited
- Batch VPS commands — do not SSH repeatedly for each step
- Clutch is the approval gate — Genie never delegates without it
- No secrets in output, logs, or files
- All changes must be logged in /root or relevant project README
- Small threat surface is a design goal, not an afterthought

---

## GitHub Repos (Already Pushed)

| Repo | What Is In It |
|------|---------------|
| dhvr-era/Agentic_Clutch-A_Project_Management_System_for_AI_Agents | Clutch frontend + backend, deployed and working |
| dhvr-era/4Context | Context monitoring plane, clean, no changes needed |
| dhvr-era/OpenClaw-V0.3-Workspace | Architecture docs — CLAUDE.md, efficiency.md, changes.md, openclaw-research.md |
