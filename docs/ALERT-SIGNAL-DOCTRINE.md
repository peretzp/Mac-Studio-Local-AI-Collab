# Alert Signal Doctrine — Fire & Wire

> Battle-station contribution from Claude Code (Opus 4.8) toward maximizing signal-to-noise across the Boris↔Peretz workstreams. This doctrine governs *how* alerts are raised, deduplicated, and routed. It is transport-agnostic (Slack, webhook, or file) and applies to any bot that reports state into the fleet — starting with the `hella-bot` health-sync feed in `#health-digest`.

## The problem (observed, not hypothetical)

The live `#health-digest` feed shows four textbook noise patterns:

1. **Level-triggered spam.** `hella-bot` posts a full PRODUCTION sync dump every cycle even when nothing changed. Near-identical payloads minutes apart.
2. **Sticky-failure re-alerting.** The `chart` source returns `HTTP 401` on every endpoint, every cycle. One broken auth state, re-announced forever.
3. **Steady-state masquerading as news.** The `bilh` source returns `HTTP 403` on appointments and "18 record types withheld" on every sync. That is a stable condition, not an event.
4. **Real edges drowning.** `note_files` moved `162 → "no data (149 failed)" → 162`. That transition is the *actual signal* — and it is buried under (1)–(3).

Noise costs three things we care about: human attention, agent tokens/compute (every dump re-summarized is wasted inference), and trust (a channel that cries wolf gets muted).

## First principle: edge-triggered, not level-triggered

**Alert on transitions, not on states.** A condition fires *once* on the edge into it, then goes silent until the edge out of it. A persistent `401` is one alert on the healthy→broken edge, then nothing until it flips back to healthy (which is itself an alert: "recovered").

State is computed by a cheap deterministic diff against the last snapshot per `(source, record_type)`. No LLM is invoked to decide whether to alert — only to *summarize* a confirmed edge.

## Severity ladder

| Level | Name | Trigger | Behavior |
|-------|------|---------|----------|
| **P1** | AUTH-BROKEN | `401`/`403` on the edge into failure for a source that was previously authenticated | FIRE immediately (interrupt) |
| **P2** | DATA-DEGRADED | Record count drops below threshold, or a fetch that previously succeeded now returns "no data / N failed" | FIRE immediately (interrupt) |
| **P3** | DRIFT | The set of withheld/partial record types changes; count moves within normal band | WIRE only (rolls into the next digest) |
| **P0** | HEALTHY | Steady state, or counts within band | SILENT (never interrupts; visible in digest footer) |
| **REC** | RECOVERED | Edge out of any P1/P2 back to healthy | FIRE once (closes the open alert) |

Thresholds live in config, not code, so Boris and Peretz can tune per source without a redeploy. Suggested defaults: count-drop ≥ 20% → P2; any auth-code appearance where the prior state was 200 → P1.

## Fire & Wire — the rhythm loop

Two channels, two cadences, one shared state:

- **FIRE** = edge-triggered interrupts (P1/P2/REC). Immediate, one message per edge, routed to the owning workstream's channel. A fire message names *exactly one changed thing* and its severity. Never a full dump.
- **WIRE** = the scheduled digest (default: one per day, or on demand). Rolls up everything — P3 drift, current counts, open P1/P2s still unresolved, and a HEALTHY footer. This is the heartbeat that keeps everyone **entangled** (shared context) without spam.

**Entangle** = the WIRE digest pulls all workstreams into one coherent state view. **Untangle** = when a FIRE edge hits, isolate the single changed fact so it is not buried in the steady-state noise. Entangling is the default resting state; untangling is what an edge does.

## Dedup & debounce

- Identical payloads within a debounce window collapse to a single line with a counter: `HTTP 401 (chart, all endpoints) ×N since <first-seen>`. The count updates in place in the digest; it does not re-fire.
- An open P1/P2 is *sticky*: it appears in every WIRE digest (so it is not forgotten) but does not re-FIRE until it changes state.
- A flapping condition (edges crossing repeatedly within a window) is coalesced into one "flapping" P2 rather than N alerts.

## Token & compute mindfulness

- The diff that decides whether to alert is deterministic and free — no inference.
- LLM summarization runs **only on a confirmed edge**, never on steady state. A day of unchanged 401s costs zero tokens.
- The WIRE digest is generated once per cadence, not per sync, and reuses the cached diff.
- Rule of thumb: if a message would say the same thing as the last one, it must not be sent — it must update a counter instead.

## Routing (Boris ↔ Peretz workstreams)

Alerts route to the owning workstream's channel (per `env.yaml` `slack.channels`):

- Health/care edges (P1/P2 on `bilh`/`chart`, appointment changes) → `#all-par-10-sky` (care coordination), tagged `[CARE]`.
- Agent/service state (dashboard/api/prompt_browser health) → the relevant per-agent machine channel, tagged `[STATUS]`.
- Anything needing a human decision → `[PROPOSAL]` with `@peretz approve?`.

A single edge produces a single message in a single channel. Cross-posting is noise.

## Compliance checklist for any reporting bot

- [ ] Emits on edges, not on every cycle.
- [ ] Carries a severity level from the ladder.
- [ ] Deduplicates sticky failures into a counter.
- [ ] Sends steady state to the digest, never as an interrupt.
- [ ] Invokes no LLM unless an edge is confirmed.
- [ ] Routes to exactly one owning channel.

## Status

Doctrine only — no bot yet enforces it (this is a docs-and-config repo). The natural first implementation target is the health-sync poster behind `hella-bot`; the "simple webhook poster" listed as a Next Step in `docs/SLACK-AGENT-COORDINATION.md` should be built to satisfy the compliance checklist above.
