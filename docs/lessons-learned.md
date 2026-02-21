# Lessons Learned

What works, what doesn't, and what we'd do differently. Drawn from 35+ session logs across 20+ named agent instances operating over two weeks.

## What works

### File-based coordination is surprisingly robust

No message queue, no pub/sub, no RPC. Just markdown files on disk. Agents read before writing, append handoffs, and check ownership. This works because:

- Files are atomic enough for our concurrency level (2-3 agents, not 200)
- Markdown is readable by humans and agents alike
- Append-only handoff logs create a natural audit trail
- `git diff` shows exactly what changed and when

We expected to need something more sophisticated. We didn't. The protocol file grew to 700+ lines and never had a merge conflict because ownership rules prevented simultaneous edits to the same file.

### One owner per file eliminates an entire class of problems

This is the most important design decision in the system. Every file has exactly one owner. The owner edits freely; everyone else proposes changes through the protocol file.

In practice, this means:
- Claude Code builds API route handlers, Codex writes the tests
- Nobody ever loses work to a merge conflict
- "Who should fix this?" is never ambiguous
- Agents don't need to lock files or coordinate timing

The overhead is small (checking the ownership ledger takes seconds) and the benefit is large (zero coordination failures on file access in two weeks of operation).

### GGO mode (execute without confirmation) changes everything

The default for all agents is "Go Go Go" — execute end-to-end without asking "shall I proceed?" Agents pause only for auth prompts, destructive actions, or genuinely unclear requirements.

This sounds risky. In practice, it's the single biggest productivity multiplier. A Claude Code session that would take 30 minutes with confirmation prompts takes 5 minutes in GGO mode. The agent builds, tests, commits, and moves on.

The safety net is not confirmation prompts — it's version control, session logs, and the ability to roll back. Git makes aggressive execution safe.

### Named agents create accountability and legibility

Each Claude Code instance picks a persona name: "The Cartographer", "The Lamplighter", "The Collector". This started as a whimsical touch but turned out to be functionally important:

- iTerm tabs show names, so the human can identify agents at a glance
- Session logs are filed by name, making the history searchable
- Handoffs are attributed to names, creating a clear chain of responsibility
- It's easier to say "The Collector finished contact dedup" than "Claude instance #7 in tab 3"

The retired agents table in the protocol file reads like a project history — each agent, its completed work, and its session log. This is more legible than git blame.

### Continuous session logging saves you from context limits

Claude Code instances run out of context. This is not a theoretical risk — it happened on day one. The "Tentacle Closer" instance hit its context limit after 26 minutes of intensive execution and couldn't complete its park protocol.

The fix: session logs are written continuously (auto-save), not just at shutdown. After every significant action — file created, commit pushed, decision made — the agent appends to its session log and rebuilds the session index.

When the Tentacle Closer's context filled up, its session log was already on disk. The next agent read the log and continued the work. Zero context was lost.

This pattern is now mandatory for all Claude Code instances.

### The env.yaml pattern makes the system portable

All machine-specific configuration lives in one YAML file: paths, ports, runtimes, services, MCP configs, GitHub accounts. Agents read `env.yaml` instead of hardcoding paths.

To move the system to a different machine: copy `env.yaml` to `env.local.yaml`, edit the paths, done. The `.gitignore` blocks `env.local.yaml` so local overrides never get committed.

This is a small thing, but it's the difference between "this only works on my machine" and "fork this, edit one file, and it works on yours."

## What doesn't work

### More than 3 concurrent agents hits diminishing returns fast

The Lamplighter's analysis was correct: 2 active agents with 1 on standby is the sweet spot. With 4+ agents:

- **Human attention fragments.** Context switching between agent tabs is real cognitive overhead. More agents = more handoffs to read, more "who's doing what?"
- **Coordination tax scales poorly.** Every shared-file edit requires read-before-write, ownership checks, and handoff logging. With 2 agents this is lightweight. With 4+ it's bureaucracy.
- **Token cost is linear.** Two agents doing overlapping work doubles cost for less than 2x output.
- **RAM pressure is real.** On a 64GB machine running Codex Desktop + Cursor + iTerm + Obsidian + 3 browsers + 3x 4K displays, you feel it at 4+ heavy processes.

The right model: one primary Claude Code instance as the daily driver. Spin up a second only for genuinely independent tasks. Use Codex for verification. Use Cursor for visual editing sessions.

### Context limits are the hardest constraint

Claude Code conversations have a finite context window. Long sessions — especially ones involving many file reads, complex reasoning, and sub-agent coordination — can exhaust context in 20-30 minutes of intense work.

When context fills up:
- The agent can't process new information
- The `/compact` command may fail ("Conversation too long")
- The park protocol may not complete
- Sub-agent results may arrive but can't be processed

Mitigations:
- Continuous session logging (the log is on disk before context fills)
- Breaking large tasks into phases that fit within one session
- Using sub-agents for parallelizable work (each sub-agent has its own context)
- Aggressive use of `/compact` before context pressure hits critical

There's no silver bullet here. Context limits are a hard constraint of current LLM architecture. The system works around them rather than solving them.

### Codex can't run servers or complete OAuth

Codex runs in a sandboxed environment. It cannot:
- Bind to local ports (`EPERM` on `listen`)
- Complete OAuth flows or click UI buttons
- Access services that require network binding

This means Codex can write tests but can't run integration tests that require a live server. It can configure MCP servers via CLI but can't complete the UI-based installation steps.

Workaround: Codex writes the code and configuration; the human or Claude Code runs the tests locally and completes UI actions.

### Session logs can get noisy

With 35+ session logs accumulated in two weeks, finding the relevant context requires searching, not browsing. The session index (SQLite FTS5) helps, but the naming convention is critical:

**Good**: `2026-02-08-cartographer-vault-completion.md`
**Bad**: `2026-02-08-session-7.md`

The naming convention was standardized after the first week: `YYYY-MM-DD-{agent-name}-{focus}`. Before that, some logs were named generically and were hard to find.

### Agents don't always know their limits

Agents are optimistic. They'll attempt tasks that turn out to be beyond their context window, require information they don't have, or need human interaction they can't perform.

Examples from real sessions:
- An agent tried to complete LinkedIn MCP login (requires manual browser interaction)
- An agent planned to transcribe 929 voice memos in one session (context limit hit at memo ~50)
- An agent wrote a handoff saying "no blockers" when the next step actually required DNS registrar credentials

The fix is not to make agents pessimistic — it's to structure work so that failures are recoverable. Continuous logging, atomic commits, and small diffs mean that when an agent overreaches, the damage is limited to one incomplete task, not a corrupted codebase.

## Design decisions we'd make again

### Markdown over databases for coordination

`agent-protocol.md` could have been a SQLite database or a JSON file. We chose markdown because:
- Every agent can read and write it natively
- Humans can read it without tooling
- `git diff` shows meaningful changes
- The append-only handoff log is a natural markdown pattern
- It's greppable

For state that needs to be queried (prompts, session index), we use SQLite. For state that needs to be read and understood, we use markdown. This split has held up well.

### Symlinks over copies

The protocol file and collab brief live at `~/` (where agents expect to find them) and are symlinked into the repo. This means:
- The repo always reflects current state without manual syncing
- Agents don't need to know about the repo to use the protocol
- There's one canonical version, not two copies that can drift

### Zero-dependency services

The API, dashboard, and prompt browser are all zero-dependency Node.js HTTP servers. No Express, no Fastify, no framework. This means:
- No `node_modules` to manage
- No security vulnerabilities in dependency trees
- Any agent can read and modify the server code without understanding a framework
- Boot time is instant

The trade-off is more boilerplate (manual routing, manual CORS, manual JSON parsing). For services this simple, it's worth it.

## Things we'd do differently next time

### Establish the protocol before any work starts

On day one, the first real work (API scaffolding) started before the coordination protocol was fully established. This worked out because only one agent was building at the time, but it could have caused conflicts if two agents had started simultaneously.

The protocol should be the first artifact created — before any code, before any configuration, before any agent starts "real" work.

### Standardize session log naming from day one

The naming convention (`YYYY-MM-DD-{agent-name}-{focus}`) wasn't established until after several generic logs had been created. Early logs like `2026-02-08-lookaround.md` don't follow the pattern and are harder to attribute.

### Build the prompt store earlier

The prompt store (SQLite FTS5 of all human prompts) was built on day one but could have been built even earlier — before the first multi-agent session. Having searchable prompt history from the very start would have made the second-day sessions smoother.

### Document the concurrency recommendation earlier

The Lamplighter's analysis of "how many agents should run concurrently" was done at the end of day one. If it had been done first, it would have saved some experimental thrashing with too many agents running at once.
