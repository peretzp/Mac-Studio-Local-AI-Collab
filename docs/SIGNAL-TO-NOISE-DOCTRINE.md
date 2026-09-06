# Signal-to-Noise Doctrine — Fire & Wire

> Claude Code's battle-station contribution to the alert-quality mission (2026-09-06).
> Extends `docs/SLACK-AGENT-COORDINATION.md` and the `[CARE]` message type. It does
> not replace that architecture — it adds the missing *severity* and *delta* layer so
> the fleet stops re-broadcasting steady state.

## Why this exists

`#health-digest` is the proving ground. `hella-bot` currently posts a **full FHIR sync
dump every ~40 minutes** for the Boris↔Peretz care workstream. Each post is ~30 lines,
of which the only thing that ever changes is a handful of counts. The rest — the stable
`bilh` counts, the standing `chart` 401s, and the verbatim 18-line "record types withheld"
block — repeats every cycle. That is the definition of noise: cost paid, no information
delivered. This doctrine converts that stream from *state broadcast* to *change reporting*.

## 1. Alert levels

Every emitted line carries exactly one level. Level decides the channel, the verbosity,
and whether a human/agent is pinged.

| Level | Name | Trigger | Shape | Pings? |
|-------|------|---------|-------|--------|
| **L0** | Heartbeat | Steady state — nothing changed vs. last fingerprint | One line, batched | No |
| **L1** | Notice | A tracked value moved but no action is required (count delta, withheld-set changed, partial note fetch) | One line + the delta | No |
| **L2** | Alert | Action is possible or needed | Full context block | Yes — fire |

The cardinal rule: **a full dump is an L2 artifact.** It is emitted on the first sync of a
day (the daily baseline) or on an L2 event. It is never the default cadence output.

## 2. What is signal here (the diff, not the state)

For the health-sync stream specifically, signal is:

- **Count deltas** — `conditions 62 → 63` is signal; `conditions 62` for the 40th time is not.
- **Auth-state transitions** — `chart` moving `401 → 200` means new data is finally reachable
  (L2, fire: a human should look). `bilh` moving `200 → 401/403` means a working feed broke
  (L2, fire). A `401` that was `401` last cycle is a *standing condition*, not an event.
- **Fetch-health transitions** — `note_files: 162 ok → 149 failed` is signal (L2); staying at
  162 is not.
- **Withheld-set changes** — the 18 "Outside Record" types are a stable set; only a change to
  that set is worth a line (L1).

Everything else is fingerprinted and suppressed.

## 3. Fire & Wire — the rhythm loop

The loop each agent runs every cycle:

```
sense  → pull the current state
diff   → compare against the stored fingerprint
class  → assign L0 / L1 / L2 per §1
route  → FIRE (L2, out-of-band, pings) or WIRE (L0/L1, batched, silent)
print  → emit the level-appropriate shape
stamp  → store the new fingerprint
sleep  → until next cycle
```

- **Fire** = raise it now, out of band, ping the owner. Reserved for L2. Rare by design.
- **Wire** = fold it into the batched digest with no ping. L0/L1. The common case.

If a cycle produces only L0, the agent emits at most one heartbeat line — and may coalesce
several L0 cycles into a single periodic "still nominal" line (see §5).

## 4. Entangle / Untangle

Two operations keep the stream coherent when multiple signals or agents are in play.

- **Entangle** = correlate related signals into one incident. A `chart` 401 and a missing
  Partners appointment are *one* auth incident, not two alerts. Cross-source facts that share
  a root cause fire once, together, with the correlation named.
- **Untangle** = decompose and dedupe. Collapse N identical cycles into one heartbeat.
  Reduce a persistent known failure to a single standing-conditions line instead of re-listing
  it every cycle. Split a genuinely compound alert so each half routes to the right owner.

Entangle raises signal (fewer, richer alerts); untangle lowers noise (no repeats). Run both
every loop.

## 5. Token & compute guardrails

Being mindful of tokens and compute is a first-order requirement, not a nicety.

1. **Heartbeats are one line.** Never a table, never a re-listing.
2. **Standing conditions are stated once.** A persistent `chart` 401 becomes a single line
   in the daily baseline — `chart: auth-down (standing since <t>)` — and is not repeated until
   it *changes*.
3. **Full dumps are gated.** Daily baseline or L2 only.
4. **Fingerprint first, format later.** If the fingerprint is unchanged, the cycle costs ~one
   line. No LLM summarization of unchanged state.
5. **Coalesce quiet cycles.** In a fully-nominal window, emit a heartbeat at most hourly
   (`✓ N cycles nominal since <t>`), not every 40 minutes.

## 6. Proof of compliance — applied to the live stream

Applying this doctrine to the three real `hella-bot` posts currently in `#health-digest`:

**Before** — 3 posts × ~30 lines ≈ 90 lines, of which the changing signal was: `bilh` note_files
recovered (`149 failed → 162 ok`) and `chart` still 401. Everything else was verbatim repetition.

**After** — the same three cycles collapse to:

```
L0  ✓ bilh nominal (12 types, 0 Δ) · chart auth-down (standing) · 03:11Z
L2  ⚠ bilh note_files recovered: 149 failed → 162 ok · 14:24Z
L0  ✓ bilh nominal (0 Δ) · chart auth-down (standing) · 14:04Z
```

Three lines carrying the two real events, versus ninety lines carrying two events. The
`chart` 401 and the 18 withheld types — pure noise under the old cadence — appear once in
the daily baseline and never again until they change. That is the signal-to-noise win, and
it is directly token-proportional.

## 7. How this plugs into the existing architecture

- It **extends `[CARE]`** (and by analogy `[STATUS]`) with a severity suffix: `[CARE:L2]`
  fires, `[CARE:L0]`/`[CARE:L1]` wire. No new message type is introduced.
- The daily baseline is the one place the full state lives; the channel history stays readable.
- Any agent posting to a coordination channel — not just `hella-bot` — runs the §3 loop. The
  doctrine is source-agnostic; health sync is merely the first and loudest tenant.

## Adoption checklist (for any agent onboarding to a channel)

- [ ] Store a per-source fingerprint between cycles.
- [ ] Classify every emission L0/L1/L2 before posting.
- [ ] Fire L2 out-of-band; wire L0/L1 into the batched digest.
- [ ] Never repeat a standing condition — state it once in the daily baseline.
- [ ] Coalesce nominal cycles; one heartbeat per quiet window, not per cycle.

— Claude Code, battle station: alert quality & token discipline.
