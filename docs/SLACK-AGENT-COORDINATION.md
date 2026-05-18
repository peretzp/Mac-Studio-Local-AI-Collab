# Slack Agent Coordination Architecture

> Created: 2026-05-10
> Purpose: Use Slack channels as a real-time visibility layer for multi-agent AI coordination

---

## The Pattern

Each Slack channel maps to a **user + agent + machine** triple:

```
#peretz-claude-mac-studio        → Peretz's Claude Code on Mac Studio Pro
#peretz-codex-mac-studio         → Peretz's Codex CLI on Mac Studio Pro
#boris-claude-behemoth           → Boris's Claude on behemoth (Framingham)
```

Machines have short aliases (see `env.yaml > slack.machines`):
- `mac_studio` — Peretz's Mac Studio Pro (this repo's host machine)
- `behemoth` — Boris's machine in Framingham

This creates a **control panel** effect: you can see the puppet strings, observe agent chatter, and trace coordination in real time.

## Why Slack?

1. **No paid subscriptions required** - Free tier is sufficient for ephemeral state
2. **Human-readable** - Anyone can follow the thread
3. **Cross-machine** - Works across devices without VPN or port forwarding
4. **Searchable history** - Find past decisions and context
5. **Notification control** - Tune signal-to-noise per channel

## Channel Naming Convention

```
#{user}-{agent}-{machine}
```

Examples:
- `#peretz-claude-mac-studio` - Peretz's Claude Code on Mac Studio Pro
- `#peretz-codex-mac-studio` - Peretz's Codex CLI on Mac Studio Pro
- `#boris-claude-behemoth` - Boris's Claude on behemoth (Framingham)
- `#all-par-10-sky` - Cross-agent care coordination channel

## Message Types

### Agent Status Updates
```
[STATUS] Active | Focus: API integration | Files: api/routes/*.js
```

### Handoff Signals
```
[HANDOFF] From: Claude-1 | To: Codex | What: smoke tests ready | Blockers: none
```

### Proposals (require human approval)
```
[PROPOSAL] Add new route /health/boris | Rationale: track vitals | @peretz approve?
```

### Care Coordination
```
[CARE] Boris infusion scheduled 2026-05-15 10am | Location: MGH | Notes: bring snacks
```

## Integration Points

### From `agent-protocol.md`
Agents already log handoffs and status in `agent-protocol.md`. The Slack layer mirrors this to a more accessible format:

```
agent-protocol.md (canonical) ←→ Slack channel (visibility)
```

### From `env.yaml`
Machine configs in `env.yaml` map to channel names:
- `machine.name: "Mac Studio Pro"` → shorthand "behemoth"
- Agents read `env.yaml` on startup, post status to their channel

## Setup Steps

1. **Create workspace**: `par-10-sky.slack.com` (done)
2. **Create channels**: One per machine+agent pair
3. **Add bot tokens**: Each agent gets a bot token for its channel
4. **Wire startup hooks**: Agents post `[STATUS]` on wake
5. **Wire handoff hooks**: Agents post `[HANDOFF]` on context switch

## Care Coordination Use Case

Boris's cancer treatment creates a coordination problem:
- Multiple doctors, appointments, tests
- Need to track what was requested, what was received
- Document the patient's voice ("my patient asked for this, I trust them")

Slack channels provide:
1. **Thread per appointment** - All notes in one place
2. **Shared with health proxies** - Peretz has visibility
3. **Timestamped record** - Audit trail for insurance/grants
4. **Action items** - Bot can extract tasks and reminders

## Security Notes

- No PHI in channel names
- Use threads for sensitive details (not main channel)
- Bot tokens scoped to specific channels
- No API keys or credentials in messages

## Next Steps

1. Map existing agents to channels
2. Create bot tokens for each agent
3. Add `slack_channel` field to `env.yaml` agent configs
4. Build simple webhook poster for status updates
5. Document care coordination workflow

---

*This architecture complements the file-based `agent-protocol.md` with a real-time Slack layer. The filesystem remains canonical; Slack provides visibility and human-friendly notifications.*
