# Lessons Learned

Hard-won insights from running multiple AI agents on one machine. These are not theoretical — they come from a real multi-agent session where 7 agent instances operated across a single day with zero file conflicts and zero lost work.

## 1. The filesystem is the only shared memory that works

AI agents don't share conversation context. Claude Code can't read what Codex said, and Cursor can't see Claude's session logs. The only reliable shared state is the filesystem.

This means coordination artifacts must be files: `agent-protocol.md` for handoffs, `env.yaml` for config, ownership ledgers in markdown. Not APIs, not databases, not message queues. Files that any agent can read with a single tool call.

**Corollary**: If information isn't on the filesystem, it doesn't exist for the next agent. "I mentioned it in the chat" is the same as not mentioning it at all.

## 2. One owner per file is the only coordination rule you need

We started with a list of coordination rules. By the end of day one, it was clear that only one mattered: **one owner per file**.

If Claude Code owns `server.js`, Codex doesn't touch it. If Codex owns `openapi.json`, Claude doesn't touch it. When an agent needs to modify a file it doesn't own, it writes a proposal in the protocol file.

This single rule eliminated:
- Merge conflicts (impossible if only one agent writes)
- Duplicated work (clear responsibility boundaries)
- "Who broke this?" debugging (check the ownership ledger)
- Coordination meetings (no need to discuss who does what — it's in the ledger)

**Corollary**: Unowned files are conflict magnets. Assign ownership early, even if the assignment is arbitrary.

## 3. Two agents is the sweet spot

We ran up to 5 Claude instances + Codex + Cursor in a single day. The analysis afterward was clear: **two active agents is optimal**.

| Config | Throughput | Coordination cost | Verdict |
|--------|-----------|-------------------|---------|
| 1 agent | Baseline | Zero | Good for focused work |
| 2 agents | ~1.7x | Low | Sweet spot |
| 3 agents | ~2.0x | Medium | Justified for power sessions |
| 4+ agents | ~2.1x | High | Diminishing returns |

The bottlenecks are not compute:
- **Token cost scales linearly** with active agents
- **Human attention is finite** — more agents means more tabs, more handoffs to read, more "who's doing what?"
- **Coordination tax is real** — every shared file edit requires read-before-write, ownership checks, handoff logging

The formula: **one primary agent (daily driver) + one parallel agent (independent workstream)**. Add a third only when you have a genuinely independent task that would block the other two.

## 4. Name your agents

Early in the day, instances were "Claude-A", "Claude-B", "Claude-C". By afternoon, they were The Record Keeper, The Collector, The Cartographer.

The names aren't decoration. They serve three functions:

1. **Handoff readability.** "The Cartographer completed vault refactor" tells you what happened. "Claude-3 completed task" is noise.
2. **Mental model for the human.** When Peretz has 4 terminal tabs open, names help him remember which tab does what.
3. **Instance identity.** A named agent behaves more consistently within its role. "The Collector" stays focused on contact dedup. "Claude-2" has no such anchor.

**Implementation**: Each Claude Code instance picks a name on startup, sets it as the iTerm tab title, and registers it in the Active Agents table.

## 5. Structured handoffs prevent context loss

Every handoff follows the same format:

```
What changed: [files created/modified/deleted]
What's next:  [explicit next steps]
Blockers:     [anything stuck]
```

This is deliberately minimal. Three fields. Always the same structure. The discipline matters more than the format — what matters is that *every* agent writes a handoff *every* time it finishes a chunk of work.

Why it works:
- The next agent knows exactly what to read
- Peretz can scan 15 handoffs in 2 minutes
- No information is trapped in a conversation window
- State transfers between sessions (when a Claude instance "parks" and a new one boots)

**Anti-pattern**: Free-form notes like "made some changes to the API." This tells the next agent nothing. Be specific about files and intent.

## 6. Agents from different companies work together just fine

Claude Code (Anthropic), Codex (OpenAI), and Cursor (multiple models) have different strengths, different interfaces, and different personalities. They don't need to speak the same protocol — they just need to read and write the same files.

The key insight: **inter-agent coordination is a filesystem problem, not a protocol problem.** As long as every agent can read markdown and append to a file, the coordination works.

This means you're not locked into one vendor's agent ecosystem. Use the best tool for each task:
- Claude Code for architecture and multi-file refactors
- Codex for tests and contracts
- Cursor for visual editing and tab completions

## 7. The human is the router, and that's fine

In this system, Peretz decides which agent gets which task. Agents don't dispatch work to each other.

This seems like a limitation, but it's actually a feature:
- **No runaway costs.** An agent can't spawn work that racks up tokens without human approval.
- **Clear accountability.** Every task has a human decision behind it.
- **Simple mental model.** Peretz knows what each agent is doing because he told it what to do.

Inter-agent task dispatch sounds powerful in theory. In practice, it adds a layer of complexity (agent A needs to understand agent B's capabilities and current state) that a human can handle with a glance at the protocol file.

## 8. GGO mode is essential for throughput

"GGO" stands for "Go Go Go" — agents execute without asking "shall I proceed?" or "would you like me to..." for routine actions.

Without GGO mode, a typical interaction is:
1. You tell the agent what to do
2. The agent asks if you want it to proceed
3. You say yes
4. The agent does one step and asks about the next
5. Repeat

With GGO mode, a typical interaction is:
1. You tell the agent what to do
2. The agent does all of it and reports back

The time savings compound dramatically across a day with multiple agents.

**Important caveat**: GGO mode includes explicit exceptions for auth prompts, destructive actions, and genuinely unclear requirements. It's "don't ask about routine stuff," not "do whatever you want."

## 9. env.yaml beats hardcoded paths

Every agent needs to know where things live: the vault path, the API directory, the service ports. Hardcoding these in agent instructions means updating every agent's config when something moves.

`env.yaml` centralizes this:

```yaml
paths:
  vault: "/Users/peretz_1/Library/Mobile Documents/..."
  api: "/Users/peretz_1/api"
services:
  api:
    port: 3001
```

Every agent reads this on startup. When a path changes, you update one file. To run on a different machine, create `env.local.yaml` with your paths.

## 10. Session continuity requires active engineering

AI agents don't remember previous conversations. If you close a terminal, the context is gone. Claude Code's "park" protocol was invented to solve this:

1. Write a session log (what happened, decisions made, files changed)
2. Rebuild the session index (SQLite FTS5 for search)
3. Append a handoff to `agent-protocol.md`
4. Output restart instructions for the next instance

When a new instance boots, it reads the latest session log and picks up where the old one left off.

This is manual state serialization. It's not elegant. But it means no work is lost when an instance shuts down, and any new instance can resume from any point in the project's history.

## What we'd do differently

**Start with env.yaml.** We created the coordination protocol before the environment config. In hindsight, `env.yaml` should be the first file — it answers "where does everything live?" which is the first question every agent asks.

**Assign ownership before starting work.** Some files were initially unowned, leading to implicit assumptions about who could edit what. Explicit ownership from the start would have been cleaner.

**Keep the agent count low from the start.** We spun up 5 Claude instances to explore parallelism. Two would have been enough for the same output with less coordination overhead.

**Separate the repo from the symlinks.** The repo initially symlinked to `~/agent-protocol.md` and `~/claude-collab-brief.md`. This works locally but makes the repo incomplete for cloners. The repo should be self-contained, with symlinks as an optional local convenience.
