# The Storage Spine — fleet storage architecture decision (2026-08-29)

*From Peretz's dispatcher seat (The Tower, anvil), for Peretz & Boris. Rendered copy:
Claude artifact "The Storage Spine" (shareable link with Peretz) + `behemoth:~/inbox/2026-08-29-storage-spine.html`.
Status: **funded by Peretz 8/29 ("we have the budget — build the cart")** — hardware selection in STORAGE-SPINE-GOLIVE.md.*

## Why now
Mirroring 3 TB of `Ore` to the Oakland DS223j burned ~2 h of agent time before a byte moved:
DSM gates `rsync --server` behind a service checkbox + `rsync_sshd_port` match, with errors that
impersonate password failures. An adversarial audit of Aug 20–29 storage decisions then found
5 wrong claims and 6 unverified assumptions across three agents — every failure the same shape:
**a transport assumed working because ssh worked; success declared at start-instant; counter-evidence
(all previously working transports avoided rsync) never interrogated.**
Consumer appliances negotiate; infrastructure obeys. The fleet needs storage that takes orders
over SSH like every other machine we own.

## Current topology
- **Oakland (Margarido)**: hearth M2 Max · anvil M3 Ultra (always-on) · ember M5 Max ·
  DS223j 16 TB (2-bay ARM) · loose drives: Ore 8T, SD4Loco 4T, Deep 6T, Yellow Submarine, Elements.
- **Framingham (PAR-10-SKY East)**: behemoth M4 Max (Boris) · koroviev M4 Pro · annushka ·
  vault Synology (Gitea :3333 + Time Machine). All reachable as `azazello` (Macs) / `peretz` (NAS).
- Mesh: Tailscale. Missing: a per-coast storage **spine** — checksummed, snapshotted, replicable,
  fully scriptable.

## The decision
**Oakland sovereign ZFS node** (TrueNAS on x86, 4-bay, ~$2K all-in, 40–60 TB usable).
`zfs send | ssh` replaces the entire rsync/tar mirror genre: incremental forever, checksummed
end-to-end, resumable, boring. The Synologys demote to what they're good at — Time Machine
targets and cold replicas. Budget ladder: Tier 0 software-only (done 8/29) → Tier 1 ~$400
(2.5 GbE + offline drive) → **Tier 2 ~$2K the node (funded)** → Tier 3 +$1.5K FRAM symmetric.

## Questions for Boris (open)
1. **Topology** — OAK→FRAM nightly replication: behemoth fronts it, or a FRAM ZFS target sooner?
   Honest sustained uplink rate?
2. **Filesystem conviction** — ZFS vs btrfs vs "appliances are fine": argue with the rec.
3. **Ownership** — should PAR-10-SKY the entity own the hardware (depreciation, capital accounts)?
4. **FRAM needs** — what does your side want stored/replicated that we're not seeing?

Reply any channel: this repo, iMessage, or drop a file in `behemoth:~/inbox/` — the dispatcher sweeps it.

## Standing rules extracted (all seats, both coasts)
- **A DSM appliance gates each transport separately** — ssh OK ⇏ scp OK ⇏ rsync OK. Probe one
  file before building any plan on a transport.
- **ev:REPORTED never graduates into an executable plan without an OBSERVED probe.**
- **"It started" ≠ "it worked"** — verify on the object (files far-side, completed-backup lists).
- Long/background transfers: `ssh -o ControlMaster=no -o ControlPath=none -o BatchMode=yes`
  (a mux master dying mid-auth poisons the socket for every later session).
- Working transports to the Oakland NAS today: tar-stream (`ore-mirror-tar`), restic-over-ssh
  (`mirror-oak-nas.sh`). rsync pending the DSM checkbox tap.

## Go-live plan (once hardware arrives)
See `docs/STORAGE-SPINE-GOLIVE.md` (cart, burn-in, pool layout, dataset map, cutover order,
validation ritual).
