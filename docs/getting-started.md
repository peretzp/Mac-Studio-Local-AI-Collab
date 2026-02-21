# Getting Started: Setting Up Your Own Multi-Agent System

This guide walks you through setting up a multi-agent AI collaboration system on your own machine. You don't need the same hardware or the same agents — the patterns are what matter.

## Prerequisites

**Hardware**: Any machine with 16GB+ RAM. The original system runs on a Mac Studio Pro (M2 Max, 64GB), but the coordination patterns work on much less. Agents are mostly API-bound (calling cloud LLMs), not compute-bound locally.

**Agents**: You need at least two AI coding agents. The specific combination doesn't matter as long as they can read and write files. Options:

| Agent | Type | Access |
|-------|------|--------|
| Claude Code | CLI | `npm install -g @anthropic-ai/claude-code` (Anthropic API key required) |
| Codex | Desktop | OpenAI Codex app |
| Cursor | IDE | Cursor Pro subscription |
| Aider | CLI | Open-source, works with multiple LLM providers |
| Continue | IDE extension | Open-source VS Code extension |

The system described in this repo uses Claude Code + Codex + Cursor, but the coordination layer is agent-agnostic. Any agent that can read and write files can participate.

## Step 1: Create the coordination files

### agent-protocol.md

Create this file in your home directory (or project root). This is the central coordination hub.

```markdown
# Agent Coordination Protocol

## Active Agents

| Agent | Name | Status | Focus |
|-------|------|--------|-------|
| (fill in as agents start) |

## Rules of Engagement

1. Read this file before editing shared resources.
2. One owner per file. Owner edits freely. Others propose changes below.
3. No parallel edits. If two agents need the same file, coordinate first.
4. Handoff format: append to Handoffs with agent name, timestamp,
   what changed, what's next, blockers.
5. NEVER commit secrets. No API keys, tokens, passwords, .env files.

## File Ownership Ledger

| File/Dir | Owner | Purpose |
|----------|-------|---------|
| (fill in as files are created) |

## Proposals

(Agents propose changes to files they don't own here)

## Handoffs

(Agents append handoffs here when they finish work)
```

The critical sections are the **ownership ledger** and **handoffs**. Everything else is supporting structure.

### env.yaml

Create this file in your project directory. Map your machine's paths:

```yaml
machine:
  name: "your-machine"
  os: "macOS / Linux / Windows"

runtimes:
  python: "3.11"
  node: "v22.0.0"

paths:
  home: "/Users/you"
  projects: "/Users/you/projects"
  # Add paths your agents will need

services:
  # List any local services agents should know about
  api:
    port: 3001
    health: "http://localhost:3001/health"
```

Agents read this file instead of hardcoding paths. When you move to a different machine, only this file changes.

## Step 2: Establish ownership rules

Before any agent starts working, decide who owns what. This is the most important step.

**The rule**: every file or directory has exactly one owner. The owner edits freely. Non-owners propose changes through `agent-protocol.md`.

Example ownership split:

```
src/api/routes/    → Agent A (implements endpoints)
src/api/tests/     → Agent B (writes tests)
src/api/docs/      → Agent B (maintains documentation)
src/api/server.js  → Agent A (server entrypoint)
```

Write this into the File Ownership Ledger in your protocol file before any work begins.

**Why this matters**: without explicit ownership, two agents will eventually edit the same file simultaneously. One agent's changes will be lost, or worse, the file will end up in an inconsistent state with half of each agent's work.

## Step 3: Configure agent onboarding

Create a `CLAUDE.md` (or equivalent) that tells each new agent instance how to start:

```markdown
# Agent Onboarding

## On startup

1. Read env.yaml to learn where things live
2. Read agent-protocol.md to see who's active and who owns what
3. Check services: curl -s localhost:3001/health
4. Read recent handoffs to understand current state

## Rules

- One owner per file — check the ledger before editing
- Append handoffs when you finish work
- Never commit secrets
```

If your agents support auto-loaded instructions (Claude Code reads `CLAUDE.md`, Codex reads `.codex/instructions.md`, Cursor reads `.cursor/rules/`), put the onboarding there so it's automatic.

## Step 4: Set up session logging

Session logs are how context persists across agent lifetimes. Create a directory for logs:

```bash
mkdir -p ~/.agent-logs/
```

Each agent session should write a log with this format:

```markdown
# Session: [descriptor]
**Date**: YYYY-MM-DD
**Agent**: [name]
**Focus**: [what this session is about]

## What happened
- (timeline of actions)

## Files changed
- (list of files created/modified/deleted)

## What's next
1. (specific next steps)

## Blockers
- (anything stuck, or "None")
```

The naming convention matters: `YYYY-MM-DD-agent-name-focus.md`. This makes logs sortable by date and searchable by agent name.

**Critical**: logs should be written continuously, not just at shutdown. If your agent runs out of context or crashes, the log should already be on disk.

## Step 5: Define GGO mode (optional but recommended)

GGO (Go Go Go) means agents execute without asking "shall I proceed?" at every step. This is a significant productivity multiplier — a 30-minute task with confirmation prompts takes 5 minutes without them.

To enable this safely:

1. **Use version control.** Every change is a commit that can be reverted.
2. **Define a deny list.** Agents can do anything except: force push, `rm -rf`, drop tables, modify auth files.
3. **Require pause for ambiguity.** If the agent genuinely doesn't know what to build, it should ask. But "shall I write this file?" is not a question worth asking.

Configure your agents for GGO mode through their settings:
- Claude Code: `~/.claude/settings.json` with `allowedTools: ["Bash(*)"]`
- Cursor: `.cursor/rules/` with "execute without confirmation" rules
- Codex: Prefix approvals for common commands (`npm test`, `npm run dev`)

## Step 6: Start with two agents

Don't start with four agents. Start with two:

1. **One primary agent** (Claude Code, Aider, or your preferred CLI agent) — handles architecture, implementation, and coordination
2. **One supporting agent** (Codex, Cursor, or equivalent) — handles testing, documentation, or visual editing

Have both agents read the protocol file on startup. Have the primary agent do the first chunk of work and write a handoff. Have the supporting agent read the handoff and do its part.

Once this loop works smoothly, consider adding a third agent for a genuinely independent workstream.

## Step 7: Build the feedback loop

After a few sessions, review your handoffs and session logs:

- Are agents duplicating work? Tighten ownership boundaries.
- Are handoffs missing critical context? Standardize the handoff format.
- Is the protocol file getting too long? Archive old handoffs into a separate file.
- Are agents hitting the same problems? Document solutions in the onboarding file.

The coordination system is itself a product that improves over iteration. Treat it like code — refactor when patterns emerge, fix bugs when coordination fails.

## Common patterns

### The ownership split for web apps

```
Frontend components → Agent A
API routes          → Agent B
Tests               → Agent C (or B)
Database migrations → Agent B
Documentation       → Agent C
CI/CD config        → Whoever set it up
```

### The handoff for long-running tasks

When a task spans multiple agent sessions:

1. First agent starts the work, writes partial results to disk, logs progress
2. First agent writes handoff: "Phase 1 complete. Files at X. Next: Phase 2 (specific steps)."
3. Second agent reads handoff, reads the partial results, continues Phase 2
4. Repeat until done

The key is that partial results are always on disk. The handoff tells the next agent where to find them and what to do next.

### The escalation pattern

When an agent hits something it can't do:
1. Agent writes what it tried, what failed, and what it thinks the fix is
2. Agent notes this as a blocker in its handoff
3. Human resolves the blocker (OAuth login, DNS config, hardware action)
4. Next agent reads the resolution and continues

Common human-required actions: OAuth flows, DNS changes, hardware connections, app store installs, credential creation.

## Checklist

- [ ] `agent-protocol.md` created with ownership ledger and handoff section
- [ ] `env.yaml` created with machine-specific paths
- [ ] Agent onboarding file created (CLAUDE.md or equivalent)
- [ ] Session log directory created
- [ ] Ownership rules decided for your project files
- [ ] First agent started, read the protocol, did work, wrote handoff
- [ ] Second agent started, read the handoff, continued the work
- [ ] Version control initialized (git) with .gitignore blocking secrets
