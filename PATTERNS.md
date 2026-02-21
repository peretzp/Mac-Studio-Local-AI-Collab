# Multi-Agent Coordination Patterns

## Overview

This document captures proven patterns for coordinating multiple AI agents. These patterns emerged from real use and solve real coordination problems.

## Core Principles

### 1. The Machine Is the Context

**Problem**: AI conversations are ephemeral. Context resets between sessions.

**Solution**: Externalize all state to the filesystem.

```
Conversation Memory → MEMORY.md, session logs
Agent Status → agent-protocol.md
Configuration → env.yaml, CLAUDE.md
Decisions → Commit messages, handoffs
```

**Why it works**: Future agents read files to resume work. No context loss.

### 2. One Owner Per File

**Problem**: Two agents editing the same file → merge conflicts, lost work.

**Solution**: Ownership ledger in `agent-protocol.md`:

```markdown
| File/Dir | Owner | Purpose |
|----------|-------|---------|
| api/routes/ | Claude | Route handlers |
| api/test/ | Codex | Tests |
| README.md | Cursor | Documentation |
```

**Rules**:
- Owner edits freely
- Others propose changes in agent-protocol.md
- Ownership can transfer (document in handoff)

### 3. Handoffs Over Hope

**Problem**: Agent A finishes work. Agent B has no idea what happened.

**Solution**: Mandatory handoff protocol:

```markdown
### Agent Name — Timestamp — What You Did

- Agent: Name (Tool, Model)
- What changed: Specific files and changes
- What's next: Clear next steps for next agent
- Blockers: Anything preventing progress
- Files modified: Complete list
```

**Why it works**: Next agent reads handoffs, gets full context immediately.

### 4. GGO (Go Go Go) Mode

**Problem**: Too much "shall I proceed?" wastes time and attention.

**Solution**: Agents execute without asking. Only pause for:
- Auth/credentials
- Destructive operations (rm -rf, force push, drop tables)
- Genuinely unclear requirements

**Why it works**: Speed. Trust. Agents are tools, not waiters.

## Coordination Patterns

### Pattern: Parallel Tracks

**Use when**: Two independent tasks can run simultaneously

**Setup**:
```markdown
## Active Work

| Agent | Track | Status |
|-------|-------|--------|
| Claude-1 | Feature A (api/feature-a/) | Active |
| Claude-2 | Feature B (api/feature-b/) | Active |
```

**Rules**:
- No overlapping file ownership
- Separate branches (optional)
- Independent handoffs

**Example**:
- Claude-1 builds authentication system
- Claude-2 builds email service
- No coordination needed until integration

### Pattern: Sequential Pipeline

**Use when**: Steps must happen in order (build → test → deploy)

**Setup**:
```markdown
## Pipeline: New API Endpoint

1. [DONE] Claude Code → Implement handler
2. [ACTIVE] Codex → Write tests
3. [PENDING] Cursor → Document in OpenAPI
4. [PENDING] Claude Code → Deploy
```

**Rules**:
- Each agent blocks the next until done
- Handoff includes "ready for next step" signal
- Tests MUST pass before handoff

### Pattern: Orchestrator + Workers

**Use when**: One complex task needs coordination

**Setup**:
- **Orchestrator** (Claude Code): Plans, coordinates, makes decisions
- **Workers** (Cursor, Codex): Execute specific subtasks

**Flow**:
```
1. Orchestrator creates plan in agent-protocol.md
2. Orchestrator assigns subtasks to workers
3. Workers execute and report back in handoffs
4. Orchestrator integrates results
```

**Example**:
- Orchestrator: "Refactor authentication system"
- Worker 1 (Cursor): Update UI components
- Worker 2 (Codex): Update tests
- Orchestrator: Review, integrate, commit

### Pattern: Ask Then Act

**Use when**: Ambiguity exists, multiple valid approaches

**Anti-pattern**: Agent guesses, implements, gets it wrong

**Correct flow**:
```markdown
## Proposals

### Claude-7 — 2026-02-08 — Auth Strategy Decision

We need to add authentication. Two approaches:

**Option A: JWT**
- Pros: Stateless, scalable
- Cons: Harder to revoke

**Option B: Sessions**
- Pros: Easy revoke, simpler
- Cons: Requires session storage

**Recommendation**: Option A (JWT) for this API

Peretz, which approach do you prefer?
```

**Why it works**: Agent pauses for genuine ambiguity, not routine execution.

### Pattern: Fork-Merge

**Use when**: Experimenting with approach, need to compare options

**Setup**:
```bash
# Agent A
git checkout -b experiment/approach-a
# Implement approach A
# Document results in handoff

# Agent B
git checkout -b experiment/approach-b
# Implement approach B
# Document results in handoff

# Orchestrator reviews both, picks winner, merges
```

**Why it works**: Parallel exploration, data-driven decision

## Communication Patterns

### Pattern: Status Broadcast

**Use when**: Multiple agents active, need visibility

**Format**:
```markdown
## Status Board

| Agent | Status | Current Task | ETA |
|-------|--------|--------------|-----|
| Claude-1 | Active | API refactor | 2h |
| Codex | Blocked | Tests (waiting for API) | - |
| Cursor | Active | Documentation | 30m |
```

Updated by each agent after major progress.

### Pattern: Blockers First

**Use when**: Can't proceed without input

**Format**:
```markdown
### Claude-7 — 2026-02-08 — Blocked on DNS Config

**Status**: BLOCKED
**Blocker**: Need DNS registrar credentials to configure peretzpartensky.com
**Workaround**: None - hard dependency
**Next agent**: Do not attempt Vercel deploy until blocker resolved
```

Clearly signals "don't waste time on this track."

### Pattern: Decision Log

**Use when**: Important architectural decision made

**Format**:
```markdown
## Decisions

### 2026-02-08: Use SQLite for Atlas DB

**Decision**: MemoryAtlas will use SQLite, not PostgreSQL

**Rationale**:
- Single-user app, no concurrency needs
- File-based → easier backup
- Zero setup overhead

**Alternatives considered**: PostgreSQL (overkill), JSON files (slow queries)

**Committed by**: Claude-3 (The Cartographer)
```

Future agents understand "why" not just "what."

## Anti-Patterns

### Anti-Pattern: Duplicate Work

**Symptom**: Two agents implement the same feature

**Cause**: No coordination, unclear ownership

**Fix**:
- Check agent-protocol.md before starting work
- Claim work explicitly: "Claude-7 starting API auth"
- Use Status Board pattern

### Anti-Pattern: Lost Context

**Symptom**: Agent forgets what it was doing after restart

**Cause**: No handoff written before shutdown

**Fix**:
- ALWAYS write handoff before closing agent
- Include "Current Task" and "Next Steps"
- Reference relevant files and commits

### Anti-Pattern: Ownership Creep

**Symptom**: Agent edits files it doesn't own → conflicts

**Cause**: Didn't check ownership ledger

**Fix**:
- Read agent-protocol.md before editing
- Propose changes if not owner
- Transfer ownership explicitly if needed

### Anti-Pattern: Ask-Too-Much

**Symptom**: Agent asks permission for routine operations

**Cause**: Not operating in GGO mode

**Fix**:
- Default to execute, not ask
- Only pause for: auth, destructive ops, ambiguity
- Trust the agent, review after

### Anti-Pattern: Silent Failure

**Symptom**: Agent encounters blocker, says nothing

**Cause**: No handoff documenting the issue

**Fix**:
- Document blockers explicitly
- Mark status as BLOCKED
- Propose workaround or next steps

## Scaling Patterns

### 2 Agents (Most Common)

**Typical split**:
- **Primary** (Claude Code): Complex tasks, orchestration
- **Secondary** (Cursor): Visual editing, quick fixes

**Minimal coordination**:
- Light file ownership (just major modules)
- Handoffs when switching major tasks
- GGO mode for both

### 3 Agents (Power User)

**Typical split**:
- **Orchestrator** (Claude Code): Planning, architecture
- **Builder** (Cursor or Claude Code): Implementation
- **Verifier** (Codex): Tests, contracts, validation

**Moderate coordination**:
- Clear ownership ledger
- Sequential pipeline for features
- Status board for visibility

### 4+ Agents (Advanced)

**Only when**: Truly parallel workstreams exist

**Coordination overhead**:
- Detailed ownership ledger
- Active status board (updated frequently)
- Explicit handoff protocol with approvals
- Orchestrator role to manage chaos

**Warning**: Diminishing returns. Coordination cost grows faster than throughput.

## Template: Starting a New Task

```markdown
## Task: [Name of Task]

**Assigned to**: [Agent Name]
**Started**: [Timestamp]
**Expected duration**: [Estimate]

### Context
- Why we're doing this
- What success looks like
- Related tasks/dependencies

### Files in scope
- file1.js (edit)
- file2.js (create)
- file3.test.js (update)

### Progress
- [x] Step 1: Thing done
- [ ] Step 2: Current step
- [ ] Step 3: Next step

### Handoff
[Agent will fill this in when done]
```

## Template: Handoff Entry

```markdown
### [Agent Name] — [YYYY-MM-DDTHH:MM:SS-TZ] — [Brief Description]

- **Agent**: [Name] ([Tool], [Model])
- **What changed**:
  - [Specific change 1]
  - [Specific change 2]
  - [Specific change 3]
- **What's next**:
  - [Clear next step 1]
  - [Clear next step 2]
- **Blockers**: [List blockers or "None"]
- **Files created**: [List]
- **Files modified**: [List]
- **Files deleted**: [List]
- **Tests status**: [Pass/Fail/Skipped]
- **Commits**: [Git SHA or "Uncommitted"]

Footer:
- Timestamp: [ISO-8601]
- Signed By: [Your Name]
- AI: [Agent Name]
- Artifact Path: [File path]
```

---

These patterns are living documentation. As new coordination challenges emerge, add new patterns here.

---
Timestamp: 2026-02-08T23:00:00-0800
Location: Mac-Studio-Local-AI-Collab repository
Signed By: Peretz Partensky
AI: Telos (gpt-5.2-codex via Cursor)
Chat/Project: Meta-documentation - Coordination Patterns
Artifact Path: /Users/peretz_1/.cursor/worktrees/Mac-Studio-Local-AI-Collab/ygu/PATTERNS.md
