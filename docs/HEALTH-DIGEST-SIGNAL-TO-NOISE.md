# Health-Digest Signal-to-Noise Doctrine

> Created: 2026-09-06
> Owner: Claude Code (see `agent-protocol.md`)
> Scope: the `#health-digest` Slack channel and the `hella-bot` health-sync digests
> Status: PROPOSAL — the digest generator lives in `galen-care` (gitea); this doc is the design it should adopt.

---

## Why this exists

The `#health-digest` channel carries `hella-bot` "Health sync — PRODUCTION" digests for two
records: **`bilh`** (Lahey / BILH — Boris's record, behemoth/Framingham) and **`chart`**
(Mass General Brigham / Partners — Peretz's record). Every sync posts a **full snapshot** of
every FHIR resource type and its count or failure. This maximizes noise:

- **Structural failures are reprinted as if they were news.** `chart` returns a wall of
  `HTTP 401` on nearly every resource type, every sync. `bilh` returns a permanent
  `HTTP 403` on `appointments` and an identical **"18 record types withheld by the server"**
  block, sync after sync, unchanged.
- **No delta.** Each message is a fresh dump, so a real state change (e.g. `note_files`
  going `149 failed` → `162 ok`, or `chart` labs going `0` → `969`) is drowned in ~15 lines
  of unchanged rows.
- **One real root cause is buried.** The `chart` 401 wall is a single actionable fact —
  *the token expired, re-auth* — currently rendered as ~9 near-identical "no data (HTTP 401)"
  lines.

The goal is not fewer syncs. It is: **surface state *changes* and *actionable* faults; suppress
steady state and known-structural limitations.**

## The three tiers

Every resource-type result is classified each sync, then routed by tier.

| Tier | Meaning | Route | Examples from live data |
|------|---------|-------|-------------------------|
| 🔴 **ACT** | Actionable fault or a new/changed failure a human can fix | Post to channel, `@`-mention owner | `chart` auth wall (token expired → re-auth); a source that was up is now fully down |
| 🟡 **DELTA** | A count or status changed vs. the last sync | Post to channel (delta lines only) | `note_files` `149 failed` → `162 ok`; `chart` labs `0` → `969`; new condition/med/report appears |
| ⚪ **STEADY** | Unchanged from last sync, or a known-structural limitation | Suppress; fold into a one-line heartbeat | `bilh` `appointments` `403`; the "18 withheld types" block; `conditions: 62` identical to last sync |

If a sync produces no ACT and no DELTA, it collapses to a single heartbeat line:

```
✓ health-sync steady — bilh 12/13 types OK, chart auth-down (unchanged since <ts>). No delta.
```

## Mechanisms

### 1. Delta engine (kills the snapshot noise)
Persist the last successful fingerprint per `(source, resource_type)` = `{count, status}`.
On each sync, emit a line **only** when the fingerprint changes. Unchanged rows are counted,
not printed. This alone removes ~80% of the current volume.

### 2. Known-issue registry (kills the structural-failure noise)
Encode expected, structural failures so they stop paging:

```yaml
known_issues:
  - source: bilh
    resource: appointments
    status: 403            # server does not expose Appointment on this proxy
    expected: true
  - source: bilh
    resource: withheld_record_types   # the 18 "Outside Record" / genomics / dental types
    count: 18
    expected: true
```

A known issue is suppressed to the STEADY footer. It only re-tiers to DELTA/ACT when it
*changes* (e.g. the 403 clears, or the withheld count moves off 18).

### 3. Auth-fault collapse (kills the 401 wall)
When ≥ N resource types from one source fail with the same auth status (401/403) in one sync,
collapse them into ONE ACT line naming the root cause and the fix, not one line per type:

```
🔴 chart: auth expired — 9/11 resource types returned 401. Re-auth the Partners/MGB token. (last good: labs 969 @ <ts>)
```

### 4. Per-record routing (separates Boris from Peretz)
`bilh` = Boris, `chart` = Peretz. Route ACT items to the owning person's care channel and keep
`#health-digest` as the combined delta view, so an alert about Peretz's token doesn't read as a
Boris alert. (Respects `env.yaml > slack` machine/user mapping; no PHI in channel names, per
`docs/SLACK-AGENT-COORDINATION.md`.)

### 5. Provenance classes (borrowed from `galen-care`)
`docs/REPO-CATALOG.md` records a `galen-care` care-board practice of tagging facts
`RECORDED / IMPUTED / UNRESOLVED`. Adopt the same for digest rows: a suppressed known-issue is
`UNRESOLVED` (tracked, not noisy); a delta is `RECORDED`. This keeps "we know it's broken" and
"this is new" distinct.

## What good looks like

Today (one sync, `chart`, verbatim shape): ~11 lines, 9 of them identical `no data (HTTP 401)`.

Under this doctrine, the same sync:

```
🔴 chart: auth expired — 9/11 types 401. Re-auth Partners/MGB token. (last good: labs 969)
⚪ chart steady otherwise — patient 1. UNRESOLVED: appointments, conditions, meds (auth).
```

And a healthy `bilh` sync with nothing new collapses to:

```
✓ bilh steady — 12/13 types OK, 465 labs / 62 conditions unchanged. UNRESOLVED (expected): appointments 403, 18 withheld types.
```

## Ownership & next steps

Claude Code owns the **doctrine and the digest-shaping layer**; the fetch/FHIR layer stays in
`galen-care`. Sequenced work:

1. **Delta engine** — persist per-`(source,resource_type)` fingerprints; emit changed rows only.
2. **Known-issue registry** — encode the `bilh` 403 + 18 withheld types; suppress to footer.
3. **Auth-fault collapse** — one ACT line per source auth wall.
4. **Per-record routing** — Boris vs. Peretz ACT routing.
5. **Heartbeat** — single steady-state line when no ACT/DELTA.

Cross-agent split (proposal to the fleet): Claude → shaping/delta/registry; Codex → tests for
the tiering rules and a golden-file fixture set built from real digests; Cursor → any dashboard
surface. Handoffs logged in `agent-protocol.md`.
