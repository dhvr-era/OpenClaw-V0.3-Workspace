# Efficiency Protocol — OpenClaw V0.3

## Efficiency Rating
**Formula**: `Useful Output Tokens / Total Tokens Consumed`
Higher ratio = more work done per dollar. Target: maximize actionable output, minimize waste.

## Checkpoint Compaction Protocol
1. At session boundaries: summarize context into compact state
2. Drop stale/resolved context — carry forward only actionable items
3. Reference .MD files by path instead of re-explaining content
4. Use `changes.md` as the single source of truth for what changed

## Token Budget Rules
- **Concise by default** — short, precise, structured responses
- **No repetition** — say it once, reference it after
- **Structured .MD over prose** — tables, lists, headers > paragraphs
- **Depth only when requested** — don't over-explain unless asked
- **No filler** — every token should carry information

## Multi-Model Strategy
| Model | Use For | Cost Tier |
|-------|---------|-----------|
| Haiku | Quick searches, file reads, simple tasks | Low |
| Sonnet | Standard development, reviews, moderate complexity | Medium |
| Opus | Complex architecture, security audits, deep analysis | High |

**Rule**: Use the cheapest model that can do the job well. Escalate only when needed.

## Session Hygiene
- Start sessions by reading `MEMORY.md` + relevant .MD context
- End sessions with checkpoint compaction summary
- Never re-derive what's already documented
- Keep conversation turns lean — one clear ask, one clear answer

## Metrics (Track Per Session)
- Total tokens consumed
- Useful output tokens (deliverables created)
- Efficiency ratio
- Files created/modified
- Tasks completed vs attempted
