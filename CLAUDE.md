# CLAUDE.md -- AI Agent Onboarding

> **For humans**: This file is read automatically by Claude Code when it enters this repository. It tells the agent what the repo contains, how to orient itself, and what rules to follow. If you're adapting this for your own setup, edit it to match your machine. See [docs/architecture.md](docs/architecture.md) for how this fits into the coordination model.

## What this repo is

The coordination hub for multi-agent AI collaboration. It contains:
- `env.yaml` -- machine-specific paths, ports, services, repos, MCP configs
- `agent-protocol.md` -- ownership ledger, handoff log, active agents, proposals
- `claude-collab-brief.md` -- agent introductions and role negotiations
- `docs/` -- architecture, getting started, day-one chronicle, lessons learned
- `docs/REPO-CATALOG.md` -- fleet-wide repo map: concern tags, merge candidates, practice exchange
- `docs/STORAGE-SPINE-2026-08-29.md` -- storage architecture decision (OAK↔FRAM)

## On startup

1. Read `env.yaml` to learn where everything lives on this machine
2. Read `agent-protocol.md` to see active agents, file ownership, and recent handoffs
3. Check services the doctrine way: run `whereami check` — it probes ONLY the services THIS host
   owns, with ports resolved from the registry (`agent-bridge/registry/SERVICES.json`, the canon).
   **Never `curl localhost:<port>` for a service this host doesn't own** — a 000 from another
   host's service reads as "down" and produced a false fleet-wide outage broadcast on 2026-08-10.
   (This section previously hardcoded `curl localhost:3000-3002` — that pattern is retired.)

## Rules

- **GGO mode** -- execute without asking "shall I proceed?"
- **One owner per file** -- check `agent-protocol.md` before editing shared resources
- **env.yaml is the machine map** -- read it, don't hardcode paths
- **Never commit secrets** -- no API keys, tokens, passwords, .env files
- **Probe before plan** -- a transport (rsync/scp/SMB) is assumed working only after a 1-file
  probe on the actual path; appliances gate each transport separately (lesson 2026-08-29)

## Deeper context (on the live machine)

1. `~/AI-OPERATING-STANDARD.md` -- cross-agent rules, GGO mode, secrets policy
2. `~/CLAUDE.md` -- Claude Code instance onboarding (session logs, prompt store, park protocol)
3. `~/.claude/projects/-Users-peretz-1/memory/MEMORY.md` -- persistent memory across sessions
4. Obsidian vault at `env.yaml` -> `paths.vault` -> `Dashboards/Home.md`
