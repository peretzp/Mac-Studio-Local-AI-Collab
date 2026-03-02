# Phase-lock and visibility — Best practices for agent convergence

**Goal:** All agents and Peretz share the same picture of state. You know where you're needed; agents know what's done and what's next. We iterate towards that.

---

## What phase-lock means

- **Single picture of truth** — No "which thread was that in?" or "did the other agent already do this?" One place (or a minimal set) to read: WHAT'S DONE, WHAT'S NEXT, WHAT NEEDS PERETZ.
- **Minimal handoff friction** — Agents append handoffs; they don't duplicate work. When you (or another agent) open the protocol, the last N handoffs tell the story.
- **Human only at decision points** — You're needed for: credentials, approval, physical action, or judgment calls. Everything else agents can do and document.

---

## Best practices we're following

### 1. Single sources of truth
- **agent-protocol.md** — Who did what, what's next, blockers. All agents read on startup; all append on handoff. No parallel edits; coordinate via Proposals if you need to edit someone else's file.
- **TASKS.md** — Prioritized list (P0–P4), owners, status. Claude Code owns updates; Cursor/Codex propose via protocol.
- **WHERE-I-AM-NEEDED.md** — Your morning read. Top 3 human actions, active threads, blockers. Agents update when they park or unblock something.

### 2. Visibility
- **Dashboard (:3000)** — Live services, links to key pages.
- **Prompt Browser (:3002)** — Session and prompt history.
- **Langfuse (:3005)** — Model usage, cost, traces (when API keys set).
- **WHERE-I-AM-NEEDED** — Aggregates "where is Peretz needed" and "what are agents doing" so you don't open five files.

### 3. Synthesis across threads
- **One handoff per chunk of work** — Agent does a thing → appends one handoff with: what changed, what's next, blockers. Next agent reads last handoffs and continues.
- **Convergence in WHERE-I-AM-NEEDED** — When an agent finishes a thread or hits a blocker that needs Peretz, they update the "Active threads" and "Blockers" sections. So all threads converge into one view.

### 4. Project management of agents
- **Ownership** — One owner per file (see agent-protocol File Ownership Ledger). Others propose.
- **No parallel edits** — If two agents need the same file, one proposes; the other executes or defers.
- **GGO** — Execute without asking "shall I proceed?" Pause only for auth, destructive action, or genuine ambiguity. Speed and transparency.

---

## How to keep iterating

- **Agents:** After meaningful work, (1) append handoff to agent-protocol, (2) update TASKS.md if you changed task status, (3) update WHERE-I-AM-NEEDED if you started/finished a thread or created a blocker for Peretz.
- **Peretz:** When you sit down, read WHERE-I-AM-NEEDED first. When you unblock something, say so in a handoff or in WHERE-I-AM-NEEDED so the next agent sees it.
- **Everyone:** Prefer adding to existing docs (merge/append) over creating new ones. Fewer places to look = closer to phase-lock.

---

## Research note: what others do

- **Single "status" or "runbook" file** — Many teams use one markdown file that gets updated by humans and bots; we're doing that with agent-protocol + WHERE-I-AM-NEEDED.
- **Async handoffs** — Agents don't need to be online together; the filesystem is the bus. We're already there.
- **Human-in-the-loop visibility** — Dashboards and "what needs me" views reduce context-switching. WHERE-I-AM-NEEDED is our first dedicated "need human" view; we'll refine it.

---

Timestamp: 2026-02-28  
AI: Cursor  
Artifact Path: Mac-Studio-Local-AI-Collab/docs/PHASE-LOCK-AND-VISIBILITY.md
