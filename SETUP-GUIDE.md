# Setup Guide: Multi-Agent AI Collaboration

## Overview

This guide helps you replicate the multi-agent AI collaboration system on your own machine. The system coordinates Claude Code, Codex, and Cursor agents through shared protocols and file ownership boundaries.

## Prerequisites

- **Hardware**: Any modern Mac (M-series recommended for performance)
- **Software**:
  - Claude Code CLI (`cc` command)
  - Codex Desktop app or CLI
  - Cursor IDE
  - Node.js (v20+)
  - Python 3.11+
  - Git with SSH keys configured

## Quick Start (30 minutes)

### 1. Fork the Repository

```bash
# Clone to your preferred location
git clone git@github.com:YOUR_USERNAME/Mac-Studio-Local-AI-Collab.git
cd Mac-Studio-Local-AI-Collab
```

### 2. Configure Your Environment

```bash
# Copy env.yaml to create your local overrides
cp env.yaml env.local.yaml

# Edit env.local.yaml with your machine's paths
# Key changes:
#   - paths.home: /Users/YOUR_USERNAME
#   - paths.vault: your Obsidian vault path (if you use Obsidian)
#   - machine.name, machine.user, machine.ip_local
```

### 3. Create Coordination Files

The symlinks in this repo point to canonical files at `~/`. Create these:

```bash
# Create agent protocol file
cat > ~/agent-protocol.md << 'EOF'
# Agent Coordination Protocol

## Active Agents

| Agent | Name | Model | Interface | Status | Focus |
|-------|------|-------|-----------|--------|-------|
| Codex | Codex | gpt-5.3-codex | Codex CLI | Active | Coordination |
| Cursor | Your Name | gpt-5.2-codex | Cursor IDE | Active | Visual editing |

## Handoffs

### Your Name (Cursor) — $(date +%Y-%m-%dT%H:%M:%S%z) — Initial Setup

- Agent: Your Name (Cursor)
- What changed: Initial setup of multi-agent coordination system
- What's next: Start building!
EOF

# Create AI operating standard
cat > ~/AI-OPERATING-STANDARD.md << 'EOF'
# AI Operating Standard

## Mode
- GGO (Go Go Go) is the default: execute without asking "shall I proceed?"
- Pause ONLY for: auth, destructive actions, unclear requirements

## Secrets Policy
- NEVER commit secrets to git
- Use .env files (gitignored) for API keys
- Use environment variables in code

## Cross-Agent Coordination
- One owner per file (check agent-protocol.md)
- Document handoffs when switching work between agents
EOF

# Create Claude Code guidance (if you use Claude Code)
cat > ~/CLAUDE.md << 'EOF'
# CLAUDE.md

## On Startup
1. Read ~/AI-OPERATING-STANDARD.md
2. Read ~/agent-protocol.md
3. Check for recent handoffs
4. Pick a unique name (e.g., "The Builder", "The Architect")
5. Register yourself in agent-protocol.md

## On Shutdown
1. Update agent-protocol.md with handoff entry
2. Document what you did, what's next, any blockers
EOF
```

### 4. Configure Your Agents

#### Claude Code

```bash
# If you have session logs enabled
mkdir -p ~/.claude/session-logs

# Configure memory location (optional)
mkdir -p ~/.claude/projects/-Users-$(whoami)/memory
echo "# MEMORY.md" > ~/.claude/projects/-Users-$(whoami)/memory/MEMORY.md
```

#### Cursor

```bash
# Create rules directory
mkdir -p ~/.cursor/rules

# Copy the ecosystem rule from this repo
cp ~/.cursor/rules/ecosystem.mdc ~/path/to/your/rules/
# Or create a minimal version pointing to your coordination files
```

#### Codex

```bash
# Create config directory
mkdir -p ~/.codex/session-logs

# Configure MCP servers as needed in Codex UI
```

### 5. Test the Coordination

Open three terminals/apps:

**Terminal 1 - Claude Code:**
```bash
cc
# In Claude: "Read ~/agent-protocol.md and introduce yourself"
```

**Terminal 2 - Codex:**
```
Open Codex app
# Ask: "Read ~/agent-protocol.md and say hello"
```

**Terminal 3 - Cursor:**
```bash
cursor .
# Use Cmd+L to open chat
# Ask: "Read ~/agent-protocol.md and register yourself"
```

Each agent should read the protocol, see the other agents, and be able to coordinate through the shared file.

## Core Patterns

### Pattern 1: File Ownership

Before any agent edits a file, check `~/agent-protocol.md` for ownership:

```markdown
## File Ownership Ledger

| File/Dir | Owner | Purpose |
|----------|-------|---------|
| ~/api/ | Claude Code | Backend API |
| ~/api/test/ | Codex | Tests |
```

### Pattern 2: Handoffs

When you finish work or switch agents, add a handoff entry:

```markdown
### Agent Name — YYYY-MM-DDTHH:MM:SS — What You Did

- Agent: Name (Tool)
- What changed: Bullet list of changes
- What's next: What the next agent should do
- Blockers: Any issues preventing progress
- Files modified: List of files
```

### Pattern 3: GGO Mode

Agents execute without asking permission. Only pause for:
- Authentication/credentials
- Destructive operations (rm -rf, force push)
- Genuinely ambiguous requirements

### Pattern 4: Secrets Management

```bash
# Create .env file (gitignored)
echo "OPENAI_API_KEY=sk-..." > ~/.env
chmod 600 ~/.env

# In code, reference via environment
process.env.OPENAI_API_KEY  # Node.js
os.environ["OPENAI_API_KEY"]  # Python
```

## Common Workflows

### Workflow: Multi-File Refactor

1. **Claude Code** (orchestrator):
   - Plans the refactor
   - Updates multiple files
   - Documents changes in agent-protocol.md

2. **Codex** (verifier):
   - Writes tests for the refactored code
   - Runs test suite
   - Reports results

3. **Cursor** (polish):
   - Fixes any linter errors
   - Improves comments/docs
   - Visual review of diffs

### Workflow: New Feature

1. **Cursor** (ideation):
   - Quick prototype in visual editor
   - Cmd+K inline edits for fast iteration

2. **Claude Code** (implementation):
   - Takes prototype and implements full feature
   - Handles multi-file architecture
   - Integrates with existing codebase

3. **Codex** (contracts):
   - Creates API specs (OpenAPI)
   - Writes integration tests
   - Documents usage

## Troubleshooting

### Agents Can't See Each Other's Work

**Symptom**: Agent B doesn't know what Agent A did

**Solution**: 
- Ensure handoffs are being written to `~/agent-protocol.md`
- Check that the file is shared (not copies in different locations)
- Use symlinks to ensure single source of truth

### File Conflicts

**Symptom**: Two agents edit the same file simultaneously

**Solution**:
- Establish file ownership in agent-protocol.md
- Use the "Proposals" section for cross-ownership changes
- Coordinate timing through handoffs

### Lost Context

**Symptom**: Agent forgets what it was doing after restart

**Solution**:
- Claude Code: Enable session logs (`~/.claude/session-logs/`)
- Codex: Use session logs feature
- Always write handoffs before closing agents
- Reference session logs in handoffs

### Secrets Leaked to Git

**Symptom**: API keys committed to repository

**Solution**:
```bash
# Rotate the compromised key immediately

# Remove from git history
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch path/to/secret" \
  --prune-empty --tag-name-filter cat -- --all

# Add to .gitignore
echo "path/to/secret" >> .gitignore

# Force push (only if safe)
git push origin --force --all
```

## Adapting for Your Use Case

### Minimal Setup (Single Agent)

If you only use one agent, you can skip most coordination:

1. Keep `env.yaml` for path management
2. Skip `agent-protocol.md` (no coordination needed)
3. Keep `AI-OPERATING-STANDARD.md` for your own reference

### Two-Agent Setup (Most Common)

Most people use two agents:

1. **Primary** (Claude Code or Codex): Orchestration, complex tasks
2. **Secondary** (Cursor): Visual editing, quick fixes

Coordination is simpler:
- Light file ownership boundaries
- Handoffs only when switching major tasks
- Shared .env for secrets

### Enterprise Setup

For teams or complex multi-project workflows:

1. Add more structure to `agent-protocol.md`:
   - Detailed ownership ledger
   - Approval workflow for cross-ownership changes
   - Test gates (tests must pass before handoff)

2. Add monitoring:
   - Service health checks
   - Automated testing after each agent's work
   - Git hooks to enforce handoff documentation

3. Add communication layer:
   - Slack/Discord bot posting agent activity
   - Dashboard showing active agents and their focus
   - Automated handoff notifications

## Next Steps

1. **Try a simple task**: Ask one agent to create a "Hello World" app and document it in agent-protocol.md
2. **Add a second agent**: Have it read the protocol and add a test
3. **Iterate**: Find what coordination patterns work for your workflow

## Resources

- [Main README](README.md) - Repository overview
- [env.yaml](env.yaml) - Example environment configuration
- [CLAUDE.md](CLAUDE.md) - Claude Code specific onboarding

---

**Questions or improvements?** Open an issue or PR at https://github.com/peretzp/Mac-Studio-Local-AI-Collab

---
Timestamp: 2026-02-08T22:45:00-0800
Location: Mac-Studio-Local-AI-Collab repository
Signed By: Peretz Partensky
AI: Telos (gpt-5.2-codex via Cursor)
Chat/Project: Meta-documentation - Setup Guide
Artifact Path: /Users/peretz_1/.cursor/worktrees/Mac-Studio-Local-AI-Collab/ygu/SETUP-GUIDE.md
