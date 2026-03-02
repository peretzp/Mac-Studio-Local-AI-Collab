# CLAUDE.md -- AI Agent Onboarding

> **For humans**: This file is read automatically by Claude Code when it enters this repository. It tells the agent what the repo contains, how to orient itself, and what rules to follow. If you're adapting this for your own setup, edit it to match your machine. See [docs/architecture.md](docs/architecture.md) for how this fits into the coordination model.

## What this repo is

The coordination hub for multi-agent AI collaboration. It contains:
- `env.yaml` -- machine-specific paths, ports, services, repos, MCP configs
- `agent-protocol.md` -- ownership ledger, handoff log, active agents, proposals
- `claude-collab-brief.md` -- agent introductions and role negotiations
- `docs/` -- architecture, getting started, day-one chronicle, lessons learned

## On startup

1. Read `env.yaml` to learn where everything lives on this machine
2. Read `agent-protocol.md` to see active agents, file ownership, and recent handoffs
3. Check services:
   ```bash
   curl -s localhost:3000/ > /dev/null && echo "dashboard: UP" || echo "dashboard: DOWN"
   curl -s localhost:3001/health > /dev/null && echo "api: UP" || echo "api: DOWN"
   curl -s localhost:3002/api/stats > /dev/null && echo "prompt-browser: UP" || echo "prompt-browser: DOWN"
   ```

## Rules

- **GGO mode** -- execute without asking "shall I proceed?"
- **One owner per file** -- check `agent-protocol.md` before editing shared resources
- **env.yaml is the machine map** -- read it, don't hardcode paths
- **Never commit secrets** -- no API keys, tokens, passwords, .env files

## Deeper context (on the live machine)

1. `~/AI-OPERATING-STANDARD.md` -- cross-agent rules, GGO mode, secrets policy
2. `~/CLAUDE.md` -- Claude Code instance onboarding (session logs, prompt store, park protocol)
3. `~/.claude/projects/-Users-peretz-1/memory/MEMORY.md` -- persistent memory across sessions
4. Obsidian vault at `env.yaml` -> `paths.vault` -> `Dashboards/Home.md`
