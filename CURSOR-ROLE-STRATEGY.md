# Cursor Role Strategy — The Integrator's Analysis

## TL;DR

Cursor is your **editor**, not your **orchestrator**. Use it for the 80% of coding that's visual — tab completions, inline edits, code navigation, visual diffs. Leave orchestration, multi-project reasoning, and session persistence to Claude Code. Leave test scaffolding and contracts to Codex.

**Monthly cost**: $20 (Pro) for unlimited tab completions + ~$20 agent usage. This is the cheapest AI token spend in your stack by a wide margin.

---

## The Three-Tool Stack: Who Does What

| Capability | Cursor | Claude Code | Codex |
|------------|--------|-------------|-------|
| **Tab completions** (autocomplete) | **Best** — unlimited, instant, $0 marginal | N/A | N/A |
| **Inline edits** (Cmd+K) | **Best** — visual, targeted, cheap | Good (Edit tool) | Good |
| **Multi-file agent refactors** | Decent (Composer) | **Best** — long context, persistent | Good |
| **Architectural reasoning** | Weak (context truncates) | **Best** — 200K real context | Good |
| **Session persistence / memory** | None | **Best** — MEMORY.md, session logs | Session logs |
| **Orchestration** | None | **Best** — agent-protocol, cross-project | Coordination role |
| **Visual code navigation** | **Best** — go-to-def, find refs, file tree | N/A (terminal) | N/A |
| **Visual diff review** | **Best** — see changes before applying | Terminal diffs | Terminal diffs |
| **Test scaffolding** | Decent | Good | **Best** (current role) |
| **Shell operations** | Terminal panel | **Best** — native shell | Sandboxed |
| **MCP tools** | Supported | Supported | Supported |
| **Cost per heavy session** | ~$2-5 (agent) + free completions | ~$20-50 (Opus) | Included in sub |

## Optimal Workflow

### 1. Start your coding day in Cursor
- Open the project folder
- Tab completions handle 80% of keystrokes — this is your single highest-ROI AI interaction
- Use Cmd+K for quick fixes: "add error handling", "rename this variable", "fix this type"
- Use the file tree + go-to-definition for navigation

### 2. Escalate to Claude Code for complex tasks
- Multi-step refactors across 5+ files
- Architectural decisions ("how should I structure auth?")
- Tasks that need memory/context from previous sessions
- Anything touching the vault, dashboards, or cross-project state
- Orchestration between agents

### 3. Use Codex for verification + contracts
- Test scaffolding (its current role)
- OpenAPI spec maintenance
- Runbooks and integration glue
- Fast multi-language support

### 4. Use Cursor Composer (Agent Mode) sparingly
- Good for: "create a new React component with these props" (bounded, visual)
- Bad for: "refactor the entire auth system" (loses context, no persistence)
- Composer burns through your $20 agent budget fast with premium models
- **Tip**: Use Auto mode (not Max) — it picks cheaper models for simple tasks

## Token Economics Breakdown

| Action | Tool | Cost |
|--------|------|------|
| 1000 tab completions | Cursor | $0 (unlimited on Pro) |
| 50 inline edits (Cmd+K) | Cursor | ~$2-3 |
| 10 Composer agent turns | Cursor (Auto) | ~$5-10 |
| 10 Composer agent turns | Cursor (Max/Opus) | ~$15-30 |
| 1 complex multi-file task | Claude Code (Opus) | ~$5-15 |
| 1 orchestration session | Claude Code (Opus) | ~$15-40 |
| Test scaffolding session | Codex | Included in sub |

**Monthly projection for Peretz's usage pattern:**
- Cursor Pro: $20/mo flat (tab completions are the core value)
- Claude Code: $100-200/mo (heavy orchestration use)
- Codex: Included in ChatGPT subscription
- **Total**: ~$140-220/mo for a 3-agent stack

**To save money:**
- Stay on Cursor Pro ($20), don't upgrade to Pro Plus ($70) — the extra $50 of agent usage isn't worth it if Claude Code handles your complex tasks
- Use Cursor Auto mode (not Max) for agent tasks — it picks the cheapest adequate model
- Avoid running Cursor Composer on Opus for tasks Claude Code does better

## Cursor Configuration Needed

### 1. MCP Servers (optional, selective)
Cursor CAN use the same MCP servers as Codex, but should it? Analysis:

| MCP Server | Worth adding to Cursor? | Why |
|------------|------------------------|-----|
| Playwright | **Yes** — useful for web testing in visual context | |
| LinkedIn | **No** — too specialized, use from Claude Code | |
| OpenAI Docs | **No** — Cursor has its own doc search | |

**Verdict**: Add Playwright only. Keep Cursor's MCP surface lean.

### 2. .cursorrules (critical)
This is the single most impactful thing to configure. It makes every Cursor interaction aware of your ecosystem.

### 3. Privacy Mode
Currently **disabled**. Your code is shared with Cursor for training. Consider enabling if working on sensitive code (SDI appeal, personal data). Settings → Privacy → Enable privacy mode.

### 4. Extensions
Currently **none installed**. Recommended minimum:
- None required — Cursor's built-in AI features replace most extension needs
- If you use ESLint/Prettier, those are worth adding for formatting

## What Cursor Should NOT Do

1. **Don't use Cursor for orchestration** — it has no memory between sessions, no agent-protocol.md integration, no session logs
2. **Don't use Cursor Composer for tasks > 5 files** — context window truncates aggressively (real limit ~70-120K despite 200K advertised)
3. **Don't run Cursor as a background agent** — it's designed for interactive, visual editing
4. **Don't duplicate Claude Code's work** — if you're already running Claude Code on a task, don't also ask Cursor Composer to do it
5. **Don't configure Cursor as "another Claude Code"** — that's paying twice for the same thing

## Integration with Agent Protocol

Cursor is different from Claude Code and Codex: it's a **foreground tool**, not a background agent. It doesn't need to:
- Appear in the Active Agents table (it's not persistent)
- Write session logs (sessions are ephemeral)
- Participate in handoffs (it doesn't carry state)

Instead, Cursor integrates by:
- Reading .cursorrules that reference the ecosystem standards
- Using MCP servers for tool access when needed
- Peretz using it as a visual frontend while Claude Code handles the backend orchestration

---

## Decision Matrix: "Which Tool Do I Use?"

```
Is it a quick edit (< 3 files)?
  YES → Cursor (Cmd+K or tab complete)
  NO ↓

Does it need memory from previous sessions?
  YES → Claude Code
  NO ↓

Does it need visual diff review?
  YES → Cursor Composer, then review visually
  NO ↓

Is it a test or contract task?
  YES → Codex
  NO ↓

Is it architecturally complex (5+ files, design decisions)?
  YES → Claude Code (plan mode)
  NO ↓

Default → Claude Code (GGO mode)
```

---

Timestamp: 2026-02-08
Signed By: Peretz Partensky
AI: Claude Code (Opus 4.6) — The Integrator
Chat/Project: Cursor Integration Strategy
Artifact Path: /Users/peretz_1/CURSOR-ROLE-STRATEGY.md
---
