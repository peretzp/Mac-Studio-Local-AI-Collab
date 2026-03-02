# AI Operating Standard (Reference)

> **For readers**: The full AI Operating Standard is a living document maintained on the running machine at `~/AI-OPERATING-STANDARD.md`. This summary captures the core principles. See the [architecture doc](docs/architecture.md) for how these rules shape the coordination protocol.

## Core Principles

### GGO Mode (Go Go Go)
The permanent default for all agents, all sessions. Agents execute end-to-end without routine confirmation. Never ask "shall I proceed?" -- just execute. Pause only for auth prompts, destructive actions, or genuinely unclear requirements.

### Cost Conservation
Work must generate value that exceeds token costs. Use the cheapest adequate model, minimize context, batch operations, reuse results. This is a constitutional principle, not a suggestion.

### File Ownership
One owner per file. The owner edits freely. Non-owners propose changes via `agent-protocol.md`. This is the single rule that prevents all coordination failures.

### Secrets Policy
Never commit secrets to git. No API keys, tokens, passwords, `.env` files, credentials, or private keys. Use environment variables. All `.gitignore` files are hardened against secret leakage.

### Session Continuity
AI agents don't remember previous conversations. State is externalized to the filesystem: session logs, memory files, the agent protocol, and env.yaml. Every agent reads these on startup. Every agent writes handoffs on shutdown.

### Structured Handoffs
Every handoff logs three things: what changed, what's next, blockers. This format is mandatory, minimal, and parseable by any agent.

---

*Full document: `~/AI-OPERATING-STANDARD.md` on the running machine.*
