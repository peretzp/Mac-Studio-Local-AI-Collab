# How It Works: A Real Day of Multi-Agent Collaboration

This is a walkthrough of what actually happened on February 8, 2026 — the first full day of multi-agent operation. Everything described below comes from real session logs and the agent protocol file. Nothing is hypothetical.

## 7:00 AM — The foundation

Peretz opens iTerm and starts a Claude Code instance. The instance reads the home directory and finds an existing Obsidian vault, a life dashboard, and a MemoryAtlas tool — but no coordination infrastructure. There's also a Codex instance running in the Codex Desktop app.

Codex has already written a collaboration brief (`claude-collab-brief.md`) proposing a structure:

> We're all pointed at the same workspace folder. We're wiring up an API on a local machine. I'm Codex, acting as a coding partner and coordinator.

The brief proposes: shared goal framing, role split to avoid overlap, a single source of truth for decisions, small reviewable diffs, and conflict handling when agents touch the same files.

Claude reads the brief and replies with its own introduction — strengths, preferred role, constraints, and a counter-proposal for coordination. This exchange happens through a shared file, not a conversation. Both agents write to the same markdown document, each adding a section.

## 8:00 AM — The protocol takes shape

Claude creates `agent-protocol.md` — the file that will become the coordination hub for everything that follows. It contains:

- An active agents table (who's running, what they're doing)
- Rules of engagement (one owner per file, handoff format, no parallel edits)
- A file ownership ledger (Claude owns routes, Codex owns tests)
- Sections for proposals and handoffs

This file exists at `~/agent-protocol.md` and gets symlinked into the collaboration repo. From this point forward, every agent reads it before doing anything.

## 9:00 AM — Building the API

With coordination in place, Claude scaffolds a Node.js API at `~/api/`:

- Zero-framework HTTP server (`server.js`)
- Route modules for four domains: atlas (voice memo data), vault (Obsidian notes), system (machine state), agents (protocol and sessions)
- SQLite integration for MemoryAtlas data
- CORS enabled for dashboard integration

14 endpoints go live on port 3001. Claude tests each one with `curl` commands and commits the results.

Meanwhile, Codex reads the agent protocol and picks up its part of the work: tests, contracts, and documentation. It creates:

- 17 smoke tests using Node's built-in test runner
- An OpenAPI 3.1 specification matching the current routes
- An operator runbook with boot, verification, and test commands

Codex writes all of this in files it owns (`~/api/test/`, `~/api/openapi.json`, `~/api/README.md`) and appends a handoff to the protocol file.

Neither agent edits the other's files. Claude builds route handlers; Codex writes tests for them. The ownership boundary is enforced by convention, not by tooling.

## 10:00 AM — Git and GitHub

Claude initializes git repos and pushes three projects to GitHub:

- `peretzp/practicelife-api`
- `peretzp/life-dashboard`
- `peretzp/memoryatlas`

Before pushing, Claude audits `.gitignore` files across all three repos to ensure no secrets (API keys, `.env` files, credentials) can be committed. This becomes Rule #7 in the agent protocol.

Claude also builds a session index — a SQLite database with FTS5 that indexes all session logs for instant search. This gets wired into the auto-resume protocol: every new Claude instance runs `session-index.js rebuild` on startup.

## 11:00 AM — The Cartographer refactors the vault

A second Claude instance starts. It picks the name "The Cartographer" and focuses on Obsidian vault cleanup:

- Merges 15 legacy directories into 7 clean ACE+PARA dirs
- Sorts 48 files from the inbox
- Deletes 4 empty directories
- Moves 16 orphaned media files
- Updates the vault dashboard

By noon, the vault refactor is complete. The Cartographer writes its handoff:

> Vault now has exactly 7 clean top-level dirs: Atlas, Claude, Dashboards, Efforts, Journal, MemoryAtlas, Resources. Zero loose files at vault root.

## 12:00 PM — The Collector builds a contact pipeline

A third Claude instance — "The Collector" — tackles contact deduplication. This is a complex data engineering task:

**Phase 1b (Export):**
- Reads contacts from 6 macOS AddressBook SQLite databases
- Parses an Outlook VCF file from 2005
- Extracts 382 iMessage VCF attachments
- Enriches with social profiles (1,737), URLs (8,653 — including 971 Facebook, 358 Twitter, 253 LinkedIn)
- Total: 9,506 contacts from 8 sources

**Phase 1c (Dedup):**
- Builds a Union-Find clustering pipeline with 4 phases:
  1. Exact email match (2,630 merges)
  2. Exact phone match (1,842 merges)
  3. Exact name match (2,639 merges)
  4. Fuzzy name match at 0.92 threshold (60 merges)
- Result: 9,506 down to 6,361 golden records (33% reduction)

**Validation:**
- MX checks on 2,317 email domains (30-thread parallel `dig`)
- Phone format validation, defunct provider detection
- 8-stage cleanup pipeline brings the final count to 6,057 records
- Tiered classification: 717 inner circle, 757 active, 862 warm, 3,496 archive

The Collector writes all of this in `~/contacts-dedup/` and appends a detailed handoff.

## 2:00 PM — Codex and Claude negotiate permissions

Codex proposes a global prefix baseline — pre-approved commands that don't need human confirmation each time:

```
["npm", "run", "dev"]
["npm", "test"]
["node", "--test"]
["curl", "-sL"]
```

This gets written into `AI-OPERATING-STANDARD.md` so all future agent sessions inherit the same permissions. The human approves once; every subsequent instance reuses the approval.

Codex also adds MCP servers (LinkedIn, Playwright, OpenAI Docs) and writes a phase-lock playbook documenting its capabilities, settings, and hard limits.

## 4:00 PM — Cursor enters the game

Peretz opens Cursor IDE. Claude Code (instance "The Integrator") audits Cursor's configuration and creates:

- `~/.cursor/rules/ecosystem.mdc` — always-on rules giving Cursor awareness of the multi-agent ecosystem
- `~/.cursor/mcp.json` — Playwright MCP only (lean surface area)
- `~/CURSOR-ROLE-STRATEGY.md` — analysis of when to use Cursor vs Claude Code vs Codex

Key finding: Cursor's highest ROI is tab completions (unlimited on Pro, $0 marginal cost). It should not be used as an orchestrator — no session memory, context truncates, not designed for it.

Cursor (running as "The Nightwatch") reads the protocol, verifies its MCP configuration, and appends its own handoff.

## 5:00 PM — The collaboration repo

Cursor creates the GitHub repo that becomes this documentation:

- `peretzp/Mac-Studio-Local-AI-Collab`
- Contains: `agent-protocol.md`, `claude-collab-brief.md`, `env.yaml`, `README.md`, `CLAUDE.md`

The env.yaml pattern is established: one YAML file that maps abstract names to concrete machine paths. When this system moves to a different machine, only env.yaml changes.

## 6:00 PM — The Lamplighter analyzes concurrency

Peretz asks all agents: "How many agents should run concurrently?" The question gets posted in the Proposals section of `agent-protocol.md`.

The Lamplighter (another Claude instance) responds with a data-driven analysis:

| Config | When to use it |
|--------|---------------|
| 1 Claude Code | Default daily driver. Handles 90% of tasks solo. |
| 1 Claude + 1 Codex | Parallel tracks (Claude building, Codex testing). |
| 1 Claude + 1 Codex + Cursor | Max config. Only when actively editing in IDE. |
| 2 Claude instances | Rarely justified. Coordination cost outweighs benefit. |

The recommendation: 2 active agents, 1 on standby. The human's attention is the bottleneck, not compute.

## End of day — the numbers

By the end of February 8:

- **7 named agent instances** operated (Record Keeper, Collector, Cartographer, Integrator, Lamplighter, Tentacle Closer, Nightwatch)
- **35+ session logs** written across the system's lifetime
- **3 GitHub repos** created and pushed
- **17 API tests** passing
- **6,057 contacts** deduplicated from 9,506 raw records
- **929 voice memos** indexed
- **7 vault directories** from 15 legacy ones
- **1 coordination protocol** holding it all together

The protocol file itself — `agent-protocol.md` — grew to 700+ lines by end of day, with 15 handoff entries documenting the full chain of agent coordination.

## What made it work

1. **The protocol file was created early.** Before any real work started, agents agreed on coordination rules. This prevented the chaos that would have come from three agents editing freely.

2. **Ownership was explicit.** Every file had exactly one owner. Agents never had to guess whether it was safe to edit something.

3. **Handoffs were structured.** Each handoff answered three questions: what changed, what's next, what's blocked. The next agent could pick up without asking the human for context.

4. **Session logs were continuous.** Agents wrote logs as they worked, not just when they finished. When one instance ran out of context (this happened — see [lessons learned](lessons-learned.md)), the log was already on disk.

5. **The human stayed in the loop.** Peretz decided which agents to start, approved permission escalations, and handled actions that required physical interaction (OAuth logins, UI clicks). The agents did the work; the human provided direction and authorization.
