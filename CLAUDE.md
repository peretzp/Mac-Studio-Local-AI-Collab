# CLAUDE.md — AI Onboarding for This Repo

## What this repo is

The **field manual** for multi-agent AI collaboration on Peretz's Mac Studio Pro. It contains:
- `env.yaml` — machine-specific paths, ports, services, repos, MCP configs
- `agent-protocol.md` — symlink to `~/agent-protocol.md` (coordination hub)
- `claude-collab-brief.md` — symlink to `~/claude-collab-brief.md`

## On startup

1. Read `env.yaml` to learn where everything lives on this machine
2. Read `agent-protocol.md` to see active agents, file ownership, and recent handoffs
3. Check services: `curl -s localhost:3000/ && curl -s localhost:3001/health && curl -s localhost:3002/api/stats`

## Rules

- **GGO mode** — execute without asking "shall I proceed?"
- **One owner per file** — check `agent-protocol.md` before editing shared resources
- **Symlinks are canonical** — `agent-protocol.md` and `claude-collab-brief.md` point to `~/`; edit the originals, not copies
- **env.yaml is committed** — it reflects Peretz's machine. Others fork and create `env.local.yaml`
- **Never commit secrets** — no API keys, tokens, passwords, .env files

## The full onboarding path (if you want deeper context)

1. `~/AI-OPERATING-STANDARD.md` — cross-agent rules, GGO mode, footer policy
2. `~/CLAUDE.md` — Claude Code instance onboarding (session logs, prompt store, park protocol)
3. `~/.claude/projects/-Users-peretz-1/memory/MEMORY.md` — persistent memory across sessions
4. Obsidian vault at the path in `env.yaml` → `paths.vault` → `Dashboards/Home.md`
