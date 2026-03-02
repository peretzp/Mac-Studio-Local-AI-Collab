# Mac Studio Local AI Collab

**One machine. Three AI agents. Zero hope-based coordination.**

This repo documents a working multi-agent AI development setup running on a single Mac Studio Pro. Claude Code, Codex, and Cursor operate simultaneously on real projects — coordinated through shared filesystem protocols, not chat windows.

This isn't a framework or a library. It's a field-tested reference architecture: the config files, coordination protocols, and lessons learned from actually running 3+ AI agents on one machine across a full working day.

## Why this exists

Most "multi-agent" demos are toy examples where agents talk to each other in a loop. This is different. These are three separate AI products — made by different companies, with different interfaces — working on the same codebase at the same time:

- **Claude Code** (Anthropic) — CLI-based, runs in iTerm2, handles architecture and multi-file refactors
- **Codex** (OpenAI) — desktop app, writes tests, contracts, and integration glue
- **Cursor** (Cursor Inc.) — IDE with AI, handles visual editing, tab completions, inline fixes

They don't share a conversation. They share a filesystem. The coordination protocol is a markdown file that every agent reads on startup and appends to when handing off work. It's boring, it's simple, and it works.

## What happened on Day One

On February 8, 2026, this system ran for the first time. Seven named agent instances operated across the day:

| Instance | Agent | What it did |
|----------|-------|-------------|
| The Record Keeper | Claude Code | Built the prompt browser, set up dashboards, established the park protocol |
| The Collector | Claude Code | Deduplicated 9,506 contacts down to 6,361 golden records |
| The Cartographer | Claude Code | Refactored the entire Obsidian vault (15 legacy dirs into clean ACE+PARA) |
| The Integrator | Claude Code | Wired Cursor into the ecosystem, wrote the role strategy |
| The Lamplighter | Claude Code | Built dashboard stoplights, live charts, agent concurrency analysis |
| The Nightwatch | Cursor/Codex | Visual editing frontend, phase-lock verification |
| Codex | Codex | API smoke tests (17 passing), OpenAPI contract, operator runbook |

Three repos were pushed to GitHub. An API was scaffolded and hardened. A vault was reorganized. Contacts were deduplicated. MCP servers were wired up. All of it coordinated through `agent-protocol.md` — a single shared file.

See [docs/day-one.md](docs/day-one.md) for the full chronicle.

## The coordination model

```
agent-protocol.md (shared state)
├── Active Agents table        — who's running right now
├── File Ownership Ledger      — one owner per file, no conflicts
├── Proposals                  — request to edit someone else's file
└── Handoffs                   — structured log: what changed, what's next, blockers
```

Every agent reads this file before touching shared resources. Every agent appends to it when done with a chunk of work. The format is deliberately simple — markdown that any AI can parse.

**Key rules:**
1. **One owner per file.** The owner edits freely. Everyone else proposes changes.
2. **No parallel edits.** If two agents need the same file, they coordinate first.
3. **Structured handoffs.** Every handoff logs: what changed, what's next, blockers.
4. **GGO mode.** Agents execute without asking "shall I proceed?" Speed over ceremony.

See [docs/architecture.md](docs/architecture.md) for the full technical design.

## The env.yaml pattern

Instead of hardcoding paths, every agent reads `env.yaml` on startup:

```yaml
machine:
  name: "Mac Studio Pro"
  arch: "arm64 (M2 Max)"
  ram: "64GB"

paths:
  vault: "/Users/you/path/to/obsidian"
  api: "/Users/you/api"

services:
  api:
    port: 3001
    health: "http://localhost:3001/health"
```

This maps abstract names to concrete paths. To run on a different machine, copy `env.yaml` to `env.local.yaml` (gitignored), edit your paths, and every agent just works.

## Repo structure

```
.
├── README.md                  # You are here
├── CLAUDE.md                  # Onboarding instructions for Claude Code instances
├── env.yaml                   # Machine config: paths, ports, services, repos, MCP
├── env.example.yaml           # Template for your own machine
├── agent-protocol.md          # Coordination hub snapshot (ownership, handoffs, proposals)
├── claude-collab-brief.md     # Agent introductions and role negotiations
├── LICENSE                    # MIT
├── .gitignore                 # Blocks secrets, local overrides, OS files
└── docs/
    ├── architecture.md        # Technical design: agents, protocols, coordination
    ├── getting-started.md     # Step-by-step guide to replicate this on your machine
    ├── day-one.md             # Chronicle of the first multi-agent session
    └── lessons-learned.md     # What actually works and what doesn't
```

## Quick start

Want to try this yourself? See [docs/getting-started.md](docs/getting-started.md) for the full guide. The short version:

1. Fork this repo
2. Copy `env.yaml` to `env.local.yaml` and edit paths for your machine
3. Create `agent-protocol.md` in your home directory
4. Point your AI agents at the repo and tell them to read `env.yaml` first
5. Give each agent a clear role and file ownership boundaries

## The agent stack

| Agent | Role | Interface | Best for |
|-------|------|-----------|----------|
| **Claude Code** | Orchestrator | CLI (iTerm2) | Architecture, multi-file refactors, session continuity |
| **Codex** | Verifier | Desktop app | Tests, contracts, runbooks, integration glue |
| **Cursor** | Editor | IDE | Tab completions, inline fixes, visual diffs |
| **Claude Desktop** | Browser | Desktop app | Vault access, LinkedIn, browser automation |

## Key insights

1. **The filesystem is the shared memory.** Agents don't need to talk to each other. They read and write files. The conversation window is ephemeral; the filesystem is permanent.

2. **File ownership prevents conflicts.** The single most important coordination rule. One owner per file. Period.

3. **Structured handoffs beat free-form notes.** Three fields: what changed, what's next, blockers. Every agent writes this on every handoff. It's the minimum viable coordination protocol.

4. **Name your agents.** Giving each instance a name (The Cartographer, The Lamplighter) makes the handoff log legible. "Claude-3 handed off to Claude-4" is noise. "The Cartographer completed vault refactor, The Integrator picking up Cursor wiring" is signal.

5. **Two agents is the sweet spot.** One primary + one parallel. Three works for a power session. Four starts to cost more in coordination overhead than it saves in throughput.

See [docs/lessons-learned.md](docs/lessons-learned.md) for the full analysis.

## Related repos

| Repo | What | URL |
|------|------|-----|
| practicelife-api | Zero-dep Node.js API (port 3001) | [peretzp/practicelife-api](https://github.com/peretzp/practicelife-api) |
| life-dashboard | Single-file dashboard (port 3000) | [peretzp/life-dashboard](https://github.com/peretzp/life-dashboard) |
| memoryatlas | Voice memo to Obsidian indexer | [peretzp/memoryatlas](https://github.com/peretzp/memoryatlas) |

## About this documentation

This repo is itself a product of the system it describes. The documentation was written, reviewed, and refined by AI agents operating under the coordination protocol — then curated by a human. The agent that manages this repo's public-facing documentation is a Cursor instance named **Telos** (purpose). If the writing reads like it was produced by someone who was there, that's because it was.

## License

MIT. Fork it, adapt it, make it yours.
