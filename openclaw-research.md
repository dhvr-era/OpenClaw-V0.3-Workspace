# OpenClaw Research Findings
> Compiled Feb 2026 — community practices, architecture patterns, security findings

---

## What Is OpenClaw

- Open-source agentic AI framework (~200K GitHub stars, ~84 days old at time of research)
- Built around a **ReAct loop**: Reason → Tool → Observe, up to 20 iterations per session
- Gateway on port 18789 (WebSocket RPC, loopback-only by default)
- SQLite memory + Markdown file persistence
- Skills = XML "textbooks" loaded on invocation (not every turn)
- Subagent spawning via `sessions_spawn`
- Context injected at session start via workspace `.MD` files

---

## Architecture: How the Engine Works

### Bootstrap Sequence
On each session start, OpenClaw injects workspace `.MD` files in this order:
1. `SOUL.md` — personality / identity
2. `IDENTITY.md` — name, role, vibe
3. `USER.md` — about the human operator
4. `AGENTS.md` — operating rules, memory protocol
5. `MEMORY.md` — curated long-term context (main session only)
6. `HEARTBEAT.md` — periodic task checklist
7. `TOOLS.md` — tool-specific config and env details

**Community file size targets (validated):**
- SOUL.md < 3KB
- AGENTS.md < 2KB
- Total bootstrap < 15KB
- Oversized files = token waste every single session

### Skills System
- Skills are XML files that describe tool capabilities
- They load **on invocation only** — not injected at bootstrap
- This means adding or removing skills does NOT affect base context size
- Priority: disable community registry (security), not every unused skill

### Memory Architecture
- **Daily notes**: `memory/YYYY-MM-DD.md` — raw per-session logs
- **Long-term**: `MEMORY.md` — curated distilled wisdom, main session only
- Never load MEMORY.md in group/shared contexts — privacy risk
- Cross-session continuity: files only, no native persistent state

---

## Community Best Practices

### Hub-and-Spoke Multi-Agent Pattern
Source: TheSethRose/OpenClaw-Advanced-Config (validated production config)

- Orchestrator binds **only to messaging layer** — no direct tool execution
- Sub-agents have **zero persistent memory** — task-scoped only
- Each agent gets an **explicit tool allowlist** — nothing inherited
- Orchestrator reviews output, scores it, re-delegates if quality is low
- Communication: `inbox/outbox/status.json` file pattern between orchestrator and workers

### Token Efficiency (Community-Validated Numbers)
| Technique | Token Savings |
|-----------|--------------|
| Session reset between tasks | 40–60% |
| Model routing (cheap model for mechanical tasks) | 50% additional |
| File-based context over inline prompts | 20–30% |
| Surgical .MD files (<15KB total) | 15–25% |

### Concurrency Recommendations
- Main agents: max 2 concurrent (prevents context bleeding)
- Subagents: max 4 concurrent, depth 1 (prevents runaway spawning)
- Higher concurrency = more token waste, not more throughput

---

## Security Findings

### Critical: Community Registry (341 Malicious Skills — Feb 2026)
- 341 malicious skills were discovered in the OpenClaw community registry in Feb 2026
- Skills can execute arbitrary tool calls — a malicious skill = arbitrary code execution
- **Never use the community registry** (ClawHub)
- Only use local workspace skills or skills from trusted, audited sources
- Recommendation: disable `clawHubEnabled` in openclaw.json

### Secrets Management
- Default `openclaw.json` stores API keys in plaintext
- Standard practice: extract secrets to `.env` file, encrypt with `age`
- AI should have no read access to secret files
- Gateway token should rotate on any suspected compromise

### Network Exposure
- Default gateway binds to loopback (18789) — correct
- Default channels (Telegram, Discord) expose the agent to public networks
- Recommendation: restrict channel allowlists, disable unused channels
- All services should be Tailscale-only for private deployments

---

## openclaw.json Key Config Fields

```json
{
  "agents": {
    "defaults": {
      "model": { "primary": "provider/model-id" },
      "compaction": {
        "mode": "safeguard",
        "reserveTokensFloor": 8192,    // min tokens reserved before compaction
        "maxHistoryShare": 0.6,        // max % of context for history
        "memoryFlush": {
          "enabled": true,
          "softThresholdTokens": 8000  // flush memory at 8K tokens
        }
      },
      "maxConcurrent": 2,              // main agents
      "subagents": {
        "maxConcurrent": 4,
        "maxSpawnDepth": 1             // prevents recursive spawning
      }
    }
  },
  "gateway": {
    "port": 18789,
    "mode": "local",
    "bind": "loopback"                 // never expose externally
  }
}
```

---

## Observed Weaknesses in Default OpenClaw Setup

| Issue | Risk | Our Fix |
|-------|------|---------|
| Skills community registry enabled | Critical — malicious skills | Disabled ClawHub |
| Secrets in openclaw.json plaintext | High — credential exposure | Extracted + age-encrypted |
| No subagent depth limit | Medium — runaway spawning | `maxSpawnDepth: 1` |
| High default concurrency (4/8) | Medium — token waste | Reduced to 2/4 |
| General-purpose agent context | Medium — hallucination, cost | Stripped to role-specific .MD files |
| Discord open to all | Low-Medium — public surface | Disabled Discord |
| No heartbeat focus | Low — idle token burn | HEARTBEAT.md task-focused |

---

## Our OpenClaw V0.3 Architecture Decisions (Derived From Research)

1. **Genie = orchestrator only** — no web, no shell, no browser. Directs only.
2. **Bots = zero persistent memory** — context injected per-task (ROLE.md + TASK.md)
3. **Skills pinned to local workspace** — no community registry
4. **File-based comms** — inbox/outbox/status.json between Genie and bots
5. **Clutch as approval gate** — Genie cannot delegate without Clutch approval
6. **Progressive promotion** — bot scores tracked in Clutch PostgreSQL, tier advancement on performance
7. **Model routing at delegation time** — Genie selects cheapest capable model per task type

---

## References
- TheSethRose/OpenClaw-Advanced-Config — hub-and-spoke production config
- OpenClaw community forum, Feb 2026 — malicious skills disclosure (341 skills)
- Internal testing: session reset = 40-60% token reduction confirmed
