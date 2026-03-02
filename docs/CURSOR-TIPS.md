# Cursor tips — Make it work better + agents and work

**Practical tips for Cursor and how to add agents or other work.** GGO.

---

## 1. Make Cursor work better

### Rules and context
- **.cursor/rules** — You have `~/.cursor/rules/ecosystem.mdc` (multi-agent, ownership, GGO). Add **project-specific** rules in each repo: e.g. `Mac-Studio-Local-AI-Collab/.cursor/rules/repo.mdc` with "This repo: docs only; no secrets; link to agent-protocol and WHERE-I-AM-NEEDED." Cursor loads rules from workspace root.
- **AGENTS.md per repo** — In repos you work in often (api, life-dashboard, memoryatlas, collab), add a short `AGENTS.md`: "Who owns what, where TASKS live, how to hand off to Claude Code." Cursor (and Claude) can read it when the folder is open.
- **Open the right folder** — For collab/visibility work, open `Mac-Studio-Local-AI-Collab`. For API work, open `api`. Smaller workspace = better context and tab completion.

### Composer and model
- **Use Auto, not always Max** — Auto picks cheaper models for simple edits; Max burns credits. Reserve Max for hard refactors or when you explicitly need it.
- **Scope the ask** — "Add a route for X" or "Fix the type in this file" works better than "refactor the whole API." Cursor context truncates; hand big refactors to Claude Code.
- **Review before accept** — Use the diff view. Reject noisy or wrong edits and re-prompt with a tighter instruction.

### MCP and skills
- **Hugging Face** — You have the plugin; use it for Hub, datasets, Gradio when the task fits. Auth once via `mcp_auth` for the HF MCP server.
- **Playwright, Beeper, Notion, Linear** — Use when the task is "browser check," "send message," "create page/ticket." Don't use Cursor for long orchestration; use it for the one-off tool call.
- **Skills** — Invoke with `/` (e.g. create-rule, create-skill). Use create-rule when you want a new Cursor rule; use create-skill when you want a reusable skill doc.

### Performance and cost
- **Tab completions are free (Pro)** — Rely on them. They're the best ROI.
- **Close unused workspaces** — Fewer open folders = less context and faster Composer.
- **Privacy mode** — If you're in sensitive code (SDI, personal data), enable Settings → Privacy so code isn't used for training.

---

## 2. Adding agents or other work

### More Cursor instances
- **One Cursor per "focus"** — e.g. one window for collab repo (docs, visibility), one for api or life-dashboard. Each has its own context; avoid one window with 10 roots.
- **Don't duplicate Claude Code** — Cursor isn't an orchestrator. For "run bootstrap," "update TASKS," "coordinate repos," use Claude Code. Cursor does edits and MCP tasks.

### More Claude Code instances
- **Spawn for a thread** — e.g. "Anvil workload," "repo cleanup," "SDI draft." Give each a clear mission and a name; they read agent-protocol and TASKS and append handoffs. See TASKS "POST-REBOOT" and "ANVIL WORKLOAD QUEUE" for what to assign.
- **One orchestrator + workers** — One Claude Code as "daily driver" (reads WHERE-I-AM-NEEDED, runs top of TASKS); others for dedicated threads (MemoryAtlas, Zhirinovsky, repo stewards). Coordinate via agent-protocol; avoid two agents editing the same file at once.

### Codex
- **Tests and contracts** — Codex owns api/test, openapi.json, runbooks. When you add routes or change behavior, have Codex (or Claude Code) update tests and contract. Don't use Cursor for broad test scaffolding.

### Dashboard and visibility
- **/docs, /briefing, /fleet** — Already in TASKS for Claude Code. Cursor can own UI copy or small front-end tweaks if proposed in agent-protocol. Big dashboard features stay with Claude Code (life-dashboard owner).

---

## 3. Repo-dedicated work (cleanup, commits)

- **See REPO-STEWARDS.md** in this repo — List of actively worked-on repos and a "steward" workflow so Claude Code (or a dedicated instance) can run cleanup and commits without stepping on you.
- **Principle:** One agent "owns" repo hygiene for a run: run `node ~/.claude/repo-scanner.js` (or git status per repo), commit with clear messages, push, append handoff. You review PRs or `git log` when you have time.

---

## 4. Quick reference

| I want to… | Use |
|------------|-----|
| Quick edit in a file | Cursor Cmd+K or tab complete |
| Multi-file refactor, orchestration, memory | Claude Code |
| Tests, OpenAPI, runbooks | Codex |
| Docs in collab repo, visibility, rules | Cursor |
| Run something on Anvil | Claude Code (LiteLLM routes; or deploy script) |
| Browser or MCP one-off | Cursor (Playwright, Beeper, Notion, etc.) |
| Repo cleanup and commits | Claude Code + REPO-STEWARDS workflow |

---

Timestamp: 2026-03-01  
AI: Cursor  
Artifact Path: Mac-Studio-Local-AI-Collab/docs/CURSOR-TIPS.md
