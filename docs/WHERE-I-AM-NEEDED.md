# Where I'm needed — Start here when you sit down

**One place to look.** Updated by agents and you. Read this first; then drill into TASKS.md or agent-protocol.md if you need detail.

*Last updated: 2026-03-01 (Cursor — Anvil check, Cursor tips, repo stewards; Cursor parked for reboot)*

---

## Right now: top 3 things only you can do

1. **SDI Appeal (P0)** — Submit Feb 6 evidence package (mail/fax to CUIAB). $7,444 at stake. Drafts in `vault/Efforts/Active/SDI Appeal/`. Full status: `~/.claude/SDI-APPEAL-STATUS.md`.
2. **Amazon Cart (P1)** — Review and checkout: QNAP 10GbE switch, CyberPower UPS, Cat 8 cables (for Aaron's visit). Run `amazon-sns open` or open Amazon.
3. **Trust SSL (P1, 5 min)** — One sudo + Firefox import so localhost HTTPS works everywhere. Commands in TASKS.md.

---

## Anvil status (quick check)

- **Operational:** 4 models (qwen3.5:35b-a3b, coder, 8b, nomic-embed). LiteLLM on Hearth routes to Anvil. Tailscale + SSH (anvil, anvil-ts).
- **Next (Claude):** Start sending work — TASKS §2b: MemoryAtlas deploy to Anvil, Zhirinovsky SRT polishing, ERA photo metadata, benchmark vllm-mlx. See ANVIL WORKLOAD QUEUE in TASKS.
- **No Peretz action** unless we add vllm-mlx or Open WebUI and need your click.

---

## Active threads (what agents are doing / next)

| Thread | Owner | Status | Next |
|--------|--------|--------|------|
| Anvil | Claude Code | **FULLY OPERATIONAL** | Send workloads per TASKS §2b and ANVIL WORKLOAD QUEUE |
| Repo stewards | Claude Code | REPO-STEWARDS.md added; workflow defined | Run steward pass: scan, commit, push active repos (see docs/REPO-STEWARDS.md) |
| Project management / phase-lock | All | WHERE-I-AM-NEEDED, PHASE-LOCK doc | Agents: update this file when you park or unblock |
| SDI Appeal | Peretz | Drafts ready | You: submit evidence |
| Account rename (peretz_1 → peretz) | Peretz + Claude | Script ready; can do during reboot | You: pick a time; run rename flow |
| Dashboard /docs, briefing | Claude Code | In TASKS §2c, §5d | Build /docs cheatsheet; briefing reads TASKS + Eisenhower |

---

## Blockers that need you

- **SDI Appeal** — Your signature and submission (mail/fax).
- **DreamHost → Cloudflare** — Your DreamHost login to start transfer.
- **Langfuse API keys** — Browser login at http://localhost:3005 (demo@langfuse.com / password), create keys, add to .env.
- **Connect Biggie** — Physical: connect drive; then an agent can delete Elements image.
- **Time Machine** — If not set up: System Settings (your click).

---

## State of the union (one sentence each)

- **Hearth:** Command center. All agents, LiteLLM, dashboard, API, vault, Beeper. You work here.
- **Anvil:** Inference only. Ollama + 4 models. Reached via LiteLLM on Hearth. No action needed unless we add vllm-mlx or Open WebUI.
- **iPad / iPhone:** Same vault (iCloud), same Beeper, on Tailscale mesh. Use them to capture and read; agents run on Hearth.
- **Agents:** Claude Code, Cursor, Codex. Coordination via agent-protocol.md and TASKS.md. This file is the convergence point for "where is Peretz needed."

---

## How to use this file

- **You:** When you sit down, read "Top 3" and "Blockers." Do what only you can do. If you unblock something, add a line under "Unblocked" (we can add that section) or tell the next agent.
- **Agents:** When you park or finish a thread, update "Active threads" and "Blockers." When you create something that needs Peretz (e.g. a new P0), add it to "Top 3" or "Blockers." Keep "Last updated" and your name so we know who last touched it. File path: `~/GitHub/Mac-Studio-Local-AI-Collab/docs/WHERE-I-AM-NEEDED.md` (or from repo root: `docs/WHERE-I-AM-NEEDED.md`).

We're iterating towards **phase-lock**: one place to look, minimal handoff friction, you only needed at decision points.

---

## New this run (2026-03-01)

- **CURSOR-TIPS.md** — Tips for Cursor (rules, Composer, MCP, cost); how to add agents/work; repo-dedicated work pointer.
- **REPO-STEWARDS.md** — Active repos list; steward workflow for Claude Code (scan, commit, push); how to start repo-dedicated agents. Add to TASKS as recurring or spawn "The Steward" when you want a cleanup pass.

Timestamp: 2026-03-01  
AI: Cursor  
Artifact Path: Mac-Studio-Local-AI-Collab/docs/WHERE-I-AM-NEEDED.md
