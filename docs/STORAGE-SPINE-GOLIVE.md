# Storage Spine — shopping cart & go-live plan (2026-08-29, funded)

*Prices are late-2025/early-2026 street estimates — verify at checkout. Checkout doctrine:
Chase Prime Visa ···7703 + apply rewards points (no EBT items here).*

## The cart (~$2,150 core, ~$2,540 with options)

| # | Item | Qty | Est. | Why |
|---|------|-----|------|-----|
| 1 | **UGREEN NASync DXP4800 Plus** (4-bay, Intel x86, 2.5+10 GbE) | 1 | ~$700 | The chassis. x86 + 10GbE port + community-proven TrueNAS target. We own root. |
| 2 | **Seagate Exos X20 20 TB** (or IronWolf Pro 20 TB) | 4 | ~$320 ea ≈ $1,280 | Pool drives. Exos = enterprise, cheaper per TB; buy from 2 sellers/batches to decorrelate failures. |
| 3 | Crucial 32 GB DDR5 SO-DIMM | 1 | ~$85 | ZFS ARC eats RAM; stock 8 GB is the one real weakness. |
| 4 | TP-Link TL-SG108-M2 (8-port 2.5 GbE) | 1 | ~$170 | Breaks the gigabit ceiling for hearth/anvil/ember → node. |
| 5 | Cat6a 3 ft ×4 | 1 | ~$25 | Links. |
| — | *Option:* WD Elements 22 TB external | 1 | ~$300 | Tier-1 offline air-gapped copy; rotates to a drawer. |
| — | *Option:* 1 TB NVMe (WD SN770) | 1 | ~$60 | TrueNAS boot + app pool; frees all 4 bays for the pool. |

Not bought (deliberately): 10 GbE switch (the DXP4800+'s 10G port can direct-attach to ember
later), second PSU/UPS (CP900AVR exists), FRAM hardware (Tier 3 — after Boris weighs in).

## Burn-in (day 0–2, while everything else proceeds)
1. Flash **TrueNAS SCALE** to NVMe/USB (UGOS SSD pulled and shelved — reversible).
2. Every drive: SMART conveyance + long test, then one `badblocks -wsv` pass (destructive — BEFORE pool creation). Reject anything with reallocated/pending sectors.
3. Record serials → `docs/STORAGE-SPINE-INVENTORY.md`; label drives physically (PT-P300BT).

## Pool + datasets (day 2)
- **Layout: RAIDZ2, 4×20 TB → ~36 TiB usable.** Rationale over RAIDZ1's ~54 TiB: same-week drives
  fail correlated; a resilver of a 20 TB drive is a multi-day window — Z2 survives a second death
  during it. Capacity still >2× the whole DS223j.
- Datasets: `tank/ore` · `tank/spark` · `tank/archive` (from Deep migration) · `tank/tm`
  (SMB Time Machine target, quota'd) · `tank/replica` (inbound from FRAM later) ·
  `tank/scratch` (no snapshots).
- Snapshots: sanoid — hourly×24, daily×30, monthly×12 on data; none on scratch.
- Access: SSH keys for anvil/hearth/ember/behemoth; SMB for TM only. Tailscale on-node.

## Cutover order (day 2–7)
1. `zfs`-side rsync/tar Ore → `tank/ore` (LAN 2.5 GbE ≈ 4–6 h vs tonight's 22 h) — verify counts+spot-hashes.
2. Retire `ore-mirror-tar` cadence; DS223j `ore-mirror` share becomes the **cold second copy** (kept, refreshed weekly).
3. hearth + anvil TM → add `tank/tm` as destination (keep vault-ma offsite as second).
4. Spark → `tank/spark`. 5. SD4Loco drained → `tank/archive` (checksum, PL.OS travels).
6. FRAM leg: nightly `syncoid` OAK→FRAM once Boris answers Q1 (behemoth-fronted or Tier-3 box).

## Validation ritual (standing, monthly — Peretz's third-party rule, applied FIRST this time)
- Pull one random file from each surface (tank/ore, DS223j cold copy, TM, offsite) → diff vs source → log in this repo.
- `zpool scrub` monthly (auto), report piped to the fleet deck.
- The go-live is DONE only when the first validation pass is green — not when the hardware powers on.
