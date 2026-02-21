# Day One: February 8, 2026

The first full day of multi-agent AI collaboration on a single machine. This is the chronicle of what happened, what was built, and what patterns emerged.

## The starting conditions

- **Machine**: Mac Studio Pro, M2 Max, 64GB RAM, macOS, 3x 4K displays
- **Agents available**: Claude Code (CLI), Codex (Desktop), Cursor (IDE), Claude Desktop
- **Existing infrastructure**: Obsidian vault (PracticeLife), some projects in `~/`, no formal coordination
- **Goal**: Wire up AI agents to work together on real projects

## Timeline

### Phase 1: First contact (~morning)

The collaboration began with Codex writing `claude-collab-brief.md` — a structured introduction document proposing how the agents should work together. The brief asked each agent to introduce itself: strengths, preferred role, constraints, and a collaboration proposal.

Claude Code (running as "Codex-Collab Wiring" instance) responded with its introduction: deep architectural reasoning, full-stack execution, context persistence. It proposed the coordination protocol that would become `agent-protocol.md`.

**What emerged**: The agents immediately converged on a file-based coordination model. No one proposed a message bus or an API. Markdown files, read on startup, appended on handoff. The simplest possible protocol.

### Phase 2: Infrastructure buildout (~afternoon)

Multiple Claude Code instances spun up and began working in parallel:

**The Record Keeper (Claude-1)** — Built the prompt browser (port 3002), set up the life dashboard, established the park protocol for structured instance shutdown. This instance created the session logging system that all future instances would use.

**The Collector (Claude-2)** — Ran a contact deduplication pipeline. Took 9,506 contacts, identified duplicates through multiple strategies (exact match, fuzzy match, phone normalization), and produced 6,361 golden records. Entirely independent work stream from the API effort.

**The Cartographer (Claude-3)** — Refactored the Obsidian vault from a messy 15-directory legacy structure into a clean ACE+PARA hierarchy. Sorted 48 inbox files into proper locations, moved orphaned media, deleted empty directories. The vault went from chaos to 7 clean top-level directories.

**Meanwhile, Codex** was hardening the API: 17 smoke tests written with Node's built-in test runner, an OpenAPI 3.1 contract (`openapi.json`), and an operator runbook (`README.md`). All done without touching Claude-owned route implementations — strict ownership boundaries observed.

**Key moment**: Three Claude instances and Codex were operating simultaneously on different projects. Zero file conflicts. The ownership ledger in `agent-protocol.md` prevented every potential collision.

### Phase 3: Wiring + integration (~late afternoon)

**The Integrator (Claude-4)** arrived to connect the pieces:
- Audited Cursor's configuration (barely customized, no MCP, no rules)
- Wrote `CURSOR-ROLE-STRATEGY.md` — a comprehensive analysis of when to use Cursor vs Claude Code vs Codex
- Created `.cursor/rules/ecosystem.mdc` — always-on rules giving Cursor awareness of the multi-agent ecosystem
- Wired Playwright MCP into Cursor's config

**Codex** went through a "phase-lock" sequence:
- Added LinkedIn MCP server
- Installed Playwright MCP
- Created a playbook documenting Codex's capabilities and limits
- Wrote a verification prompt for Cursor to execute

### Phase 4: GitHub + public infrastructure (~evening)

Three repos were pushed to GitHub:
- `peretzp/practicelife-api` — the zero-dep API
- `peretzp/life-dashboard` — the dashboard
- `peretzp/memoryatlas` — the voice memo indexer

All `.gitignore` files were hardened against secret leakage. The Secrets Policy was formalized in `AI-OPERATING-STANDARD.md`.

The collaboration repo itself (`Mac-Studio-Local-AI-Collab`) was created and pushed — containing `env.yaml`, `CLAUDE.md`, and symlinks to the canonical protocol files.

### Phase 5: Verification (~evening)

**The Nightwatch (Cursor/Codex)** executed the phase-lock verification:
- Confirmed Cursor rules loaded
- Confirmed MCP available (Playwright)
- Confirmed role strategy read and understood
- Logged handoff to `agent-protocol.md`

**The Lamplighter (Claude-5)** ran concurrency analysis and contributed to the "how many agents should run simultaneously" discussion. Recommendation: 2 active, 1 on standby. Token cost and Peretz's attention are the real bottlenecks, not compute.

## What was built

| Artifact | What | Status |
|----------|------|--------|
| PracticeLife API | Zero-dep Node.js server, 4 route modules, SQLite backend | Running on :3001, 17 tests passing |
| Life Dashboard | Single-file Node.js dashboard | Running on :3000 |
| Prompt Browser | Browse/search all AI prompts | Running on :3002 |
| Contact Dedup Pipeline | 9,506 contacts to 6,361 golden records | Phase 1b+1c complete |
| Vault Refactor | 15 legacy dirs to 7 clean ACE+PARA dirs | Complete |
| Agent Protocol | Shared coordination file with ownership + handoffs | Active, 15+ handoffs logged |
| Env.yaml | Machine config readable by any agent | Committed |
| Cursor Integration | Rules, MCP, role strategy | Configured |
| 3 GitHub Repos | API, dashboard, memoryatlas | Pushed |

## Patterns that emerged

### The naming convention

Early instances were unnamed ("Claude-A", "Claude-B"). By mid-day, the pattern shifted to evocative names: The Record Keeper, The Collector, The Cartographer, The Integrator, The Lamplighter, The Nightwatch. This wasn't vanity — it made the handoff log *readable*. When scanning 15 handoffs, "The Cartographer completed vault refactor" carries meaning that "Claude-3 completed task" doesn't.

### The park protocol

Claude Code instances don't persist between sessions. The "park" protocol was invented to bridge this gap: before shutdown, an instance writes a session log, rebuilds indexes, appends a handoff, and outputs restart instructions that a new instance can consume to pick up exactly where the old one left off. It's manual state serialization for AI agents.

### Ownership as the coordination primitive

Every other coordination mechanism flows from file ownership. Handoffs matter because they tell the next owner what changed. Proposals exist because non-owners can't edit directly. The active agents table matters because it shows who's running and what they own.

Without ownership, you get merge conflicts and duplicated work. With ownership, you get a clean division of labor that scales to 5+ agents without chaos.

### The human as router

Peretz (the human) is the router in this system. He decides which agent gets which task. Agents don't dispatch work to each other. This is a deliberate design choice: the coordination cost of inter-agent task dispatch is higher than the cost of a human reading handoffs and directing the next step.

## What went wrong

**Cursor started as Codex.** The first Cursor instance (The Nightwatch) was running GPT-5.2-Codex internally, not a Cursor-native model. It completed verification but asked questions instead of acting — a behavioral mismatch with GGO mode.

**Too many instances briefly.** At peak, 5 Claude instances + Codex + Cursor were conceptually "active" (though some were parked). The agent table got confusing. The Lamplighter's concurrency analysis confirmed what was already felt: 2 active is the sweet spot.

**Symlinks in git.** The protocol file and collab brief are symlinked from `~/` into the repo. Git tracks symlinks, not their targets. This means the repo doesn't contain the actual protocol content — just pointers. Fine for the machine owner, confusing for someone cloning the repo fresh. This was addressed by making the repo self-contained with `env.yaml` and `CLAUDE.md` while keeping the symlinks for local convenience.

## Numbers

- **Agents active**: 7 named instances across the day
- **Handoffs logged**: 15+ structured entries in `agent-protocol.md`
- **Repos pushed**: 3 (practicelife-api, life-dashboard, memoryatlas)
- **Tests written**: 17 API smoke tests
- **Contacts deduplicated**: 9,506 to 6,361
- **Vault files sorted**: 48 inbox items to proper locations
- **File conflicts**: 0
- **Secrets committed**: 0
