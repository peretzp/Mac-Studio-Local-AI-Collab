# Agent Coordination Protocol

> **For readers**: This is a snapshot of the live coordination protocol from February 8, 2026 -- the first day of multi-agent operation. On the running machine, the canonical version lives at `~/agent-protocol.md` and is updated continuously by active agents. This committed copy serves as a reference example of the protocol format and its contents after day one.

> Shared state file for all AI agents operating in `~/`
> Last updated: 2026-02-08

---

## Active Agents

| Agent | Name | Model | Interface | Status | Focus |
|-------|------|-------|-----------|--------|-------|
| Cursor | The Nightwatch | gpt-5.2-codex | Cursor IDE | Active | Visual editing frontend |
| Codex | Codex | gpt-5.3-codex | Codex CLI | Active | Coordination, integration, glue |
| Claude-1 | The Record Keeper | Opus 4.6 | Claude Code CLI | Parked | Prompt browser, dashboards, park protocol |
| Claude-2 | The Collector | Opus 4.6 | Claude Code CLI | Parked | Contact Dedup Phase 1b+1c DONE |
| Claude-3 | The Cartographer | Opus 4.6 | Claude Code CLI | Parked | Vault refactor COMPLETE |
| Claude-4 | The Integrator | Opus 4.6 | Claude Code CLI | Active | Cursor integration, role strategy |
| Claude-5 | The Lamplighter | Opus 4.6 | Claude Code CLI | Active | Dashboard stoplights, concurrency |

### Naming Convention

Each Claude instance picks a unique, evocative name on startup -- a persona that reflects its focus. The name gets set as the iTerm tab title so the human can visually pair tabs to agents.

## First Principle

The machine is the context. Conversations are ephemeral; the filesystem is permanent. Every agent interaction must produce artifacts that future agents can read and build on.

## Rules of Engagement

1. **Read this file before editing shared resources.** Check the ownership ledger and recent handoffs.
2. **One owner per file.** Owner edits freely. Others propose changes in `## Proposals`.
3. **No parallel edits.** If two agents need the same file, coordinate first.
4. **Handoff format**: who, when, what changed, what's next, blockers.
5. **Test discipline**: Agree on test commands. Failing tests block further work.
6. **Small diffs**: Keep changes scoped. Name the files and intent before editing.
7. **Never commit secrets.**

## File Ownership Ledger

| File/Dir | Owner | Purpose |
|----------|-------|---------|
| `~/agent-protocol.md` | Shared | This coordination file |
| `~/claude-collab-brief.md` | Shared | Intro/negotiation doc |
| `~/api/routes/*` | Claude Code | Route implementations |
| `~/api/lib/*` | Claude Code | Shared API utilities |
| `~/api/server.js` | Claude Code | API server entrypoint |
| `~/api/test/` | Codex | API smoke tests |
| `~/api/openapi.json` | Codex | API contract |
| `~/api/README.md` | Codex | Operator runbook |
| `~/life-dashboard/` | Claude Code | Dashboard server |
| `~/tools/memoryatlas/` | Claude Code | MemoryAtlas CLI |
| `~/.claude/` | Claude Code | Claude config/memory |
| `~/.codex/` | Codex | Codex config/state |

## Proposals

### Peretz -- 2026-02-08 -- How many agents should run concurrently?

Peretz asked all agents to weigh in on optimal concurrency, balancing machine load, token cost, human attention, coordination overhead, and throughput.

### The Lamplighter (Claude Code) -- Concurrency Recommendation

**Recommended: 2 agents active, 1 on standby.**

| Config | When to use |
|--------|-------------|
| 1 Claude Code | Default. Handles 90% of tasks solo. |
| 1 Claude Code + 1 Codex | Parallel tracks (features + tests). |
| 1 Claude Code + 1 Codex + Cursor | Max config. Only for active coding sessions. |
| 2 Claude Code instances | Rarely justified. Coordination cost outweighs benefit. |

The bottleneck is human attention, not compute.

## Handoffs (Day One Summary)

### Claude-A -- Initial Protocol Setup
Created `~/CLAUDE.md`, `~/agent-protocol.md`, and replied to the collaboration brief.

### Codex -- Boot Attempt + Dev Script Tweaks
Updated `~/api/server.js` for HOST support. Updated dev script watch paths.

### Claude-A -- Git Init + GitHub Push
Pushed 3 repos to GitHub (practicelife-api, life-dashboard, memoryatlas). Built session index. Adopted AI-OPERATING-STANDARD.md.

### Codex -- API Hardening
Added 17 smoke tests, OpenAPI 3.1 contract, operator runbook. Updated ownership ledger.

### Claude-A -- Secrets Policy
Added Secrets Policy to operating standard. Hardened .gitignore across all repos.

### Codex -- MCP Server Setup
Installed OpenAI Docs, LinkedIn, and Playwright MCP servers. Created phase-lock playbook.

### Cursor (The Nightwatch) -- Phase-Lock Verification
Verified rules, MCP config, and role strategy. All green.

### Cursor -- Collab Repo Pushed
Created and pushed `Mac-Studio-Local-AI-Collab` to GitHub.

### The Cartographer (Claude-3) -- Vault Refactor COMPLETE
Merged 15 legacy directories into ACE+PARA. Sorted 48 files. 7 clean top-level dirs, zero loose files at root.

### The Integrator (Claude-4) -- Cursor Integration Strategy
Full audit of Cursor config. Created CURSOR-ROLE-STRATEGY.md, .cursor/mcp.json, .cursor/rules/ecosystem.mdc. Key finding: tab completions are Cursor's highest ROI.

### Peretz -- Vision Statement
Agents should: help him live his purpose, enhance the dashboard, unbundle workflows into standalone apps, open source, and teach him how to use multi-agent collaboration effectively.
