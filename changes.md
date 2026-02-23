# Changes Log — OpenClaw V0.3
> Append-only. Every change gets logged here.

---

## 2026-02-23 — Session: Foundation Setup

### Phase 0: Dev Environment
- [x] Created `.gitconfig` — identity: dhvr-era / dheeruerabelly@gmail.com, defaults: main branch, autoSetupRemote
- [x] Dev environment audited — SSH keys in `.ssh/` + `DO_2026/`, no prior .gitconfig, `.claude/` config was empty

### Phase 1: Sandbox Lockdown
- [x] Created `CLAUDE.md` — AI boundary rules: workspace-only access, no external reads, VPS only on instruction
- [x] Configured `settings.json` — allow workspace R/W, deny SSH keys + credentials paths
- [x] Updated `MEMORY.md` — added sandbox rules, efficiency protocol, compliance frameworks, git identity

### Phase 2: Foundation Docs
- [x] Created `efficiency.md` — token efficiency rating, compaction protocol, multi-model strategy, session hygiene
- [x] Created `changes.md` — this file

### VPS Audit
- [x] Connected via Tailscale (100.64.0.1) — no SSH key needed
- [x] Mapped 3 systems: 4Context (Python beacon+hub), Clutch (pulled), OpenClaw (agentic engine)
- [x] Found 8 active services across ports 80, 3000, 4000, 4001, 8000, 18789, 18792, 8080
- [x] Found exposed credentials in openclaw.json output (API keys, bot tokens) — NOT stored locally

### VPS Security Lockdown
- [x] **Killed Mission Control** — PM2 stopped + deleted, no longer running
- [x] **Removed Nginx public proxy** — default site deleted, nginx reloaded
- [x] **Removed UFW port 80 public** — confirmed `Connection refused` from public IP
- [x] **All services locked to Tailscale only** — ports 80, 3000, 4000, 4001, 8000, 18789, 18792 on `tailscale0` interface only
- [x] **Remaining public**: SSH (22) + Headscale (8080, required for Tailscale mesh)
- [x] Mission Control dashboard redundant (Clutch replaces it) — workspace data preserved

### Credential Isolation
- [x] Installed `age` encryption on VPS
- [x] Extracted 4 secrets from `openclaw.json` → `.secrets.env` (OpenRouter, Google AI, Gateway token, Discord token)
- [x] Scrubbed `openclaw.json` — all secrets replaced with `${ENV_VAR}` references
- [x] User encrypted `.secrets.env` → `.secrets.env.age` (passphrase-protected)
- [x] Plaintext `.secrets.env` shredded — AI can no longer read secrets

## 2026-02-23 — Session: Genie Architecture

### Phase 1: Genie Context + Config
- [x] **SOUL.md** — added `## Role` section (orchestrator identity, never executes)
- [x] **AGENTS.md** — appended Genie core loop, delegation rules, approval gate
- [x] **HEARTBEAT.md** — added 4 specific checks (Clutch queue, bot reports, 4Context alerts, scores)
- [x] **TOOLS.md** — added service endpoints + blocked tools list
- [x] **USER.md** — updated name (dhvr-era), added active projects + priorities
- [x] **openclaw.json** — hardened: maxConcurrent 4→2, subagents 8→4, maxSpawnDepth 1, reserveTokensFloor 8192, maxHistoryShare 0.6, Discord disabled
- [x] **BOT-REGISTRY.md** — created at `workspace/agents/BOT-REGISTRY.md` — scraper-bot-01 Tier 1 Entry

### Phase 2: Scraper Bot
- [x] **agents/scraper/ROLE.md** — job definition, allowed/blocked tools, output format
- [x] **agents/scraper/TASK-TEMPLATE.md** — per-task injection template

### Pending
- [ ] Rotate exposed API keys on respective platforms (user action: OpenRouter, Google, Telegram, Discord)
- [ ] Setup Vaultwarden (self-hosted password manager + authenticator)
- [x] **Clutch deployed** — all 4 bugs fixed, Postgres installed, systemd service running on http://100.64.0.1:4002
  - server.ts: port 4000→4002, bound to tailscale0, static file serving added
  - db.ts: JS-in-SQL seed bug fixed, added bot_scores/bot_promotions/project_reviews tables, seeded Genie+Scraper Bot
  - Postgres: installed, agents_db + agent_user created
  - node_modules: fresh Linux install on both frontend + backend
  - Frontend: built (Vite, 454KB bundle), served from same origin
  - UFW: port 4002 allowed on tailscale0 only
- [ ] Wire Genie ↔ Clutch (task lifecycle API)
- [ ] Wire 4Context → OpenClaw budget webhook
- [ ] Create `ai-standards-audit.md` (6 frameworks)
- [ ] Create security/compliance/architecture .MD docs
