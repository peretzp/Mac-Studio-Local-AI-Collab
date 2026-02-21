# Mac Studio Local AI Collab

Multi-agent AI collaboration on a local Mac Studio Pro. Claude Code, Codex, and Cursor working together on real projects — coordinated through shared protocols, not hope.

## What this repo is

A **field manual** for AI agents operating on one machine. Any agent that drops into this repo knows:

- **Where things live** — [`env.yaml`](env.yaml) maps every path, port, repo, and service
- **How agents coordinate** — [`agent-protocol.md`](agent-protocol.md) tracks ownership, handoffs, and proposals
- **What the rules are** — who owns what files, how to avoid conflicts, how to hand off work
- **What's running** — services, MCP servers, background processes

📚 **New here?** Start with the [Setup Guide](SETUP-GUIDE.md) to replicate this system on your machine.

🎯 **Looking for patterns?** See [Coordination Patterns](PATTERNS.md) for proven multi-agent workflows.

## Quick start (for a new AI agent)

```bash
# 1. Read the environment
cat env.yaml

# 2. Read the coordination protocol
cat agent-protocol.md

# 3. Check what services are running
curl -s localhost:3000/ > /dev/null && echo "dashboard: UP" || echo "dashboard: DOWN"
curl -s localhost:3001/health && echo " api: UP" || echo "api: DOWN"
curl -s localhost:3002/api/stats > /dev/null && echo "prompt-browser: UP" || echo "prompt-browser: DOWN"

# 4. Read the collaboration brief
cat claude-collab-brief.md
```

## File structure

```
.
├── README.md              # You're here
├── SETUP-GUIDE.md         # Step-by-step setup for your machine
├── PATTERNS.md            # Proven coordination patterns
├── env.yaml               # Machine-specific paths, ports, services, repos
├── env.local.yaml         # (gitignored) Local overrides for other machines
├── CLAUDE.md              # AI onboarding checklist
├── agent-protocol.md      # → symlink to ~/agent-protocol.md (canonical)
├── claude-collab-brief.md # → symlink to ~/claude-collab-brief.md (canonical)
└── .gitignore
```

## The env.yaml pattern

`env.yaml` is the single source of truth for machine configuration. It maps abstract names to concrete paths:

```yaml
paths:
  vault: "/Users/peretz_1/Library/Mobile Documents/..."
  api: "/Users/peretz_1/api"

services:
  api:
    port: 3001
    health: "http://localhost:3001/health"
```

**To run on a different machine**: copy `env.yaml` to `env.local.yaml`, edit paths, and have your agents read the local version first.

## The agent stack

| Agent | Role | Interface |
|-------|------|-----------|
| **Claude Code** | Orchestration, architecture, multi-file refactors | CLI (iTerm2) |
| **Codex** | Tests, contracts, integration glue | Codex Desktop |
| **Cursor** | Visual editing, tab completions, inline fixes | Cursor IDE |
| **Claude Desktop** | Vault access, LinkedIn, browser automation | Desktop app |

## Coordination model

- **One owner per file** — avoids merge conflicts between agents
- **Handoff protocol** — agents log what changed, what's next, blockers
- **Symlinked shared files** — `agent-protocol.md` and `claude-collab-brief.md` live at `~/` and are symlinked here so the repo always reflects current state
- **GGO mode** — execute without routine confirmation. Speed over ceremony.

## Related repos

| Repo | What | URL |
|------|------|-----|
| practicelife-api | Node.js API (zero-dep, port 3001) | [peretzp/practicelife-api](https://github.com/peretzp/practicelife-api) |
| life-dashboard | Single-file dashboard (port 3000) | [peretzp/life-dashboard](https://github.com/peretzp/life-dashboard) |
| memoryatlas | Voice memo → Obsidian indexer | [peretzp/memoryatlas](https://github.com/peretzp/memoryatlas) |

## Getting Started

1. **Read this README** to understand the system
2. **Follow the [Setup Guide](SETUP-GUIDE.md)** to adapt it for your machine (30 min)
3. **Study the [Patterns](PATTERNS.md)** to learn coordination workflows
4. **Try a simple task** with two agents coordinating through `agent-protocol.md`

## Contributing

This repository documents real patterns from real use. Contributions welcome:

- **Patterns**: Share coordination patterns that worked for you
- **Tools**: Document integration with other AI tools (Claude Desktop, Windsurf, etc.)
- **Improvements**: Better onboarding, clearer docs, useful scripts

Open issues or PRs at https://github.com/peretzp/Mac-Studio-Local-AI-Collab

## License

This is a personal workflow repo. Fork it if the patterns are useful to you.
