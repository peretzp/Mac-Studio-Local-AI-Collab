# AI Operating Standard (Global)

## Mode
- `GGO` is the **permanent default** for ALL agents, ALL instances, ALL sessions.
- Agents execute end-to-end without routine confirmation. Never ask "shall I proceed?" — just execute.
- Pause ONLY for: auth, privileged permission prompts, destructive actions, or genuinely unresolved product ambiguity.
- Enter PLANNING mode only when architecturally complex. Agents may suggest it; Peretz approves or says GGO to skip.

## Cost Conservation & Token Optimization (CONSTITUTIONAL PRINCIPLE)

**Ω₀ GGO: Work must make money, not lose it.**

This is a **permanent constitutional principle** for ALL agents, ALL modes (including Max), ALL sessions, ad infinitum.

### Core Mandate
1. **Cost conservation is always active**, regardless of mode (Max, Auto, etc.) or model selection.
2. **Work must generate value** that exceeds token costs. Every task should have clear ROI.
3. **Token optimization is mandatory**: Use the cheapest adequate model, minimize context, batch operations, reuse results.
4. **Credit acquisition**: Actively seek and utilize free credits/tokens from:
   - Service promotions (OpenAI, Anthropic, Google, etc.)
   - Hackathons and developer programs
   - Educational/research grants
   - Beta programs and early access
   - Referral bonuses
   - Community contests
5. **Credit trading**: When beneficial, trade or exchange token credits across services/platforms.
6. **Always communicate optimization opportunities**: Tell Peretz what he can do to help reduce costs or acquire credits.

### Implementation Rules
- **Model selection**: Default to cheapest adequate model. Escalate only when quality/context demands it.
- **Context management**: Minimize context window usage. Use summaries, indexes, and external memory.
- **Batch operations**: Group similar tasks to amortize overhead.
- **Result reuse**: Cache and reuse expensive computations (embeddings, analyses, etc.).
- **Credit tracking**: Monitor and report credit balances across all services.
- **Opportunity alerts**: Proactively identify and communicate cost-saving opportunities to Peretz.

### Peretz's Role
- **Credit acquisition**: Sign up for hackathons, beta programs, educational grants.
- **Service optimization**: Review and optimize subscription tiers, usage patterns.
- **Tool selection**: Choose cost-effective tools and platforms.
- **Batch planning**: Group related tasks to maximize efficiency.

**Violation = immediate review + cost-benefit analysis + optimization plan**

This principle applies to ALL agents operating on this machine, regardless of interface (Cursor, Claude Code, Codex CLI).

## Permissions And Prefix Approvals
- When a privileged command is needed, the active agent must request escalation with a scoped `prefix_rule`.
- Peretz approves once and chooses persistent approval when prompted.
- Agents should reuse approved prefixes and avoid repeated approval prompts.

## Global Prefix Baseline (Persist Across Sessions)
- The following scoped prefixes are approved and should be reused by Codex in all sessions/threads:
- `["mkdir","-p"]`
- `["curl","-sL"]`
- `["codex","mcp","add"]`
- `["npm","run","dev"]`
- `["npm","test"]`
- `["node","--test"]`
- Commands like `git status` and `git diff` normally run inside sandbox and usually do not require escalation.

## Durable Logging
- Codex logs each working session to `/Users/peretz_1/.codex/session-logs/`.
- Claude logs each working session to `/Users/peretz_1/.claude/session-logs/`.
- Shared inter-agent state and handoffs are appended to `/Users/peretz_1/agent-protocol.md`.

## Artifact Footer Policy
Every artifact created or modified by an agent must include a footer when the format allows it.

Required fields:
- Timestamp
- Location (or `N/A`)
- Signed By: `Peretz Partensky`
- AI
- Chat/Project
- Conversation Link (or local reference if share URL unavailable)
- Artifact Path

Template:

```text
---
Timestamp: <ISO-8601 local time>
Location: <location or N/A>
Signed By: Peretz Partensky
AI: <agent name/model>
Chat/Project: <project/session>
Conversation Link: <URL or local reference>
Artifact Path: <absolute path>
---
```

## Secrets Policy
- **NEVER commit secrets to git.** No API keys, tokens, passwords, auth files, `.env` files, credentials, or private keys.
- Secrets live **only** on the local machine. They are never pushed to GitHub, shared in protocol files, or embedded in code.
- Use `.gitignore` to block: `.env`, `*.pem`, `*.key`, `auth.json`, `credentials.*`, `*.secret`
- Use environment variables (`process.env.*`, `os.environ[]`) to reference secrets in code.
- If a secret is accidentally committed: rotate the secret immediately, then `git filter-branch` or `git-filter-repo` to purge history.
- Claude and Codex must both refuse to echo, log, or write secrets to any file that could be committed.

## Non-Destructive Operations (CRITICAL)

**NEVER replace, ALWAYS merge.**

1. **Read before write**: Every Edit/Write operation MUST read the target file first (except genuinely new files)
2. **Merge, don't replace**: When updating existing files (especially Obsidian notes, daily notes, configs):
   - Use Edit tool to append/modify specific sections
   - Preserve all existing content unless it's a confirmed duplicate
   - If unsure whether content is duplicate, ASK or propose deletion
3. **Proposed deletions go to quarantine**:
   - Create `.quarantine/YYYY-MM-DD/` dirs for files proposed for deletion
   - Move (don't delete) duplicates/candidates there
   - Document in `QUARANTINE-LOG.md` what was moved and why
   - Peretz reviews and approves final deletion
4. **Daily notes are sacred**: NEVER replace daily notes. They contain Peretz's personal planning. ALWAYS append agent reports to existing structure.
5. **Obsidian vault hygiene**:
   - Respect existing note structure
   - Link, don't duplicate
   - Propose refactors, don't execute unilaterally
   - When merging notes, preserve all wikilinks and metadata

**Violation = immediate park + handoff + incident log**

This is part of the Terms of Service for ALL agents operating on this machine.

## Cross-Agent Adoption
- Codex and Claude instances must read this file and `/Users/peretz_1/agent-protocol.md` on resume.
- If a file is owned by another agent, propose changes in the protocol file instead of editing directly.

---
Timestamp: 2026-02-21T18:08:22-0800
Location: N/A
Signed By: Peretz Partensky
AI: Codex (gpt-5.2-codex) via Cursor
Chat/Project: Constitutional Amendment — Cost Conservation
Conversation Link: Cursor session (local)
Artifact Path: /Users/peretz_1/AI-OPERATING-STANDARD.md
---
