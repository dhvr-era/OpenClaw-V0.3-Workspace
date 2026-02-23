# CLAUDE.md — OpenClaw V0.3 AI Boundary & Operating Rules

## Workspace Boundary (HARD RULE)
- AI operates ONLY within `D:\node03\202602_OpenClaw_V0.3\`
- NO reading, writing, or executing outside this directory
- NO access to: SSH keys, `.ssh/`, `DO_2026/`, `.credentials.json`, other user dirs
- NO access to other projects under `D:\node03\` (Articles, Projects, Test, White Papers)

## VPS Access
- SSH to VPS ONLY when user explicitly instructs
- Never store or echo SSH keys, IPs, or credentials in output
- Log all VPS operations in `changes.md`

## Operating Protocol
- ALL changes must be logged in `changes.md` (append-only)
- Quality > Quantity — concise, deterministic, high-signal output
- Reference existing .MD files instead of re-explaining context
- No file creation outside workspace boundary

## Efficiency Protocol
- Follow `efficiency.md` for token budget and compaction rules
- Use checkpoint compaction at session boundaries
- Minimal redundancy — structured .MD over prose
- Short, precise responses unless depth explicitly requested

## Security Protocol
- Security by design, not bolt-on
- All architecture decisions must consider OWASP Top 10 + OWASP AI Top 10
- Compliance mapping against: NIST AI RMF, EU AI Act, ISO 42001, ISO 27001, SOC2
- Never expose sensitive data in responses or files
