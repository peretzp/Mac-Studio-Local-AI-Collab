# FLEET REPO CATALOG — visibility, concern tags, merge candidates, practice exchange
*The Tower, 2026-08-29. Companion to REPO-UNIFICATION-PLAN.md (anvil `~/agents/dispatcher/`) —
that file holds remotes/auth/migration mechanics; THIS file is the map both coasts pull.
Refresh ritual: any agent touching repo topology updates this table in the same commit.*

## The catalog (14 repos + 2 bare archives, both coasts, surveyed 8/29)

| Repo | Concern | Canonical home | Coasts | State 8/29 |
|---|---|---|---|---|
| **colab** (github: Mac-Studio-Local-AI-Collab) | 🧭 fleet-bootstrap / doctrine | github (→ +gitea mirror) | OAK+FRAM | in sync both coasts (df06694) |
| **galen-care** | 🩺 Boris clinical | gitea | OAK+FRAM | OAK sync; FRAM 22 behind (auth gate) |
| **fleet-train** | 🚂 outbound governance | gitea | OAK (FRAM: add) | needs fetch-reconcile (split tips) |
| **agent-bridge** | 🌉 Mike↔Peretz mail | github+gitea(public) | OAK+Mike | live |
| **agency-correspondence** | 📜 tax/legal ceremonies | gitea (kmikeym/) | OAK+Mike | in sync (efcb62e) |
| **kmikeym-house** | 🏠 Mike's house manual | gitea (kmikeym/) | OAK+Mike | anvil BEHIND — fetch |
| **treehouse-kevin** | 🌳 Kevin co-housing | gitea | OAK+Kevin | in sync (8/8) |
| **treehouse-network** | 🌳 Kevin co-housing | **NONE — no remote** | anvil only | single-copy ⚠️ |
| **biostack** | 🧬 biohacking (Rachael Dec-retreat) | **NONE — no remote** | anvil only | single-copy ⚠️ health-adjacent → private gitea |
| **a2agents.com** | 🤖 a2agents public | github (a2agents org) | OAK | on a claude/* branch |
| **life-dashboard** | 📊 life-stack product | github | hearth+FRAM(stale) | FRAM clone Feb-stale |
| **practicelife-api** | 📊 life-stack product | github | hearth+FRAM(stale) | FRAM clone Feb-stale |
| **life-timeline** | 📊 life-stack product | github | FRAM(ref) | stale reference |
| **download-router** | 📊 life-stack product | github | FRAM(ref) | stale reference |
| **memoryatlas** | 🎙 voice/life-log | github + `nas` remote (ember!) | ember+FRAM(ref) | FRAM can't auth github |
| *(bare)* colab.git @ NAS | 🗄 UNRELATED 136-commit coast-bridge lineage | archive | vault | PRESERVE — never overwrite |
| *(bare)* galen-care.git @ NAS | 🗄 stale mirror | retire/re-mirror | vault | 170 behind, pure ancestor |

## Merge / consolidation candidates (proposals — Peretz's word before any execution)
1. **treehouse-kevin + treehouse-network → one `treehouse` repo** (same concern, same partner;
   network is remote-less and invisible to Kevin today). Keep kmikeym-house separate — it's
   Mike's binding manual, consumed not co-owned.
2. **colab ⊕ NAS colab.git lineage**: graft the 136-commit coast-bridge history as a
   `coastbridge-legacy` branch (`--allow-unrelated-histories`) or archive-rename on the NAS.
   Either way it becomes visible instead of a landmine.
3. **life-stack umbrella** (life-dashboard, practicelife-api, life-timeline, download-router):
   NOT a code merge — a meta-repo/README in colab linking them + one owner statement each;
   retire FRAM's stale reference clones rather than refresh them.
4. **gitea Mac-Studio-Local-AI-Collab**: push github main there, delete its 3 stale claude/*
   ancestor branches → one repo, two mirrors, zero divergence.
5. **fleet-train**: fetch + merge (never rebase the shared train), then FRAM gets a clone —
   Boris currently cannot see fleet state at all.

## Practice exchange — the best part of each repo, and where it should spread
| Practice | Lives in | Spread to |
|---|---|---|
| Outbound train: every send reviewed, released on a human's word, never "sent" without permalink | fleet-train (GOVERNANCE.md, SIGNING.md) | agent-bridge sends; Kevin/Naama BD threads |
| Drafter + blind-verifier + human tiebreak on documents | agency-correspondence (CP21B ceremony) | galen-care summaries; SDI packet already uses it |
| Care-board with provenance classes (RECORDED/IMPUTED/UNRESOLVED) | galen-care | any medical/legal fact table fleet-wide |
| `check_pages` before commit; repo-as-binding-manual voice | kmikeym-house (Mike) | colab doctrine docs; treehouse manual |
| Beads (`bd`) issue tracking — work discoverable via `bd ready` | treehouse-kevin | biostack, life-stack repos |
| `env.yaml` machine map — read, don't hardcode | colab | every repo with machine-specific paths |
| ev-classes on every claim (OBSERVED/REPORTED/INFERRED) | concerns ledger + this survey | all repo docs, both coasts |
| Registry-resolved health checks (`whereami`, SERVICES.json) — never hardcoded curl | anvil doctrine | colab CLAUDE.md (fixed in this commit), FRAM playbooks |

## Standing gates (from REPO-UNIFICATION-PLAN — unchanged)
Gate 0: behemoth git auth (helper stacking) — until fixed, FRAM can't pull canon.
Open questions for Peretz+Boris: rename → `colab`? Boris's own gitea identity? legacy-lineage
graft vs archive? stale claude/* branch deletion? biostack/treehouse-network gitea onboarding.
