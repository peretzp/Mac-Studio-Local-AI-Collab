# Getting Started

How to set up multi-agent AI collaboration on your own machine.

## Prerequisites

You need at least two of the following:

- **Claude Code** — [claude.ai/code](https://claude.ai/code) (CLI, requires Claude Pro/Team)
- **Codex** — [codex.openai.com](https://codex.openai.com) (Desktop app, requires OpenAI API access)
- **Cursor** — [cursor.com](https://cursor.com) (IDE, free tier works but Pro recommended)

You also need:
- A Unix-like machine (macOS or Linux)
- Git
- Node.js (for the example services)
- A terminal you can run multiple tabs in

## Step 1: Fork and configure

Fork this repo, then clone it:

```bash
git clone git@github.com:YOUR_USERNAME/Mac-Studio-Local-AI-Collab.git
cd Mac-Studio-Local-AI-Collab
```

Create your local environment config:

```bash
cp env.yaml env.local.yaml
```

Edit `env.local.yaml` with your machine's paths. Every path in the file needs to point to real locations on your system. Remove sections that don't apply (e.g., if you don't have an Obsidian vault, remove the vault path).

## Step 2: Create the coordination file

Create `agent-protocol.md` in your home directory:

```bash
cat > ~/agent-protocol.md << 'EOF'
# Agent Coordination Protocol

> Shared state file for all AI agents operating in ~/

---

## Active Agents

| Agent | Name | Model | Interface | Status | Focus |
|-------|------|-------|-----------|--------|-------|

## Rules of Engagement

1. **Read this file before editing shared resources.**
2. **One owner per file.** Owner edits freely. Others propose changes below.
3. **No parallel edits.** If two agents need the same file, coordinate first.
4. **Handoff format**: Agent name, timestamp, what changed, what's next, blockers.
5. **Never commit secrets.**

## File Ownership Ledger

| File/Dir | Owner | Purpose | Notes |
|----------|-------|---------|-------|
| ~/agent-protocol.md | Shared | This file | All agents read/append |

## Proposals

<!-- Agents: append proposals here -->

## Handoffs

<!-- Agents: append handoffs here -->
EOF
```

## Step 3: Configure your agents

### Claude Code

Add these instructions to Claude Code's onboarding (via `CLAUDE.md` in your project root or `~/.claude/` config):

```markdown
On startup:
1. Read env.yaml (or env.local.yaml if it exists) to learn where things live
2. Read ~/agent-protocol.md to see active agents and file ownership
3. Check for recent handoffs before starting work
4. Append your status to the Active Agents table

Before editing any file:
- Check the ownership ledger in agent-protocol.md
- If you don't own the file, append a proposal instead of editing

When finishing work:
- Append a handoff to agent-protocol.md with: what changed, what's next, blockers
```

### Codex

Tell Codex to read the coordination files on startup. If using Codex Desktop, you can paste this as a system instruction or include it in a project-level config:

```
Before starting work, read these files:
1. env.yaml — machine configuration
2. ~/agent-protocol.md — coordination protocol

Follow the file ownership rules. Append handoffs when done.
```

### Cursor

Create `.cursor/rules/ecosystem.mdc` in your project:

```markdown
# Multi-Agent Ecosystem

You are one of multiple AI agents operating on this machine.

Before editing shared files, check ~/agent-protocol.md for ownership.
Read env.yaml to know where things live.

Key rules:
- One owner per file
- Check ownership before editing
- Append handoffs to agent-protocol.md when done
```

## Step 4: Assign file ownership

Before agents start working, populate the ownership ledger. Be explicit:

```markdown
| File/Dir | Owner | Purpose |
|----------|-------|---------|
| ~/project/src/ | Claude Code | Core implementation |
| ~/project/tests/ | Codex | Test suite |
| ~/project/README.md | Cursor | Documentation |
| ~/agent-protocol.md | Shared | Coordination |
```

The ownership boundaries don't have to be perfect on day one. Adjust as you learn where conflicts happen.

## Step 5: Run your first multi-agent session

1. **Start Agent A** (e.g., Claude Code). Give it a concrete task: "Build the API endpoints for user management." Watch it read the protocol, check ownership, do its work, and write a handoff.

2. **Start Agent B** (e.g., Codex). Give it a complementary task: "Write tests for the user management API." It should read the handoff from Agent A and know exactly what to test.

3. **Review the handoffs.** Open `~/agent-protocol.md` and read what both agents logged. This is your coordination record.

## Tips for effective multi-agent work

**Start with two agents, not three.** The coordination overhead with three agents is real. Get comfortable with two first.

**Give agents independent tasks.** "Claude builds the API, Codex writes the tests" works great. "Claude and Codex both refactor the auth system" will cause conflicts.

**Name your instances.** Tell Claude Code to pick a name. It makes the handoff log dramatically more readable.

**Use GGO mode.** Tell agents to execute without asking permission for routine actions. The confirmation dialog is the enemy of throughput.

**Read the handoffs yourself.** The protocol file is your audit trail. Skim it between tasks to stay oriented.

**Keep the ownership ledger current.** When you create new files, immediately assign ownership. Unowned files are conflict magnets.

## Adapting the env.yaml

The `env.yaml` in this repo reflects one specific machine. Here's what to change for yours:

| Section | What to edit |
|---------|-------------|
| `machine` | Your hostname, user, OS, RAM |
| `paths` | Every absolute path — these are machine-specific |
| `services` | Ports and dirs for your services (or remove if you don't run them) |
| `repos` | Your GitHub remotes and local clone paths |
| `mcp` | MCP server configs for your agents (or remove unused ones) |
| `github` | Your GitHub username and SSH key type |
| `ollama` | Your local models (or remove if you don't use Ollama) |

## What you don't need

You don't need all the services, repos, or MCP servers listed in `env.yaml`. The core pattern only requires:

1. `env.yaml` — machine config that agents read
2. `agent-protocol.md` — shared coordination state
3. File ownership rules — one owner per file
4. Handoff discipline — log what changed, what's next, blockers

Everything else is optional infrastructure you can add as your workflow demands it.
